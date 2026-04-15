---
title: RenderFormer
sources:
  - "[[raw/articles/RenderFormer Transformer based Neural Rendering of Triangle Meshes with Global Illumination]]"
tags:
  - neural-rendering
  - transformer
  - global-illumination
  - computer-graphics
  - rendering-equation
created: 2026-04-15
updated: 2026-04-15
type: entity
---

# RenderFormer

A [[Deep Learning - Transformer|transformer]]-based neural rendering pipeline (Zeng, Dong, Peers, Wu, Tong — SIGGRAPH 2025) that directly renders an image from a triangle-mesh scene description with full global illumination, **without per-scene training or fine-tuning**. Reframes rendering as a sequence-to-sequence problem: a sequence of triangle tokens (with reflectance) is mapped to a sequence of pixel-patch tokens, bypassing both Monte Carlo path tracing and rasterization.

## Key idea

Rather than encode the Rendering Equation [Kajiya 1986] into the solver (as path tracing, radiosity, or PRT all do), RenderFormer learns a rendering pipeline **from data with minimal priors**. One triangle token per scene triangle → transformer produces a triangle token holding the triangle's converged outgoing-radiance distribution → second transformer gathers those tokens into ray-bundle tokens → MLP/DPT decodes log-HDR pixel values.

The whole pipeline is:

- **Non-recursive** — solves light transport in a single forward pass, no Monte Carlo, no noise.
- **Fully neural** — no rasterization, ray tracing, or ray marching anywhere in the critical path.
- **Differentiable** — every component is learnable, opening inverse-rendering directions.
- **Scene-agnostic** — train once, inference on novel meshes with no fine-tuning.

## Two-stage architecture

### View-independent stage (12 layers)

Resolves **triangle-to-triangle light transport**. Input: a sequence of triangle tokens + 16 register tokens [Darcet et al. 2024]. Output: transformed triangle tokens encoding each triangle's overall outgoing radiance.

- 768-d tokens, 12 layers, 6 heads, d_ff = 768×4, LLaMA-style pre-norm with RMS-Normalization, SwiGLU, QK-Normalization, full bidirectional self-attention.
- Runs in **bf16** precision; flash-attention-2 for speed.

### View-dependent stage (6 layers)

Transforms **ray-bundle tokens** into output pixel patches. Each output token covers an 8×8 pixel patch; the virtual camera is specified by the ray bundle passing through those 64 pixel centers.

- Same transformer recipe, but each layer adds a **cross-attention** over the view-independent triangle tokens (so the ray-bundle queries "which triangles matter for these pixels").
- Runs in **tf32** precision (higher precision needed for convergence vs. view-independent bf16).
- Operates in **camera coordinates**, not world coordinates — vertex positions transformed to camera space before positional embedding; avoids learning the world-to-camera transform and improves accuracy (Table 1 rows 5-6).
- Final decoder: self-attention between ray-bundle tokens + a dense vision transformer (DPT) [Ranftl et al. 2021] over the last 4 layers' features → log(x+1)-encoded HDR RGB. Both self-attention and DPT measurably improve results (Table 1 rows 1-4).

## Triangle token embedding

Everything relevant to rendering packed into one 768-d vector per triangle:

- **Geometry** → relative 3D spatial positional encoding (see below). Shape is *implicit* in the three vertex positions; no separate "shape" feature.
- **Normals** → per-vertex shading normals, NeRF-style positional encoding (6 frequencies), linear-projected to 768-d + RMS-Norm.
- **Material** → stacked 10-d vector of diffuse albedo (3), specular albedo (3), roughness (1), emission (3); GGX microfacet BRDF [Walter et al. 2007]. Linear-projected to 768-d + RMS-Norm.
- Added together into the final triangle token.

## Relative 3D spatial positional encoding (the novel bit)

Standard transformer positional encoding injects **1D sequence index**. For rendering, the sequence order is irrelevant (swap two triangles → same output), but the **3D world position** of each triangle is load-bearing: two triangles with identical materials/shapes at different locations contribute differently to global light transport. Plus: translating the whole scene shouldn't change the image.

RenderFormer adapts [[Deep Learning - Rotary Positional Embedding|RoPE]] to 3D vertex positions:

- Concatenate the three vertex positions into a 9-d vector.
- Duplicate each element 6 times and multiply by 6 frequencies `{1.0, 1.3797, 1.9037, 2.6265, 3.6239, 5.0}` (log-spaced 1→5) → 54 scaled frequencies.
- Encode each as sin/cos → block-diagonal rotation matrix where each 2×2 block is one sin/cos pair.
- Apply the block-rotation to the first 108 of each head's 128 coefficients (6 heads × 128 = 768); leave the remaining 20 per head unchanged.
- Applied to tokens at **each attention layer**, RoPE-style, so attention sees *relative* 3D positions and the model is automatically translation-invariant.

**Rotation invariance is NOT achieved** — SO(3) is not commutative, so RoPE-style decomposition doesn't cleanly extend. Training instead uses random rotation augmentation (via RoMa [Brégier 2021], applied on-the-fly, no re-rendering needed) to get rotational robustness.

Register tokens get the relative positional encoding at the **average vertex position** so they're also translation-invariant.

**Ablation:** attempted to embed triangle position additively via NeRF-style encoding like the normals — training unstable, converges to bad local minima. The RoPE-style relative encoding is load-bearing.

## Training

- **Hardware:** 8 × NVIDIA A100 40GB. FlashAttention-2, Liger Kernel.
- **Optimizer:** AdamW, batch size 128, linear warmup to lr = 1e-4 over 8k steps, then cosine decay.
- **Curriculum:** 500k iters at 256×256 with ≤1536 tris (~5 days), then 100k iters at 512×512 with ≤4096 tris (~3 days).
- **Loss:** L1 on log-transformed HDR images (avoids bright-pixel domination) + 0.05 · LPIPS on a tonemapped `clamp(log I / log 2, 0, 1)` version.
- **Rotation augmentation:** random scene rotation per-sample.

### Training data

- 16M HDR images total (8M at 256², 8M at 512²) from 2M synthetic scenes, 4 viewpoints each.
- Scenes: 1-3 random **Objaverse** [Deitke et al. 2023] objects dropped into one of **4 template scenes** (ground + back + side walls, randomly translated/rotated/scaled).
- Camera: outside scene bounding box, FOV ∈ [30°, 60°], distance ∈ [1.5, 2.0] bbox units, aimed near center with perturbation.
- Lights: 1-8 diffuse-emitting triangles, intensity ∈ [2500, 5000] W/unit², distance ∈ [2.1, 2.7].
- Materials: per-object or per-triangle (1:1), RGB diffuse + mono specular summing to ∈ [0.9, 1.0], roughness log-sampled in [0.01, 1.0], flat or Phong-shaded (coin flip).
- Objaverse meshes remeshed via SDF + marching cubes → Qslim to 256-3072 faces.
- Reference rendered with Blender Cycles, 4096 spp adaptive + denoise.

## Performance

At 512² on a single A100, pure PyTorch with pre-cached kernels:

| Scene | #tris | Cycles (adaptive+denoise) | Cycles (4096 spp) | RenderFormer | View-indep. | View-dep. |
|-------|------:|-------------------------:|-----------------:|-------------:|------------:|----------:|
| Fig 1a | 5366 | 3.97 s | 12.05 s | 0.076 s | 0.028 s | 0.048 s |
| Fig 1b | 4400 | 4.73 s | 11.21 s | 0.061 s | 0.019 s | 0.043 s |
| Fig 1c | 4527 | 3.77 s | 9.95 s | 0.062 s | 0.019 s | 0.043 s |
| Fig 1d | 7321 | 2.71 s | 7.83 s | 0.098 s | 0.043 s | 0.055 s |

Roughly 40-100× faster than Cycles at matching-spp quality; equal-time comparison (Cycles at 26 spp, no denoise) shows RenderFormer has much lower noise. Further speedups possible: reuse the view-independent pass for static scenes (amortize across camera moves), batch 48 animation frames at once.

## Capabilities & effects modeled

- Multiple specular reflections (up to ~3 bounces consistently; higher-order drops due to training scarcity)
- Soft and hard shadows, including sub-triangle-granularity detail
- Diffuse inter-reflections
- Glossy reflections
- Multiple light sources (up to 8, matching training)
- Robust to scene translation (by design) and rotation (by augmentation)

## Limitations (well-documented in the paper)

- **Triangle budget:** ≤4096 due to O(N²) attention. Larger meshes work but lose fine detail; fails gracefully.
- **Triangle size:** trained on uniformly-sized triangles. Oversize triangles degrade shading and shadows — token has to encode too much, and the degradation *propagates into reflections* (back-wall mirror of a bad pedestal also looks wrong). Suggests pre-meshing/subdivision before inference.
- **Lights ≤8**, **outside scene**, **white only**, **limited size range**. Linearity-of-light-transport trick recovers many cases: render single-color/single-light/subdivided-light images and composite.
- **Camera outside scene** — triangles behind the camera have never been seen.
- **Fixed 512×512 resolution.** Higher resolutions fail gracefully, errors concentrate at depth discontinuities.
- **Constant per-triangle reflectance** (no textures). Exploratory extension: rasterize 13-channel material to a 32×32 isosceles-right triangle → 13,312-d → 768-d token. Works but blurred; longer tokens may help.
- **Higher-order (>3) specular bounces** unreliable — training-data scarcity, not architectural.
- **Rotation invariance not architectural** (SO(3) non-commutativity); relies on data augmentation.

## Ablation highlights (Table 1, all at 256²)

- **View-dependent stage components:** full stage PSNR 29.77 → without DPT 29.75 (SSIM drops more) → without self-attention 29.70 → without both 29.15. Both contribute.
- **World-space vs camera-space view-dependent stage:** 28.98 vs 29.77. Camera space is clearly better.
- **Model size** (768d/205M → 384d/45M): monotonic quality drop, PSNR 29.77 → 27.87.
- **Layer ratio** (total 18): 12+6 → 29.77, 9+9 → 30.11, 6+12 → **30.38**, 0+18 → 28.28. **Eliminating the view-independent stage fails**; more view-dependent layers help quality but cost more at runtime. Paper ships 12+6 as accuracy-vs-speed balance (~25% faster, ~5% accuracy loss vs best).

### Complexity of each stage

- View-independent: **O(#tris²)** — attention over all triangle pairs.
- View-dependent: **O(#bundles² + #bundles × #tris)** — self-attention over ray bundles + cross-attention to triangles.

Plus a hardware-dependent factor from the bf16/tf32 precision split.

## What each stage actually learns

Auxiliary probe: a small MLP + cross-attention layer (with frozen view-independent weights) is trained to decode each transformed triangle token into a 32×32 RGB texture. Result: the view-independent stage **already resolves most diffuse light transport and shadows** — the transformed tokens contain per-triangle shading maps with sub-triangle shadow detail.

For the view-dependent stage, cross-attention weights visualized per ray-bundle show attention concentrated on **directly visible triangles and triangles around the reflected direction**, with the distribution widening as roughness increases — i.e., the model has learned specular-lobe-like attention shapes without being told to.

## Relation to other work

- **Monte Carlo path tracing** (Kajiya, Dutré, Pharr) — physically explicit; RenderFormer replaces the integral with a learned transform.
- **Radiosity / Neural Radiosity** [Hadadan et al. 2021] / **PRT** — precompute light transport per scene; RenderFormer trains once across scenes.
- **RenderNet / Neural Voxel Renderer** — voxel-based, local shading, single light; no global illumination. RenderFormer has GI and takes native triangle input.
- **NeRFs** [Mildenhall et al. 2021] — learn a scene as an implicit function; need per-scene training. RenderFormer takes triangle input and has no per-scene training.
- **LVSM** [Jin et al. 2024] — transformer for view interpolation using ray tokenization (RenderFormer reuses the ray-bundle tokenization idea for camera pose).
- **Vision Transformers need registers** [Darcet et al. 2024] — source of the 16 register tokens.

## Future directions (paper's own)

- Hierarchical / sparse attention (LoD, BVH-guided attention) to break the 4096-triangle ceiling.
- More expressive materials (transparency, subsurface scattering) — no architectural barrier.
- Environment and non-diffuse lighting.
- Inverse rendering — the differentiability makes this natural.

## Sources

- Chong Zeng, Yue Dong, Pieter Peers, Hongzhi Wu, Xin Tong. *RenderFormer: Transformer-based Neural Rendering of Triangle Meshes with Global Illumination.* SIGGRAPH Conference Papers '25.
- Affiliations: Zhejiang University State Key Lab of CAD & CG; Microsoft Research Asia; College of William & Mary.

## Related

- [[Graphics - Neural Rendering]] — the broader topic this slots into.
- [[Deep Learning - Transformer]] — base architecture, with novel 3D positional encoding.
- [[Deep Learning - Attention Mechanisms]] — cross-attention and self-attention both load-bearing here.

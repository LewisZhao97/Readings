---
title: Neural Rendering
sources:
  - "[[raw/articles/RenderFormer Transformer based Neural Rendering of Triangle Meshes with Global Illumination]]"
tags:
  - neural-rendering
  - computer-graphics
  - rendering-equation
  - global-illumination
created: 2026-04-15
updated: 2026-04-15
type: topic
---

# Neural Rendering

**Neural rendering** replaces the simulation of light transport with a learned neural process. Rather than solving the Rendering Equation [Kajiya 1986] via Monte Carlo integration, finite elements, or rasterization, a neural model predicts the image (or intermediate radiance quantities) directly.

This is a broad umbrella — any system that uses neural networks somewhere in the image-synthesis pipeline is arguably "neural rendering" — but there are meaningful subfamilies distinguished by **what the model replaces**, **what scene representation it takes**, and **whether it needs per-scene training**.

## The Rendering Equation (the thing being replaced)

$$
L_o(x, \omega_o) = L_e(x, \omega_o) + \int_\Omega f_r(x, \omega_i, \omega_o)\, L_i(x, \omega_i)\, (\omega_i \cdot n)\, d\omega_i
$$

Outgoing radiance at $x$ in direction $\omega_o$ = emitted + integrated incident radiance weighted by the BRDF and cosine term. The integral is **recursive** (incident radiance is some other surface's outgoing radiance), which is why classical solvers are iterative or path-tracing-based.

A neural renderer can sidestep the equation entirely (learn the mapping from scene → image) or bake it into the architecture (e.g., precompute the transport operator once per scene).

## Axes of comparison

| Axis | Options |
|---|---|
| **Scene representation** | Triangle mesh, voxel grid, point cloud, implicit neural field (NeRF), Gaussian splat, RGB-D, images |
| **Training scope** | Per-scene (overfits the current scene) vs. generalizing (train once, render many scenes) |
| **Light transport** | Local shading only / single bounce / full GI |
| **Neural role** | End-to-end image synthesis / neural module inside classical pipeline (denoiser, cache) / learned scene representation + classical renderer |
| **Differentiability** | Fully differentiable (useful for inverse rendering) vs. only partially |

## Families

### Classical pipeline + neural augmentation

Keep the physical renderer, plug in a neural module:

- **Neural denoisers** — predict clean image from low-spp path-traced input [Bako et al. 2017].
- **Neural caches** — cache radiance in learned structures [Müller et al. 2021; Coomans et al. 2024].

These leave the Rendering Equation visible in the pipeline; they just accelerate parts of it.

### Neural radiosity / PRT

Learn a scene-specific transport operator — once trained, any viewpoint or light can be rendered cheaply. Expensive **per-scene** precomputation.

- **Neural Radiosity** [Hadadan et al. 2021] — neural-network radiosity variant; handles arbitrary BRDFs, unlike classical radiosity (which is diffuse-only).
- **Precomputed Radiance Transfer** [Sloan et al. 2023] and neural variants [Rainer et al. 2022; Gao et al. 2022] — factor out the light-transport matrix, replace the fixed lighting basis (e.g., spherical harmonics) with a learned embedding.

### Implicit neural scene representations

The scene *is* the neural network; rendering is ray-marching the net:

- **NeRF** [Mildenhall et al. 2021] — MLP maps (position, direction) → (density, radiance); volumetric rendering produces the image. Captures photorealistic scene reconstructions from multi-view photos but needs **per-scene training**, is limited to the captured subject, and editing is non-trivial [Granskog et al. 2020, 2021; Haque et al. 2023; Yuan et al. 2022; Zheng et al. 2024].
- **Gaussian splats** — millions of learned 3D Gaussians, differentiable rasterization; similar per-scene training regime.

### Generalizing neural renderers over explicit geometry

Take a classical scene (mesh/voxel) as input, learn a rendering pipeline, train once on many scenes, render novel scenes without retraining:

- **RenderNet** [Nguyen-Phuoc et al. 2018] — voxel in, convolutional rendering out; local shading, single point light.
- **Neural Voxel Rendering** [Rematas & Ferrari 2020] — voxel in, similar scope.
- **Sanzenbacher et al. 2020** — point-cloud-based global light transport + screen-space shading; trained per-scene (static or dynamic).
- **[[Graphics - RenderFormer|RenderFormer]]** [Zeng et al. 2025] — **triangle-mesh in, transformer-based, full GI, no per-scene training.** First system combining all four.

### Transformer-based view synthesis

Use [[Deep Learning - Transformer|transformers]] to aggregate features along rays, between views, or directly over pixel patches:

- NerFormer [Reizenstein et al. 2021] — epipolar-constrained attention for feature volumes.
- IBRNet [Wang et al. 2021a] — transformer-estimated density along rays.
- View-interpolation methods [Liang et al. 2024; Sajjadi et al. 2022; Suhail et al. 2022; Varma et al. 2022] — attention over ray features and across views.
- LVSM [Jin et al. 2024] — directly transforms input pixel patches to view-interpolated output patches; camera specified via per-pixel ray tokens. RenderFormer reuses this ray-bundle tokenization.

## Why RenderFormer is a distinctive point in this space

The combination:

- Classical triangle-mesh input (compatible with existing DCC tool chains)
- Full global illumination
- **No per-scene training**
- Fully transformer-based, fully differentiable

…is unusual. Most neural renderers pick a subset. See [[Graphics - RenderFormer]] for details.

## Rendering Equation — what neural renderers preserve or discard

| Approach | Explicitly models Rendering Equation? | Recursive? |
|---|---|---|
| Monte Carlo path tracing | Yes | Yes |
| Radiosity (classical) | Yes (finite-element) | Yes (iterative) |
| Neural Radiosity / PRT | Yes (learned operator) | Equation solved during training |
| NeRF | No — learns volumetric radiance field | No (single forward pass per ray) |
| RenderNet / Neural Voxel | No — learns shading directly | No |
| RenderFormer | No — learns transport transform directly | No (single two-stage forward pass) |

## Open challenges

- **Scalability of attention** to large scenes — RenderFormer's quadratic cost caps it at ~4k triangles. Hierarchical / sparse attention using existing graphics structures (BVH, LoD) is an obvious direction.
- **Material breadth** — most learned systems are tied to the BRDF family seen at training. GGX is common; transparency and subsurface scattering are largely open.
- **Lighting generality** — environment lighting, emissive textures, many-light rendering.
- **Inverse rendering** — differentiable neural renderers open the door to scene recovery from images, but the full generalizing pipeline hasn't been pushed there yet.
- **Editing** — NeRF-style implicit representations are hard to edit compared to triangle meshes; RenderFormer-style systems inherit edit-ability but inherit triangle-budget limits.

## Related

- [[Graphics - RenderFormer]] — the instance that motivates this topic.
- [[Deep Learning - Transformer]] — architecture underlying the transformer-based neural renderers.
- [[Graphics - Ray Marching]] — a classical rendering technique without neural components; useful contrast.
- Glossary: **Rendering Equation**, **Global Illumination**, **BRDF**, **GGX**, **Neural Rendering**.

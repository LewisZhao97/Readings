---
title: "RenderFormer: Transformer-based Neural Rendering of Triangle Meshes with Global Illumination — Summary"
sources:
  - "[[raw/articles/RenderFormer Transformer based Neural Rendering of Triangle Meshes with Global Illumination]]"
tags: [neural-rendering, transformer, global-illumination, computer-graphics, rendering-equation]
created: 2026-04-14
updated: 2026-04-15
type: summary
recommend: yes
distil_times: 1
---

## Overview

RenderFormer (SIGGRAPH 2025, Zeng et al.) is a neural rendering pipeline that directly renders images from triangle-mesh scenes with full global illumination, without per-scene training or fine-tuning. It reframes rendering as a sequence-to-sequence transformer task: a sequence of triangle tokens (with reflectance) is mapped to a sequence of pixel-patch tokens, bypassing Monte Carlo path tracing and rasterization entirely.

## Key Points

- **Two-stage transformer architecture:**
  - *View-independent stage* (12 layers): triangle-to-triangle light transport via self-attention; outputs per-triangle radiance features.
  - *View-dependent stage* (6 layers): transforms 8×8 ray-bundle tokens into HDR pixel patches via cross-attention to triangle tokens, followed by a dense vision transformer (DPT).
- **Relative 3D spatial positional encoding** adapted from RoPE on triangle vertex positions — makes the network translation-invariant (but not rotation-invariant; rotation handled via data augmentation).
- **Triangle token embedding:** per-vertex normals (NeRF positional encoding) + GGX microfacet BRDF parameters (diffuse/specular albedo, roughness) + emission, projected to 768-d.
- **Camera-coordinate view-dependent stage** improves accuracy vs. world coordinates.
- **16 register tokens** added for global information / noise removal.
- **Training:** 8 A100 GPUs, AdamW, L1 (log-space) + 0.05·LPIPS loss; 500k iters at 256² (≤1536 tris, ~5 days) + 100k fine-tune at 512² (≤4096 tris, ~3 days). Uses FlashAttention-2, Liger Kernel, bf16/tf32.
- **Training data:** 16M HDR images from 2M scenes composed from Objaverse objects in 4 template scenes, rendered with Blender Cycles 4096 spp.
- **Results:** handles soft/hard shadows, multiple specular interreflections, diffuse indirect, glossy reflections, multiple lights. ~0.06–0.1 s/frame vs. Blender Cycles ~4 s (adaptive+denoise) or ~10 s (4096 spp).
- **Limitations:** ≤4096 triangles (quadratic attention cost), single BRDF model, no textures (per-triangle params), ≤8 diffuse light sources, 512² fixed resolution, camera outside scene bounding box.

## Entities Mentioned

- **People:** Chong Zeng, Yue Dong, Pieter Peers, Hongzhi Wu, Xin Tong
- **Institutions:** Microsoft Research Asia, Zhejiang University (State Key Lab of CAD & CG), College of William & Mary
- **Models/architectures:** Transformer, RoPE, LLaMA-style pre-norm, RMS-Normalization, SwiGLU, QK-Normalization, FlashAttention-2, Dense Prediction Transformer (DPT), NeRF (positional encoding)
- **Tools/datasets:** Blender Cycles, Objaverse, Liger Kernel, RoMa, Qslim
- **Concepts:** Rendering Equation (Kajiya 1986), GGX microfacet BRDF (Walter 2007), Radiosity, Neural Radiosity, Precomputed Radiance Transfer (PRT), NeRF, LPIPS, FLIP

## Related Topics

- [[Deep Learning - Attention Mechanisms]] — transformer self/cross-attention
- [[Deep Learning - Transformer]] — base architecture RenderFormer adapts
- [[Graphics - Ray Marching]] — alternative rendering paradigm
- Potential new topics: Neural Rendering, Global Illumination, Rendering Equation, BRDF models

## Recommend Reason

**yes** — This is a novel, well-executed application of transformer architectures outside NLP/vision, bridging computer graphics and deep learning. It introduces non-obvious design choices (relative 3D RoPE on vertex positions, camera-space view-dependent stage, ray-bundle tokenization, two-stage split mirroring view-independent/view-dependent light transport) that are worth distilling into entity and topic pages. Strong candidate for new entity pages (RenderFormer, GGX BRDF), topic pages (Neural Rendering, Global Illumination), and cross-links to the existing Transformer entity showing a concrete non-language domain application.

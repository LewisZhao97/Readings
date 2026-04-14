---
title: "Image Guided Depth Super-Resolution for Spacewarp in XR Applications — Summary"
sources:
  - "[[raw/articles/Image Guided Depth Super-Resolution for Spacewarp in XR Applications]]"
tags: [xr, space-warp, depth-map, depth-super-resolution, bilateral-filter, hmd, remote-rendering]
created: 2026-04-14
updated: 2026-04-14
type: summary
recommend: partial
distil_times: 1
---

## Overview

Peri & Xiong (Samsung Research America, IEEE 2021) present an image-guided depth super-resolution algorithm for split-rendering XR pipelines: the device sends a **regular-grid downsampled** depth map plus full-resolution color, and the HMD upsamples depth via a bilateral-filter-style weighted average using spatial, color-intensity, and 3D-pose-derived depth weights. The reconstructed depth map drives Space-Warp reprojection with better edge preservation than bilinear interpolation.

## Key Points

- **Problem:** Wireless split rendering (5G/WiGig) still hits latency limits; transmitting full-res depth doubles image bandwidth. "Binning" (regular sub-sampling every 2/4/8 px) compresses but loses detail; naive averaging causes artifacts.
- **Algorithm — three Gaussian-kernel weights applied per-pixel over a neighborhood $N$:**
  - Spatial: $W_s = G_{\sigma_s}(\|p-q\|)$
  - Intensity (color-image guide): $W_r = G_{\sigma_r}(I_p - I_q)$
  - Depth (from estimated 3D pose via camera model): $W_d = G_{\sigma_d}(D_p - D_q)$
  - Output: $D_p = \frac{\sum_{q\in N} W_s W_r W_d D_q}{\sum_{q\in N} W_s W_r W_d}$
- **Iterative refinement:** if the depth candidate is not "good enough," update 3D poses with it and re-solve.
- **Conceptually:** a joint/cross bilateral filter (Tomasi & Manduchi; Durand & Dorsey) applied to depth upsampling, with color image as guidance.
- **Spacewarp reprojection:** standard 2D → 3D with depth + new head pose; hole-filling for disocclusions. Implemented in OpenGL ES on GPU.
- **Results:** At scale factors 0.125 / 0.0625 / 0.03125 / 0.015625, super-resolution preserves edges vs. blurred bilinear; at 0.125 nearly matches ground truth; smaller factors show block-like artifacts tunable via σ. Spacewarp runs ~60 fps at 1200×1080 on Samsung S20+ (Snapdragon 865, Adreno 650) — similar fps to bilinear (dominated by other pipeline stages).
- **Hardware:** tested on Samsung S10, Note10, Note10+, S20, S20+.

## Entities Mentioned

- **People:** Christopher Peri, Yingen Xiong
- **Organizations:** Samsung Research America
- **Concepts:** Space-Warp / Timewarp / 3D reprojection, bilateral filter (Tomasi & Manduchi 1998; Durand & Dorsey 2002), joint bilateral upsampling, depth super-resolution, depth completion, HMD split rendering, 5G / WiGig (IEEE 802.11ay)
- **Hardware:** Samsung S20+, Snapdragon 865, Adreno 650; tested on S10/Note10/Note10+/S20
- **Tools:** Blender (ground-truth color+depth), OpenGL ES

## Related Topics

- Companion/successor paper: [[XR - Space Warp with Depth Propagation]] — **same authors, same problem, improved approach**. The propagation paper replaces regular-grid downsampling with adaptive feature/edge/contour-based sparse point extraction and frames reconstruction as an optimization rather than a one-shot filter. If you read only one, prefer the propagation paper.
- Potential new topics: Bilateral Filtering for Depth, XR Remote Rendering, Space-Warp

## Recommend Reason

**partial** — This is essentially a joint bilateral upsampler applied to depth maps for XR Space-Warp; the novelty is the application and the 3D-pose-derived depth weight, not the filter itself. Highly redundant with the later **Space Warp with Depth Propagation** paper (same authors, same venue family, superseded approach). Distil only if you want a dedicated Bilateral Filter entity, or want a comparison page between the two papers. Otherwise, ingest summary is sufficient and the propagation paper carries the story. Equations 1–8 are the substantive content.

---
title: "Space Warp with Depth Propagation in XR Applications — Summary"
sources:
  - "[[raw/articles/Space Warp with Depth Propagation in XR Applications]]"
tags: [xr, space-warp, depth-map, depth-propagation, hmd, remote-rendering, computer-graphics]
created: 2026-04-14
updated: 2026-04-14
type: summary
recommend: partial
distil_times: 1
---

## Overview

Xiong & Peri (Samsung Research America, IEEE 2021) present a pipeline for reducing depth-map bandwidth in split-rendering XR systems: the device side extracts a sparse set of high-value depth points guided by image features (keypoints, edges, contours), and the HMD reconstructs a dense depth map via image-guided depth propagation, which is then used in Space-Warp late-stage reprojection. It outperforms bilinear and multilevel B-spline interpolation, preserving object boundaries with fewer artifacts in reprojected frames.

## Key Points

- **Problem:** 5G/wireless XR offloads rendering from HMD, but transmitting full-res depth alongside color doubles bandwidth and adds latency. Regular-grid depth downsampling smooths edges → Space-Warp artifacts.
- **Pipeline — device side (adaptive depth-point extraction):**
  1. Multi-scale feature-point extraction (Gaussian pyramid + DoG/LoG).
  2. Multi-scale edge extraction (Canny/Sobel in pyramid).
  3. Object and image contour detection.
  4. Verify each candidate against depth map (drop points whose depth barely changes vs. neighbors).
  5. Integrate, deduplicate, and assign confidence-score levels → sparse depth point set; transmit via WiFi (level selection adapts to link quality).
- **Pipeline — HMD side (dense depth recovery via depth propagation):** Optimize a cost function with:
  - Data term: keep sparse depths unchanged (weighted L2).
  - Smooth term: propagate neighbor depths into each pixel, weighted by color-image + original-depth-map similarity (bilateral-style).
  - $J(d) = \mu_s W_s \sum_p \|d(p) - d_s(p)\|^2 + \mu_m \sum_{(p,q)\in N} W_{pq} \|d(p) - d(q)\|^2$
- **Space-Warp:** NDC → 3D reprojection via $P\,S_i\,S_r^{-1}\,P^{-1}$ using predicted head pose; includes a hole-filling step.
- **Results:** At 16512 / 4435 / 1858 / 1062 sparse points (from a 1024² map), depth propagation yields clean boundaries; both bilinear and B-spline produce blurred edges, missing contour pixels, and distorted reprojected frames (non-straight boundaries).
- **Implementation:** OpenGL ES on Samsung Note 10+ (Exynos 9825), tested on S10/Note10/S20/S20+.
- **Cost:** Higher compute than bilinear, but lower total cost than transmitting higher-res depth over wireless.

## Entities Mentioned

- **People:** Yingen Xiong, Christopher Peri
- **Organizations:** Samsung Research America
- **Concepts:** Space-Warp / late-stage reprojection, Asynchronous Spacewarp (Oculus), post-rendering 3D warping (McMillan & Bishop), plenoptic modeling, SIFT, SURF, Canny edge detection, multilevel B-spline interpolation (Lee et al.), bilateral filtering, image-guided depth propagation, head-mounted device (HMD), 6 DOF head pose, NDC (normalized device coordinates)
- **Hardware/tools:** Samsung Note 10+, Exynos 9825, OpenGL ES, Blender (ground-truth generation)

## Related Topics

- Companion paper: [[XR - Image Guided Depth Super-Resolution]] — earlier paper by same authors, simpler regular-grid downsample + bilateral-style super-resolution approach. This depth-propagation paper is the **adaptive, feature-aware successor**.
- Potential new topics: XR Remote Rendering, Space-Warp / Late-Stage Reprojection, Depth Map Densification

## Recommend Reason

**partial** — The core ideas (feature-guided sparse depth + bilateral-style propagation) are solid but are direct, fairly-standard combinations of known techniques (SIFT/Canny/contour detection + bilateral filtering as an optimization). Distil if building an XR-rendering topic cluster: it would support good entity pages (Space-Warp, Depth Propagation) and pairs cleanly with the super-resolution paper for a comparison. Skip the deep distil if the knowledge base is focused elsewhere — the summary captures the algorithm well enough. Equations 1–4 and the device/HMD split are the non-obvious, distil-worthy parts.

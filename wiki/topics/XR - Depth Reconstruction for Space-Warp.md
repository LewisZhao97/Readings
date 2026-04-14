---
title: "Depth Reconstruction for Space-Warp in Split-Rendered XR"
sources:
  - "[[raw/articles/Space Warp with Depth Propagation in XR Applications]]"
  - "[[raw/articles/Image Guided Depth Super-Resolution for Spacewarp in XR Applications]]"
tags: [xr, space-warp, depth-map, remote-rendering, hmd]
created: 2026-04-14
updated: 2026-04-14
type: topic
---

# Depth Reconstruction for Space-Warp in Split-Rendered XR

How to send a compressed depth representation from a remote renderer to an HMD and reconstruct a full-resolution depth map good enough to drive [[XR - Space-Warp|Space-Warp]] reprojection. Sourced from two consecutive Samsung Research America papers by Yingen Xiong and Christopher Peri.

For the side-by-side algorithmic comparison of the two reconstruction approaches, see [[XR - Depth Super-Resolution vs Depth Propagation]].

## The Problem

With 5G, WiGig (IEEE 802.11ay), and similar high-bandwidth low-latency links, rendering is moved off the HMD to a tethered device or cloud, keeping the headset small and light but introducing round-trip latency between head-pose update and displayed pixels. Space-Warp re-warps the last rendered frame using the freshest head pose — but it **requires depth** to do so correctly.

Transmitting a full-resolution depth map alongside the color image roughly **doubles wireless bandwidth**. So: send a compressed depth representation; reconstruct on the HMD with enough fidelity for clean reprojection.

### Why naive resampling fails

Regular-grid down/up-sampling (bilinear) smooths over depth discontinuities:

- Blurred / zigzag object boundaries
- Missing pixels on edges and contours
- Non-straight straight lines after reprojection
- Artifacts that grow as sub-sampling becomes more aggressive

Multilevel B-spline interpolation (Lee et al. 1997) preserves edges slightly better but distributes false depths *inside* objects, also producing reprojection artifacts. Any depth-only method ignores the co-transmitted full-resolution color image, which is the strongest cue for where depth discontinuities actually lie.

## Split Pipeline

Both papers follow the same device ↔ HMD split:

```
[Rendering Device]                              [HMD]
  render color + depth
  compress depth  ──── WiFi / 5G ─────▶  receive color + compressed depth
                                         reconstruct full-resolution depth
                                         Space-Warp reproject with new head pose
                                         hole-fill disocclusions
                                         display
```

What changes between papers is the **compression scheme** (device side) and the **reconstruction algorithm** (HMD side). The core insight shared by both: **use the full-resolution color image as a guide** when reconstructing depth. Color edges and intensity discontinuities correlate strongly with depth discontinuities, so color-guided weighting preserves object boundaries in a way pure-depth interpolation cannot.

## Two Reconstruction Approaches

Both are detailed — with equations — in [[XR - Depth Super-Resolution vs Depth Propagation]]. Summary:

- **Image-guided depth super-resolution** (Peri & Xiong 2021, earlier): device sends a regular-grid sub-sampled depth map; HMD reconstructs via a [[Graphics - Bilateral Filter|joint bilateral filter]] with three Gaussian weights (spatial, color-intensity, pose-derived depth).
- **Space-Warp with depth propagation** (Xiong & Peri 2021, later): device extracts adaptive sparse points at feature/edge/contour locations with confidence scores; HMD reconstructs via global optimization (data term anchors sparse depths, smooth term propagates through bilateral-style neighborhood weights).

The second paper supersedes the first — feature-aware sampling puts points where depth changes, and hard-anchoring the sparse points prevents the drift that a pure weighted average incurs.

## Space-Warp Reprojection Math

Given the reconstructed dense depth, both papers reproject the color frame identically.

Normalize screen → NDC:

$$
(x_{ndc}, y_{ndc}, z_{ndc})^T = (2x_{sc}/w - 1,\; 2y_{sc}/h - 1,\; d)^T
$$

Reproject with predicted head pose:

$$
(x_p, y_p, z_p, w_p)^T = P\, S_i\, S_r^{-1}\, P^{-1}\, (x_{ndc}, y_{ndc}, z_{ndc}, 1)^T
$$

where $P$ is the projection matrix, $S_r$ the current head pose, $S_i$ the predicted pose for the new frame. Disocclusions leave holes that a subsequent fill pass patches. See [[XR - Space-Warp]] for more on the technique and its lineage.

## Implementation & Performance (shared)

- OpenGL ES on mobile GPU (offload CPU).
- Tested on Samsung S10, Note10, Note10+, S20, S20+. Paper A reports ~60 fps at 1200×1080 on S20+ (Snapdragon 865, Adreno 650) — Space-Warp cost is dominated by other pipeline stages, so bilinear vs. super-resolution are indistinguishable in fps.
- Paper B uses Samsung Note 10+ (Exynos 9825) as the primary test device.
- Ground-truth color + depth generated with Blender; grayscale depth maps, near/far planes 0.037/0.083 m, brightness proportional to distance.
- Paper B notes the added compute cost vs. bilinear is outweighed by the cost of sending higher-resolution depth over wireless.

## Open Questions / Limitations

- Both papers validate on **Blender-synthetic** color+depth only; no real captured depth (sensor noise), no textured scenes stress-testing.
- No head-to-head fps numbers for the two reconstruction methods; optimization (B) typically costs more per frame than one-shot filtering (A).
- Confidence-level adaptation in B is described qualitatively ("good WiFi" vs. "bad WiFi") without a closed-loop rate-control scheme.
- Hole-filling after reprojection is mentioned but never described — a known hard sub-problem (disocclusion inpainting).
- No comparison against learning-based depth completion / super-resolution, which the super-resolution paper briefly acknowledges as promising but error-prone for XR.

## Connections

- **[[XR - Space-Warp]]** — the reprojection technique these papers feed.
- **[[Graphics - Bilateral Filter]]** — weighting scheme underlying both reconstructions.
- **[[XR - Depth Super-Resolution vs Depth Propagation]]** — detailed algorithmic comparison.
- **[[Graphics - Ray Marching]]** — sibling rendering topic.
- Related concepts not yet their own page: **SIFT / SURF** keypoints, **Canny** edge detection, **multilevel B-spline** scattered-data interpolation, **Asynchronous Spacewarp** (Oculus), **plenoptic modeling** / **post-rendering 3D warping** (McMillan & Bishop).

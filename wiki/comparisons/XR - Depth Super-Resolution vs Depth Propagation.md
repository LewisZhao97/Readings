---
title: "Depth Super-Resolution vs Depth Propagation for XR Space-Warp"
sources:
  - "[[raw/articles/Image Guided Depth Super-Resolution for Spacewarp in XR Applications]]"
  - "[[raw/articles/Space Warp with Depth Propagation in XR Applications]]"
tags: [xr, space-warp, depth-map, depth-super-resolution, depth-propagation, bilateral-filter]
created: 2026-04-14
updated: 2026-04-14
type: comparison
---

# Image-Guided Depth Super-Resolution vs Image-Guided Depth Propagation

Two consecutive Samsung Research America papers by Xiong & Peri attacking the same problem — reconstructing a dense depth map on the HMD from a compressed representation sent over wireless — with different device-side sampling and different HMD-side reconstruction. Problem framing, pipeline, and Space-Warp math are shared and documented in [[XR - Depth Reconstruction for Space-Warp]]. This page focuses on the algorithmic diff.

## TL;DR

Paper A (Super-Resolution) is a **joint bilateral filter** on regular-grid-sampled depth. Paper B (Propagation) is a **global optimization** on adaptively-selected sparse points placed at feature/edge/contour locations. B supersedes A: better sample placement and hard anchoring of known-good depths.

## Side-by-Side

| Dimension | A: Super-Resolution (earlier) | B: Depth Propagation (later) |
|---|---|---|
| Paper | Peri & Xiong 2021 (IEEE 9427716) | Xiong & Peri 2021 (IEEE 9666145) |
| Device compression | Regular-grid sub-sample ("binning" — every 2/4/8 px) | Feature-guided sparse points: keypoints + edges + contours |
| Sample placement | Uniform across frame | Concentrated at depth discontinuities |
| Per-point metadata | None | Confidence scores, level tiers |
| Bandwidth adaptivity | Choose scale factor | Choose confidence-level cutoff |
| HMD reconstruction | Joint bilateral filter — per-pixel weighted average | Global optimization: data term + smooth term |
| Anchoring of received samples | None (all pixels reweighted) | Hard anchor via data term |
| Guidance signal | Color image + pose-derived depth | Color image + original depth map |
| Baselines beaten | Bilinear | Bilinear **and** multilevel B-spline |
| Reprojection math | Identical | Identical |
| Relationship | Predecessor | Successor; same framework, improved on both ends |

## Algorithm A — Joint Bilateral Filter (Super-Resolution)

Device sends a regular-grid sub-sampled depth map. HMD seeds a higher-resolution estimate with nearest-neighbor / bilinear, then refines per-pixel:

$$
D_p = \frac{\sum_{q \in N} W_s\, W_r\, W_d\, D_q}{\sum_{q \in N} W_s\, W_r\, W_d}
$$

with three Gaussian weights per neighbor $q$:

- **Spatial:** $W_s = G_{\sigma_s}(\|p-q\|)$
- **Color intensity:** $W_r = G_{\sigma_r}(I_p - I_q)$
- **Depth (pose-derived):** $W_d = G_{\sigma_d}(D_p - D_q)$ — depths computed from 3D poses derived via an assumed camera model on the current depth estimate

Iterate: if the output is not satisfactory, recompute 3D poses from the new depth and refilter. Conceptually a color-guided bilateral filter with a pose-aware depth weight.

**Strengths:** one-shot, GPU-friendly, preserves edges far better than bilinear. At scale factor 0.125, output nearly matches ground truth.

**Weaknesses:** uniform sampling wastes budget on flat regions; received samples aren't anchored — the filter can smear them. Block-like artifacts appear at aggressive sub-sampling (0.015625).

## Algorithm B — Variational Depth Propagation

### Device side: adaptive sparse depth extraction

1. **Multi-scale feature points** — Gaussian pyramid, DoG / LoG per level (SIFT/SURF-style).
2. **Multi-scale edges** — Canny / Sobel per pyramid level.
3. **Object & image contours.**
4. **Depth verification** — drop any candidate whose depth change vs. neighbors is near zero (keep only points that actually signal a depth discontinuity).
5. **Integration & dedup** — merge across detectors, remove overlapping/close points, assign confidence scores.
6. **Level selection** — transmit higher-confidence levels only when the WiFi budget is tight; transmit more for complex scenes. Contour labels are reusable downstream (object recognition, occlusion reasoning).

### HMD side: optimization

Minimize

$$
J(d) = \mu_s W_s \sum_p \|d(p) - d_s(p)\|_2^2 \;+\; \mu_m \sum_{(p,q) \in N} W_{pq}\, \|d(p) - d(q)\|_2^2
$$

- First (data) term: keep the received sparse depths $d_s(p)$ unchanged (weighted L2).
- Second (smooth) term: propagate neighbor depths into each pixel, weighted by $W_{pq}$ derived from color image + original depth map similarity (bilateral-flavored).
- $\mu_s, \mu_m$ balance the terms.

$d(p) = \arg\min_d J(d)$ gives the final dense depth.

**Strengths:** samples placed where depth changes — no wasted budget; hard anchoring prevents drift; global optimization propagates correct depths across larger regions coherently; beats both bilinear and multilevel B-spline.

**Weaknesses:** heavier compute than a local filter (though the paper argues still cheaper than transmitting full-res depth); feature-extraction cost on the device side; no closed-loop bandwidth control specified.

## Why B Improves Over A

- **Adaptive sampling:** a regular grid spends samples on flat regions and misses thin features; feature-guided sampling concentrates bits where they carry information (edges, contours).
- **Hard anchoring:** A's weighted average can smear even the received samples. B's data term pins them, so reconstruction error is strictly in the propagated interior.
- **Global optimization vs. local filter:** A fills each pixel from its neighborhood only; B can propagate a known depth across a flat object region coherently.
- **Stronger baseline beaten:** A only compares against bilinear; B additionally beats multilevel B-spline, which itself beats bilinear.

## When A Might Still Be Preferred

- Strict per-frame compute budget on a very weak GPU where optimization is infeasible.
- No feature-detection budget on the device side.
- Scenes dominated by smooth depth variation where a regular grid is adequate.

In the XR latency regime both papers target, B is the better default.

## Connections

- [[XR - Depth Reconstruction for Space-Warp]] — shared problem framing, pipeline, reprojection math.
- [[XR - Space-Warp]] — the reprojection technique downstream of both methods.
- [[Graphics - Bilateral Filter]] — weighting primitive both methods reduce to.

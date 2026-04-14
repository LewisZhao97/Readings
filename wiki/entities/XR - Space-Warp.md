---
title: "Space-Warp"
sources:
  - "[[raw/articles/Space Warp with Depth Propagation in XR Applications]]"
  - "[[raw/articles/Image Guided Depth Super-Resolution for Spacewarp in XR Applications]]"
tags: [xr, space-warp, reprojection, hmd, remote-rendering]
created: 2026-04-14
updated: 2026-04-14
type: entity
---

# Space-Warp

**Space-Warp** (a.k.a. late-stage reprojection, post-rendering 3D warping) is a technique for re-rendering an already-rendered frame to reflect a fresher head pose, using per-pixel depth to do 3D reprojection rather than re-rendering from scratch. Used in XR to hide rendering latency, particularly in split/remote-rendering setups where the rendered image has to travel from a tethered device or cloud to the HMD.

## Why It Exists

Any gap between when the head pose was sampled and when pixels hit the display produces judder and sickness. Re-rendering the full frame with the newest pose is expensive; Space-Warp instead reprojects the existing color frame using depth and the fresh pose — much cheaper, and fast enough to run on the HMD's GPU every display refresh.

Especially critical for wireless XR: 5G / WiGig (IEEE 802.11ay) enable off-HMD rendering for thin-and-light form factors, but round-trip latency makes un-warped frames unusable.

## Mechanics

Given a rendered color frame, a per-pixel depth map, the head pose at render time $S_r$, and a predicted head pose for the new display time $S_i$:

Normalize screen → NDC:

$$
(x_{ndc}, y_{ndc}, z_{ndc})^T = (2x_{sc}/w - 1,\; 2y_{sc}/h - 1,\; d)^T
$$

Reproject:

$$
(x_p, y_p, z_p, w_p)^T = P\, S_i\, S_r^{-1}\, P^{-1}\, (x_{ndc}, y_{ndc}, z_{ndc}, 1)^T
$$

where $P$ is the projection matrix. Applied per pixel of the source frame.

**Disocclusions** (pixels visible from the new pose but not the old) leave holes that a subsequent hole-fill step patches.

## Dependency on Depth

Space-Warp is only as good as its depth map. Poor depth → blurred boundaries, non-straight lines, ghosting around object silhouettes after warp. In split rendering, transmitting full-resolution depth alongside color roughly doubles bandwidth, so depth is usually compressed and reconstructed on the HMD. See [[XR - Depth Reconstruction for Space-Warp]] for approaches, and [[XR - Depth Super-Resolution vs Depth Propagation]] for the two Samsung methods compared.

## Variants & Lineage

- **Post-rendering 3D warping** — McMillan & Bishop 1997; depth-image-based reprojection for interactive 3D graphics.
- **Plenoptic modeling** — McMillan & Bishop 1995; image-based rendering foundation.
- **Asynchronous Spacewarp (ASW)** — Oculus 2016; runtime reprojection to maintain target frame rate on the Rift.
- **Timewarp** — simpler rotational-only variant that doesn't require depth. Space-Warp generalizes to 6-DOF translation, which does.

## Implementation Notes

- Runs on GPU via shader / compute. In both Samsung papers: OpenGL ES on Snapdragon 865 / Exynos 9825, ~60 fps at 1200×1080.
- Hole-fill pass is required in practice but seldom published in detail.
- Benefits from prediction quality of $S_i$; beyond a certain prediction horizon, warp artifacts exceed the latency cost being hidden.

## Sources

- Xiong & Peri, *Space Warp with Depth Propagation in XR Applications* — uses feature-based sparse depth + propagation to feed Space-Warp.
- Peri & Xiong, *Image Guided Depth Super-Resolution for Spacewarp in XR Applications* — uses joint bilateral super-resolution for the same purpose.

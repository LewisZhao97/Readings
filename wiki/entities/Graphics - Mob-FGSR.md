---
title: "Mob-FGSR"
sources:
  - "[[raw/articles/Mob-FGSR Frame Generation and Super Resolution for Mobile Real-Time Rendering]]"
  - "[[raw/articles/Motion Vector-Based Frame Generation for Real-Time Rendering]]"
tags:
  - mobile-rendering
  - frame-generation
  - super-resolution
  - motion-vectors
  - splatting
  - no-neural-net
  - siggraph-2024
created: 2026-04-20
updated: 2026-04-20
type: entity
---

# Mob-FGSR

**Mob-FGSR** (Yang, Zhu, Zhuge, Qiu, Li, Yuzhong Yan, Xu, Ling-Qi Yan, Jin — SIGGRAPH 2024) is a lightweight, **non-neural** framework for combined **frame generation** and **super-resolution** on mobile GPUs. Same input contract as DLSS/FSR (color + depth + motion vectors at two reference times), but no runtime neural network — entirely classical operators plus offline-learned LUT weights. Ships commercially in OnePlus Ace 3 Pro (June 2024) and OnePlus 13 (October 2024) for *Genshin Impact* and *Honkai: Star Rail* at 120 FPS.

## Why It Exists

DLSS 2 / FSR 2 / DLSS 3 FG depend on features of high-end desktop GPUs: optical-flow hardware on NVIDIA RTX, neural-net throughput on dedicated matrix units. Mobile GPUs have neither. Mob-FGSR asks: under strict no-neural-net-at-runtime and no-optical-flow-hardware constraints, how close can you get? Answer: close enough to halve shading work on shipping phones, and rival DLSS 2 / FSR 2 quality on forward-shaded Unity and deferred-shaded UE scenes.

## Architecture

Three composable stages; two frame-generation modes share the MV reconstruction front end.

### 1. Splat-based Motion Vector Reconstruction (shared front end)

- Inputs at times 0 and 1: color, depth, motion vectors from the game engine's G-buffer.
- **Forward splat** of MVs and depth to an arbitrary intermediate time α. Each pixel's MV, together with depth, is scattered to its predicted destination at time α.
- **Depth-aware atomic resolution:** foreground wins collisions via an atomic depth test (GPU-era analogue of McMillan's occlusion-compatible scan order — same trick used in [[XR - Space-Warp]]).
- **Hole filling:** splatted MV fields show grid-like gaps; a thin-object detector + mean filter fills them.
- Supports both linear and quadratic (uniform-acceleration) motion trajectories.

### 2a. Interpolation model (`Ours-I`)

- Reconstruct bidirectional MVs M₀→α and M₁→α.
- Warp color + depth from frames 0 and 1 forward/backward.
- **Blend** using brightness (B) and depth (D) deltas to classify each pixel:
  - *Shading change* (color differs, depth matches) → lerp both frames.
  - *Disocclusion* (depth differs) → pick from the appropriate single frame to avoid ghosting.
  - *Stable* → prefer temporally-closer frame.

### 2b. Extrapolation model (`Ours-E`)

- Predict single-direction MV M₁→(1+α).
- Backward-warp frame 1 only; no future frame to consult.
- Simplified disocclusion-fill module. Lower quality than interpolation but lower latency.

### 3. Super-Resolution module (`Ours-SR`, composable with either FG mode)

- Standard temporal-accumulation SR: align history SR frame to current via LUT-based backward warp, rectify invalid history pixels, blend with current low-res frame.
- **LUT-based resampling** is the paper's SR innovation: learned 16-tap weights indexed by sub-pixel offsets (dₓ, d_y) live in a lookup table. Matches bicubic quality; faster than bicubic on mobile because the expensive polynomial is precomputed offline and replaced with a table lookup at runtime.
- Composable outputs: `Ours-ISR` = interpolation + SR; `Ours-ESR` = extrapolation + SR.

## On-device Performance

Snapdragon 8 Gen 3, synthetic rendering-load Android app:

| Config | FPS |
|--------|-----|
| No supersampling | ~22 |
| + 2-frame generation | ~50 |
| + FG + 2× super-resolution | >110 |

Roughly **5× throughput** at delivered resolution — the reason it's shipping in consumer phones.

## Design Constraint: No Neural Networks at Runtime

All runtime operations are classical: splatting (atomic scatter), backward warp, mean filter, blend, LUT lookup. Data-driven optimization is offline only — it learns:

- The 16-tap LUT weights for resampling.
- The blend parameters a, b in the SR accumulation equation.

This is *the* load-bearing design choice; everything else follows from it. Neural FG would not run on mobile.

## Compared and Improved-Upon By

- **[[Graphics - Motion Vector-Based Frame Generation]]** (Ha, Ahn, Yoon — Pacific Graphics 2025) explicitly builds on Mob-FGSR's splat-MV front end and replaces the heuristic blend/inpaint back end with a deep-learning flow-refinement decoder. Head-to-head on Ha et al.'s UE4 dataset: Mob-FGSR **35.17 PSNR / 0.979 SSIM**, Ha et al. **37.56 / 0.985**. The gap is concentrated in disocclusions, shadows, reflections, and lighting changes — the non-geometric effects Mob-FGSR's heuristic blend handles bluntly. Mob-FGSR remains vastly faster (~1 ms vs 24 ms on an A6000) — the deep-learning paper is not mobile-real-time.
- Mob-FGSR's own baselines: 3DWarp (Mark et al. 1997), BSR (Yang et al. 2011), AFME (Holmes & Wicks 2020) for FG; TSR (Epic 2022), FSR 2 (AMD 2022), DLSS 2 (Liu 2020) for SR. Reported to outperform lightweight solutions and rival high-end GPU methods.

## Relationship to [[XR - Space-Warp]]

Same input triple (color + depth + MVs) and same forward-scatter primitive (depth-aware atomic splat). Different problem:

- **Mob-FGSR** interpolates *between* two rendered frames at the same pose to halve shading cost in games.
- **Space-Warp** extrapolates *from* one rendered frame to a fresher head pose to hide XR latency.

Both do forward-scatter for the same reasons articulated in the Space-Warp entity (motion anchored at source; depth resolves multi-source-to-one-destination; gather requires an ill-posed inverse).

## Commercial Deployment

- OnePlus Ace 3 Pro (June 2024) — first Android phone to run *Genshin Impact* at stable 120 FPS via "frame prediction."
- OnePlus 13 (October 2024) — extends to *Honkai: Star Rail*.

This is rare for a SIGGRAPH graphics paper and is worth calling out as an exemplar: the design constraints (no neural net, LUT-based ops, forward-only) survived contact with shipping requirements intact.

## Entities / Authors

- Sipeng Yang, Qingchuan Zhu, Junhao Zhuge, Qiang Qiu, Chen Li, Yuzhong Yan, Huihui Xu (OPPO / OnePlus side).
- Ling-Qi Yan (UC Santa Barbara), Xiaogang Jin (Zhejiang University) — academic co-authors.

## Caveat on the source

The KB's current raw file is the project page, not the full SIGGRAPH PDF. Architectural detail (exact blend-module formulation, full loss expressions, per-scene quantitative tables, ablations) is summarized but not reproduced. For deeper distil, fetch `SIGGRAPH_Conf_Mob_FGSR.pdf` linked on `mob-fgsr.github.io`.

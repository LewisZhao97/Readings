---
title: "Frame Generation for Real-Time Rendering"
sources:
  - "[[raw/articles/Mob-FGSR Frame Generation and Super Resolution for Mobile Real-Time Rendering]]"
  - "[[raw/articles/Motion Vector-Based Frame Generation for Real-Time Rendering]]"
tags:
  - frame-generation
  - frame-interpolation
  - motion-vectors
  - real-time-rendering
  - mobile-rendering
  - deep-learning
created: 2026-04-20
updated: 2026-04-20
type: topic
---

# Frame Generation for Real-Time Rendering

**Frame generation** (FG) synthesizes additional frames in between (interpolation) or beyond (extrapolation) the frames actually rendered by the engine, letting the GPU render at fraction-rate while displaying at full rate. All modern approaches consume roughly the same inputs from the engine — **color + depth + motion vectors** — and differ in how they reconstruct the synthetic frame.

## The Input Contract

Rendered FG is distinguished from video frame interpolation (VFI) by what's available from the engine for free:

- **Motion vectors (MVs):** per-pixel NDC-space displacement between frames, derived in the rasterizer pass. Exact (geometric) — not estimated optical flow. Available in every modern engine (UE4/5, Unity, custom).
- **Depth:** linear or NDC depth from the Z-buffer. Enables depth-ordered compositing and parallax-aware reprojection.
- **Color:** the shaded frame itself.

A consequence: rendered FG has *privileged* input that general VFI methods (which must estimate optical flow from color alone) don't. The research consensus — confirmed by the ablation in [[Graphics - Motion Vector-Based Frame Generation]] — is that **engine-provided MVs are a stronger prior than estimated optical flow**, even when the optical-flow model is modern (LiteFlowNet, RAFT). Game FG methods should use the MVs.

## The Two Modes

### Interpolation (bidirectional, one-frame lag)

Given rendered frames at t=0 and t=1, synthesize a frame at t=α (0 < α < 1).

- Pros: highest quality. Both keyframes are known, so bidirectional flow is available; disocclusions can usually be filled from whichever frame sees that region.
- Cons: adds ~1 frame of latency — the player sees t=α after t=1 is available.

### Extrapolation (forward-only, zero lag)

Given frame at t=1, synthesize frame at t=1+α.

- Pros: no added latency; can hide latency (this is the [[XR - Space-Warp]] family).
- Cons: lower quality; disocclusions in the direction of motion must be inpainted without a future reference; quadratic extrapolation error grows with α.

## The Landscape

| Method | Neural runtime? | Mode | Target hardware | Status |
|---|---|---|---|---|
| NVIDIA **DLSS 3 FG** | Yes (+ optical-flow accelerator) | Interpolation | RTX 40+ | Shipping |
| AMD **FSR 3 FG** / AFMF | Partial | Interp / Extrap | GeForce/Radeon | Shipping |
| Intel **XeSS-FG** | Yes | Interpolation | Arc | Shipping |
| [[Graphics - Mob-FGSR]] (SIGGRAPH 2024) | **No** (LUT only) | Interp + Extrap | Mobile (Snapdragon 8 Gen 3) | Shipping (OnePlus Ace 3 Pro, OnePlus 13) |
| [[Graphics - Motion Vector-Based Frame Generation]] (Ha et al., PG 2025) | Yes | Interpolation | Desktop GPU (not yet mobile) | Research |
| **AppSW** / `XR_FB_space_warp` | No | Extrapolation | Quest 2+ | Shipping (OpenXR ext) |
| **ASW 2.0** (Oculus 2019) | No | Extrapolation | Rift PC | Shipping |
| **ExtraNet** [GFL*21] | Yes | Extrapolation | Desktop GPU | Research |
| **NFI** [BDM*21], **KBI** [BDO*23] | Yes | Interpolation w/ target-frame MVs | Desktop GPU | Research |

## The Axes

Two orthogonal axes organize the space:

### 1. Neural vs Classical at Runtime

- **Classical** (Mob-FGSR, AppSW, ASW 2.0, BSR, 3DWarp): splatting, warping, filters, LUT lookups. Runs on any mobile GPU without dedicated ML silicon. Quality ceiling limited by heuristic disocclusion-fill and blend logic.
- **Neural** (DLSS 3 FG, FSR 3, Ha et al., NFI, KBI): convolutional / transformer refinement of flow and/or color. Higher quality, especially on shadows, reflections, lighting, particles — effects MVs don't capture. Currently desktop-GPU only for research; shipping methods use vendor-specific accelerators.

[[Graphics - Motion Vector-Based Frame Generation]] straddles these: uses engine MVs as a classical *prior* inside a neural refinement network. On Ha et al.'s UE4 dataset, this narrows the gap between the two poles (37.56 PSNR vs Mob-FGSR's 35.17 vs AMT-G's 34.43) and suggests the neural pole has a long runway for quality before it saturates.

### 2. Requires-Target-Frame-G-buffer vs Keyframe-Only

- **Target-G-buffer methods** (NFI, KBI, Learnable MVs) render an *additional* G-buffer at the intermediate time to get precise bilateral MVs. Highest quality, but invasive engine integration and expensive.
- **Keyframe-only methods** (Mob-FGSR, Ha et al., DLSS 3 FG, FSR 3 FG, AppSW) take only inputs from the rendered frames. Drop-in integration. Must reconstruct or estimate the intermediate-frame motion — this is where splat-based MV reconstruction (forward-scatter under linear or quadratic motion models) becomes the canonical primitive.

Keyframe-only is winning in practice — it's what every shipping product does. Research (Ha et al.) has shown you don't have to give up much quality by staying keyframe-only if you refine the splatted MVs with a learned network.

## The Canonical Primitive: Depth-Aware Forward Splat

Every keyframe-only method includes a version of this step (ASW/AppSW, Mob-FGSR, Ha et al., post-render warp for cloud gaming):

```
for each pixel p in keyframe:
    destination = splat(p, MV, t)  // t-weighted interpolation or extrapolation
    atomic_min_depth(destination, depth[p])  // foreground wins
    scatter color / MV / etc. to destination
```

Post-processing fills the grid-like gaps splat leaves (thin-object detection + mean filter in Mob-FGSR). It's the same GPU-era analogue of McMillan's *occlusion-compatible scan order* described in the [[XR - Space-Warp]] entity.

## Why Scatter (Not Gather)

Motion is anchored at the **source** keyframe (that's where the MV was computed). Gathering requires the inverse flow, which is ill-posed at occluders and undefined at disocclusions. Interpolation methods with *two* known frames can afford gather (they have bidirectional flow); extrapolation methods with one known frame must scatter. See [[XR - Space-Warp]] §Scatter-not-Gather for the full argument — Mob-FGSR and Ha et al. both ultimately combine the scattered flows with backward warping in the decoder, but the flow *reconstruction* itself is scatter.

## The Non-Geometric-Effect Problem

MVs capture *geometric* motion (vertices transformed between frames). They miss:

- **Shadows** — cast-shadow positions change as objects move, but the shadow-receiving pixels don't have MVs for the shadow.
- **Reflections / glossy highlights** — specular response is view-dependent.
- **Lighting changes** — dynamic sun, moving light sources.
- **Particles, transparency, texture animations** — often composited from multiple depths.

Mob-FGSR handles these with a heuristic brightness+depth classifier that routes each pixel to one of lerp / frame-pick / temporal-preference. Ha et al. show this leaves ~2 PSNR on the table that a learned decoder recovers. The deep-learning pole's biggest quality win, per their "Full model vs w/o Motion Encoder" ablation, is on exactly these non-geometric regions.

## Adjacent Topic: Latency Mitigation via Reprojection

Frame generation shares its core primitive — reprojection under a motion field — with latency mitigation ([[XR - Space-Warp]], [[Graphics - Post-Render Warp]] for cloud gaming). The difference is intent:

- FG interpolates between rendered frames at the same pose → **higher perceived framerate**.
- Space-Warp / late-warp extrapolate from one rendered frame to a fresher pose → **lower perceived latency**.

Both use depth-aware forward splat over per-pixel motion fields. Expect these to converge architecturally as cloud-gaming stacks adopt MV reconstruction for both roles.

## Open Problems

- **Mobile-real-time deep-learning FG.** Ha et al. show a 5.4 M-param network wins on quality but takes 24 ms/1080p on an A6000 — not mobile budget. An architectural search under mobile constraints, or knowledge distillation from Ha et al. back into Mob-FGSR's LUT-based primitives, is an obvious next step.
- **Unified motion representation.** Games, VR, and cloud-gaming late-warp all want MV + depth, but at different time horizons (½ frame / next-pose / 80 ms). A common engine-side contract would let one MV reconstruction feed all three.
- **Disocclusion inpainting under extrapolation.** Shared across AppSW, ASW 2.0, Mob-FGSR extrapolation mode, post-render warp's RN condition. Mostly ad-hoc today.
- **Quality metrics.** PSNR/SSIM don't capture motion artifacts (judder, wobble on thin objects, ghosting on disocclusions). Mob-FGSR and Ha et al. both lean on figure-based qualitative comparison for these — an objective metric would accelerate the field.

## Sources (so far)

- [[Graphics - Mob-FGSR]] — non-neural mobile pole, shipping.
- [[Graphics - Motion Vector-Based Frame Generation]] — deep-learning desktop pole, research.
- [[XR - Space-Warp]] — extrapolation sibling family (VR).
- [[Graphics - Post-Render Warp]] — extrapolation sibling family (cloud gaming).

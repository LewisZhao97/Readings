---
title: "Motion Vector-Based Frame Generation (Ha, Ahn, Yoon)"
sources:
  - "[[raw/articles/Motion Vector-Based Frame Generation for Real-Time Rendering]]"
tags:
  - frame-generation
  - frame-interpolation
  - motion-vectors
  - deep-learning
  - pacific-graphics-2025
  - optical-flow
created: 2026-04-20
updated: 2026-04-20
type: entity
---

# Motion Vector-Based Frame Generation (Ha, Ahn, Yoon)

Ha, Ahn, Yoon (SAIT + KAIST, Pacific Graphics 2025). A **deep-learning frame-interpolation** method for rendered content that uses splatted motion vectors as *guiding flows* inside a learned bilateral-flow estimator. On a custom UE4 gaming dataset, beats Mob-FGSR by **+2.39 PSNR** and beats the strongest video-frame-interpolation baseline (AMT-G) by **+3.13 PSNR** while using ~5× fewer parameters. Not yet mobile-real-time, but positioned as the neural endpoint of the mobile-vs-desktop frame-generation axis.

## Core Idea — MVs as Guiding Flows, Not as Final Flow

Rendered game engines produce exact per-pixel motion vectors for free (from the G-buffer). General video frame interpolation (VFI) doesn't have this and must estimate optical flow — which is unreliable under large or complex motion. Prior rendered-FG work splits into two camps:

1. **Neural approaches that assume target-frame MVs** (NFI, KBI, Learnable MVs) — highest quality, but require rendering an extra G-buffer for the intermediate frame. Awkward to integrate with existing engine interfaces; significant overhead.
2. **No-neural-net approaches using only keyframe MVs** (e.g. [[Graphics - Mob-FGSR]]) — engine-friendly, but heuristic inpaint/blend misses non-geometric effects (shadows, reflections, particles, lighting changes).

Ha et al. take the keyframe-only input contract of Mob-FGSR but feed the splatted MVs into a *deep-learning flow-refinement network* instead of a heuristic blend.

## Architecture

Two parallel feature pyramids feeding a combined decoder.

### Preprocessing (Mob-FGSR-style)

- Input: color `I_0, I_1`, motion vector `M_1`, depth `D_1` from the rendered reference frames.
- Splat `M_1` forward under linear-motion assumption `P_t = P_1 − (1−t)·M_1` to produce bidirectional approximations `M_{t→0}, M_{t→1}`.
- Depth-aware atomic resolution for collisions; disocclusion holes filled directly from `M_1`.
- Warp both images to time t using these splatted MVs.

### Motion Context Encoder (new contribution)

Pyramid encoder `E_m` receives `[W(I_0, M_{t→0}), W(I_1, M_{t→1}), M_{t→0}, M_{t→1}]` at each of 4 levels, producing multi-scale refined guiding flows `m_0^k, m_1^k` and flow features `Φ_t^k`. Channel widths 32 / 48 / 72 / 96. Each block: two 3×3 convs with strides 2 and 1.

The aligned images let the encoder extract *confidence information* about the splatted MVs — essentially, where the MV prior is reliable vs where it needs big residual corrections.

### Image Context Pyramid (standard)

`E_I` extracts a 4-level feature pyramid from each input frame: `X_0^k, X_1^k`.

### Combined Pyramid Decoder (IFRBlock-style)

At each level k:

- Flow merge: `f̄^k = m^k + f^k` — motion-vector prior plus learned task-specific residual.
- Backward-warp image features `X_0^k, X_1^k` using the merged flow.
- IFRBlock: six 3×3 convs + one deconv updates intermediate features `X_t^k` and flow residuals `f_{t→0}^k, f_{t→1}^k`.
- Time embedding `T` (one channel filled with t) is injected at the top level for arbitrary-time interpolation.

Final synthesis uses the standard flow-based VFI formulation:

```
I_t = M · W(I_0, f_{t→0}) + (1 − M) · W(I_1, f_{t→1}) + R
```

with sigmoid merge mask M and 3-channel residual R.

### Losses

- Charbonnier loss `L_char = (x² + ε²)^α` — smooth L2 variant.
- Census loss `L_cen` (Meister et al. 2018) — soft Hamming distance between census-transformed patches; robust to illumination changes.
- `L = λ_char · L_char + λ_cen · L_cen` with both λ = 1.

## Quantitative Results (custom UE4 dataset, 1080p)

| Method | Neural | PSNR / SSIM | Params (M) | Time (ms) |
|---|---|---|---|---|
| Mob-FGSR [YZZ*24] | No | 35.17 / 0.979 | – | ~1 |
| IFRNet-S | Yes | 33.54 / 0.959 | 2.8 | 18 |
| IFRNet-L | Yes | 34.37 / 0.962 | 19.7 | 49 |
| FILM [RKT*22] | Yes | 34.05 / 0.961 | – | – |
| AMT-S | Yes | 33.70 / 0.958 | 3.0 | 38 |
| AMT-G | Yes | 34.43 / 0.963 | 30.6 | 164 |
| **Ours** | Yes | **37.56 / 0.985** | **5.4** | **24** |

Timings on NVIDIA A6000. At 720p: 11 ms/frame, 1.8 GB GPU memory.

## Ablation (same dataset)

| Variant | PSNR / SSIM |
|---|---|
| Baseline (MV splat + equal-weight blend) | 34.54 / 0.976 |
| w/ External flows (LiteFlowNet) instead of MVs | 35.16 / 0.976 |
| w/o Motion encoder (raw splatted MVs into decoder) | 35.42 / 0.974 |
| w/o Aligned input images in motion encoder | 35.82 / 0.976 |
| Full model | **37.56 / 0.985** |

Two publishable-in-themselves observations:

1. **Rendered MVs > estimated optical flow as a prior.** The "External Flows" variant feeds LiteFlowNet-estimated optical flow instead of game-engine MVs; it *beats* IFRNet-L (35.16 vs 34.37) but *loses* to the MV version (35.16 vs 37.56). Game engines should prefer their own MVs over learned flow — a clear KB-level finding on the optical-flow-vs-motion-vector distinction.
2. **The motion encoder carries most of the win.** Going from the "w/o Motion Encoder" variant (35.42) to the full model (37.56) is +2.14 dB — the biggest single ablation delta. The aligned-image input to that encoder is the second-biggest (+1.74 dB).

## Dataset

Seven UE4 Marketplace scenes, 1080p, 9× MSAA ground truth:

- **Training:** Factory (8), Infiltrator (12), Subway Train (4), Sun Temple (4), Zen Garden (5) — sequences of 200 triplets each.
- **Test:** Abandoned Apartment (2), Factory (4), Infiltrator (7), Subway Sequencer (4), Subway Train (2), Sun Temple (3).

Motion vectors and depth were exported via custom compute shaders integrated into the UE4 renderer. Crucially, **target-frame G-buffers are not used** — simulating the standard engine workflow where only keyframe MVs are available.

## Failure Modes (authors' own)

Thin structures under large displacement (e.g., a pole undergoing big motion) still produce artifacts. Authors attribute this to neural capacity, not algorithm design, and flag mobile-hardware optimization as future work.

## Relationship to Adjacent Entities

- **[[Graphics - Mob-FGSR]]** — direct progenitor. Ha et al. explicitly describe their front end as "Following the approach of [YZZ*24]." The paper is effectively "Mob-FGSR front end + deep-learning back end."
- **[[XR - Space-Warp]]** — shares the depth-aware splat primitive but for extrapolation-to-pose, not interpolation-between-frames.
- **[[Graphics - Neural Rendering]]** — sits at the same neural-vs-classical axis, on the neural side; Mob-FGSR is the non-neural counterpoint.

## References Cited

Mob-FGSR [YZZ*24], IFRNet [KJL*22], AMT [LZH*23], FILM [RKT*22], RAFT [TD20], LiteFlowNet [HTL18], PWC-Net [SYLK18], FastFlowNet [KSY21], FlowNet [FDI*15], SPyNet [RB17], Softmax splatting [NL20], BMBC [PKLK20], DLSS 2 [Liu20], ExtraNet [GFL*21], NFI [BDM*21], KBI [BDO*23], Learnable MVs [WZH*23], ExtraSS [WKZ*23], Temporally Reliable MVs for Ray Tracing [ZLY*21], Neural Supersampling [XNC*20], Layered Depth Images [SGHS98].

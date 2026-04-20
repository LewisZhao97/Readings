---
title: "Motion Vector-Based Frame Generation for Real-Time Rendering — Summary"
sources:
  - "[[raw/articles/Motion Vector-Based Frame Generation for Real-Time Rendering]]"
tags:
  - frame-generation
  - frame-interpolation
  - motion-vectors
  - deep-learning
  - real-time-rendering
  - pacific-graphics-2025
  - optical-flow
  - game-engines
created: 2026-04-20
updated: 2026-04-20
type: summary
recommend: yes
distil_times: 1
---

# Motion Vector-Based Frame Generation — Summary

Ha, Ahn, Yoon (SAIT + KAIST, Pacific Graphics 2025). A **deep-learning** frame-interpolation method for rendered content that uses splatted motion vectors as **guiding flows** inside a learned bilateral-flow estimator — explicitly improving on Mob-FGSR and beating IFRNet/AMT/FILM on a custom UE4 gaming dataset (37.56 PSNR vs Mob-FGSR's 35.17, +3.13 dB over AMT-G). 5.4 M params, 24 ms per 1080p frame on an A6000.

## Key points

- **Problem framing:** standard VFI networks (IFRNet, AMT, FILM) estimate bilateral flows from image pyramids; they struggle with *large motions* and *thin fast objects* because pyramid depth limits the receptive field, and early-stage errors compound. Rendering has motion vectors for free — but MVs alone miss non-geometric effects (shadows, reflections, particles, lighting, transparency) and have errors at occlusions/disocclusions.
- **Core contribution — MVs as guiding flows, not as final flow:**
  - Follow Mob-FGSR for the front end: splat frame-1 MVs forward to target time *t* (linear-motion assumption, `Pₜ = P₁ − (1−t)·M₁`), depth-aware atomic resolution of splat collisions, MVs from keyframes fill disocclusions.
  - *Departure:* splatted MVs are treated as **rough priors**, not ground truth. Feed into a **motion context encoder** alongside the pair of images warped to *t* using those splatted MVs. Encoder produces multi-scale refined flows `m_0^k, m_1^k` and flow features `Φ_t^k`.
- **Image context pyramid + combined decoder:**
  - Parallel image-context encoder produces multi-scale features `X_0^k, X_1^k`.
  - Pyramid decoder (IFRBlock style from IFRNet) jointly refines intermediate features `X_t^k` and residual bilateral flows `f_{t→0}^k, f_{t→1}^k`. At each level, the merged flow is `f̄^k = m^k + f^k` — motion-vector prior plus learned task-specific residual.
  - Decoder receives warped image features, intermediate features, motion features, and a time embedding T. Final synthesis follows standard flow-based VFI: `I_t = M·W(I_0, f_{t→0}) + (1−M)·W(I_1, f_{t→1}) + R` with mask M and residual R.
- **Quantitative results (custom UE4 dataset, 7 scenes, 1080p):**
  - Ours: **37.56 PSNR / 0.985 SSIM**, 5.4 M params, 24 ms.
  - Mob-FGSR: 35.17 / 0.979, ~1 ms (no NN, no flow, so can't handle shadows/reflections/disocclusion cleanly).
  - AMT-G: 34.43 / 0.963, 30.6 M params, 164 ms.
  - IFRNet-L: 34.37 / 0.962, 19.7 M, 49 ms.
  - FILM: 34.05 / 0.961.
  - The Baseline (MV splat + equal-weight blend, no NN) hits 34.54 / 0.976 — already beating IFRNet-L. Shows how strong rendered-side MVs are as a prior.
- **Ablation (all on the same dataset):**
  - Baseline (MV splat, direct blend): 34.54.
  - w/ External flows from LiteFlowNet instead of MVs: 35.16 — MV priors beat estimated optical flow.
  - w/o Motion encoder (inject raw splatted MVs directly into pyramid decoders): 35.42.
  - w/o Aligned input images in motion encoder: 35.82.
  - Full: 37.56. Every component contributes; motion-encoder refinement is the largest single lever (+2.14 dB over the no-encoder variant).
- **Dataset:** 7 UE4 Marketplace scenes (Abandoned Apartment, Factory, Infiltrator, Subway Sequencer, Subway Train, Sun Temple, Zen Garden), 1080p, 9× MSAA ground truth, custom compute-shader MV + depth export. Target-frame G-buffers are deliberately **not** used (simulating standard commercial rendering workflow; MVs/depth only for I-frames).
- **Limitations** (authors' own): large-motion thin structures (e.g., a thin pole undergoing big displacement) still produce artifacts; neural capacity, not algorithm design, is the ceiling. Deployment optimization across hardware is future work.
- **Relationship to the KB's ongoing thread:** directly builds on Mob-FGSR's splat-MV front end (same preprocessing step, explicitly cited), replaces the heuristic blend/inpaint back end with a learned flow-refinement decoder. Pushes the "frame generation for real-time rendering" axis from *no-NN-at-runtime* (Mob-FGSR) to *deep-learning-on-desktop-GPUs* — not yet mobile-real-time at 24 ms on an A6000, but the authors flag lightweighting as the path forward.

## Entities mentioned

- **Authors / affiliations:** Inwoo Ha (SAIT + KAIST), Young Chun Ahn (SAIT — Samsung Advanced Institute of Technology), Sung-eui Yoon (KAIST).
- **Compared methods:** IFRNet [KJL*22] (three sizes S/B/L), AMT [LZH*23] (S/L/G), FILM [RKT*22], Mob-FGSR [YZZ*24], LiteFlowNet (as external-flow baseline).
- **Related methods cited:** DLSS 2 [Liu20], DLSS 3, FSR 2/3, Intel XeSS, TAAU, ExtraNet [GFL*21], NFI [BDM*21], KBI [BDO*23], Learnable Motion Vectors [WZH*23], ExtraSS [WKZ*23], RAFT [TD20], PWC-Net, FlowNet, SPyNet, FastFlowNet, Temporally Reliable MVs for Ray Tracing [ZLY*21].
- **Engines / hardware:** Unreal Engine 4, NVIDIA A6000 (4× for training, 1× for inference timing), AdamW optimizer, Charbonnier + census losses.

## Related topics (existing wiki)

- **[[Graphics - Mob-FGSR]]** — direct predecessor whose splat-MV preprocessing is reused verbatim. This paper is effectively "what if we keep Mob-FGSR's front end but replace its heuristic back end with a learned flow-refinement network." Strongest cross-link.
- **[[XR - Space-Warp]]** — shares the "use depth + MVs to synthesize a frame at a new time" structure, but Space-Warp does scatter-based extrapolation from a single frame for VR reprojection, whereas this is gather-based interpolation between two frames for games. Useful contrast.
- **[[Graphics - Neural Rendering]]** — this sits at the same neural-vs-classical axis Mob-FGSR also plotted; Ha et al. are now the "neural" endpoint on that axis for FG specifically.
- **The pending `Graphics - Frame Generation for Real-Time Rendering` topic** (flagged in the Mob-FGSR summary) — Ha et al. is a second load-bearing entry for that page.

## Potential new wiki pages

- **Entity:** `Graphics - Motion Vector-Based Frame Generation (Ha et al.)` — method architecture (motion context encoder, image context pyramid, combined decoder, flow merge), quantitative results, ablation table, dataset spec.
- **Topic:** `Graphics - Frame Generation for Real-Time Rendering` — broad area spanning DLSS 3 FG, FSR 3 FG, Mob-FGSR, Ha et al., AppSW, ExtraNet. Ha et al. provides a clean neural vs non-neural axis.
- **Comparison candidate:** `Mob-FGSR vs MV-Based Frame Generation (Ha et al.)` — natural pairing; paper already quantifies the gap on a shared dataset.
- **Glossary:** Motion Splatting, Bilateral Flow, Disocclusion, IFRNet/IFRBlock, Optical Flow vs Motion Vector (the paper's whole framing pivots on this distinction).

## Recommend Reason

**`yes`** — distil-worthy and very timely:

1. **Directly advances the 2026-04-20 frame-generation thread.** We just ingested Mob-FGSR as the non-neural mobile pole; this is the deep-learning desktop pole on the same problem. Head-to-head numbers on a shared dataset are already in the paper — a comparison page almost writes itself.
2. **Self-contained with a clear architecture and strong ablations.** Unlike Mob-FGSR (where the raw is only the project page), this raw is the full paper body including quantitative results, loss formulations, and four ablation variants. Distil has everything it needs.
3. **Cross-links an important sub-topic: MV-as-prior vs estimated-optical-flow.** The "External Flows" ablation (35.16 with LiteFlowNet vs 37.56 with MVs) is a crisp, publishable-in-itself finding on why game engines should prefer their own MVs over learned flow — worth surfacing as its own glossary entry or topic paragraph.

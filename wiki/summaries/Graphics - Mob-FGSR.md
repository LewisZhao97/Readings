---
title: "Mob-FGSR: Frame Generation and Super Resolution for Mobile Real-Time Rendering — Summary"
sources:
  - "[[raw/articles/Mob-FGSR Frame Generation and Super Resolution for Mobile Real-Time Rendering]]"
tags:
  - mobile-rendering
  - frame-generation
  - super-resolution
  - motion-vectors
  - real-time-rendering
  - siggraph-2024
  - splatting
  - no-neural-net
created: 2026-04-20
updated: 2026-04-20
type: summary
recommend: yes
distil_times: 0
---

# Mob-FGSR — Summary

Yang, Zhu, Zhuge, Qiu, Li, Yan, Xu, Yan, Jin (SIGGRAPH 2024). A lightweight, **non-neural** framework for combined frame generation and super-resolution, purpose-built for mobile GPUs that lack the optical-flow hardware and ML acceleration DLSS/FSR2/XeSS rely on. Ships commercially — OnePlus Ace 3 Pro (Jun 2024) and OnePlus 13 (Oct 2024) use it to run *Genshin Impact* / *Honkai: Star Rail* at 120 FPS. Two runtime modes (interpolation and extrapolation) + an SR module that plugs into either.

> ⚠️ **Source limitation:** the raw file is the project page (`mob-fgsr.github.io`), not the full SIGGRAPH PDF. Method overview and results are summarized; architectural detail, loss formulations, full ablations, and per-scene numbers are absent. Any `/distil` should fetch the PDF linked on the page (`SIGGRAPH_Conf_Mob_FGSR.pdf`) before doing deep synthesis.

## Key points

- **Problem framing:** DLSS 2 / FSR 2 / TSR assume high-end desktop GPUs with dedicated hardware (optical-flow acceleration on RTX; sufficient compute for neural nets). Mobile GPUs have neither. The paper targets *the same supersampling problem* (halve shading work, upscale + interpolate) under mobile compute constraints and **without neural nets at runtime**.
- **Inputs required from the renderer:** color, depth, motion vectors (MVs) at two reference times (frames 0 and 1). Same input shape as DLSS 2 / FSR 2 / AppSW — the intent is drop-in replaceability on the engine side.
- **Core technical contribution — splat-based MV reconstruction:**
  - MVs and depth from I-frames 0 and 1 are **scattered (splatted) forward** to synthesize MVs at an arbitrary intermediate time α.
  - Depth-aware atomic splatting ensures foreground pixels win over background at collisions.
  - Splatted MV fields naturally have grid-like gaps; a thin-object detector + mean-filter fills them.
  - Supports both linear and quadratic (uniform-acceleration) motion trajectories.
- **Two runtime models (share the MV reconstruction front end):**
  1. **Interpolation** — synthesize bidirectional MVs M₀→α and M₁→α, warp both real frames forward/backward, blend them. Blend logic uses depth (D) and brightness (B) deltas to classify each pixel as *shading change* (lerp), *disocclusion* (pick from the appropriate frame to avoid ghosting), or *stable* (prefer temporally-closer frame).
  2. **Extrapolation** — predict single-direction MV M₁→(1+α), backward-warp frame 1, then run a simplified disocclusion-filling module. No access to a future frame, so artifacts are inherently worse — this is the latency-hiding mode.
- **Super-resolution module** — standard temporal-accumulation SR: align history SR frame to current via LUT-based backward warp, rectify invalid history pixels, blend with current low-res frame. Innovation is the **LUT-based resampling**: learned 16-tap weights indexed by sub-pixel offsets (dₓ, d_y) in a lookup table — matches bicubic quality but is faster to evaluate on mobile. SR cleanly composes with either FG mode → `Ours-ISR` and `Ours-ESR`.
- **No neural networks at runtime.** Data-driven optimization is used *offline* to learn LUT weights and blend parameters, but the runtime path is all classical ops (splatting, warping, filtering, LUT lookup). This is the entire reason it runs on mobile.
- **Reported on-device performance (Snapdragon 8 Gen 3):**
  - Baseline (no SS): ~22 FPS under the synthetic rendering load.
  - + 2-frame generation: ~50 FPS.
  - + FG + 2× super-resolution: >110 FPS.
  - Roughly 5× throughput at delivered resolution.
- **Evaluation baselines:** against 3DWarp [Mark et al. 1997], BSR [Yang et al. 2011], AFME [Holmes & Wicks 2020] on FG; TSR [Epic 2022], FSR 2 [AMD 2022], DLSS 2 [Liu 2020] on SR. Unity scenes (forward shading: Street View, Meadows, Hilly Area, Dragon Park) + UE scenes (deferred: Bunker, Western Town).
- **Commercial deployment (rare for SIGGRAPH graphics):** shipped in OnePlus Ace 3 Pro (June 2024) and OnePlus 13 (October 2024), marketed as "frame prediction" for *Genshin Impact* at 120 FPS.

## Entities mentioned

- **Authors / affiliations:** Yang, Zhu, Zhuge, Qiu, Li, Yuzhong Yan, Xu (OPPO / OnePlus presumably — ships in OnePlus phones), Ling-Qi Yan (UC Santa Barbara), Xiaogang Jin (Zhejiang University).
- **Compared techniques:**
  - Frame generation: 3DWarp (Mark et al. 1997), BSR (Yang et al. 2011), AFME (Holmes & Wicks 2020).
  - Super-resolution: TSR (Epic's temporal super-resolution in Unreal, 2022), FSR 2 (AMD FidelityFX Super Resolution 2, 2022), DLSS 2 (NVIDIA Deep Learning Super Sampling 2, Liu 2020).
- **Platforms:** Qualcomm Snapdragon 8 Gen 3 (benchmark target); Unity (forward shading scenes); Unreal Engine (deferred shading scenes).
- **Commercial products:** OnePlus Ace 3 Pro, OnePlus 13, Genshin Impact, Honkai: Star Rail.

## Related topics (existing wiki)

- **Very close to the current thread** — this is the academic / mobile-phone analogue of what AppSW does for VR. Same input shape (color + depth + MVs), same goal (halve render load), different execution (splatting + LUT vs rasterizer-pass MVs + runtime PTW). The 2026-04-20 feed (`raw/feeds/2026-04-20.md`) asked for an AppSW ↔ ASW cross-compare; Mob-FGSR is a third interesting axis on that same diagram — *application-level* frame generation without specialized hardware.
- `[[XR - Space-Warp]]` — AppSW on Quest is a close cousin, especially the motion-vector extrapolation path. Mob-FGSR isn't VR, so pose-correction / TimeWarp coupling isn't relevant, but the frame-synthesis pipeline is very similar.
- `[[XR - Depth Reconstruction for Space-Warp]]` — Xiong & Peri also target mobile-friendly depth handling; different problem (depth bandwidth reduction for remote rendering) but shares the "mobile GPU can't do what desktop GPU can" framing.
- `[[Graphics - Neural Rendering]]` — Mob-FGSR is the *classical* pole of a neural-vs-classical axis; a good contrast piece for the neural-rendering topic page.
- `[[Graphics - Bilateral Filter]]` — the blend-logic section (use brightness + depth to classify pixels) is a close cousin to bilateral filtering's intensity-space + spatial-space weighting.

## Potential new wiki pages

- **Entity:** `Graphics - Mob-FGSR` — the framework, splat-based MV reconstruction, interpolation vs extrapolation pipeline, LUT-based SR, on-device numbers.
- **Topic:** `Graphics - Frame Generation for Real-Time Rendering` — broad area covering DLSS 3 FG, FSR 3 FG, AFMF, Mob-FGSR, AppSW (as an XR-specific variant). Could accumulate fast if more FG sources are ingested.
- **Topic update:** extend the pending AppSW/ASW cross-compare (if created) with a third column for Mob-FGSR — same problem shape, non-VR mobile.
- **Glossary:** Motion Splatting, Disocclusion, Frame Generation, Super-Resolution (graphics sense), Temporal Accumulation.
- **Comparison candidate:** `Mob-FGSR vs DLSS 2 vs FSR 2` — the paper's own §Results material would bootstrap this, though full numbers require the PDF.

## Recommend Reason

**`yes`** — distil-worthy for two reasons:

1. **Directly complements the current AppSW/ASW investigation.** The 2026-04-20 feed explicitly asked for an AppSW ↔ ASW cross-compare; Mob-FGSR broadens the axis to *non-VR mobile* frame generation, and its splat-based MV reconstruction is a technically distinct approach from both ASW's video-encoder block matching and AppSW's rasterizer-pass MVs. A comparison page that didn't mention it would be incomplete.
2. **Rare SIGGRAPH-to-production story.** Most SIGGRAPH graphics papers never ship; this one runs *Genshin Impact* at 120 FPS on consumer phones within months of publication. That makes the paper's design constraints (no neural net at runtime, LUT-based ops, mobile-first) unusually load-bearing — worth capturing as an entity so later KB entries can point at it as an exemplar of "academic work that survived contact with shipping constraints."

**Caveat for distil:** the raw file is only the project page. Before deep distil, fetch the actual SIGGRAPH PDF (linked as `SIGGRAPH_Conf_Mob_FGSR.pdf` from the project page) — that's where the architecture details, loss formulations, per-scene quantitative results, and ablations live. The project page is enough to summarize; it's not enough to distil into high-quality entity/topic pages.

Skip-list for distil focus: the marketing-style OnePlus commercial-application section (note it as deployment evidence, don't try to synthesize technical content from phone product pages).

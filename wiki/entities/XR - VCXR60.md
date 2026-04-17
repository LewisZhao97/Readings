---
title: "VCXR60 Dataset"
sources:
  - "[[raw/articles/Vibe Coding XR Accelerating AI + XR Prototyping with XR Blocks and Gemini]]"
tags:
  - xr
  - ai-xr
  - benchmark
  - llm-code-generation
  - vibe-coding
created: 2026-04-17
updated: 2026-04-17
type: entity
---

# VCXR60

A pilot benchmark for intent-driven XR code generation, introduced alongside [[XR - Vibe Coding XR]] (Du, Hersh, Li, Numan et al. 2026). Sixty natural-language XR prompts + an automated headless-browser harness that executes generated code and reports a pass / fail verdict.

## Dataset composition

- **60 prompts** sourced from four one-hour workshops with **20 participants** total.
- Prompts span toys (*"make a bunch of bubbles that pop when I touch them"*, Prompt 004) through detailed system specifications (Prompt 060 "Superpower Hands" is ≈200 words of particle-physics, cone-attraction geometry, billboarded sparkle quads, depth occlusion, and a fireworks finale).
- Category coverage visible in the prompt list (Appendix A): educational (Euler's theorem, chemistry, guitar tab tutor, planetary gearbox, four-bar linkage, math AR graph), games (Chrome Dino XR, tower defense, ping pong, butterfly catching, Space Invaders, Pacman, escape room, stickman sketch), ambient / aesthetic (chill cats, EDM concert, chunky holographic horse, matrix mesh, art exhibition), tools (XR pomodoro, sticky notes, home decorator, yoga instructor, plant doctor), and physics-heavy (marble run, waterfall, weather, voxel garden, vine growth).

## Evaluation harness

- **Runtime:** headless Chromium driven by [Playwright](https://playwright.dev).
- **Pass definition:** zero runtime errors during generated-code execution. Authors justify this by noting that in 3D graphics *"the rendering engine itself serves as an implicit correctness validator — runtime errors during rendering indicate code failures"* (§4).
- **Metric:** pass@1, following the HumanEval [Chen et al. 2021] tradition.
- **Protocol:** each prompt run 5 times; error logs monitored; a "pass" is zero-error execution.

## The rendering-engine-as-implicit-validator pattern

Worth highlighting as a transferable idea. For a codegen target with a strict runtime (WebGL rendering, shader compilation, WebXR session lifecycle), the runtime's own error output acts as a correctness oracle. This sidesteps the unit-test authorship problem that usually makes codegen benchmarks expensive to build: HumanEval needed hand-written tests; VCXR60 needed none — the browser's error console is the test. The trade-off: the oracle catches hard failures but not semantic mismatches (a volleyball that looks like a medicine ball would still "pass").

## Baseline results (XR Blocks v0.11.0)

Measured in 5 runs each, averaged:

| Model | Thinking | Median time (s) | IQR | Pass@1 |
|-------|----------|----------------:|----:|-------:|
| gemini-3.1-pro | High | 86.02 | 32.12 | **95.5%** |
| gemini-3.1-pro | Low  | 33.39 |  8.60 | 94.1% |
| gemini-3-flash | High | 22.26 |  4.66 | 87.8% |
| gemini-3-flash | Low  | 17.30 |  4.00 | 87.4% |

Every prompt achieved ≥1 successful run across the 5 attempts. Pattern: smaller/faster models are "good enough" for simple prompts (dandelion) but drop off on complex multi-system prompts (Superpower Hands, Neon Dodge Arena). The cost–quality knob is whether you pay ~86 s median latency for Pro+High to get an extra 8 pp of pass rate.

## Iteration story

- **XR Blocks v0.1.0:** ~70% pass@1.
- **10 major releases over 6 months**, driven by error-log analysis of which prompts failed and why.
- **v0.11.0:** 95.5% pass@1 (gemini-3.1-pro, High).

Failure-mode breakdown (from §4): most early errors were (a) framework edge-case bugs surfacing under LLM-generated code rather than (b) pure model hallucination. Framework polish + tighter grounding closed the gap together.

## Stated limitations

- **Limited coverage** — 60 prompts is small; doesn't span the AI + XR design space.
- **Viability ≠ quality** — pass/fail doesn't measure usability, aesthetics, or interaction fidelity.
- **One-shot only** — no evaluation of iterative / multi-turn refinement scenarios.
- **No ablation** — doesn't separate the contribution of the XR Blocks framework from the underlying three.js, nor does it quantify the token savings relative to raw WebGL/WebXR API usage.
- **No isolation of model vs framework improvement** — the 70% → 95.5% jump over 6 months could be framework fixes, model upgrades (Gemini 3.1-pro post-dates v0.1.0), or both.

## Related

- [[XR - Vibe Coding XR]] — the system whose viability this benchmark was designed to measure.
- [[XR - XR Blocks]] — the framework under test.
- [[LLM - Code Generation Benchmarks]] — the lineage it sits in.

---
title: "Vibe Coding XR"
sources:
  - "[[raw/articles/Vibe Coding XR Accelerating AI + XR Prototyping with XR Blocks and Gemini]]"
tags:
  - xr
  - ai-xr
  - vibe-coding
  - llm-code-generation
  - webxr
  - xr-blocks
  - gemini
created: 2026-04-17
updated: 2026-04-17
type: entity
---

# Vibe Coding XR

An end-to-end rapid-prototyping workflow by Google XR Labs (Du, Hersh, Li, Numan et al. 2026) that translates a natural-language prompt into a running WebXR application in under 90 seconds. Built on top of [[XR - XR Blocks|XR Blocks]] (`xrblocks.js` v0.11.0) and Gemini (tested through v3.1-pro).

- **Web UI / "XR Blocks Gem":** https://xrblocks.github.io/gem
- **Source:** https://github.com/google/xrblocks
- **Companion paper to:** [[XR - XR Blocks]] (the earlier vision/white paper). Vibe Coding XR is the measured, working implementation of what that paper sketched under "Vibe Coding for XR."

## User flow

1. User types a prompt (e.g. *"create a beautiful dandelion that blows away when I pick it up"*) into the XR Blocks Gem web interface, running in Chrome on a Galaxy XR headset.
2. Gemini generates a single-file `index.html` WebXR experience, grounded in the XR Blocks API and templates (see [System Prompt Architecture](#system-prompt-architecture)).
3. The user reviews reasoning + code, then presses **Enter XR** to deploy to the headset, or previews on the desktop **simulated reality** environment first.

## Why this works: the grain-alignment argument

Previous LLM-XR codegen attempts (LLMR, DreamCodeVR, DreamGarden) target Unity or Unreal, where the API surface is a massive deeply-nested syntax tree. The paper's claim: LLMs hallucinate over such surfaces because the *grain* of the API does not match the grain of LLM reasoning tokens.

The fix is not a better model — it is an **abstraction layer whose API surface is semantically dense and matches natural-language concepts 1:1**. This is the [[XR - XR Blocks#the-reality-model|Reality Model]]: `user`, `world`, `agents`. A prompt like "make the ball react to my hand and the room geometry" translates almost line-by-line into XR Blocks calls, without the LLM having to reconstruct how `Rigidbody` in Unity interacts with `XRHandTrackingSubsystem`.

## System Prompt Architecture

The system prompt (Appendix C of the paper) instructs Gemini as "an expert Creative Technologist specialized in three.js, WebXR, and the XR Blocks (xrblocks) library." Four components, applied in order:

1. **Persona & guidelines** — domain-expert persona, room-scale XR best practices (spatial layout, human-scale proportions, interaction distances).
2. **Dependency management** (marked *Critical* in the prompt) — explicit version pinning. *"Use the specific versions below. Do NOT hallucinate newer versions."* Prevents the failure mode where the model invents a plausible-looking import that points to a non-existent version.
3. **Coding standards** — strict architectural constraints:
   - Single-file `index.html`.
   - All logic inside `class MyScript extends xb.Script`, within `<script type="module">`.
   - Use `init()` / `update()` lifecycle; do NOT call `requestAnimationFrame` manually.
   - Interactions via `onSelectStart` / `onSelectEnd` / `onSelecting`.
   - `y` is up, `z` is forward; place objects at `z = -this.user.objectDistance`.
   - Spatial UI in 3D, not 2D overlay divs.
4. **Reference examples** — curated templates, samples, demos, gallery examples from XR Blocks. This is the "grounding corpus": full source code of `xrblocks.js` and 34 open-source templates are inlined. Acts as a technical constraint ensuring API adherence.
5. **Planning directive** — before emitting code, outline `xb.Script` class structure, members needed, and `init()` vs `update()` logic.

The paper's framing: these four pieces together *teach* the Reality Model to the LLM inside one context window — persona sets behavior, dependency rules + coding standards set invariants, reference examples provide grounded exemplars, and the planning directive forces explicit intermediate reasoning.

Full prompt + XR Blocks examples: https://xrblocks.github.io/prompts.

## Simulated-to-Extended Reality Loop

A core HCI contribution. Creators iterate on generated scripts in a **desktop "simulated reality"** environment — a browser window with simulated hand/body inputs — before deploying to a physical Android XR headset. Directly targets the "constant don/doff" friction that plagues XR prototyping cycles: visual validation of physics, logic, and layout happens at desktop speed, headset testing is reserved for genuinely embodied aspects (gesture feel, depth-collision, room-scale UX).

## Application scenarios (from §3.4)

Generated from single prompts, working on the first shot:

| Scenario | Prompt sketch | Notable system use |
|----------|---------------|---------------------|
| **AS1 Math Tutor** | *"visualize Euler's theorem in geometry and explain vertices, edges, and facets"* | LLM autonomously picked tetrahedron, cube, octahedron; pinch to trigger highlighting |
| **AS2 Physics Lab** | Interactive scale with labeled weights | Physics + labels + pinch-and-drop |
| **AS3 Immersive Chemistry** | *"ignite methane / ethylene / acetylene"* | 3D volumetric effects, educational cards |
| **AS4 XR Sports** | *"let me play volleyball with hands and collide with my environment"* | Textured ball, physical-hand collision, room-geometry collision via depth mesh |
| **AS5 Chrome Dino XR** | *"create the Chrome Dino game in XR"* | Voxelized characters, translucent lane, audio cues |

The range — from a straightforward dandelion (AS-null, Figure 1) to the 60th VCXR60 prompt ("superpower hands," ≈200 words specifying particle physics, cone-attraction geometry, fireworks finale) — is wide enough that the 95.5% pass rate covers genuine system-building, not just toy demos.

## Evaluation — see [[XR - VCXR60]]

Pass@1 and latency baselines are reported against the new [[XR - VCXR60|VCXR60 benchmark]]:

| Backend | Thinking | Median time | IQR | Pass@1 |
|---------|----------|-------------|-----|--------|
| gemini-3.1-pro | High | 86.02 s | 32.12 | **95.5%** |
| gemini-3.1-pro | Low  | 33.39 s |  8.60 | 94.1% |
| gemini-3-flash | High | 22.26 s |  4.66 | 87.8% |
| gemini-3-flash | Low  | 17.30 s |  4.00 | 87.4% |

All averaged over 5 runs on 60 prompts. Every test prompt achieved ≥1 successful run across the 5 runs.

The authors report the trajectory matters more than the absolute number: XR Blocks v0.1.0 was **~70% pass@1**; after 10 major releases over 6 months (informed by error-log analysis), v0.11.0 hits 95.5%. Most early failures were (a) framework bugs surfacing under generated code and (b) LLM API hallucinations of non-existent methods. Fixing (a) and constraining (b) via tighter grounding closed the gap.

## Stated limitations (§5)

- **Stochastic failures** — LLMs occasionally misinterpret physical constraints, produce invalid syntax, or fail complex interaction logic. Larger models + more thinking mitigate; don't eliminate.
- **Platform trade-offs** — WebXR can't match Unity/Unreal throughput; cloud LLMs add network latency. Future work: LLM-driven cross-compiler to native engines (Godot mentioned).
- **Viability ≠ quality** — pass@1 metric captures only "does it run without runtime errors." Doesn't assess usability, aesthetics, or interaction fidelity.
- **One-shot only** — multi-turn iterative refinement is untested; may expose distinct failure modes in context management and incremental code coherence.
- **Doesn't isolate XR Blocks contribution** — no ablation measuring tokens saved vs raw three.js API usage.
- **Framework gaps** — no ready-to-use accessibility or safety modules.

## Envisioned next steps

- ImageNet-scale prompt-to-interaction dataset for XR.
- Multimodal prompting: gaze, micro-gestures, cross-device interactions as additional guidance channels beyond text.
- Hybrid natural-language + spatial-UI prompting.
- Human-in-the-loop co-creation studies.
- LLM-driven cross-compilation from XR Blocks scripts to native-engine code.

## Related

- [[XR - XR Blocks]] — the framework this workflow depends on; Vibe Coding XR is its "proof of shipping."
- [[XR - VCXR60]] — the benchmark accompanying this workflow.
- [[XR - AI + XR Integration]] — broader topic; this is the highest-signal concrete datapoint in that area to date.
- [[LLM - Code Generation Benchmarks]] — lineage: HumanEval → VCXR60.
- [[LLM - Meta-Harness]] / [[LLM - Harness Engineering]] — same pattern (system prompt + grounding corpus + constrained output), different target domain (XR scripts vs. SWE-bench tasks).

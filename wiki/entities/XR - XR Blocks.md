---
title: "XR Blocks"
sources:
  - "[[raw/articles/XR Blocks Accelerating Human-Centered AI + XR Innovation]]"
  - "[[raw/articles/Vibe Coding XR Accelerating AI + XR Prototyping with XR Blocks and Gemini]]"
tags:
  - xr
  - ai-xr
  - toolkit
  - sdk
  - webxr
  - three-js
  - gemini
  - vibe-coding
created: 2026-04-15
updated: 2026-04-17
type: entity
---

# XR Blocks

A cross-platform framework by Google XR Labs (Li, Numan et al. 2025; Du, Hersh et al. 2026) for prototyping human-centered AI + XR applications. Initial implementation is **`xrblocks.js`**, a lightweight WebXR library built on three.js, TensorFlow, and Gemini. The stated mission: **"reduce friction from idea to reality"** by providing high-level abstractions that separate the *what* of an interaction from the *how* of low-level perception, rendering, and AI plumbing.

- **Source code:** https://github.com/google/xrblocks
- **Docs / live samples:** https://xrblocks.github.io
- **Vibe-coding front-end (XR Blocks Gem):** https://xrblocks.github.io/gem
- **Status as of 2026-04:** v0.11.0, ~6 months past the v0.1.0 white-paper release; 34 curated templates ship with the framework and serve double duty as LLM grounding examples. See [[XR - Vibe Coding XR]].

## Motivation

AI has a mature ecosystem flywheel: frameworks (PyTorch, JAX), benchmarks (ImageNet, LMArena), model hubs (Hugging Face). XR has no equivalent — research prototypes break between engine versions, deploying old work (DepthLab, 2020) to new hardware is non-trivial, and integrating novel AI models into XR requires gluing low-level sensor/rendering/inference systems manually. XR Blocks tries to be the substrate that makes XR research **reproducible on the open web**.

## The Reality Model

A unified, coherent representation of blended reality that the application's **Script** operates on (rather than disconnected sensor streams). First-class primitives:

| Primitive | What it exposes |
|-----------|-----------------|
| **User** | Hands, gaze, avatar |
| **World** | Depth, estimated lighting, objects, tracked humans |
| **Interface** | 2D panels, 3D assets, UI affordances |
| **Context** ∗ | Environment, activities, interaction history |
| **Agents** ∗ | AI-driven entities with `useTool`, `remember` |
| **Peers** ∗ | Remote human participants as first-class entities |

∗ = partially implemented / aspirational.

## Core Engine (modules)

Realizes the Reality Model:

- **Perception & Input** — `camera`, `depth`, `sound`, `input` (normalizes device inputs into the Interaction Grammar)
- **AI as utility** — `ai.query`, `ai.runModel`; complemented by `agent.useTool`, `agent.remember`
- **Experience / Visualization** — `ux.selectable`, `ux.draggable`, `ui` (panels/buttons), `effect` (occlusion, shadows)
- **Physics** — applied to virtual objects against live depth-map geometry

## Interaction Grammar

A deliberate two-level input model:

- **Explicit events** — direct low-level inputs (touch, click)
- **Implicit intents** — higher-level interpretations (gesture, voice command, `user.isSelectingAt(object)`)

Creators script against **intent**, not raw events. Example Script from the paper:

```js
for (const object of world.objects) {
  if (user.isSelectingAt(object)) {
    const prompt = `Write a poem with ${object.name}.`;
    const poem = agent.query(prompt);
    this.textView.setText(poem);
  }
}
```

This orchestrates `input`, `world`, `agent`, and `ui` modules from ~5 lines.

## Design Principles

1. **Embrace simplicity and readability** — inspired by the Zen of Python. A Script should read like a high-level description of the desired experience.
2. **Prioritize the creator experience** — absorb incidental complexity of sensor fusion, cross-platform logic, model integration.
3. **Pragmatism over completeness** — "worse is better" (Gabriel 1991). A comprehensive framework would be obsolete on release; favor modular and adaptable over complete.

## Demonstrated Applications

- **XR Realism:** depth-aware ballpit physics; geometry-aware shadows; 3D Gaussian splatting with occlusion; lighting-estimated virtual objects.
- **XR Interactions:** immersive emoji / rock-paper-scissors with custom ML gesture models; dynamic swipe recognition; physical-world touch & grab.
- **AI + XR:**
  - **XR-Objects** — long-pinch → `ai` module identifies real-world objects → virtual buttons attach to them for queries. Turns passive objects into programmable interfaces.
  - **Sensible Agent** — proactive/unobtrusive AR assistant built on XR Blocks; perception supplied by Reality Model, intervention logic by `ai` module, peripheral notifications by `ui`. Authors cite this as proof the framework frees researchers to focus on social/cognitive questions instead of plumbing.
  - **Gemini Live** integration; **Gemini Canvas** collaboration; poem-generation with real-world camera input.

## Relation to prior frameworks

| Prior art | Contrast with XR Blocks |
|-----------|-------------------------|
| **Unity / Unreal / Godot** | High ceiling, high floor — complex and slow for prototyping; external AI integration is friction-heavy; API churn across versions |
| **MRTK (HoloLens), XRI (Unity), VRTK, MRTK3** | Unity-native, production-grade UI controls. XR Blocks is web-native and AI-first |
| **A-Frame** | Declarative WebXR via HTML ECS. XR Blocks is programmatic, focused on AI-driven interaction, not declarative scene-graph |
| **three.js, Babylon.js** | Low-level WebGL/WebXR rendering. XR Blocks builds **on** three.js, adds an abstraction layer above it |
| **MediaPipe, Visual Blocks (Rapsai)** | Streamline AI-pipeline authoring; limited XR access to sensors / natural input. XR Blocks extends their spirit into XR |
| **AI2-THOR, Habitat** | Offline embodied-agent training environments. XR Blocks is for real-time human-in-the-loop experiences |
| **LLMR, Thing2Reality** | Natural-language manipulation of 3D worlds. XR Blocks generalizes this with multi-modal (language + gesture + sensing) input |
| **ARKit, ARCore** | Low-level tracking SDKs; leave interaction patterns to the developer. XR Blocks provides those patterns |

## Stated Limitations

- **System design vs implementation:** `agent`, `peers`, `context` are sketches; Sensible Agent is the furthest-along concrete example. Accessibility / SIIDs (Situationally Induced Impairments and Disabilities) noted as a future direction.
- **Performance ceiling of WebXR:** can't match Unity/Unreal native rendering performance; cloud AI adds network latency. Future work: hybrid native/web, on-device distillation, LLM-driven cross-compiler to native engines.
- **Abstraction ceiling:** high-level abstractions trade expressivity for prototyping speed. Need an "escape hatch" for expert devs writing custom shaders / raw sensor access without breaking the framework.
- **WebXR privacy model:** no raw eye/face tracking from the browser (native OpenXR has it). Authors envision future browser-level privacy-preserving sandboxed APIs.

## Vibe Coding XR — the 2026 follow-up

The 2025 white paper's central speculative thread ("Vibe Coding for XR") was operationalized in the Du et al. 2026 follow-up, [[XR - Vibe Coding XR]]. Key deltas that update the status of this entity:

- **From sketch to shipped:** what the white paper rendered as a ~5-line pseudocode example is now a production web app (XR Blocks Gem) that turns a natural-language prompt into a running WebXR experience in under 90 seconds.
- **The Reality Model earned its keep as an LLM-interface layer.** The 2025 paper argued the Reality Model would be easier for *humans* to author against; the 2026 paper reports the stronger result that it is also easier for *LLMs*, because its surface is semantically dense and maps 1:1 onto natural-language concepts. Using XR Blocks instead of raw Unity/Unreal API surfaces is the core reason pass@1 is achievable at all.
- **The "grounding corpus":** the 34 open-source templates (Figure 4 in both papers) are injected into the system prompt as reference examples. Together with the `xrblocks.js` source itself, they act as a technical constraint preventing API hallucination.
- **Measured iteration story:** VCXR60 pass@1 rose from **~70% (v0.1.0) to 95.5% (v0.11.0)** over 6 months / 10 major releases. Most early failures were framework edge-case bugs surfacing under LLM-generated code, not pure model hallucinations — the framework itself was the bigger lever.
- **Benchmark spun out:** [[XR - VCXR60]] (60 prompts, 20 participants, 4 workshops, Playwright headless-browser harness with `pass@1` metric).

## Forward-looking ideas (still speculative as of 2026-04)

Status deltas noted since 2025:

- ~~**Vibe coding for XR**~~ → *shipped*; see [[XR - Vibe Coding XR]].
- **Differentiable / co-adaptive realities** — TextGrad/ToolGrad-style gradient optimization of lighting, geometry, agent behavior to an affective target ("make this room feel relaxing"). Still speculative.
- **Learnable interaction grammars** — systems that learn a user's idiosyncratic gestural vocabulary over time. Still speculative.
- **Multi-sensory synthesis** — haptics, EEG via WebUSB/Arduino extension. Still speculative.
- **LLM-driven cross-compiler to native engines** (Godot called out by name in the 2026 paper) — speculative; a path out of the WebXR performance ceiling.
- **Multimodal prompting** beyond text — gaze, micro-gestures, cross-device inputs as guidance channels for the vibe-coding workflow. Flagged as next step.
- **Accessibility and safety modules** — explicitly absent in v0.11.0; the 2026 paper lists this as a meaningful gap given the framework's broad-participation goals.

## Related

- [[XR - Vibe Coding XR]] — the concrete workflow built on this framework
- [[XR - VCXR60]] — the benchmark measuring it
- [[XR - AI + XR Integration]] — broader topic this entity anchors
- [[LLM - Code Generation Benchmarks]] — benchmark family VCXR60 sits in
- [[Knowledge Management - LLM Powered Knowledge Management]] — shares the "LLM orchestrates a structured model" pattern, just in a different domain
- [[XR - Space-Warp]] — unrelated XR subproblem (reprojection), but shows the fragmentation the paper complains about

---
title: "XR Blocks"
sources:
  - "[[raw/articles/XR Blocks Accelerating Human-Centered AI + XR Innovation]]"
tags:
  - xr
  - ai-xr
  - toolkit
  - sdk
  - webxr
  - three-js
  - gemini
created: 2026-04-15
updated: 2026-04-15
type: entity
---

# XR Blocks

A cross-platform framework by Google XR Labs (Li, Numan et al. 2025) for prototyping human-centered AI + XR applications. Initial implementation is **`xrblocks.js`**, a lightweight WebXR library built on three.js, TensorFlow, and Gemini. The stated mission: **"reduce friction from idea to reality"** by providing high-level abstractions that separate the *what* of an interaction from the *how* of low-level perception, rendering, and AI plumbing.

- **Source code:** https://github.com/google/xrblocks
- **Docs / live samples:** https://xrblocks.github.io
- **Status:** directional white paper + early SDK. Several primitives marked ∗ in the paper are aspirational (not yet in the GitHub).

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

## Forward-looking ideas (speculative)

- **Vibe coding for XR:** natural-language prompts compiled to XR Blocks Scripts ("pinch an object, agent generates a poem of it" → the 5-line example above).
- **Differentiable / co-adaptive realities:** TextGrad/ToolGrad-style gradient optimization of lighting, geometry, agent behavior to an affective target ("make this room feel relaxing").
- **Learnable interaction grammars:** systems that learn a user's idiosyncratic gestural vocabulary over time.
- **Multi-sensory synthesis:** haptics, EEG via WebUSB/Arduino extension.

Marked as research directions, not current capability.

## Related

- [[XR - AI + XR Integration]] — broader topic this entity anchors
- [[Knowledge Management - LLM Powered Knowledge Management]] — shares the "LLM orchestrates a structured model" pattern, just in a different domain
- [[XR - Space-Warp]] — unrelated XR subproblem (reprojection), but shows the fragmentation the paper complains about

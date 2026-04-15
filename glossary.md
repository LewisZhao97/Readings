# Glossary

> Key terms and definitions across the knowledge base.

## A

- **Attention (Scaled Dot-Product)** — `softmax(QK^T / sqrt(d_k))V`. The core primitive of the [[Deep Learning - Transformer|Transformer]]. See [[Deep Learning - Attention Mechanisms]].

## B

- **BRDF (Bidirectional Reflectance Distribution Function)** — Function $f_r(x, \omega_i, \omega_o)$ describing how much light from direction $\omega_i$ is reflected toward $\omega_o$ at surface point $x$. Core term in the Rendering Equation. See [[Graphics - Neural Rendering]].
- **Bilateral Filter** — Edge-preserving smoothing filter weighting neighbors by spatial proximity *and* intensity similarity. Joint/cross variants use a separate guide image (e.g., color-guided depth upsampling). See [[Graphics - Bilateral Filter]].

## D

- **Depth Propagation** — Reconstruction of a dense depth map from sparse anchor points via optimization with data (anchor) + smooth (neighbor) terms, weighted by color-image similarity. See [[XR - Depth Reconstruction for Space-Warp]].

## G

- **Generative Reality** — Design paradigm in which generative AI is applied not just to conversational agents but to the fabric of XR environments: depth, lighting, agents, UI all co-synthesized against a high-level intent. Coined by [[XR - XR Blocks]]. See [[XR - AI + XR Integration]].
- **GGX** — Microfacet normal-distribution BRDF [Walter et al. 2007], parameterized by diffuse albedo, specular albedo, and roughness. The reflectance model used by [[Graphics - RenderFormer]].
- **Global Illumination (GI)** — Light transport accounting for *all* bounces (direct + indirect), including diffuse inter-reflection, specular inter-reflection, soft shadows, caustics. Classical solvers: path tracing, radiosity. See [[Graphics - Neural Rendering]].

## H

- **Harness** — The code wrapping an LLM that determines context, memory, tool access, and scaffolding. Often a larger performance lever than the model itself. See [[LLM - Harness Engineering]].

## I

- **Interaction Grammar** — Two-level input model distinguishing **explicit events** (touch, click — low-level) from **implicit intents** (gesture, voice command, `user.isSelectingAt(object)` — high-level interpretations). Creators script against intent. See [[XR - XR Blocks]], [[XR - AI + XR Integration]].

## L

- **LLM Wiki Pattern** — A methodology where an LLM incrementally builds and maintains a persistent wiki from ingested sources, rather than re-deriving knowledge per query via RAG. See [[Knowledge Management - LLM Powered Knowledge Management]].

## M

- **Meta-Harness** — An outer-loop system that automates harness design by giving a coding-agent proposer full filesystem access to prior candidates. See [[LLM - Meta-Harness]].
- **Multi-Head Attention** — h parallel attention heads operating in separate projected subspaces, concatenated at output. See [[Deep Learning - Attention Mechanisms]].
- **Memex** — Vannevar Bush's 1945 concept of a personal, curated knowledge store with associative trails between documents. See [[Knowledge Management - Memex]].

## N

- **Neural Rendering** — Replacing light-transport simulation with a learned neural process. Many variants: per-scene (NeRF, Gaussian splats) vs. generalizing (RenderFormer, RenderNet); explicit geometry (triangles/voxels) vs. implicit fields. See [[Graphics - Neural Rendering]].

## O

- **Obsidian** — Markdown-based note-taking app with graph view, used as the browsing interface in the LLM Wiki pattern. See [[Tools - Obsidian]].

## R

- **RenderFormer** — Transformer-based neural rendering pipeline (Zeng et al. 2025) that takes a triangle mesh and produces a globally-illuminated image without per-scene training. See [[Graphics - RenderFormer]].
- **Rendering Equation** — Kajiya 1986. $L_o(x, \omega_o) = L_e + \int_\Omega f_r\, L_i\, (\omega_i \cdot n)\, d\omega_i$. The recursive integral that classical renderers solve and that neural renderers aim to replace. See [[Graphics - Neural Rendering]].
- **RoPE (Rotary Position Embedding)** — Encodes token position as a rotation applied to Q and K at every attention layer; the Q·K^T dot product then depends only on relative offsets. Extended by RenderFormer to 3D vertex coordinates for scene geometry. See [[Deep Learning - Attention Mechanisms]].
- **Ray Marching** — A rendering technique that steps along rays using signed distance functions to find surface intersections. Also called sphere tracing. See [[Graphics - Ray Marching]].


- **Reality Model** — Unified, coherent representation of blended XR reality with first-class primitives (`user`, `world`, `interface`, `context`, `agents`, `peers`) that application logic (the Script) operates on, instead of raw disconnected sensor streams. Introduced by [[XR - XR Blocks]]. See [[XR - AI + XR Integration]].

- **RAG (Retrieval-Augmented Generation)** — An approach where an LLM retrieves relevant document chunks at query time and generates answers from scratch. Contrasted with the LLM Wiki pattern. See [[Knowledge Management - LLM Powered Knowledge Management]].

## P

- **Positional Encoding** — Sinusoidal (or learned) vectors added to token embeddings to inject order information into an otherwise permutation-invariant attention model. See [[Deep Learning - Attention Mechanisms]].

## S

- **Space-Warp** — Depth-based late-stage reprojection of a rendered frame to a fresher head pose, used in XR to hide rendering latency. See [[XR - Space-Warp]].
- **SDF (Signed Distance Function)** — A mathematical function that returns the signed distance from a point to the nearest surface. Negative = inside, zero = on surface, positive = outside. Used in ray marching. See [[Graphics - Ray Marching]].
- **Self-Attention** — Attention where queries, keys, and values all come from the same sequence — each position attends over every other. See [[Deep Learning - Attention Mechanisms]].
- **Sphere Tracing** — A ray marching optimization where the step size equals the SDF value at each point, guaranteeing no overshoot. See [[Graphics - Ray Marching]].

## T

- **Transformer** — Sequence model built entirely from attention, introduced in "Attention Is All You Need" (Vaswani et al. 2017). See [[Deep Learning - Transformer]].

## V

- **Vibe Coding** — Intent-driven creation paradigm: a user expresses a high-level natural-language goal and a tool (e.g., Gemini Canvas, Cursor) generates a working application. XR Blocks aspires to extend vibe coding into XR, where the prompt compiles into a Script orchestrating perception, AI, and UI modules. See [[XR - XR Blocks]], [[XR - AI + XR Integration]].

## W

- **WebXR** — W3C standard browser API exposing VR/AR device capabilities (pose, input, depth, lighting) to web applications. Trades off native sensor access (no raw eye/face tracking by default) for cross-platform reach and privacy. See [[XR - XR Blocks]].

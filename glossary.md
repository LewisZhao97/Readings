# Glossary

> Key terms and definitions across the knowledge base.

## A

- **Attention (Scaled Dot-Product)** — `softmax(QK^T / sqrt(d_k))V`. The core primitive of the [[Deep Learning - Transformer|Transformer]]. See [[Deep Learning - Attention Mechanisms]].

## B

- **Bilateral Filter** — Edge-preserving smoothing filter weighting neighbors by spatial proximity *and* intensity similarity. Joint/cross variants use a separate guide image (e.g., color-guided depth upsampling). See [[Graphics - Bilateral Filter]].

## D

- **Depth Propagation** — Reconstruction of a dense depth map from sparse anchor points via optimization with data (anchor) + smooth (neighbor) terms, weighted by color-image similarity. See [[XR - Depth Reconstruction for Space-Warp]].

## H

- **Harness** — The code wrapping an LLM that determines context, memory, tool access, and scaffolding. Often a larger performance lever than the model itself. See [[LLM - Harness Engineering]].

## L

- **LLM Wiki Pattern** — A methodology where an LLM incrementally builds and maintains a persistent wiki from ingested sources, rather than re-deriving knowledge per query via RAG. See [[Knowledge Management - LLM Powered Knowledge Management]].

## M

- **Meta-Harness** — An outer-loop system that automates harness design by giving a coding-agent proposer full filesystem access to prior candidates. See [[LLM - Meta-Harness]].
- **Multi-Head Attention** — h parallel attention heads operating in separate projected subspaces, concatenated at output. See [[Deep Learning - Attention Mechanisms]].
- **Memex** — Vannevar Bush's 1945 concept of a personal, curated knowledge store with associative trails between documents. See [[Knowledge Management - Memex]].

## O

- **Obsidian** — Markdown-based note-taking app with graph view, used as the browsing interface in the LLM Wiki pattern. See [[Tools - Obsidian]].

## R

- **Ray Marching** — A rendering technique that steps along rays using signed distance functions to find surface intersections. Also called sphere tracing. See [[Graphics - Ray Marching]].


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

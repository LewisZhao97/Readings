# Log

## [2026-04-17] ingest | World Mouse — Tütüncü, Gonzalez-Franco, Patel, Gonzalez

- **Source:** https://arxiv.org/html/2603.10984v1 (CHI '26)
- **Raw:** `raw/articles/World Mouse Exploring Interactions with a Cross-Reality Cursor.md` (already in raw/ — downloaded manually from 2026-04-17 feed)
- **Summary:** `wiki/summaries/XR - World Mouse.md`
- **Recommend:** partial (vision + prototype paper; no user study. Worth a small entity page + cross-links to AI+XR integration for the deixis argument)

## [2026-04-10] ingest | LLM Wiki — Andrej Karpathy

- **Source:** https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- **Raw:** `raw/articles/LLM-Karpathy_LLM_Wiki.md`
- **Status:** pending distillation

## [2026-04-11] distil | LLM Wiki — Andrej Karpathy

- **Source:** `raw/articles/LLM-Karpathy_LLM_Wiki.md`
- **Wiki pages created:** `wiki/summaries/LLM-Karpathy_LLM_Wiki.md`, `wiki/entities/AI-Andrej_Karpathy.md`, `wiki/entities/Knowledge_Management-Memex.md`, `wiki/topics/Knowledge_Management-LLM_Powered_Knowledge_Management.md`
- **Wiki pages updated:** none
- **Glossary terms added:** LLM Wiki Pattern, Memex, RAG

## [2026-04-11] ingest | Ray Marching — Michael Walczyk

- **Source:** https://michaelwalczyk.com/blog-ray-marching.html
- **Raw:** `raw/articles/Ray Marching.md`
- **Summary:** `wiki/summaries/Graphics-Ray_Marching.md`
- **Recommend:** partial

## [2026-04-11] distil | LLM Wiki — Andrej Karpathy (pass 2)

- **Source:** `raw/articles/Karpathy LLM Wiki.md`
- **Wiki pages created:** `wiki/entities/Tools-Obsidian.md`
- **Wiki pages updated:** `wiki/entities/AI-Andrej_Karpathy.md`, `wiki/entities/Knowledge_Management-Memex.md`, `wiki/topics/Knowledge_Management-LLM_Powered_Knowledge_Management.md`
- **Glossary terms added:** Obsidian
- **distil_times:** 1 → 2

## [2026-04-12] ingest | Attention Is All You Need — Vaswani et al.

- **Source:** https://arxiv.org/html/1706.03762v7
- **Raw:** `raw/articles/Attention Is All You Need.md`
- **Summary:** `wiki/summaries/Deep_Learning-Attention_Is_All_You_Need.md`
- **Recommend:** yes

## [2026-04-12] ingest | Meta-Harness: End-to-End Optimization of Model Harnesses — Lee, Khattab et al.

- **Source:** https://arxiv.org/html/2603.28052v1
- **Raw:** `raw/articles/Meta-Harness End-to-End Optimization of Model Harnesses.md`
- **Summary:** `wiki/summaries/LLM-Meta_Harness.md`
- **Recommend:** yes

## [2026-04-12] distil | Attention Is All You Need — Vaswani et al.

- **Source:** `raw/articles/Attention Is All You Need.md`
- **Wiki pages created:** `wiki/entities/Deep Learning - Transformer.md`, `wiki/topics/Deep Learning - Attention Mechanisms.md`
- **Wiki pages updated:** none
- **Glossary terms added:** Attention (Scaled Dot-Product), Multi-Head Attention, Positional Encoding, Self-Attention, Transformer
- **distil_times:** 0 → 1

## [2026-04-12] distil | Meta-Harness — Lee, Khattab et al.

- **Source:** `raw/articles/Meta-Harness End-to-End Optimization of Model Harnesses.md`
- **Wiki pages created:** `wiki/entities/LLM - Meta-Harness.md`, `wiki/topics/LLM - Harness Engineering.md`
- **Wiki pages updated:** none
- **Glossary terms added:** Harness, Meta-Harness
- **distil_times:** 0 → 1

## [2026-04-12] distil | Attention Is All You Need — Vaswani et al. (pass 2)

- **Source:** `raw/articles/Attention Is All You Need.md` (read in full, chunked)
- **Wiki pages created:** none
- **Wiki pages updated:** `wiki/entities/Deep Learning - Transformer.md`, `wiki/topics/Deep Learning - Attention Mechanisms.md` — significantly expanded with hyperparameters, training recipe, ablation details, complexity table, attention visualization findings
- **Glossary terms added:** none
- **distil_times:** 1 → 2
- **Reason:** first pass distilled from the summary rather than the raw; second pass reads the raw directly per updated skill.

## [2026-04-12] distil | Meta-Harness — Lee, Khattab et al. (pass 2)

- **Source:** `raw/articles/Meta-Harness End-to-End Optimization of Model Harnesses.md` (read in full, chunked)
- **Wiki pages created:** none
- **Wiki pages updated:** `wiki/entities/LLM - Meta-Harness.md`, `wiki/topics/LLM - Harness Engineering.md` — expanded with formal objective, search loop algorithm, Table 1 feedback comparison, full ablation, per-domain empirical results, qualitative proposer narrative (TerminalBench-2 iterations 1–3), practical tips from Appendix D
- **Glossary terms added:** none
- **distil_times:** 1 → 2
- **Reason:** re-distil from raw per updated skill.

## [2026-04-14] ingest | RenderFormer — Zeng et al.

- **Source:** https://dl.acm.org/doi/10.1145/3721238.3730595
- **Raw:** `raw/articles/RenderFormer Transformer based Neural Rendering of Triangle Meshes with Global Illumination.md`
- **Summary:** `wiki/summaries/Graphics - RenderFormer.md`
- **Recommend:** yes

## [2026-04-14] ingest | Space Warp with Depth Propagation in XR Applications — Xiong & Peri

- **Source:** https://ieeexplore.ieee.org/document/9666145
- **Raw:** `raw/articles/Space Warp with Depth Propagation in XR Applications.md`
- **Summary:** `wiki/summaries/XR - Space Warp with Depth Propagation.md`
- **Recommend:** partial

## [2026-04-14] ingest | Image Guided Depth Super-Resolution for Spacewarp in XR Applications — Peri & Xiong

- **Source:** https://ieeexplore.ieee.org/document/9427716
- **Raw:** `raw/articles/Image Guided Depth Super-Resolution for Spacewarp in XR Applications.md`
- **Summary:** `wiki/summaries/XR - Image Guided Depth Super-Resolution.md`
- **Recommend:** partial
- **Note:** companion/predecessor to the depth-propagation paper above — same authors, same problem (reduce depth bandwidth for XR Space-Warp), superseded approach (regular-grid downsample + joint bilateral upsampling vs. adaptive feature-based sparse points + optimization-based propagation).

## [2026-04-14] distil | Xiong & Peri XR depth-reconstruction pair (combined)

- **Sources:** `raw/articles/Space Warp with Depth Propagation in XR Applications.md`, `raw/articles/Image Guided Depth Super-Resolution for Spacewarp in XR Applications.md` (read in full)
- **Wiki pages created:**
  - `wiki/topics/XR - Depth Reconstruction for Space-Warp.md` (unified topic — shared problem, pipeline, reprojection math, limitations)
  - `wiki/comparisons/XR - Depth Super-Resolution vs Depth Propagation.md` (side-by-side algorithmic diff)
  - `wiki/entities/XR - Space-Warp.md` (the reprojection technique)
  - `wiki/entities/Graphics - Bilateral Filter.md` (shared primitive)
- **Wiki pages updated:** none
- **Glossary terms added:** Space-Warp, Bilateral Filter, Depth Propagation
- **distil_times:** 0 → 1 (both summaries)
- **Reason:** user requested combined distil — shared content consolidated into one topic page; algorithmic diff split into a comparison page per user follow-up.

## [2026-04-15] distil | RenderFormer — Zeng et al.

- **Source:** `raw/articles/RenderFormer Transformer based Neural Rendering of Triangle Meshes with Global Illumination.md` (read in full, chunked)
- **Wiki pages created:**
  - `wiki/entities/Graphics - RenderFormer.md` (full architecture, training, ablations, limitations, probe results)
  - `wiki/topics/Graphics - Neural Rendering.md` (broad area — families of neural rendering, rendering-equation relationships, open challenges)
- **Wiki pages updated:**
  - `wiki/entities/Deep Learning - Transformer.md` — added modern-recipe deltas (RMS-Norm, SwiGLU, QK-Norm, FlashAttention-2, register tokens) and a "use beyond language" section linking to RenderFormer
  - `wiki/topics/Deep Learning - Attention Mechanisms.md` — added RoPE + 3D spatial RoPE to positional-encoding section, added cross-attention-across-heterogeneous-sequences section with RenderFormer as example
- **Glossary terms added:** BRDF, GGX, Global Illumination, Neural Rendering, RenderFormer, Rendering Equation, RoPE
- **distil_times:** 0 → 1

## [2026-04-15] ingest | XR Blocks: Accelerating Human-Centered AI + XR Innovation — Li, Numan et al.

- **Source:** https://arxiv.org/html/2509.25504v1
- **Raw:** `raw/articles/XR Blocks Accelerating Human-Centered AI + XR Innovation.md`
- **Summary:** `wiki/summaries/XR - XR Blocks.md`
- **Recommend:** partial
- **Note:** directional white paper / vision piece, not an evaluated research contribution. Value is in the conceptual model (Reality Model, Interaction Grammar, Script) and the curated XR-framework landscape, not the (small, partially aspirational) SDK surface.

## [2026-04-15] distil | XR Blocks — Li, Numan et al.

- **Source:** `raw/articles/XR Blocks Accelerating Human-Centered AI + XR Innovation.md` (read in full)
- **Wiki pages created:**
  - `wiki/entities/XR - XR Blocks.md` (framework entity — Reality Model, Core Engine modules, Interaction Grammar, design principles, demos, landscape comparison, limitations)
  - `wiki/topics/XR - AI + XR Integration.md` (broad topic — ecosystem-flywheel argument, Reality Model pattern, explicit vs implicit intents, exemplars, open challenges)
- **Wiki pages updated:** none
- **Glossary terms added:** Generative Reality, Interaction Grammar, Reality Model, Vibe Coding, WebXR
- **distil_times:** 0 → 1
- **Reason:** partial-recommend white paper; distilled the conceptual vocabulary and framework-landscape map per ingest plan. Skipped speculative sections (differentiable realities, learnable grammars) beyond noting them as future directions.

## [2026-04-16] feed | XR real-time rendering & ray-tracing

- **Feed file:** `raw/feeds/2026-04-16.md`
- **Query:** latest papers on XR real-time rendering / XR ray-tracing (ACM SIGGRAPH, IEEE, arXiv, blog posts)
- **Candidates:** 10 (all `new` — none already ingested)
- **Top picks:** GRTX (arXiv 2601.20429, Jan 2026); VR-Splatting (PACMCGIT / I3D 2025, DOI 10.1145/3728302); Radiance Fields in XR survey (arXiv 2508.04326 / IEEE TVCG 2025).

## [2026-04-17] feed | latest XR research

- **Feed file:** `raw/feeds/2026-04-17.md`
- **Query:** latest XR research (open web — no source allowlist)
- **Candidates:** 11 (all `new`; close KB adjacency: Vibe Coding XR → `[[XR - XR Blocks]]`, Geometry Aware Passthrough → `[[XR - Depth Reconstruction for Space-Warp]]`)
- **Top picks:** How Do We Evaluate Experiences in Immersive Environments? (CHI '26, DOI 10.1145/3772318.3790724); World Mouse (arXiv 2603.10984); Vibe Coding XR (arXiv 2603.24591).
- **Note:** IEEE VR 2026 papers page not yet populated — re-run once list publishes.

## [2026-04-17] ingest | Vibe Coding XR — Du, Hersh et al.

- **Source:** https://arxiv.org/html/2603.24591v2
- **Raw:** `raw/articles/Vibe Coding XR Accelerating AI + XR Prototyping with XR Blocks and Gemini.md`
- **Summary:** `wiki/summaries/XR - Vibe Coding XR.md`
- **Recommend:** yes
- **Note:** concrete companion / measured follow-up to the earlier XR Blocks white paper (`[[XR - XR Blocks]]`). Reports 95.5% pass@1 on new VCXR60 benchmark with gemini-3.1-pro.

## [2026-04-17] distil | Vibe Coding XR — Du, Hersh et al.

- **Source:** `raw/articles/Vibe Coding XR Accelerating AI + XR Prototyping with XR Blocks and Gemini.md` (read in full, chunked)
- **Wiki pages created:**
  - `wiki/entities/XR - Vibe Coding XR.md` (workflow entity — system-prompt architecture, simulated-to-extended-reality loop, application scenarios, limitations)
  - `wiki/entities/XR - VCXR60.md` (benchmark entity — composition, harness, pass@1 results, iteration story)
  - `wiki/topics/LLM - Code Generation Benchmarks.md` (HumanEval → VCXR60 lineage; runtime-as-implicit-validator pattern; pass@k metric)
- **Wiki pages updated:**
  - `wiki/entities/XR - XR Blocks.md` — added Vibe Coding XR section, shipped-vs-speculative status updates, v0.11.0 info, grounding-corpus insight
  - `wiki/topics/XR - AI + XR Integration.md` — added Reality-Model-as-LLM-interface, grounding-corpus+constrained-system-prompt, and runtime-as-validator design patterns; updated exemplars; moved vibe-coding from "forward-looking" to "delivered"
- **Glossary terms added:** Pass@1 / pass@k, VCXR60; updated Vibe Coding entry to link to Vibe Coding XR entity.
- **distil_times:** 0 → 1

## [2026-04-17] ingest | Autoresearch — Andrej Karpathy

- **Source:** https://github.com/karpathy/autoresearch
- **Raw:** `raw/articles/Karpathy Autoresearch AI agents running research on single-GPU nanochat training automatically.md`
- **Summary:** `wiki/summaries/LLM - Autoresearch.md`
- **Recommend:** partial
- **Note:** GitHub README / research-sandbox by Karpathy. Conceptually an open-source instance of the Meta-Harness outer-loop pattern (agent edits code → runs fixed-budget experiment → logs → iterates), applied to model training rather than LLM harness. Novel moves worth distilling: `program.md` as a programmable "skill" for a research organization, 5-min fixed wall-clock budget for cross-architecture comparability, `val_bpb` as vocab-independent metric.

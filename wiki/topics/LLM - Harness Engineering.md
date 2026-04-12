---
title: Harness Engineering
sources:
  - "[[raw/articles/Meta-Harness End-to-End Optimization of Model Harnesses]]"
tags:
  - LLM
  - harness-engineering
  - prompt-engineering
  - automated-optimization
created: 2026-04-12
updated: 2026-04-12
type: topic
---

# Harness Engineering

A **harness** is the code surrounding an LLM that determines, at each step, what context, memory, retrieved documents, tool access, and orchestration logic the model sees. Harness engineering is the practice of designing and tuning that wrapper code. Historically manual; increasingly automated (see [[LLM - Meta-Harness]]).

Meta-Harness formalizes the definition: a harness is a **stateful program** that wraps a language model, constructs prompts, consumes model responses, and updates state. The harness optimization objective is to maximize expected reward over the task distribution with the model held fixed.

## Why harnesses matter

- On the same benchmark with the same model, swapping harnesses can produce a **6× performance gap** (Tian et al. 2026, SWE-bench mobile).
- Despite this leverage, harness engineering has remained largely manual: practitioners inspect failures, iterate on prompts, retrieval strategies, and tool-use logic by hand.
- Industry activity: OpenAI, Anthropic, and the Martin Fowler blog have all published harness-engineering writeups (2025–2026), signaling an emerging named discipline.

## Why prior text optimizers fall short

Text optimizers try to automate harness/prompt tuning but treat prompts or short templates as the search space and compress feedback aggressively. Their shared limitation: they strip out the information needed to trace long-horizon failures back to earlier harness decisions.

| Method | History | Log content | MTok/iter |
|--------|---------|-------------|----------:|
| OPRO | Window | past (solution, score) pairs | 0.002 |
| TextGrad | Last | textual feedback on current artifact | 0.015 |
| AlphaEvolve | Window | program database + eval scores | 0.022 |
| GEPA | Summary | reflective rollout-trace feedback | 0.008 |
| Feedback Descent | Summary | comparison + textual feedback | 0.012 |
| TTT-Discover | Window | previous solution fragment | 0.026 |
| **Meta-Harness** | **Full** | **all logs and scores** | **10.0** |

Harnesses act over long horizons — a single decision about what to store, when to retrieve it, or how to present it can cascade through many reasoning steps. Compressed feedback (scalar scores, short summaries, fixed critique templates) removes exactly the signal needed to trace such failures.

## The Meta-Harness thesis

The fix is not better summaries — it is **no summarization at all**. Give a coding-agent proposer full filesystem access to prior harness code, execution traces, and scores, and let it decide what to inspect.

Empirically validated by ablation (see [[LLM - Meta-Harness]]):

- Scores-only: 34.6 median accuracy
- Scores + LLM-generated summary: 34.9 (no meaningful gain — summaries do **not** recover the missing signal)
- Full trace access: 50.0

## Design principles emerging from Meta-Harness

- **Search in code space, not prompt space.** Programs are readable; brittle overfitting is visible as hard-coded class mappings or ugly if-chains in a way that weight-space overfitting is not.
- **Preserve raw diagnostic signal.** Do not pre-digest traces; let the proposer choose what to read via `grep`/`cat` on the filesystem.
- **Accumulate, don't overwrite.** Every candidate's artifacts stay available; the proposer can compare across many prior runs simultaneously.
- **Delegate diagnosis to the coding agent.** Minimal imposed structure — no parent-selection rule, no fixed critique template. The proposer's causal reasoning over the log is what drives improvement.
- **Automate evaluation out-of-band.** The proposer should not run evals; a separate process scores candidates and writes to the filesystem.

## Practical tips (Meta-Harness Appendix D)

Engineering lessons from building harness-search systems:

- **Write a good skill.** The skill text is the strongest lever on whether the loop works. Constrain **outputs and safety-relevant behavior**, not the proposer's diagnosis procedure. Expect 3–5 short debug runs to iterate on the skill before a full run.
- **Start with a baseline and a search set that is hard for it.** Filter for examples the baseline gets wrong, or pick a diverse subset of difficult instances. A saturated eval has nothing to optimize against.
- **Keep search sets small but discriminative.** ~50 full evaluations per run, 50–100 examples per eval.
- **Log everything in a queryable format.** Machine-readable (JSON), hierarchical, consistent file names, regex-friendly.
- **Add a small CLI** for listing the Pareto frontier, showing top-k, diffing pairs. Saves the proposer tokens that would otherwise be spent navigating.
- **Lightweight validation before expensive evals.** A trivial test (import module, instantiate class, call methods on a tiny example set) catches most malformed candidates in seconds.

## Connections

- Parallels [[Knowledge Management - LLM Powered Knowledge Management|LLM-powered knowledge management]]: both treat the filesystem as durable, inspectable memory for an LLM rather than compressing everything through a context window.
- Close to external-memory / adaptive-access work (retrieval-augmented generation, memory-based agents, recursive language models) in spirit — the principle is that large context should be *accessed* adaptively, not packed into one prompt.
- Distinct from executable code search (AlphaEvolve, OpenEvolve) because harnesses are stateful programs that accumulate experience across examples, not stateless functions with a single scalar objective.
- Distinct from prompt orchestration frameworks (LMQL, LangChain, DSPy): those give developers abstractions for *composing* LLM programs; harness engineering optimizes the *implementation* of those programs.

## Related

- [[LLM - Meta-Harness]]
- [[LLM - Meta Harness]] (summary)

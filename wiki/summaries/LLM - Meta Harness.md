---
title: Meta-Harness End-to-End Optimization of Model Harnesses — Summary
sources:
  - "[[Meta-Harness End-to-End Optimization of Model Harnesses]]"
tags:
  - LLM
  - automated-optimization
  - harness-engineering
  - coding-agents
  - meta-learning
  - prompt-engineering
created: 2026-04-12
updated: 2026-04-12
type: summary
recommend: yes
distil_times: 2
---

# Meta-Harness: End-to-End Optimization of Model Harnesses — Lee, Khattab et al.

## Summary

Introduces Meta-Harness, an outer-loop system that automatically searches over LLM harness code (the code that determines what information to store, retrieve, and present to a model). Unlike prior text optimizers that compress feedback into scalar scores or short summaries, Meta-Harness gives a coding-agent proposer full filesystem access to all prior candidates' source code, execution traces, and scores. Demonstrates improvements across online text classification (+7.7 points over ACE with 4x fewer tokens), retrieval-augmented math reasoning (+4.7 points on IMO-level problems across 5 held-out models), and agentic coding (surpasses hand-engineered baselines on TerminalBench-2).

## Key Points

- **Core insight:** LLM performance depends heavily on the harness (the code wrapping the model) — changing the harness alone can produce a 6x performance gap on the same benchmark. Yet harness engineering is still done manually.
- **Why existing text optimizers fail:** they compress feedback too aggressively — memoryless, scalar-score-only, or restricted to short templates/summaries. Harnesses act over long horizons where compressed feedback removes the information needed to trace failures.
- **Meta-Harness design:** an agentic proposer (Claude Code with Opus-4.6) with full filesystem access to a growing log of prior harness code, scores, and execution traces. The proposer reads a median of 82 files per iteration, referencing 20+ prior candidates per step.
- **Search loop:** propose new harness -> evaluate on tasks -> log everything (code, scores, traces) to filesystem -> repeat. Minimal imposed structure — diagnosis and proposal logic delegated entirely to the coding agent.
- **Key ablation finding:** full access to execution traces is the critical ingredient. Scores-only (34.6 median) and scores-plus-summary (34.9) dramatically underperform the full interface (50.0 median). Summaries don't recover the missing signal.
- **Code-space search advantages:** harnesses are readable Python programs, so overfitting is inspectable (brittle if-chains visible on inspection). Coding models tend to propose coherent algorithms rather than brittle hard-coded solutions.
- **Generalization:** discovered harnesses transfer to out-of-distribution tasks (9 unseen classification datasets) and unseen models (5 held-out models in math reasoning).
- **Practical scale:** single evaluation can produce up to 10M tokens of diagnostic information — roughly 3 orders of magnitude beyond prior text optimizer feedback budgets.
- **Qualitative behavior:** proposer demonstrates causal reasoning — identifies confounded edits across iterations, isolates likely causal changes, shifts toward safer modifications after repeated regressions.

## Entities Mentioned

- Yoonho Lee (Stanford), Omar Khattab (MIT), Chelsea Finn (Stanford) — key authors
- Claude Code / Claude Opus 4.6 — used as the proposer agent
- TerminalBench-2 — agentic coding benchmark
- ACE (Agentic Context Engineering) — baseline harness system
- OpenEvolve, TTT-Discover, OPRO, TextGrad, GEPA — compared text optimization methods
- Terminus-KIRA — hand-engineered baseline agent

## Related

- Potential topics: harness engineering, automated prompt/code optimization, coding agents, meta-learning for LLM systems

## Recommend Reason

**Yes.** This paper formalizes and automates a practice most LLM practitioners do by hand — optimizing the code around a model. The key insight that *full diagnostic access* (not summaries) is what makes optimization work is practically important. The three-domain evaluation is thorough, the ablation cleanly isolates the filesystem-access advantage, and the qualitative proposer analysis showing causal reasoning over prior failures is genuinely novel. Directly relevant to anyone building LLM-powered systems, including this knowledge base's own tooling.

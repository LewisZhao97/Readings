---
title: Meta-Harness
sources:
  - "[[raw/articles/Meta-Harness End-to-End Optimization of Model Harnesses]]"
tags:
  - LLM
  - harness-engineering
  - automated-optimization
  - coding-agents
created: 2026-04-12
updated: 2026-04-12
type: entity
---

# Meta-Harness

An outer-loop optimization system introduced by Lee, Nair, Zhang, Lee, Khattab, Finn (Stanford / MIT / KRAFTON) that automates the design of **LLM harnesses** — the code wrapping a model that decides what to store, retrieve, and show at inference. Its distinguishing design choice: hand a coding-agent proposer **full filesystem access** to every prior candidate's source, execution traces, and scores, rather than compressing feedback into scalar scores or short summaries as prior text optimizers do.

Meta-Harness is itself a harness — a program that decides what context the proposer model sees during search.

## Formal objective

Let `M` be a frozen LLM and `X` a task distribution. A harness `H` is a stateful program that wraps `M`, constructs prompts, consumes model responses, and updates state. Executing a rollout yields trajectory `τ ~ p_M(H, x)` with reward `r(τ, x)`. The objective:

```
H* = argmax_H  E_{x~X, τ~p_M(H,x)}  r(τ, x)
```

When multiple objectives are relevant (e.g., accuracy and context cost), candidates are ranked by Pareto dominance.

## Search loop

The loop is deliberately minimal — diagnosis is delegated to the coding agent:

1. **Initialize** a population `H` with baseline harnesses (zero-shot, few-shot, ACE, etc.) and an empty filesystem `D`.
2. **Evaluate each seed** on the search set; write code, scores, traces into `D`.
3. **Iterate:**
   - Proposer queries `D` via terminal tools (`grep`, `cat`) — `D` is typically larger than the proposer's context window, so it is accessed adaptively, not ingested in one shot.
   - Proposer proposes `k` new harnesses.
   - Each candidate must pass a lightweight interface-validation test before paying the full-eval cost.
   - Evaluated candidates are logged in a new directory (code + scores + traces).
4. **Return** the Pareto frontier across evaluated harnesses.

No parent-selection rule is imposed. The proposer is free to inspect any prior harness. It never sees test-set results.

## Proposer and scale

- **Proposer P:** Claude Code with Opus-4.6, guided by a minimal domain-specific skill describing directory layout, CLI commands, output format, and forbidden files — but **not** how to diagnose.
- **Typical run:** ~60 harnesses over ~20 iterations; wall-clock a few hours.
- **Access pattern (TerminalBench-2 run):** median **82 files read/iteration** (range 69–99), breakdown:
  - 41% prior harness source code
  - 40% execution traces
  - 6% score/summary files
  - 13% other
- **Feedback scale:** up to **10M tokens** of diagnostic info per evaluation — roughly three orders of magnitude above the largest feedback budgets in prior text optimizers (100–30K tokens/step for OPRO, TextGrad, GEPA, AlphaEvolve, Feedback Descent, TTT-Discover).

## Key ablation — traces are the critical ingredient

Online text classification (Table 3):

| Proposer interface | Scores | Code | Summary | Traces | Median | Best | Runs > zero-shot |
|--------------------|:-----:|:----:|:-------:|:------:|-------:|-----:|-----------------:|
| Scores only | ✓ | ✓ | × | × | 34.6 | 41.3 | 26 |
| Scores + summary | ✓ | ✓ | ✓ | × | 34.9 | 38.7 | 23 |
| **Full (Meta-Harness)** | ✓ | ✓ | — | ✓ | **50.0** | **56.7** | **39** |

Summaries **do not** recover the missing signal; they may even hurt by compressing away diagnostically useful details. Full execution traces are the single most important component of the interface.

## Empirical results

### Online text classification (GPT-OSS-120B)

Datasets: LawBench (215 classes), Symptom2Disease (22 classes), USPTO-50k (180 classes). 20 evolution iterations, 2 candidates/iter.

- Meta-Harness: **48.6%** accuracy at **11.4K** context tokens.
- ACE: 40.9% at 50.8K tokens (+7.7 points with 4× fewer tokens).
- MCE: 40.0% (+8.6 points).
- vs other text optimizers: matches OpenEvolve / TTT-Discover final accuracy in **10× fewer evaluations**, then surpasses them by >10 points by run end (Table 4 — Meta-Harness 50.0 / 56.7 median/best vs best other ≤ 45.6 best).
- **OOD generalization:** on 9 unseen classification datasets, Meta-Harness averages 73.1% vs ACE 70.2%, best on 6/9 datasets. Naïvely adding more few-shot examples beyond 32 hurts on 7/9 tasks.

### Retrieval-augmented math reasoning

Corpus: ≥500K solved problems, deduplicated and decontaminated. Search set: 250 Olympiad-difficulty problems. Search ran 40 iterations → 109 candidates. A single selected harness was evaluated on 200 held-out IMO-level problems across 5 models not used in search.

- Average gain over no-retriever baseline: **+4.7 points**.
- Beats BM25 retrieval by 1.3 points overall.
- Avoids the regressions that dense retrieval and random few-shot show on some models.
- Notably, the harness runs on the **same BM25 stack** as the sparse baseline — gains are purely from search over retrieval policy code.

### Agentic coding — TerminalBench-2

89 long-horizon autonomous tasks. Search + eval on the same benchmark (used as a discovery problem); overfitting checked by manual inspection and regex leak audits.

- **Opus 4.6:** 76.4% pass rate, **#2 overall** (behind only ForgeCode 81.8%, which authors could not reproduce from the public repo). Beats Terminus-KIRA (74.7%), Capy (75.3%).
- **Haiku 4.5:** 37.6%, **#1** — beats Goose (35.5%) by 2.1 points.

## Qualitative behavior — causal reasoning over prior failures

The TerminalBench-2 search log (Appendix A) shows the proposer does more than random local edits:

- **Iterations 1–2:** both bundle structural fixes (marker stripping + loop breaker; removing double-confirmation completion) with a cleanup-oriented prompt rewrite. Both regress sharply from the 64.4% Terminus-KIRA baseline.
- **Iteration 3 (the causal step):** proposer explicitly notes the common factor across the two failures is the prompt rewrite, not the bugfixes. Reverts the prompt, tests only the marker-stripping and loop-breaker. Still slightly under (63.3%, −1.1 pp) but loses much less — supporting the confound diagnosis.
- **Later iterations:** the proposer continues to isolate effects, shifting toward safer additive modifications. The winning candidate emerges from this disciplined narrowing.

This is framed in the paper as evidence that filesystem access enables the proposer to form and test **causal hypotheses** over prior failures, rather than gradient-descending on scalar feedback.

## Design advantages

- **Readable artifacts:** harnesses are Python programs; overfitting shows up as brittle if-chains or hard-coded class mappings, visible on inspection.
- **Coherent proposals:** coding models tend to propose coherent algorithmic structures rather than brittle hard-codes — a natural regularization bias.
- **Pareto search comes for free:** the proposer can target multi-objective preferences (e.g., accuracy vs context cost) by free-form code changes.
- **Transferable harnesses:** discovered solutions generalize across OOD tasks and unseen models without retraining.
- **Improves with proposer quality:** because the outer loop is minimal, Meta-Harness automatically benefits from stronger future coding agents.

## Compared systems (from Table 1 and Related Work)

- **OPRO:** window of past (solution, score) pairs; ~2K tokens/iter.
- **TextGrad:** textual feedback on the current artifact only; ~15K tokens/iter.
- **AlphaEvolve / OpenEvolve:** program database with evaluation scores; ~22K tokens/iter. Designed for stateless single-function optimization, not stateful harnesses.
- **GEPA:** reflective feedback from rollout traces, one candidate at a time; ~8K tokens/iter. Closest on feedback richness, but fixed critique format.
- **Feedback Descent:** pairwise comparison + textual feedback; ~12K tokens/iter.
- **TTT-Discover:** previous solution fragment window; ~26K tokens/iter.
- **Meta-Harness:** full logs and scores; ~10M tokens/iter.

## Limitations and future work

- Evaluated with one particularly strong coding-agent proposer (Claude Code + Opus 4.6). Behavior with other proposers remains open.
- Three domains tested; generalization to other settings is plausible but unverified.
- Natural extension: co-evolve harness and model weights.

## Related

- [[LLM - Harness Engineering]]
- [[LLM - Meta Harness]] (summary)

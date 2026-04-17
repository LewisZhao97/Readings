---
title: Autoresearch — Summary
sources:
  - "[[Karpathy Autoresearch AI agents running research on single-GPU nanochat training automatically]]"
tags:
  - llm
  - ai-agents
  - autonomous-research
  - nanochat
  - training
  - meta-research
created: 2026-04-17
updated: 2026-04-17
type: summary
recommend: partial
distil_times: 0
---

# Autoresearch — Andrej Karpathy

## Summary

A small GitHub repository that turns a single-GPU LLM training setup (a simplified fork of `nanochat`) into an autonomous research sandbox for coding agents. The agent iteratively edits `train.py`, runs a fixed 5-minute training, reads the validation bits-per-byte, then keeps or discards the change — ~100 experiments per overnight run. The researcher's role shifts from editing Python to editing `program.md`, a Markdown "skill" that programs the agent's research organization rather than the model itself.

## Key Points

- **Core flip:** the human no longer edits `train.py`; the agent does. The human programs the *agent's context* (`program.md`), not the model. The repo positions `program.md` as "essentially a super lightweight 'skill'".
- **Three-file design:** `prepare.py` (fixed — data prep, tokenizer, dataloader, eval), `train.py` (agent-editable — model + optimizer + loop, "everything fair game"), `program.md` (human-editable — baseline agent instructions).
- **Fixed 5-minute wall-clock budget** per experiment, excluding compile/startup. Rationale: makes every run — across architecture, batch size, optimizer changes — directly comparable on the same platform, and forces the search to find the best model *for that budget*. Trade-off: results are not comparable across different hardware.
- **Metric:** `val_bpb` (validation bits per byte) — lower is better. Chosen because it is vocab-size-independent, so architectural changes that shift tokenization are fairly compared.
- **Baseline setup:** GPT model, Muon + AdamW optimizers, `DEPTH=8`, `vocab_size=8192`, `WINDOW_PATTERN="SSSL"` (alternating banded + full attention), `MAX_SEQ_LEN` tuned for H100. Tested on a single H100; CPU/MPS deliberately not supported to keep the code minimal.
- **Agent harness:** Claude Code / Codex / etc. with all permissions disabled(?), pointed at `program.md` with a one-line kickoff prompt. The repo is agent-agnostic.
- **Self-modifying research narrative:** the README frames this as "how it all began" for a future where research is run entirely by autonomous agent swarms — a rhetorical framing but consistent with the repo's real purpose of probing what the minimal autonomous-research primitive looks like.
- **Scale-down guidance for small GPUs / Macbooks:** TinyStories dataset, drop `vocab_size` to 4096/2048/bytes, lower `MAX_SEQ_LEN` to 256, reduce `DEPTH` to 4, switch `WINDOW_PATTERN` to just "L", shrink `TOTAL_BATCH_SIZE` to ~2^14.

## Entities Mentioned

- Andrej Karpathy — author; former OpenAI, former Tesla AI; see [[AI - Andrej Karpathy]].
- nanochat — parent repository (full-featured, multi-platform LLM training baseline).
- Muon — newer optimizer used alongside AdamW in the baseline `train.py`.
- Claude Code / Codex — example agent harnesses the repo is designed for.
- TinyStories (karpathy/tinystories-gpt4-clean) — recommended low-entropy dataset for small-compute forks.

## Related

- **Meta-Harness** ([[LLM - Meta-Harness]]) — Autoresearch is an open-source, domain-specific instance of the same pattern Meta-Harness formalizes: an outer-loop coding-agent proposer with filesystem access to prior experiments, code, and scores. Meta-Harness optimizes LLM harnesses; Autoresearch optimizes the trained model itself, but the architecture (agent edits code → runs → logs → iterates) is the same.
- **LLM Wiki pattern** ([[Knowledge Management - LLM Powered Knowledge Management]]) — same author; both treat a Markdown file as the programmable interface between human and agent (here `program.md`, there the wiki notes).
- Potential topics: autonomous AI research, experiment-loop design, fixed-budget comparability in ML.

## Recommend Reason

**Partial.** The repo's body is mostly install instructions — no novel algorithm, no empirical results, no user study. But three conceptual moves are worth distilling: (1) `program.md` as a programmable *skill* for a research agent, reframing "researcher" as "prompt engineer of a research organization"; (2) fixed-time-budget as the comparability trick that makes cross-architecture search tractable; (3) `val_bpb` as a vocab-independent metric for honest architectural comparison. These pair naturally with the [[LLM - Meta-Harness]] entry — Autoresearch is a concrete, public example of the same outer-loop pattern. Distil should extract those concepts and cross-link; skip the install instructions and scale-down tuning recipe.

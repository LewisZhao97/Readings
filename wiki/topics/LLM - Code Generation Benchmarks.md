---
title: "LLM Code Generation Benchmarks"
sources:
  - "[[raw/articles/Vibe Coding XR Accelerating AI + XR Prototyping with XR Blocks and Gemini]]"
tags:
  - llm
  - code-generation
  - benchmark
  - evaluation
created: 2026-04-17
updated: 2026-04-17
type: topic
---

# LLM Code Generation Benchmarks

Benchmarks for measuring how well a language model can produce runnable, correct code from natural-language intent. The family has a common skeleton (a set of prompts + an automated oracle that says "this code passed" or "this code failed") but differs sharply in what the oracle is and what "passed" means.

## Lineage

- **HumanEval** (Chen et al. 2021, OpenAI). 164 Python problems; oracle is a hand-authored unit test suite run against the generated function. `pass@k` = probability that at least one of `k` samples passes all tests. Established the pass@k metric vocabulary.
- **MBPP, APPS, CodeContests, BigCodeBench, LiveCodeBench, SWE-bench, SWE-Gym, TerminalBench** — widening scope from isolated functions to competitive programming, multi-file repos, full software-engineering tasks, terminal workflows. Each extension usually needs hand-written oracles or curated bug fixes.
- **WebDev Arena** (Vichare et al. 2025) — live leaderboard for full web-app generation; human-preference-based rather than unit-test-based.
- **VCXR60** ([[XR - VCXR60]], Du et al. 2026) — 60 XR prompts in natural language; **no hand-authored tests**. The WebGL/WebXR rendering engine's own runtime errors serve as the oracle (see pattern below).

## The rendering-engine-as-implicit-validator pattern

[[XR - VCXR60|VCXR60]]'s key methodological move: for codegen targets where a strict runtime exists — graphics, shader compilation, browser JS, type-checker, linter, even a parser — the runtime's own error output can serve as an automatic correctness oracle. This sidesteps the unit-test authorship bottleneck that makes HumanEval-style benchmarks expensive to build.

The pattern:

- **Target needs a runtime with hard failure modes.** (WebGL throws on bad shader or bad geometry; Python crashes on AttributeError; TypeScript fails typecheck.)
- **Run generated code in a headless harness.** (Playwright + headless Chromium for the web case.)
- **Pass = zero runtime errors.** Fail = any error surfaces during execution.

Benefits: cheap to build, cheap to extend (adding a prompt ≠ writing a test), language-agnostic.

Limits: the oracle catches **hard failures** — code that crashes, won't parse, won't compile, won't render. It does not catch **semantic mismatches**: a volleyball that's labeled "ball" but looks like a cube will pass; a Pacman clone where ghosts don't move will pass if the game at least renders. Aesthetics, usability, interaction fidelity all slip through. The authors explicitly call this out as VCXR60's biggest weakness.

## Pass@k — the metric

`pass@k` = probability that at least one of `k` independently drawn samples passes the oracle. Estimated from `n ≥ k` samples per problem. Pass@1 is the most common single-number summary.

VCXR60 uses `n = 5` samples per prompt; every prompt hit ≥1 success across 5 attempts, so **pass@5 is effectively 100%** — the interesting variation is in pass@1.

## Cost-quality curves

The Vibe Coding XR evaluation (Table 1) is a rare tidy demonstration of a model × thinking-budget × pass-rate Pareto frontier inside a single family:

| Model         | Thinking | Median latency | Pass@1 |
|---------------|----------|---------------:|-------:|
| gemini-3.1-pro | High    | 86.02 s        | 95.5% |
| gemini-3.1-pro | Low     | 33.39 s        | 94.1% |
| gemini-3-flash | High    | 22.26 s        | 87.8% |
| gemini-3-flash | Low     | 17.30 s        | 87.4% |

Pro + High is the pass-rate winner; Flash + Low is ≈5× faster at an ~8 pp quality cost. Thinking budget matters more for complex prompts (multi-system apps with physics + interaction + animation) than for simple ones.

## Framework vs model: the 70% → 95.5% story

Du et al. report that VCXR60 pass@1 started at ≈70% on XR Blocks v0.1.0 and reached 95.5% on v0.11.0 six months later. Because both the framework and the underlying Gemini model improved in that window, the study does **not** isolate how much of the 25-pp gain came from framework polish vs. model upgrades. Still, they attribute *most* early failures to (a) framework edge-case bugs surfacing under LLM-generated code rather than (b) pure API hallucination — consistent with the broader observation that the *API surface* of a codegen target is often a bigger lever than the model's raw capability. See [[LLM - Harness Engineering]] for the same pattern at the agent-task level.

## Related

- [[LLM - Harness Engineering]] — same "shape your codegen target to the model's grain" insight, different target (agent harnesses vs. codegen APIs).
- [[LLM - Meta-Harness]] — one approach to automating the "shape your target" search.
- [[XR - Vibe Coding XR]] — the system VCXR60 was built to evaluate.
- [[XR - VCXR60]] — the benchmark itself.

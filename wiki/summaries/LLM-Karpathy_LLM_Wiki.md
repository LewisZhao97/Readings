---
title: LLM Wiki — Summary
sources:
  - "[[Karpathy LLM Wiki]]"
tags:
  - knowledge-management
  - wiki
  - methodology
  - RAG-alternative
created: 2026-04-10
updated: 2026-04-11
type: summary
recommend: yes
distil_times: 2
---

# LLM Wiki — Andrej Karpathy

## Summary

Andrej Karpathy proposes a pattern for building personal knowledge bases where an LLM incrementally builds and maintains a persistent wiki from ingested sources, rather than re-deriving knowledge per query via RAG. The human curates sources and asks questions; the LLM handles all bookkeeping.

## Key Points

- **Core problem with RAG:** knowledge is rediscovered from scratch on every question — no accumulation
- **LLM Wiki alternative:** LLM reads new sources, integrates them into a persistent wiki — updating entity pages, revising summaries, noting contradictions. Knowledge compiles once.
- **Three-layer architecture:** raw sources (immutable, human-curated) → wiki (LLM-generated markdown) → schema (conventions doc like CLAUDE.md)
- **Three operations:** ingest (process new source → touch ~10-15 pages), query (answer from wiki, file good answers back), lint (health-check for contradictions, orphans, staleness)
- **Two navigation files:** index (content catalog, delegation map) and log (chronological append-only record)
- **Why it works:** humans abandon wikis because maintenance burden grows faster than value. LLMs eliminate maintenance cost — they don't get bored, don't forget cross-references, can touch 15 files in one pass.
- **Historical connection:** related to Vannevar Bush's [[entities/Knowledge_Management-Memex|Memex]] (1945) — the missing piece was who does maintenance.

## Entities Mentioned

- [[entities/AI-Andrej_Karpathy|Andrej Karpathy]] — author
- [[entities/Knowledge_Management-Memex|Memex]] — historical predecessor concept
- RAG (Retrieval-Augmented Generation) — contrasted approach

## Related

- [[topics/Knowledge_Management-LLM_Powered_Knowledge_Management|LLM-Powered Knowledge Management]]

## Recommend Reason

**Yes.** This is a foundational methodology document directly relevant to how this knowledge base is built. It articulates the core insight (compile once vs. re-derive per query), provides a clear three-layer architecture, defines the key operations, and explains why LLMs solve the maintenance problem that kills human wikis. Worth multiple distillation passes — the practical details (tooling, indexing strategy, schema co-evolution) reward deeper reading.

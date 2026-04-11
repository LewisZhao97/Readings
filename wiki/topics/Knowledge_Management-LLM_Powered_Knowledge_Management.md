---
title: LLM-Powered Knowledge Management
sources:
  - "[[Karpathy LLM Wiki]]"
tags:
  - knowledge-management
  - wiki
  - LLM
  - RAG
  - methodology
created: 2026-04-11
updated: 2026-04-11
type: topic
---

# LLM-Powered Knowledge Management

Using LLMs to build, maintain, and query persistent knowledge bases — as opposed to traditional RAG which re-derives knowledge per query.

## Core Idea

Traditional approach (RAG): store documents → retrieve chunks at query time → generate answer from scratch. No knowledge accumulates. NotebookLM, ChatGPT file uploads, and most RAG systems work this way.

LLM Wiki approach: store documents → LLM reads and **integrates** into a persistent wiki → wiki compounds over time. Cross-references, contradictions, and synthesis are maintained continuously.

The key difference: the wiki is a persistent, compounding artifact. A single new source might touch 10-15 wiki pages.

## Architecture Pattern

Three layers identified by [[entities/AI-Andrej_Karpathy|Karpathy]]:

1. **Raw sources** — immutable collection of original documents. Human curates what goes in. LLM never modifies.
2. **Wiki** — LLM-generated knowledge artifacts. Multiple types serve different purposes:
   - **Summaries** — faithful 1:1 summary of each ingested source
   - **Entity pages** — accumulate knowledge about a specific thing across ALL sources
   - **Topic pages** — synthesize a concept area across sources
   - **Comparisons** — X vs Y analysis when meaningful
   - **Synthesis** — high-level cross-cutting analysis (emerges after many sources)
3. **Schema** — configuration document defining structure, conventions, and workflows. Co-evolved between human and LLM over time.

## Operations

| Operation | Purpose | Frequency |
|-----------|---------|-----------|
| **Ingest** | Capture source + produce summary with recommendation | Per new source |
| **Distil** | Deep-process raw source → create/update wiki pages | Per recommended source (repeatable) |
| **Query** | Answer questions from wiki; good answers file back as new pages | On demand |
| **Lint** | Health-check: contradictions, orphans, staleness | Periodic |

## Key Insight: Filing Answers Back

Good query answers — comparisons, analyses, discovered connections — should be filed back into the wiki as new pages. This way explorations compound in the knowledge base just like ingested sources do. Knowledge shouldn't disappear into chat history.

## Indexing Strategy

- **Index file** — content-oriented catalog, read first to find relevant pages. Works at moderate scale (~100 sources, ~hundreds of pages) without needing embedding-based RAG infrastructure.
- **Log file** — chronological append-only record. Parseable with simple tools.
- At larger scale, consider local search tools like qmd (hybrid BM25/vector search with LLM re-ranking).

## Why LLMs Solve the Maintenance Problem

> "Humans abandon wikis because maintenance burden grows faster than value."

LLMs eliminate the bookkeeping cost: updating cross-references, keeping summaries current, noting contradictions, maintaining consistency across pages. The human focuses on curation, direction, and thinking.

## Tooling Ecosystem

- **Obsidian** — browsing interface with graph view, Web Clipper for ingesting articles, Dataview for frontmatter queries, Marp for slide decks
- **Git** — version history, branching, collaboration for free
- **qmd** — local markdown search engine for larger wikis

## Historical Context

The pattern is spiritually connected to [[entities/Knowledge_Management-Memex|Vannevar Bush's Memex]] (1945) — a vision of personal knowledge with associative trails. The missing piece was maintenance automation.

## Sources

- [[summaries/LLM-Karpathy_LLM_Wiki|LLM Wiki — Karpathy]]

## Core Idea

Traditional approach (RAG): store documents → retrieve chunks at query time → generate answer from scratch. No knowledge accumulates.

LLM Wiki approach: store documents → LLM reads and **integrates** into a persistent wiki → wiki compounds over time. Cross-references, contradictions, and synthesis are maintained continuously.

## Architecture Pattern

Three layers identified by [[entities/AI-Andrej_Karpathy|Karpathy]]:

1. **Raw sources** — immutable collection of original documents. Human curates what goes in. LLM never modifies.
2. **Wiki** — LLM-generated knowledge artifacts. Multiple types serve different purposes:
   - **Source summaries** — faithful 1:1 summary of each ingested source
   - **Entity pages** — accumulate knowledge about a specific thing across ALL sources
   - **Topic pages** — synthesize a concept area across sources
   - **Comparisons** — X vs Y analysis when meaningful
   - **Synthesis** — high-level cross-cutting analysis (emerges after many sources)
3. **Schema** — configuration document defining structure, conventions, and workflows

## Operations

| Operation | Purpose | Frequency |
|-----------|---------|-----------|
| **Ingest** | Capture source into raw storage | Per new source |
| **Distil** | Process raw source → create/update wiki pages | Per new source |
| **Query** | Answer questions from the wiki | On demand |
| **Lint** | Health-check: contradictions, orphans, staleness | Periodic |

## Why LLMs Solve the Maintenance Problem

> "Humans abandon wikis because maintenance burden grows faster than value."

LLMs eliminate the bookkeeping cost: updating cross-references, keeping summaries current, noting contradictions, maintaining consistency across pages. The human focuses on curation, direction, and thinking.

## Historical Context

The pattern is spiritually connected to [[entities/Knowledge_Management-Memex|Vannevar Bush's Memex]] (1945) — a vision of personal knowledge with associative trails. The missing piece was maintenance automation.

## Sources

- [[summaries/LLM-Karpathy_LLM_Wiki|LLM Wiki — Karpathy]]

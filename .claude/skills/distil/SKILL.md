---
name: distil
description: Deep-process an ingested source into wiki knowledge. Creates/updates entity pages, topic pages, and more. Run after /ingest when recommend is yes or partial.
user_invocable: true
---

# Distil Source

Deep-process an ingested source into wiki knowledge. Conventions (naming, frontmatter, directory structure) are defined in CLAUDE.md — follow them, don't repeat them here.

## Input

The user specifies which source to distil. If not specified, check `wiki/log.md` for recent ingests and ask.

## Steps

1. **Read the raw source in full.** The raw is the source of truth — distil must extract knowledge from it, not from the summary. If the raw exceeds the Read tool's token limit, read it in chunks using `offset`/`limit` until the whole body is covered. The existing summary may be read first as orienting context (what the article is about, where the key sections are), but it must NOT substitute for reading the raw.
2. **Read existing context.** Read `wiki/_wiki-index.md`, then relevant sub-indexes to find pages that may need updating.
3. **Extract and organize:** entities, topics, terminology, connections, contradictions, potential comparisons.
4. **Create or update entity pages** in `wiki/entities/`. Entity pages accumulate across ALL sources — never overwrite, always add.
5. **Create or update topic pages** in `wiki/topics/`.
6. **Optionally create comparison pages** if two entities warrant a side-by-side diff.
7. **Flag potential synthesis opportunities to the user — do NOT auto-create.** A synthesis page is cross-cutting and rare. Scan for cases where the distil contributes a new data point to a pattern that:
   - spans **≥3 existing entity/topic pages** (horizontal — not just a single source's thesis),
   - is **not fully argued in any single source** already in the KB (otherwise it's a topic, not a synthesis),
   - would meaningfully help a reader navigate the accumulated mass (skip if the KB is still too thin for the synthesis to pay for itself).
   When these conditions hold, include a "Potential synthesis" note in the final presentation describing the candidate (title, the ≥3 pages it would pull from, one-sentence thesis) and ask the user whether to create it. Do not write the synthesis page unprompted.
8. **Update all indexes** (sub-indexes + `_wiki-index.md` stats).
9. **Update glossary** with new terms.
10. **Increment `distil_times`** in the summary file. If a closer reading of the raw reveals the summary is inaccurate or misleading, correct the summary body as well — the summary should remain a faithful orienting snapshot.
11. **Log it** in `wiki/log.md`.

## Rules

- A single source may create/update many wiki pages — that's expected
- Prefer updating existing pages over creating near-duplicates
- Entity pages ACCUMULATE — never overwrite
- Cross-reference with `[[links]]`
- Update `updated` date in frontmatter of modified articles
- Do NOT touch raw files (ingest already handled that)

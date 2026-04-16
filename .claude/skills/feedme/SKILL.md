---
name: feedme
description: Search the web for recent, relevant papers/content on a topic or author. Writes a ranked candidate list to raw/feeds/YYYY-MM-DD.md for the user to selectively /ingest.
user_invocable: true
---

# Feed Me

Discover new reading candidates via web search. Produces a dated candidate list in `raw/feeds/` — not an ingest. User reviews, picks what to `/ingest`. Conventions (naming, frontmatter, directory structure) are defined in CLAUDE.md — follow them, don't repeat them here.

## Input

`/feedme <topic or author>` — free-text. Examples:
- `/feedme neural rendering 2026`
- `/feedme Andrej Karpathy`
- `/feedme XR depth reconstruction`

If no argument, ask the user. Do not default.

## Steps

1. **Search.** Use Exa (`mcp__exa__web_search_exa`) or `WebSearch` for recent content matching the argument. Prefer the last ~7 days; include older items only if highly relevant and absent from the KB. Use multiple queries if needed — don't rely on one phrasing.
2. **Fetch metadata** for each candidate: title, authors, venue/source, publication date, URL, 1–2 sentence hook. For papers, also fetch the **DOI** (e.g. `10.1145/3721238.3730595`) — check arXiv, publisher page (ACM/IEEE/Springer), or Crossref. If no DOI exists (e.g. preprint-only or blog post), record `DOI: none`.
3. **Cross-check duplicates.** Compare candidate titles/URLs against `raw/articles/` filenames and `wiki/summaries/_summaries-index.md`. Mark matches as `already-ingested` with a link to the existing summary. Do NOT exclude them.
4. **Rank** by relevance × signal (citations, conference, engagement). Keep 5–10 — quality over quantity.
5. **Write** to `raw/feeds/YYYY-MM-DD.md`. If that file exists, use `YYYY-MM-DD_2.md` (then `_3`, …) — feeds are immutable snapshots. Pick topical tags for the frontmatter that describe the subject of the query and candidates (e.g. `neural-rendering`, `llm-agents`) — not the generic word `feed`. Sections:
   - **Query** — argument as given + actual search queries used.
   - **Candidates** — ranked list; each item: title, authors, venue/date, URL, **DOI** (for papers; `none` if N/A), hook, why-matched, status (`new` / `already-ingested → [[link]]`).
   - **Top picks** — 2–3 items with brief reason.
6. **Log** in `wiki/log.md`: `## [YYYY-MM-DD] feed | <query>`.
7. **Present** top picks to the user. Do NOT auto-ingest.

## Rules

- Feed files are immutable once written — re-run into a new file, never edit.
- Do NOT restrict to existing entities/topics — feeds are a surface for *new* areas.
- If nothing notable turns up, say so explicitly. Do not pad.
- Only write to `raw/feeds/` and append to `log.md`. Do not touch `raw/articles/` or wiki pages.
- Record the exact search queries at the top of the feed file — reproducibility.

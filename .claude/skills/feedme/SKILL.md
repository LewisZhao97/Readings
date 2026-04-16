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

1. **Parse sources from the query.** Scan for named venues / source types (e.g. "ACM SIGGRAPH", "IEEE VR", "TVCG", "arXiv", "blog posts"). If any are named, treat them as a **hard allowlist** — candidates from outside must be excluded. For each, derive:
   - a **domain list**: SIGGRAPH → `dl.acm.org`, `siggraph.org`, `s2025.siggraph.org`, `realtimerendering.com`; IEEE → `ieeexplore.ieee.org`; arXiv → `arxiv.org`; CVF → `openaccess.thecvf.com`; blogs → leave open.
   - an **index URL** when the venue has a curated list: e.g. `https://www.realtimerendering.com/kesen/sig<YYYY>.html` (Ke-Sen Huang's SIGGRAPH tracker), IEEE VR / TVCG accepted-papers pages, `https://arxiv.org/list/<category>/recent`.
2. **Search — two passes.**
   - **(a) Keyword search with domain filter.** If step 1 produced a domain allowlist, use `WebSearch` with `allowed_domains` set to that list (native hard filter). Without named sources, use `mcp__exa__web_search_exa` for open-web semantic search. Either way, run multiple query phrasings — don't rely on one.
   - **(b) Index-page fetch.** For each known index URL from step 1, fetch it directly with `WebFetch` or `mcp__exa__web_fetch_exa` and scan for on-topic items. Venue index pages are curated lists that keyword search frequently misses — this pass is how SIGGRAPH/IEEE-VR papers get fully represented.
   Prefer content from the last ~7 days; include older items only if highly relevant and absent from the KB.
3. **Fetch metadata** for each candidate: title, authors, venue/source, publication date, URL, 1–2 sentence hook. For papers, also fetch the **DOI** (e.g. `10.1145/3721238.3730595`) — check arXiv, publisher page (ACM/IEEE/Springer), or Crossref. If no DOI exists (e.g. preprint-only or blog post), record `DOI: none`.
4. **Cross-check duplicates.** Compare candidate titles/URLs against `raw/articles/` filenames and `wiki/summaries/_summaries-index.md`. Mark matches as `already-ingested` with a link to the existing summary. Do NOT exclude them.
5. **Rank** by relevance × signal (citations, conference, engagement). Keep 5–10 — quality over quantity.
6. **Write** to `raw/feeds/YYYY-MM-DD.md`. If that file exists, use `YYYY-MM-DD_2.md` (then `_3`, …) — feeds are immutable snapshots. Pick topical tags for the frontmatter that describe the subject of the query and candidates (e.g. `neural-rendering`, `llm-agents`) — not the generic word `feed`. Sections:
   - **Query** — argument as given + actual search queries used + source allowlist (if any).
   - **Candidates** — ranked list; each item: title, authors, venue/date, URL, **DOI** (for papers; `none` if N/A), hook, why-matched, status (`new` / `already-ingested → [[link]]`).
   - **Top picks** — 2–3 items with brief reason.
7. **Log** in `wiki/log.md`: `## [YYYY-MM-DD] feed | <query>`.
8. **Present** top picks to the user. Do NOT auto-ingest.

## Rules

- Feed files are immutable once written — re-run into a new file, never edit.
- Do NOT restrict to existing entities/topics — feeds are a surface for *new* areas.
- When the user names sources, that IS a hard filter — drop candidates from outside the allowlist even if they look relevant. If nothing on-allowlist matches, say so; do not substitute off-allowlist items.
- If nothing notable turns up, say so explicitly. Do not pad.
- Only write to `raw/feeds/` and append to `log.md`. Do not touch `raw/articles/` or wiki pages.
- Record the exact search queries AND the source allowlist at the top of the feed file — reproducibility.

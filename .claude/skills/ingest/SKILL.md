---
name: ingest
description: Read a raw source, produce a summary with recommendation and reason. User then decides whether to /distil.
user_invocable: true
---

# Ingest Source

Process a new raw source into the knowledge base. Conventions (naming, frontmatter, directory structure) are defined in CLAUDE.md — follow them, don't repeat them here.

## Input

The user specifies which raw file to ingest, a URL, or a title (often from a recent `/feedme` candidate list).

**Resolving the input:**
1. If a **raw file path** is given → use it.
2. If a **title** is given (no path, no URL) → glob `raw/articles/` for files matching key words from the title. The user's workflow is to `/feedme`, then manually download picks into `raw/articles/` before asking to ingest, so the file is usually already there.
   - If found → proceed with that file.
   - If not found → tell the user it's not in `raw/` and ask them to download it. Do NOT auto-fetch.
3. If a **URL** is explicitly given as the argument → fetch with WebFetch and save to `raw/articles/` (the one case where auto-fetch is authorized).

## Steps

1. **Read the raw file.** If `status: true`, inform the user. Otherwise proceed.
2. **Refine tags.** Review the raw file's `tags` — the user's may be approximate. Replace with accurate, consistent tags after reading the content.
3. **Create summary** in `wiki/summaries/`. Include:
   - Concise overview (2-3 sentences)
   - Key points as bullet list
   - Entities mentioned
   - Related topics
   - **"Recommend Reason" section at the end** explaining why to distil (or not)
4. **Update raw frontmatter:** `status: true`, `ingested: YYYY-MM-DD`
5. **Update indexes:** `_summaries-index.md` and `_wiki-index.md` stats
6. **Log it** in `wiki/log.md`
7. **Present to user.** Show summary and recommendation. Ask if they want to `/distil`.

## Rules

- Do NOT modify body content of raw files — only frontmatter (`status`, `ingested`, `tags`)
- Do NOT create entity, topic, comparison, or synthesis pages (that's `/distil`)
- Summary is always created regardless of recommendation
- Only create a raw file when the user explicitly provides a URL as the argument. A bare title is NOT a fetch request — look in `raw/` first, ask if absent.

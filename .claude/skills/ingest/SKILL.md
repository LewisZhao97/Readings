---
name: ingest
description: Read a raw source, produce a summary with recommendation and reason. User then decides whether to /distil.
user_invocable: true
---

# Ingest Source

Process a new raw source into the knowledge base. Conventions (naming, frontmatter, directory structure) are defined in CLAUDE.md — follow them, don't repeat them here.

## Input

The user specifies which raw file to ingest, or provides a URL (fetch with WebFetch and save to `raw/`).

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
- Only exception for creating raw files: user provides a URL

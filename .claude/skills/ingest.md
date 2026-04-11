---
name: ingest
description: Read a raw source, produce a summary with recommendation and reason. User then decides whether to /distil.
user_invocable: true
---

# Ingest Source

You are ingesting a new source into the knowledge base.

## Input

The user has already copied or downloaded the original source content into a file under `raw/<source-type>/` with frontmatter. They tell you which file to ingest.

Alternatively, the user may provide a URL — fetch it with WebFetch and save faithfully to the appropriate `raw/` directory with frontmatter.

## Raw file frontmatter template

```yaml
---
title: "..."
source: "URL or citation"
author: "..."
date: YYYY-MM-DD
tags: [...]
status: pending
ingested: none
---
```

- `status: pending` + `ingested: none` = not yet processed
- `status: done` + `ingested: YYYY-MM-DD` = processed

## Steps

1. **Read the raw file.** Read from `raw/<source-type>/`. Check frontmatter: if `status: pending` and `ingested: none`, proceed. If already done, inform the user.

2. **Create summary.** Read the full source and create a summary file in `wiki/summaries/` following the naming convention `<Field>-<Title_With_Underscores>.md`. The summary includes:
   - A concise overview (2-3 sentences)
   - Key points as bullet list
   - Entities mentioned
   - A `recommend` field in frontmatter: `yes`, `partial`, or `no`
   - **A "Recommend Reason" section at the end of the body** explaining why you recommend (or don't) further distillation

Summary frontmatter:
```yaml
---
title: "... — Summary"
sources:
  - "[[raw source filename]]"
tags: [...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: summary
recommend: yes | partial | no
distil_times: 0
---
```

3. **Update raw frontmatter.** Change `status: pending` → `status: done` and `ingested: none` → `ingested: YYYY-MM-DD` (today's date).

4. **Update indexes.**
   - Add entry to `wiki/summaries/_summaries-index.md`
   - Update stats in `wiki/_wiki-index.md`

5. **Log it.** Append an entry to `wiki/log.md`:

```markdown
## [YYYY-MM-DD] ingest | <title>

- **Source:** <url or citation>
- **Raw:** `raw/<source-type>/<filename>.md`
- **Summary:** `wiki/summaries/<Field>-<Title>.md`
- **Recommend:** yes | partial | no
```

6. **Present to user.** Show the summary and recommendation reason. Ask if they want to `/distil` for deeper processing.

## Rules

- Do NOT modify the body content of raw files — only update `status` and `ingested` in frontmatter
- Do NOT create entity, topic, comparison, or synthesis pages (that's `/distil`)
- The summary file is always created regardless of recommendation
- Always include a "Recommend Reason" section at the end of the summary body
- Only exception for creating raw files: if the user provides a URL, you may create the raw file

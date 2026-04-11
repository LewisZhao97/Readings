---
name: distil
description: Deep-process an ingested source into wiki knowledge. Creates/updates entity pages, topic pages, and more. Run after /ingest when recommend is yes or partial.
user_invocable: true
---

# Distil Source

You are distilling an ingested source into deeper wiki knowledge. The source has already been ingested (summary exists in `wiki/summaries/`).

## Input

The user specifies which source to distil. If not specified:
- Check `wiki/log.md` or scan `raw/` for files with `status: pending`
- List them with their recommend status and ask which to process

## Steps

1. **Read the raw source.** Read the file from `raw/<source-type>/<Field>-<Title>.md`.

2. **Read existing context.** Read `wiki/_wiki-index.md` (delegation map). Then read relevant sub-indexes (`_entities-index.md`, `_topics-index.md`, etc.) to check for existing pages that may need updating.

3. **Extract and organize.** Identify:
   - Key concepts and findings
   - Entities mentioned (people, tools, models, companies)
   - Broader topics this connects to
   - New terminology (for glossary)
   - Connections to existing wiki articles
   - Contradictions with existing knowledge
   - Potential comparisons worth making

4. **Create or update entity pages** in `wiki/entities/`:
   - Named: `<Field>-<Entity_Name>.md`
   - If entity page exists: add new information from this source
   - If new: create entity page
   - Entity pages accumulate knowledge from ALL sources — don't overwrite, add

5. **Create or update topic pages** in `wiki/topics/`:
   - Named: `<Field>-<Topic_Name>.md`
   - If topic page exists: integrate new information
   - If new: create topic page

6. **Optionally create comparison or synthesis pages** if the source warrants it.

7. **Update all indexes:**
   - `wiki/entities/_entities-index.md` — add new entities
   - `wiki/topics/_topics-index.md` — add new topics
   - `wiki/comparisons/_comparisons-index.md` or `wiki/synthesis/_synthesis-index.md` if applicable
   - `wiki/_wiki-index.md` — update stats

8. **Update glossary.** Add new terms to `glossary.md` with links to defining wiki articles.

9. **Increment distil_times.** In the corresponding summary file (`wiki/summaries/`), increment the `distil_times` field by 1. Re-distilling the same source is encouraged — each pass may surface new connections or nuances.

10. **Log it.** Append to `wiki/log.md`:

```markdown
## [YYYY-MM-DD] distil | <title>

- **Source:** `raw/<source-type>/<Field>-<Title>.md`
- **Wiki pages created:** <list>
- **Wiki pages updated:** <list>
- **Glossary terms added:** <list>
```

## Rules

- A single source may create/update many wiki pages — that's expected
- Each wiki article should be self-contained but cross-referenced with `[[links]]`
- Prefer updating existing entity/topic pages over creating near-duplicates
- Entity pages ACCUMULATE across sources — never overwrite, always add
- Always update the `updated` date in frontmatter of modified articles

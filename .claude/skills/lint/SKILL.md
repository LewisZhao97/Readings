---
name: lint
description: Health-check the wiki for contradictions, orphans, stale claims, missing links, index sync issues, and pending sources.
user_invocable: true
---

# Lint Knowledge Base

You are performing a health check on the knowledge base.

## Steps

1. **Scan raw sources.** Find any with `status: pending` — these haven't been distilled yet.

2. **Check naming convention.** Verify files follow the conventions defined in CLAUDE.md.

3. **Scan wiki articles.** Read all articles across all knowledge types and check for:
   - **Broken links:** `[[links]]` pointing to non-existent pages
   - **Orphan pages:** articles with no inbound links from other articles
   - **Missing pages:** concepts frequently referenced but lacking their own article
   - **Stale content:** articles whose sources have been superseded by newer sources
   - **Contradictions:** conflicting claims across different articles
   - **Missing cross-references:** articles on related topics that don't link to each other
   - **Frontmatter issues:** missing or malformed fields, missing `type` or `recommend` field

4. **Check indexes.** For each sub-index (`_summaries-index.md`, `_entities-index.md`, `_topics-index.md`, `_comparisons-index.md`, `_synthesis-index.md`):
   - Verify it lists all actual files in its directory
   - Verify it has no entries for deleted/missing pages
   - Check `wiki/_wiki-index.md` stats are accurate

5. **Check glossary.** Verify terms in `glossary.md` link to existing wiki articles.

6. **Report.** Present findings organized by severity:
   - **Errors:** broken links, missing frontmatter, indexes out of sync
   - **Warnings:** orphan pages, pending sources, missing cross-references
   - **Suggestions:** potential new articles, entities worth creating, topics to expand

7. **Log it.** Append to `wiki/log.md`:

```markdown
## [YYYY-MM-DD] lint

- **Articles scanned:** <count>
- **Errors:** <count>
- **Warnings:** <count>
- **Suggestions:** <count>
```

## Rules

- This is a read-only audit — do NOT fix issues automatically
- Present findings clearly so the user can decide what to address

---
name: query
description: Answer questions using the wiki delegation map. Reads only the pages needed to minimize context usage.
user_invocable: true
---

# Query Knowledge Base

You are answering a question using the knowledge base.

## Steps

1. **Read the delegation map.** Read `wiki/_wiki-index.md` to determine which knowledge type(s) are relevant:
   - "What did article X say?" → `summaries/`
   - "What is X?" / "Who is X?" → `entities/`
   - "Explain X" / "How does X work?" → `topics/`
   - "How does X compare to Y?" → `comparisons/`
   - "What's the big picture on X?" → `synthesis/`

2. **Read the sub-index.** Read the relevant `_<type>-index.md` to find specific pages.

3. **Read only the needed pages.** Do NOT load everything — read only the pages identified in step 2. Follow `[[links]]` to related pages only if the initial pages don't fully answer the question.

4. **Synthesize.** Answer the question with citations to wiki pages.

5. **Optionally file back.** If the answer is a substantial synthesis, ask the user if they'd like to save it as a new wiki page.

## Rules

- Always cite which wiki articles informed the answer
- If the wiki doesn't contain enough information, say so clearly
- Don't fabricate information that isn't in the wiki or raw sources
- Minimize context usage — use the delegation map, don't scan everything

---
name: note
description: Summarize the current conversation into a field-categorized note under raw/notes/<field>/. For preserving substantive discussion (sources, evidence, decisions) before the session ends.
user_invocable: true
---

# Note

Capture the substantive part of the current conversation into `raw/notes/<field>/`. Conventions (naming, frontmatter) are in CLAUDE.md.

## When to use

Use for multi-turn discussions that produced new reasoning, sources, or decisions worth preserving — not lookups, not external source material (use `/ingest`), not content that belongs on a wiki page (use `/distil`).

## Input

`/note` — infer field and title.
`/note <title>` — user gives title, skill infers field.
`/note <field>/<title>` — both given.

If the conversation spans multiple distinct topics, ask which to capture.

## Steps

1. **Pick the field folder.** Match the field vocabulary the wiki already uses (prefixes like `LLM - ...`, `XR - ...`, `Graphics - ...`) translated to lowercase-hyphen directory names. Existing folders first; create a new one only if no existing field fits.
2. **Name the file.** Match the style of sibling notes in the chosen folder. If the name collides, ask whether to append or rename.
3. **Write the note** with:
   - Lightweight frontmatter: `title`, `date`, `type: note`, `tags`.
   - **Context** — what prompted the discussion, linking to any triggering note/article/wiki page.
   - **Body** — the substance. Prefer quoted primary sources over paraphrase.
   - **Actions taken** — concrete changes made during the conversation (wiki edits, commits). Link, don't duplicate.
   - **Open questions** — flagged follow-ups. Omit if none.
   - **References** — URLs, DOIs, citations.
4. **Present** the path and a one-line summary. Ask if anything should be added.

## Rules

- Do not touch `wiki/`, `wiki/log.md`, or any sub-indexes. Notes are not a logged operation.
- Skip small talk, tool mechanics, and anything re-derivable from the wiki.
- If a wiki edit was made in the conversation, link to it in *Actions taken* — do not restate its content in the note body.
- If the user has uncommitted wiki edits, remind them at the end.

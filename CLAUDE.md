# Readings — Personal Knowledge Base

A personal wiki-style knowledge base. Source materials live in `raw/` (immutable), then get distilled into structured, interlinked wiki articles in `wiki/`. The LLM handles bookkeeping — cross-references, indexing, consistency.

## Directory Structure

```
/
├── CLAUDE.md                    # This file — schema and conventions
├── raw/                         # Layer 1: Original sources (immutable body)
│   ├── articles/ papers/ notes/ tweets/ assets/ ...
├── wiki/                        # Layer 2: Distilled knowledge
│   ├── _wiki-index.md           # Top-level delegation map — read FIRST
│   ├── log.md                   # Chronological append-only log
│   ├── summaries/               # 1:1 per ingested source
│   ├── entities/                # People, tools, models, concepts (accumulates across sources)
│   ├── topics/                  # Broad concept areas synthesized across sources
│   ├── comparisons/             # X vs Y
│   └── synthesis/               # Rare cross-cutting analysis
└── glossary.md                  # Cross-cutting term definitions
```

Each wiki subdirectory has a `_<type>-index.md` sub-index.

## Wiki Knowledge Types

| Type | Purpose | Query pattern |
|------|---------|---------------|
| **Summary** | Summary of ONE source, with read recommendation | "What did article X say?" |
| **Entity** | A specific thing: person, model, tool. Accumulates across ALL sources. | "What is X?" |
| **Topic** | Broad concept area synthesized across sources | "Explain X" |
| **Comparison** | X vs Y | "How does X compare to Y?" |
| **Synthesis** | High-level cross-cutting. Rare. | "Big picture on X?" |

### Query delegation flow

1. Read `wiki/_wiki-index.md` → find knowledge type(s)
2. Read `<type>/_<type>-index.md` → find specific page(s)
3. Read only the relevant pages

Keeps context minimal at scale.

## File Naming

- **Raw files:** user-named, no enforced convention.
- **Wiki pages** (except indexes/`log.md`/`glossary.md`): `<Field> - <Full Name>.md` — e.g., `Deep Learning - Attention Is All You Need.md`. Field is the domain area (`LLM`, `AI`, `Deep Learning`, `Knowledge Management`, …).
- **Index files:** `_<type>-index.md` — underscore prefix sorts them first.

## Frontmatter

**All tags use YAML block-list style** (`- tag` per line). Never flow style (`[a, b, c]`). Obsidian-native.

### Raw sources (`raw/**/*.md`)

```yaml
---
title: "Article or paper title"
source: "URL or citation"
author: "Author name(s)"
date: 2026-04-10            # Date of the original source
tags:
  - tag1
status: true | false        # true = ingested
ingested: none | 2026-04-10 # date when ingested
---
```

LLM updates `status`, `ingested`, and refines `tags` during ingest — never touches body content.

### Wiki summaries (`wiki/summaries/*.md`)

```yaml
---
title: "Article title — Summary"
sources:
  - "[[raw/articles/Source_File]]"
tags:
  - tag1
created: 2026-04-10
updated: 2026-04-10
type: summary
recommend: yes | partial | no
distil_times: 0
---
```

- `recommend`: `yes` = deep-read worthwhile · `partial` = some bits · `no` = summary suffices.
- `distil_times`: increments each `/distil` pass; re-distilling surfaces missed detail.

### Wiki articles (`wiki/{entities,topics,comparisons,synthesis}/*.md`)

```yaml
---
title: "Article title"
sources:
  - "[[raw/articles/Source_File]]"
tags:
  - tag1
created: 2026-04-10
updated: 2026-04-10
type: entity | topic | comparison | synthesis
---
```

### Glossary (`glossary.md`)

Inline entries. Each term links back to the wiki article(s) defining it.

## Core Skills

- `/ingest` — raw → summary with recommendation.
- `/distil` — summary → entities/topics/comparisons; updates indexes, glossary, cross-links.
- `/query` — answer via delegation map.
- `/lint` — health check.

Each skill's details live in its own `SKILL.md`.

## Conventions

- Wiki articles use `[[internal links]]` for cross-references.
- Each article self-contained but cross-referenced.
- Entity pages **accumulate** across sources — never overwrite.
- Keep indexes in sync after every distil.
- `log.md` is append-only; entries use `## [YYYY-MM-DD] <operation> | <title>`.
- Bump `updated:` in frontmatter when modifying a wiki page.

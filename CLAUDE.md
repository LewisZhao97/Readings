# Readings — Personal Knowledge Base

## Project Purpose

A personal wiki-style knowledge base. Source materials are saved into `raw/` (immutable), then distilled into structured, interlinked wiki articles in `wiki/`. The LLM handles all bookkeeping — cross-references, indexing, consistency.

## Directory Structure

```
/
├── CLAUDE.md                    # Schema — conventions, workflows, frontmatter spec (this file)
├── raw/                         # Layer 1: Original sources (immutable — LLM never modifies)
│   ├── articles/                # Web articles, blog posts
│   ├── papers/                  # Academic papers
│   ├── notes/                   # Personal notes, meeting notes
│   ├── tweets/                  # X/Twitter threads
│   ├── assets/                  # Images, attachments
│   └── ...                      # Add source-type folders as needed
├── wiki/                        # Layer 2: Distilled knowledge (LLM writes)
│   ├── _wiki-index.md           # Top-level delegation map — read this FIRST
│   ├── log.md                   # Chronological append-only log
│   ├── summaries/               # 1:1 summaries of each ingested source (created during ingest)
│   │   └── _summaries-index.md  # Sub-index for summaries
│   ├── entities/                # Pages about specific things (people, tools, models, concepts)
│   │   └── _entities-index.md   # Sub-index for entities
│   ├── topics/                  # Broad concept areas synthesized across sources
│   │   └── _topics-index.md     # Sub-index for topics
│   ├── comparisons/             # X vs Y analysis
│   │   └── _comparisons-index.md # Sub-index for comparisons
│   └── synthesis/               # High-level cross-cutting analysis
│       └── _synthesis-index.md  # Sub-index for synthesis
└── glossary.md                  # Cross-cutting term definitions
```

## Raw Sources (`raw/`)

Organized by **source type** (where the content came from), not by topic. The user copies or downloads the exact original content into a file. Raw files are immutable — the LLM never modifies the body content after ingest. Only the `status` field in frontmatter is updated.

## Wiki Knowledge Types (`wiki/`)

Organized by **knowledge type** for efficient AI-agent routing (delegation map):

| Type | Directory | Purpose | Query pattern |
|------|-----------|---------|---------------|
| **Summary** | `summaries/` | Quick summary of ONE ingested source, with read recommendation | "What did article X say?" |
| **Entity** | `entities/` | A specific thing: person, model, tool, company. Accumulates across ALL sources. | "What is X?" / "Who is X?" |
| **Topic** | `topics/` | Broad concept area synthesized across sources | "Explain X" / "How does X work?" |
| **Comparison** | `comparisons/` | X vs Y analysis | "How does X compare to Y?" |
| **Synthesis** | `synthesis/` | High-level cross-cutting analysis. Rare — emerges after many sources. | "What's the big picture on X?" |

### Query delegation flow

```
1. Read wiki/_wiki-index.md          → find which knowledge type(s) to check
2. Read <type>/_<type>-index.md      → find specific page(s) within that type
3. Read only the relevant pages      → answer the question
```

This keeps context usage minimal even with hundreds of articles.

## File Naming Convention

### Raw source files

Raw source files in `raw/` are named by the user manually. No enforced convention — the user drops files as-is.

### Wiki files

All wiki markdown files (summaries, entities, topics, comparisons, synthesis — but NOT index files, `log.md`, or `glossary.md`) follow:

```
<Field> - <Full Name>.md
```

- **Field** — the main topic/domain area (e.g., `LLM`, `AI`, `Deep Learning`, `Knowledge Management`)
- **Full Name** — the article's full name, natural spacing
- Separated by ` - ` (space-hyphen-space)

Examples:
- `LLM - Karpathy LLM Wiki.md`
- `AI - Andrej Karpathy.md`
- `Knowledge Management - Memex.md`
- `Deep Learning - Attention Is All You Need.md`

### Index files

Index files use: `_<type>-index.md` (underscore prefix, lowercase with hyphens). The underscore ensures they sort first in every directory.

Examples:
- `_wiki-index.md`
- `_summaries-index.md`
- `_entities-index.md`

## Frontmatter Specs

### Raw sources (`raw/**/*.md`)

```yaml
---
title: "Article or paper title"
source: "URL or citation"
author: "Author name(s)"
date: 2026-04-10            # Date of the original source
tags: [tag1, tag2]
status: pending | done      # pending = not yet processed, done = ingested
ingested: none | 2026-04-10 # none = not processed, date = when ingested
---
```

The user manages raw files and their frontmatter. The LLM updates `status`, `ingested`, and **refines `tags`** during ingest — never modifies body content. The user's initial tags may be approximate; the LLM should replace them with accurate, consistent tags after reading the source.

### Wiki summaries (`wiki/summaries/*.md`)

```yaml
---
title: "Article title — Summary"
sources:
  - "[[raw/articles/LLM-Source_File]]"  # Links back to raw sources
tags: [tag1, tag2]
created: 2026-04-10
updated: 2026-04-10
type: summary
recommend: yes | partial | no
distil_times: 0
---
```

- `recommend`: `yes` — high-value, worth deep reading | `partial` — some useful bits | `no` — summary is sufficient
- `distil_times`: starts at 0 on ingest, incremented each time `/distil` is run on this source. Re-distilling deepens understanding — each pass may surface new entities, connections, or nuances missed before.

### Wiki articles (`wiki/{entities,topics,comparisons,synthesis}/*.md`)

```yaml
---
title: "Article title"
sources:
  - "[[raw/articles/LLM-Source_File]]"  # Links back to raw sources
tags: [tag1, tag2]
created: 2026-04-10
updated: 2026-04-10
type: entity | topic | comparison | synthesis
---
```

### Glossary (`glossary.md`)

Glossary entries are inline (not separate files). Each term links back to the wiki article(s) that define it.

## Core Operations (Skills)

### 1. Ingest (`/ingest`)

Process a raw source file and produce a summary. The skill:

- Adds proper frontmatter to the raw file (body content stays untouched)
- Reads the source and creates a **summary** in `wiki/summaries/` with a `recommend` field
- Updates `wiki/summaries/_summaries-index.md` and `wiki/_wiki-index.md`
- Appends an entry to `wiki/log.md`
- Presents the summary and recommendation to the user

The user then decides whether to `/distil` based on the recommendation:
- `recommend: yes` → proceed to `/distil`
- `recommend: partial` → user's call
- `recommend: no` → summary is sufficient, no further processing needed

### 2. Distil (`/distil`)

Deep-process an ingested source into wiki knowledge. Run after `/ingest` when the user wants deeper processing. The skill:

- Creates or updates **entity pages** in `wiki/entities/`
- Creates or updates **topic pages** in `wiki/topics/`
- Optionally creates **comparison** or **synthesis** pages if warranted
- Updates all relevant index files (top-level + sub-indexes)
- Updates `glossary.md` with new terms
- Adds `[[cross-references]]` to related existing articles
- Sets raw source `status: pending` → `status: ingested`
- Appends an entry to `wiki/log.md`

A single source may create/update many wiki pages. Prefer one source at a time.

### 3. Query (`/query`)

Answer questions from the wiki using the delegation map. The skill:

- Reads `wiki/_wiki-index.md` to determine which knowledge type to check
- Reads the relevant sub-index
- Reads only the specific pages needed
- Synthesizes an answer with citations
- Optionally files good answers back as new wiki pages

### 4. Lint (`/lint`)

Health-check the wiki. The skill scans for:

- Contradictions between pages
- Stale claims superseded by newer sources
- Orphan pages with no inbound links
- Concepts mentioned but lacking their own page
- Missing cross-references
- Raw sources still in `status: pending`
- Broken `[[links]]`
- Sub-indexes out of sync with actual files

## Conventions

- All markdown files use YAML frontmatter
- Wiki articles use `[[internal links]]` for cross-references
- Use clear, concise language — accessibility without sacrificing accuracy
- Each article should be self-contained but cross-referenced
- Tag every article for discoverability
- Keep all `index.md` files up to date after every distil
- Keep `wiki/log.md` as an append-only chronological record
- Log entries use format: `## [YYYY-MM-DD] <operation> | <title>`
- Glossary terms link back to the articles that define them
- Entity pages should accumulate knowledge from ALL sources, not just the latest one

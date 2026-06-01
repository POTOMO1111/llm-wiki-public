# LLM Wiki — Schema

## Purpose

This is a personal research knowledge base maintained by an LLM. The focus is **academic research and paper reading** — papers, preprints, technical articles, and related commentary are ingested as raw sources, and their content is compiled into a persistent, interlinked wiki.

The human curates sources and asks questions. The LLM writes and maintains all wiki pages.

## Directory Structure

```
vault/
├── raw/                      # Immutable source documents — NEVER modified by LLM
│                             # Papers (.pdf), articles (.md), notes, etc. all placed here flat
└── wiki/                     # LLM-maintained wiki
    ├── index.md              # Content catalog — updated on every ingest
    ├── log.md                # Append-only operation log
    ├── overview.md           # High-level synthesis of the research area
    ├── sources/              # One summary page per source
    ├── concepts/             # Research concepts, methods, theories
    └── entities/             # Authors, institutions, datasets, models, benchmarks
```

## Page Formats

### Frontmatter (all wiki pages)
```yaml
---
title: Page Title
type: source | concept | entity | synthesis | overview
tags: []
sources: []        # slugs of source pages this page draws from
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

### Source summary page (`vault/wiki/sources/<slug>.md`)
```
# <Paper/Article Title>

| Field | Value |
|-------|-------|
| Authors | ... |
| Venue | ... (conference, journal, blog) |
| Year | ... |
| Raw file | vault/raw/filename |

## Key Contributions
- Bullet points: 3–7 most important claims, findings, or techniques

## Summary
Prose summary (2–5 paragraphs). What problem does it address, what approach is taken, what are the results, what are the limitations?

## Methodology
Key technical details relevant to understanding and citing the work.

## Connections
- Updates/extends: [[concept-or-source]]
- Contradicts: [[concept-or-source]] — brief note on the contradiction
- See also: [[related-page]]

## Open Questions
Questions this paper raises that the wiki hasn't addressed yet.
```

### Concept page (`vault/wiki/concepts/<slug>.md`)
```
# <Concept Name>

Brief definition (1–2 sentences).

## Current Understanding
The best synthesis of what is known, with inline citations [[source-slug]].

## Competing Views
Where papers/authors disagree. Don't silently resolve contradictions — name them.

## Key Papers
- [[source-slug]] — one-line summary of its contribution to this concept

## Related
- [[concept-or-entity]]
```

### Entity page (`vault/wiki/entities/<slug>.md`)
Used for: authors, research groups, institutions, datasets, models, benchmarks, software tools.
```
# <Entity Name>

Type: Author | Institution | Dataset | Model | Benchmark | Tool

Brief description.

## Key Facts
Cited facts about this entity.

## Appearances
Papers/articles where this entity plays a significant role, with brief context.

## Related
- [[related-entity-or-concept]]
```

## Cross-referencing

- Use wikilinks: `[[page-title]]` (Obsidian-compatible)
- When linking, use display text when the slug alone is ambiguous: `[[slug|Display Name]]`
- Link a term the first time it appears in a section — don't repeat links in the same section
- Prefer linking to the most specific relevant page (entity > concept > source)

## index.md Format

Organized by category. Each entry is one line:
```
- [[slug|Page Title]] — one-line description `[N sources]`
```

Sections: **Sources**, **Concepts**, **Entities**, **Synthesis**

## log.md Format

Append-only. Each entry:
```markdown
## [YYYY-MM-DD] <operation> | <title>
Brief summary of what was done.
```
Operations: `ingest`, `query`, `lint`

## Naming Conventions

- Filenames: lowercase, hyphen-separated (`attention-mechanism.md`, `vaswani-2017.md`)
- No spaces or special characters in filenames
- Source pages named as `<first-author>-<year>[-<keyword>].md` (e.g., `vaswani-2017-attention.md`)
- Concept pages named by the concept (`chain-of-thought.md`)
- Entity pages named by the entity (`geoffrey-hinton.md`, `imagenet.md`)

## LLM Behavioral Rules

1. **Never modify vault/raw/** — treat everything there as immutable source of truth
2. **Always update index.md** after creating or significantly changing a wiki page
3. **Always append to log.md** after each ingest, query, or lint pass
4. **Prefer updating existing pages** over creating new ones for minor additions
5. **Cite sources** — every factual claim in concept/entity pages must link to a source page
6. **Flag contradictions explicitly** — use a "Competing Views" or "Contradicts" section; don't silently pick a side
7. **One concept per page** — don't merge distinct concepts to save space
8. **Update overview.md** when a major new theme emerges from an ingest
9. **Language** — write wiki pages in the same language as the primary source; use English for page slugs and frontmatter

## Operations

### Ingest (see `/ingest-paper`)
1. Read the source from vault/raw/
2. Discuss key takeaways with the user if helpful
3. Write a source summary page in vault/wiki/sources/
4. Create or update concept pages touched by the source
5. Create or update entity pages (authors, datasets, models) introduced by the source
6. Update vault/wiki/index.md
7. Optionally update vault/wiki/overview.md if the source shifts the big picture
8. Append an entry to vault/wiki/log.md

### Query (see `/query`)
1. Read vault/wiki/index.md to find relevant pages
2. Read the relevant pages
3. Synthesize an answer with citations ([[source-page]])
4. If the answer is worth keeping, offer to save it as a synthesis page in vault/wiki/

### Lint (see `/lint`)
1. Read all pages via index.md
2. Report: contradictions, orphan pages, missing cross-references, stale claims, gaps
3. Suggest new sources to look for
4. Append a lint entry to log.md

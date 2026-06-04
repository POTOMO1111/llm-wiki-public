# Skill: ingest-paper

Ingest a source document from `vault/raw/` into the wiki.

## Trigger

`/ingest-paper [filename]`

If `filename` is provided, process that file. If omitted, list available files in `vault/raw/` and ask the user which to ingest.

## Steps

### 1. Read the source
- Read the file from `vault/raw/<filename>`
- For PDFs, extract text with `pdftotext "path" - 2>&1`; if output exceeds 30KB it may be saved to a temp file — read that with the Read tool
- Read title/abstract/conclusion first, then methodology and results
- Note any figures or tables referenced — offer to view key images if relevant

### 2. Brief discussion (optional)
- Summarize the source: problem, approach, key results, limitations (4–5 sentences)
- Ask the user: anything specific to emphasize? Related sources already in the wiki to connect to?

### 3. Write a source summary page
Create `vault/wiki/sources/<slug>.md` using the source page format from CLAUDE.md.
- Slug: `<first-author-lastname>-<year>[-<short-keyword>].md` (e.g., `vaswani-2017-attention.md`)
- Fill in all fields: Authors, Venue, Year, Key Contributions, Summary, Methodology, Connections, Open Questions
- After the metadata table and before `## Key Contributions`, insert a **6-question quick summary block**:

```markdown
## ひとめでわかる要点

1. **どんなもの？** （1〜2文）
2. **先行研究と比べてどこがすごいの？** （1〜2文）
3. **技術や手法の"キモ"はどこにある？** （1〜2文）
4. **どうやって有効だと検証した？** （1〜2文）
5. **議論はあるか？** （1〜2文）
6. **次に読むべき論文はあるか？** （1〜2文）
```

### 4. Update concept pages
For each major concept, method, dataset, or benchmark the source uses or introduces:
- If a concept page exists: update Current Understanding with new evidence, add to Key Papers, flag contradictions
- If no concept page exists and the concept is substantial (will be referenced by future sources): create one

### 5. Update entity pages
For key entities:
- **Authors**: create or update author entity pages for primary contributors
- **Datasets**: create or update dataset pages; note how this source uses them
- **Models/architectures**: create or update model pages
- **Benchmarks**: create or update benchmark pages; record reported scores

### 6. Update index.md
Add all new/updated pages under the appropriate section.

### 7. Update overview.md (conditional)
If the source is a significant contribution that shifts the field understanding, update `vault/wiki/overview.md` — add it to Emerging Themes or update the synthesis.

### 8. Rename and archive the source file to `vault/ingested/`
After the source page is created with slug `<slug>.md`, **rename the source file to match the slug, then move it from `vault/raw/` to `vault/ingested/`**:

1. **Rename**: `vault/raw/<original-filename>.<ext>` → `vault/raw/<slug>.<ext>` (preserve the original extension: `.pdf`, `.md`, etc.)
2. **Move**: `vault/raw/<slug>.<ext>` → `vault/ingested/<slug>.<ext>`
3. **Update the source page's "Raw file" field** to reflect the new path: `vault/ingested/<slug>.<ext>`

These two operations can be combined into a single move-with-rename (e.g., PowerShell `Move-Item -Path <src> -Destination vault/ingested/<slug>.<ext>`, or `git mv` if the repo is git-tracked).

**Purpose**:
- `vault/raw/` now contains **only files pending ingest** — you can tell at a glance (no LLM tokens needed) which papers are still TODO.
- `vault/ingested/` is **append-only archive**: files are never modified or removed after entering. This protects the immutable source of truth even if `vault/raw/` is later edited by hand.
- The move is the **only allowed write** that touches a successfully-ingested source file. File contents themselves are never altered.

Example: ingesting `vault/raw/DDPM.pdf` and creating `vault/wiki/sources/ho-2020-ddpm.md`
→ rename + move to `vault/ingested/ho-2020-ddpm.pdf`
→ source page table contains: `| Raw file | vault/ingested/ho-2020-ddpm.pdf |`

### 9. Append to log.md
```
## [YYYY-MM-DD] ingest | <Title>
Source: vault/ingested/<slug>.<ext>   (after rename + move)
Venue: <conference/journal/blog>, <year>
Pages touched: <comma-separated list of wiki pages created or updated>
```

## Notes
- **Source file content is immutable**. The only allowed write operations are (a) the rename + move from `vault/raw/` to `vault/ingested/` performed in Step 8, and (b) reads. Never edit file contents in either directory.
- **`vault/raw/` = pending ingest queue**. **`vault/ingested/` = archive of completed ingests**. If a paper is already in `vault/ingested/<slug>.<ext>`, it is already ingested — no need to re-process.
- For sources with many authors, focus entity pages on the primary/corresponding authors
- Always record the venue and year — these matter for situating claims in time
- If a source replicates, extends, or contradicts prior work already in the wiki, make that connection explicit in both source pages
- Write wiki pages in the same language as the primary source; use English for page slugs and frontmatter

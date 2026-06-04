---
name: lint
description: Health-check the wiki to find contradictions, gaps, orphans, and stale content.
---

# Skill: lint

Health-check the wiki: find contradictions, gaps, orphans, and stale content.

## Trigger

`/lint`

## Steps

### 1. Read the index
- Read `vault/wiki/index.md` to enumerate all wiki pages

### 2. Read all wiki pages
- Read every page listed in the index
- Focus on concept pages (most likely to accumulate issues) and source pages

### 3. Check for issues — report each category

#### Contradictions
- Claims in one concept page that conflict with claims in another, or with a source page
- Different papers reporting conflicting results on the same question
- Mark severity: **critical** (direct factual conflict), **moderate** (methodological disagreement), **minor** (emphasis difference)

#### Orphan pages
- Pages with no inbound wikilinks from other wiki pages
- These are isolated and hard to discover — suggest which existing pages should link to them

#### Missing concept pages
- Concepts mentioned (linked or unlinked) across multiple pages but without their own page
- Prioritize concepts referenced in 3+ pages

#### Missing entity pages
- Frequently mentioned authors, datasets, or models without a dedicated page

#### Stale claims
- Claims in concept pages that may have been superseded by newer sources in the wiki
- Flag where a newer source contradicts or qualifies an older claim that hasn't been updated

#### Broken or missing cross-references
- Source pages not linked from any concept/entity page
- Concept pages with "Key Papers" entries that don't exist in the wiki yet

#### Data gaps
- Important sub-topics in the research area that have zero coverage in the wiki
- Suggest what kind of source would fill the gap

### 4. Suggest new sources
Based on what's missing, suggest:
- Foundational papers that should be ingested
- Survey/review papers that would strengthen synthesis pages
- Blog posts or technical articles that explain gaps accessibly

### 5. Prioritized action list
End with a short prioritized list:
1. Issues to fix immediately (contradictions, broken links)
2. Pages to create next (high-value missing concepts/entities)
3. Sources to seek out (gaps in coverage)

### 6. Append to log.md
```
## [YYYY-MM-DD] lint | Wiki health check
Issues found: <N contradictions, N orphans, N gaps>
Key actions: <brief summary>
```

## Notes
- The goal is not to achieve a perfect wiki — it's to know where the weak spots are
- Flag but don't automatically resolve contradictions; let the user decide
- A lint pass is most useful after ingesting 5+ new sources

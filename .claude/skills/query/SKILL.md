# Skill: query

Answer a question against the wiki and optionally file the answer as a synthesis page.

## Trigger

`/query <question>`

## Steps

### 1. Read the index
- Read `vault/wiki/index.md` to get the full catalog of available pages

### 2. Identify relevant pages
- From the index, identify which pages are relevant to the question
- Prioritize: concept pages > source pages > entity pages

### 3. Read relevant pages
- Read the identified pages from `vault/wiki/`
- If a concept page references source pages you haven't read, read those too

### 4. Synthesize the answer
Structure the answer based on the question type:

| Question type | Format |
|---|---|
| Factual / definition | Short prose with citations |
| Comparison | Markdown table + commentary |
| Literature survey | Structured prose by theme, with citation list |
| Contradiction / debate | "View A (papers) vs View B (papers)" structure |
| Hypothesis / open question | Summary of evidence + explicit gaps |

Always include inline citations using wikilinks: `[[source-slug]]`

### 5. Offer to file the answer
If the answer is substantial (a comparison, a survey, a useful synthesis):
- Ask the user: "Should I save this as a synthesis page?"
- If yes: create `vault/wiki/<descriptive-slug>.md` with `type: synthesis` in frontmatter
- Update `index.md` under the **Synthesis** section
- Append to `log.md`:
  ```
  ## [YYYY-MM-DD] query | <Question summary>
  Saved as: vault/wiki/<slug>.md
  ```
- If not saved, still append a brief log entry

### 6. Suggest follow-up
After answering, suggest:
- Related pages worth reading
- Gaps in the wiki that a new source could fill
- Follow-up questions worth asking

## Notes
- If the wiki doesn't have enough information to answer well, say so explicitly — don't hallucinate
- Note when an answer relies on a single source vs. convergent evidence from multiple sources
- For questions about recent developments, flag if the wiki's sources may be outdated

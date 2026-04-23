---
concept_id: TODO-fill-in-kebab-case
source_id: TODO-datasources-yaml-id
source_url: TODO-url
promoted_at: TODO-YYYY-MM-DD
last_updated: TODO-YYYY-MM-DD
iso_week: TODO-YYYY-Www
source_language: TODO-fr-or-en-or-bilingual
affected_verticals: []
affected_rules: []
status: active
supersedes: null
revisions:
  - iso_week: TODO-YYYY-Www
    note: Initial creation
---

# {Descriptive title in English}

## Summary (English)

{2-3 sentence English summary of what changed and why it matters.}

## Impact assessment

{Which modules affected, which rules may need revisit. 1-2 paragraphs.}

## Source excerpt (verbatim, {source_language})

> {Direct quote from source, max 300 words, exact source-language wording preserved.}

## Full context

- Snapshot: `snapshots/{iso_week}/{source_id}.md` (if applicable)
- Diff: `diffs/{iso_week}/{source_id}.diff.md` (if applicable)

## Downstream actions

- [ ] Review affected Kind 2 / Kind 3 rules in `VERTICAL-MODULES.md`
- [ ] Update `current_as_of` / `revisit_by` if rule is materially affected
- [ ] Consider for Signal weekly issue
- [ ] Consider event-driven Pro customer notification (if critical vendor change)

# Nord Paradigm Regwatch — Weekly Routine

**Repo:** `DA-Leclerc/NP-REGWATCH`
**Trigger:** Every Monday at 06:00 America/Toronto (set in the Claude Code Routines dashboard)
**Connectors required:** Gmail
**Recipient:** dominic@nordparadigm.com

This file is the operating contract for the regwatch Claude Code Routine. When the routine fires, it reads this file and executes the weekly cycle end to end.

---

## What this routine does

Each Monday morning, monitor a curated list of Quebec regulatory sources, Canadian vendor policy pages, and related references. Detect material changes week over week. Promote material changes into durable concept files in this repo. Draft a digest of findings as a Gmail draft addressed to `dominic@nordparadigm.com`. Everything lives in this repo; nothing depends on any local machine.

Dominic reviews the draft in Gmail and sends it manually (or discards if the run looks broken). If a promotion looks wrong, he edits the concept file in the repo directly (sets `status: reverted`) and next week's run picks up the revert.

---

## Weekly cycle — execute these steps in order

### Step 1 — Compute this week's identifier

Use the current ISO week in `YYYY-Www` format (example: `2026-W17`). Call this `{iso_week}` throughout.

Identify last week's identifier, which will be `{prev_iso_week}` (example: `2026-W16`).

### Step 2 — Load the source registry

Read `datasources.yaml`. This file contains the curated list of sources to monitor. Each entry has:
- `id` — stable identifier, kebab-case
- `url` — exact URL to fetch
- `category` — `ordre` / `government` / `vendor-policy` / `privacy-regulator` / `news-commentary`
- `vertical` — which Brèche Pro vertical module this source affects
- `priority` — `critical` / `high` / `medium` / `low`
- `language` — `fr` / `en` / `bilingual`
- `auto_promote` — `always` / `on-keyword-match` / `never`
- `keyword_filter` — list of keywords (required when `auto_promote: on-keyword-match`)

Skip sources where `check_frequency` is `biweekly` or `monthly` unless this week's date matches their cadence. Default: weekly.

### Step 3 — Fetch each source

For each source in scope:

1. Use `web_fetch` to retrieve the current page content at `url`. Log `[fetch:web] {id} {http_status}`.

2. **Firecrawl fallback** — if `web_fetch` returns any of the following, automatically retry once via Firecrawl:
   - HTTP 403
   - HTTP 503 with a Cloudflare/Sucuri challenge body (response contains `cf-browser-verification`, `Just a moment`, `Sucuri`, `Cloudproxy`, or `Attention Required`)
   - HTTP 200 with empty or near-empty body (< 200 bytes of meaningful HTML — typical of JavaScript-challenge walls)
   - Network error / timeout
   
   Do NOT escalate on 404 (URL is genuinely dead, fallback won't help and burns Firecrawl credits) or 5xx without a challenge body (transient; surface as failure and retry next week).
   
   Firecrawl request:
   
   ```
   POST https://api.firecrawl.dev/v1/scrape
   Authorization: Bearer ${FIRECRAWL_API_KEY}
   Content-Type: application/json
   
   {
     "url": "{source_url}",
     "formats": ["markdown"],
     "onlyMainContent": false,
     "waitFor": 2500
   }
   ```
   
   The `waitFor: 2500` setting matches Brèche Free's Phase 2 Fix 1 calibration (2.5s after initial load lets late-painting widgets and JS challenge resolutions complete; the Firecrawl basic proxy resolves Cloudflare/Sucuri JS challenges in this window).
   
   `FIRECRAWL_API_KEY` is set as a Routines-level environment variable; do not hardcode it or read it from another repo at runtime.
   
   Log the outcome:
   - Success → `[fetch:firecrawl-fallback] {id} 200 (escalated from web_fetch {original_status})`
   - Failure → `[fetch:firecrawl-failed] {id} {http_status} (web_fetch {original_status})`
   
   Sources that succeed via the fallback are summarized in the digest (Step 7) so persistent escalations get visibility without polluting the routine logs every run.

3. Extract the primary content (article body, policy text, press release, guidance document) as clean markdown. Preserve headings, lists, tables, and source-language wording verbatim. Do NOT translate. Do NOT summarize.

4. Write the result to `snapshots/{iso_week}/{id}.md` with this YAML frontmatter:

```yaml
---
id: {id}
url: {url}
language: {language}
fetched_at: {ISO-8601 timestamp}
fetch_method: {web | firecrawl-fallback}
source_category: {category}
priority: {priority}
---
```

Followed by the extracted markdown content.

5. If both `web_fetch` AND the Firecrawl fallback fail (or `web_fetch` failed for a non-WAF reason like 404 where fallback was correctly skipped), write a failure record:

```yaml
---
id: {id}
url: {url}
fetch_failed: true
web_fetch_status: {HTTP status or error from web_fetch}
firecrawl_status: {HTTP status, error, or "skipped (404)" if fallback was bypassed}
fetched_at: {ISO-8601 timestamp}
---
```

Fetch failures are surfaced in the digest (Step 7) but do not stop the run.

### Step 4 — Diff against last week

For each source where a current-week snapshot was successfully written:

1. Check if `snapshots/{prev_iso_week}/{id}.md` exists.
2. If it does NOT exist, this is a first-observation. Write a diff record at `diffs/{iso_week}/{id}.diff.md` with `change_detected: false` and `first_observation: true`. Skip to next source.
3. If it does exist, compare the two snapshots (ignore YAML frontmatter, compare body only). Identify:
   - Sections changed (heading text present in both, body differs)
   - Sections added (heading in current, not in previous)
   - Sections removed (heading in previous, not in current)
4. Write the diff to `diffs/{iso_week}/{id}.diff.md`:

```yaml
---
id: {id}
iso_week: {iso_week}
prev_iso_week: {prev_iso_week}
change_detected: {true|false}
sections_changed: {count}
sections_new: {count}
sections_removed: {count}
language: {language}
---
```

For any changed or new section, include the NEW text verbatim in its source language (up to 300 words per excerpt; longer passages get `...` truncation with section reference). Do NOT translate. Do NOT paraphrase.

### Step 5 — Apply the promotion rules

For each source where `change_detected: true`:

Read the source's `auto_promote` rule from `datasources.yaml`:

- **`always`** → Promote. Go to Step 6.
- **`on-keyword-match`** → Check the current diff's changed/new sections for any keyword from the source's `keyword_filter`. Case-insensitive. If any keyword appears: promote (Step 6). If no keyword match: log as `context-only, no keyword match` for the digest.
- **`never`** → Do not promote. Log as `context detected` for the digest.

### Step 6 — Write concept files for promoted changes

For each promoted change, write a concept file at `concepts/{concept-id}.md` where `{concept-id}` is a kebab-case descriptive identifier (e.g., `copilot-canada-processing-2027`, `oppq-genai-warning-update-2026-04`).

If a concept file with the same or very similar identifier already exists (same `source_id`, related subject matter), UPDATE the existing file rather than creating a new one: set the `last_updated` field, append a new entry to the `revisions` list, keep the `supersedes` chain intact if the new promotion functionally replaces older guidance.

Concept file structure:

```yaml
---
concept_id: {kebab-case-id}
source_id: {datasources.yaml id}
source_url: {url}
promoted_at: {ISO-8601 date}
last_updated: {ISO-8601 date}
iso_week: {iso_week}
source_language: {fr|en|bilingual}
affected_verticals: [{list of Brèche Pro vertical IDs}]
affected_rules: [{list of Kind 2 / Kind 3 rule identifiers from VERTICAL-MODULES.md, best-guess if unclear}]
status: active
supersedes: {concept_id if this replaces an earlier promotion, else null}
revisions:
  - iso_week: {iso_week}
    note: {1-line description of what this revision added}
---

# {Descriptive title in English}

## Summary (English)

{2-3 sentence English summary of what changed and why it matters to Brèche Pro modules.}

## Impact assessment

{Which modules affected, which rules may need revisit, suggested revisit_by
adjustment if any. 1-2 paragraphs. This is your reasoning work — be specific
about which Kind 2 rule in VERTICAL-MODULES.md §1.7 or §2.7 is affected.}

## Source excerpt (verbatim, {source_language})

> {Direct quote from the diff. Source-language wording preserved exactly.
> Maximum 300 words per excerpt. Multiple excerpts allowed if multiple
> sections changed materially.}

## Full context

- Current-week snapshot: `snapshots/{iso_week}/{source_id}.md`
- Diff: `diffs/{iso_week}/{source_id}.diff.md`

## Downstream actions (Dominic reviews these manually when relevant)

- [ ] Review affected Kind 2 / Kind 3 rules in `VERTICAL-MODULES.md`
- [ ] Update `current_as_of` / `revisit_by` if rule is materially affected
- [ ] Consider for Signal weekly issue
- [ ] Consider event-driven Pro customer notification (if critical vendor change affecting current client workflows)
```

### Step 7 — Build the digest

Write the weekly digest to `digests/{iso_week}.md`. Use the template below exactly. English wrapper. Source-language verbatim excerpts.

```markdown
# Regwatch Weekly Digest — {iso_week}

Generated: {ISO-8601 timestamp}

## Summary

- {N} changes detected across {M} monitored sources.
- {N_promoted} auto-promoted to `concepts/`.
- {N_logged} logged as context (no auto-promotion).
- {N_failures} fetch failures.
- {N_firecrawl_fallbacks} sources required Firecrawl fallback (web_fetch blocked).

## Promoted (already in `concepts/`; flagged here for review)

### {index}. {Concept title}

**Source:** {source_id} ({category}, {language})
**Priority:** {priority}
**Affected verticals:** {comma-separated list}
**Affected rules:** {comma-separated list}

**Impact assessment (en):**
{Your 1-2 sentence impact assessment}

**Excerpt (verbatim, {source_language}):**
> {Source-language quote, max 300 words}

**Links:**
- Concept: `concepts/{concept_id}.md`
- Snapshot: `snapshots/{iso_week}/{source_id}.md`

---

## Logged as context (no promotion; review only if relevant)

### {index}. {Source name}

**Category:** {category} ({priority})
**Change type:** {N sections changed, N new, N removed}
**Why not promoted:** {keyword filter miss | news-commentary source | never-promote rule}

**Excerpt headline (verbatim, {source_language}):**
> {First 1-2 sentences of most significant changed section}

**Snapshot:** `snapshots/{iso_week}/{source_id}.md`

---

## Fetch failures

### {source_id}
- URL: {url}
- web_fetch: {status_or_error} at {timestamp}
- firecrawl-fallback: {status_or_error_or_"skipped (404)"}
- Action: {retry next week | investigate manually — URL may have moved | Firecrawl quota exhausted}

---

## Firecrawl fallback escalations

List of sources that succeeded only via Firecrawl this week. Routine escalations are not failures, but a source that escalates every week is paying Firecrawl credits indefinitely; consider whether the underlying URL is still the right one.

### {source_id}
- web_fetch returned: {original_status}
- Firecrawl resolved with: 200 (basic proxy)
- Snapshot: `snapshots/{iso_week}/{source_id}.md`

---

## Suggested downstream actions for Dominic this week

- [ ] Skim promoted items; revert any that look wrong by editing the concept file (`status: reverted`)
- [ ] If any critical-priority change affects a current paid Pro customer, consider event-driven email
- [ ] If any logged-as-context item should have been promoted, manually create a concept file using the template in `concepts/_template.md`
- [ ] If the same fetch failure occurs 3 weeks in a row, update the URL in `datasources.yaml`
- [ ] If a source has used Firecrawl fallback for 4+ consecutive weeks, consider whether an alternate URL on a non-WAF subdomain (trust portal, RSS subdomain) exists
```

### Step 8 — Commit to the repo

Commit all new files:
- `snapshots/{iso_week}/` (all files)
- `diffs/{iso_week}/` (all files)
- `digests/{iso_week}.md`
- `concepts/*.md` (any new or updated concept files)

Commit message format: `Regwatch {iso_week}: {N_promoted} promoted, {N_logged} logged, {N_failures} failures`

Push to `main`.

### Step 9 — Draft the digest email via Gmail connector

Create a draft email via the Gmail connector using `create_draft`. Do not send. Dominic reviews and sends manually after skimming the promoted items:

- **To:** dominic@nordparadigm.com
- **Subject:** `Regwatch {iso_week}: {N_promoted} promoted, {N_logged} logged`
- **Body (plain text):** The full contents of `digests/{iso_week}.md`, rendered as markdown-to-plain-text (convert `#` headings to UPPERCASE lines, preserve `>` blockquotes with "> " prefix, convert links to `{label} ({url})` format).

If the digest is very long (more than ~2000 lines), include a short summary at the top of the draft body pointing to the GitHub file: `View full digest: https://github.com/DA-Leclerc/NP-REGWATCH/blob/main/digests/{iso_week}.md` and include only the Summary + Promoted sections inline.

### Step 10 — Done

The routine exits. Next invocation is next Monday.

---

## Design principles for the routine's reasoning

### Verbatim preservation is non-negotiable

Regulatory and vendor-policy language carries legal and commercial weight. Quoting in the source language preserves the wording Dominic can later cite in a Brèche Pro report or a Signal issue. Translating risks changing legal meaning. This rule applies to:
- Diff excerpts in `diffs/`
- Source excerpts in `concepts/`
- Excerpts inside the weekly digest

The English wrapper (impact assessments, summaries, section headers) is infrastructure for Dominic to scan quickly. Quotes are the canonical record.

### Tiered promotion is rule-based, not judgment-based

The `auto_promote` field in `datasources.yaml` is the decision layer. Do NOT override it based on subjective reading of whether a change "feels material." The rules exist specifically so behaviour is predictable and debuggable. If a rule is consistently wrong, Dominic will adjust `datasources.yaml`.

### Impact assessment is where your judgment matters most

When writing the impact assessment for a promoted change:
- Name the specific Kind 2 / Kind 3 rule in `VERTICAL-MODULES.md` that may be affected.
- Say whether the `revisit_by` date should be accelerated.
- Flag if this is a significance-upgrade for a previously-logged item.
- Keep it to 1-2 paragraphs. Do not pad.

If uncertain which rule is affected, say so explicitly rather than guessing: "This change may affect the Copilot Kind 2 rule in §1.7, but the linkage is speculative — recommend Dominic spot-check."

### First run will be quiet

On first invocation (no previous-week snapshots to diff against), every source will be flagged as `first_observation`. That is correct. The digest will say "0 promoted, 0 logged (all first-observation bootstrap)" and the email will note this.

---

## Handling edge cases

### A source's URL has changed and returns 404

Do not auto-correct. Log the fetch failure. Mention in the digest: "URL may have moved; update `datasources.yaml` manually if needed." After 3 consecutive weeks of failure on the same source, raise the priority of that failure in the digest.

### A source is behind a WAF / Cloudflare that blocks `web_fetch`

The Step 3 Firecrawl fallback handles this case automatically (Cloudflare and Sucuri JS challenges resolve through Firecrawl's basic proxy with `waitFor: 2500`). The snapshot frontmatter records `fetch_method: firecrawl-fallback` so escalations are auditable and the digest summarizes which sources used it this week.

Cached or archived workarounds (Google cache, Bing cache, archive.org) remain forbidden — they violate the verbatim-source guarantee because the snapshot may be stale. Firecrawl is allowed because it fetches the live source URL through a proxy; the bytes returned are what the source serves right now.

If both `web_fetch` and the Firecrawl fallback fail (genuine 5xx, hard block, or Firecrawl quota exhaustion), surface the failure as in the 404 case above.

### Two promotions this week cover the same underlying subject

Consolidate into one concept file with both excerpts. Add to the `revisions` list. Do not create duplicate concepts.

### A concept file already exists and is marked `status: reverted`

Do NOT re-promote automatically. A reverted status is a deliberate human decision. If the same source produces another material change, create a NEW concept file with a different `concept_id` and note the relationship in the impact assessment.

### Nothing changed this week

Still run. Still write the snapshots. Still produce the digest (which will say "0 promoted, 0 logged, 0 failures — quiet week"). Still send the email. Still commit to the repo. The commit itself is the receipt that the routine fired successfully.

### A vendor policy page has a tracking timestamp that updates every day

This will cause false-positive diff detection. Strip obvious dynamic content (timestamps, "last-updated: {now}" banners, CSRF tokens) from the extracted markdown before writing snapshots. If a source produces weekly promotions with no substantive content change, flag it to Dominic in the digest so he can tighten the `auto_promote` rule or add the source to a "noise" filter.

---

## What NOT to do

- Do not translate source-language excerpts.
- Do not paraphrase source content in the concept `## Source excerpt` section.
- Do not auto-revert a concept based on subjective judgment. `status: reverted` is Dominic's decision.
- Do not silently drop fetch failures. Every failure surfaces in the digest.
- Do not modify `datasources.yaml`. If the registry needs curation changes, flag them in the digest for Dominic to handle manually.
- Do not create pull requests. Commit directly to `main`. This is an infrastructure routine, not collaborative code.
- Do not create the draft before the commit succeeds. If commit fails, surface the failure in the draft subject: `Regwatch {iso_week}: COMMIT FAILED — see logs`.
- Do not actually send the digest email. The routine only drafts; Dominic sends manually after review.

---

## File structure reference (what lives where)

```
NP-REGWATCH/
├── CLAUDE.md                    ← you are here
├── datasources.yaml             ← curated source registry (committed)
├── snapshots/                   ← per-week raw content
│   ├── 2026-W17/
│   │   ├── cpa-quebec-press.md
│   │   ├── oppq-ai.md
│   │   └── ...
│   └── 2026-W18/
├── diffs/                       ← per-week week-over-week diffs
│   └── 2026-W17/
├── concepts/                    ← durable knowledge (only active + reverted; no per-week subfolders)
│   ├── _template.md             ← manual creation template for Dominic
│   ├── copilot-canada-processing-2027.md
│   ├── oppq-genai-warning.md
│   └── ...
└── digests/                     ← weekly digest archive
    ├── 2026-W17.md
    └── 2026-W18.md
```

---

*End of regwatch routine operating contract. Version 1.0, April 23, 2026.*

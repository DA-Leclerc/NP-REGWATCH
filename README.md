# NP-REGWATCH

Nord Paradigm's regulatory-watch system. Monitors 40 curated sources across Quebec ordres, Canadian regulators, and AI vendor policy pages. Detects material changes week over week. Promotes material findings into durable concept files. Delivers a weekly digest email.

Runs as a Claude Code Routine on Anthropic's infrastructure. No local machine required.

## Repository structure

```
NP-REGWATCH/
├── CLAUDE.md                    ← operating contract for the routine
├── README.md                    ← you are here
├── datasources.yaml             ← curated source registry
├── snapshots/                   ← weekly raw content (routine writes)
├── diffs/                       ← week-over-week diffs (routine writes)
├── concepts/                    ← durable promoted findings
│   └── _template.md             ← manual creation template
└── digests/                     ← weekly digest archive
```

## How it works

**Every Monday at 06:00 America/Toronto,** a Claude Code Routine fires. The routine:

1. Reads `datasources.yaml` for the curated source list.
2. Fetches current content from each source.
3. Diffs each source against the previous week's snapshot.
4. Applies tiered auto-promotion rules (rule-based, not judgment-based).
5. For promoted changes, writes a concept file to `concepts/`.
6. Builds a weekly digest and commits everything to the repo.
7. Emails the digest to `dominic@nordparadigm.com`.

Full operating contract: see `CLAUDE.md`.

## Review workflow

**Monday morning:** skim the digest email in under 5 minutes. If every promoted concept looks reasonable, do nothing — the repo is already updated.

**If a promotion looks wrong:**
1. Open the concept file in `concepts/`.
2. Change `status: active` to `status: reverted`.
3. Add a note in the `revisions:` list explaining why.
4. Commit.
5. Next week's routine picks up the reverted state automatically.

**If something should have been promoted but wasn't:**
1. Copy `concepts/_template.md` to `concepts/{kebab-case-descriptive-id}.md`.
2. Fill in the template fields.
3. Commit.
4. The concept is now part of the durable knowledge base.

## Integration with Brèche Pro

Concepts in this repo feed Brèche Pro's vertical modules. When Pro generates a report, it reads the current state of `concepts/` to ground its Kind 2 / Kind 3 regulatory claims. Concept files with `status: active` are live; `status: reverted` are ignored.

The integration is pull-based: Pro's build step reads this repo. This repo never pushes to Pro.

## Curating the source list

`datasources.yaml` is version-controlled. Edit it directly to:
- Add a new source (new ordre, new vendor page, new regulator)
- Drop a dead source (URL moved, site shut down, no longer relevant)
- Change auto-promotion behaviour (tighten or loosen rules based on noise level)
- Adjust priority or check frequency

Quarterly review is recommended. First Monday of January, April, July, October. 15 minutes.

## Setup

See `CLAUDE.md` for routine configuration instructions. The routine needs:
- Read/write access to this repo
- Gmail connector (for digest email)
- `web_fetch` tool access (for source scraping)

One-time setup at `claude.ai/code/routines`. After that, fully unattended.

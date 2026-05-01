# 40-projects/

**Multi-thread projects** — anything with sustained context, multiple decisions, and an evolving timeline.

A project page is the running ledger for one project: status, decisions made, open questions, key artifacts, related notes. Dumps that mention a project add `## Notes — YYYY-MM-DD` sections to its page over time.

## Frontmatter

```markdown
---
type: project
status: active | paused | done | blocked
started: YYYY-MM-DD
---
```

## Suggested structure

```markdown
---
type: project
status: active
started: 2026-04-15
---

# Project name

## Goal

One paragraph: what success looks like.

## Status

Current state, last updated date.

## Key decisions

- 2026-04-20: chose X over Y because [...]
- 2026-04-22: deferred Z until [...]

## Open questions

- [ ] How to handle [...]?

## Artifacts
- [[related concept]]
- `code/<project>/` if applicable
- External links

## Notes — YYYY-MM-DD

(Verbatim content from `raw/dumps/` lands here when relevant.)
```

## Project pages this vault is likely to have

Based on your current life:
- `phd-applications.md` — target schools, deadlines, lab research, SOP drafts.
- `the-seventh-bar.md` — music project: practice log, set ideas, live performance plans.
- `apartment-beaumont.md` — lease details, move logistics, address changes needed.
- `h1b-sponsorship.md` — petition status, AC21 timing, prevailing wage notes.
- `contract-extension-meta.md` — manager comms, immigration-separated thread.
- `resume-rebuild.md` — phased rebuild, framing decisions, draft links.
- `slip-detection-pipeline.md` — research pipeline, code in `code/slip/`, related papers.
- `cr-v-collision.md` — repair decisions, costs, status.

(Pages don't need to exist until there's something to put in them.)

## When to spin up a project page

- The thing has more than one decision attached to it, OR
- It has a timeline beyond a single day, OR
- You'll come back to it more than twice.

If it's "send one email and done" → it doesn't need a project page. Note it in `10-daily/`.

## Filename convention

`<lowercase-hyphenated-slug>.md`. E.g., `phd-applications.md`, not `PhD Applications.md`. Easier to type and link.

## Distinct from

- `20-concepts/` — concepts are timeless ideas; projects have status and a timeline.
- `50-courses/` — courses are a special kind of project with a fixed structure (modules, assignments, exam date). Use `50-courses/` for those.

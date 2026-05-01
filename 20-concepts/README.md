# 20-concepts/

**The wiki proper for technical and research concepts.** Each file is a long-lived, maintained page about one idea. Linked aggressively via `[[wikilinks]]`.

This is where understanding accumulates — papers are integrated *into* concept pages, dumps add `## Notes` sections, contradictions are tracked, examples build up over time.

## Template

`templates/concept.md`:

```markdown
---
type: concept
tags:
---

# Concept name

## Definition

## Intuition

## Mathematical statement

## Why it matters

## Connections
-

## References
-
```

## Frontmatter conventions

- `type: concept` — required.
- `status: stub` — optional; signals the page exists but the canonical sections aren't filled in yet.
- `last-updated: YYYY-MM-DD` — added on every edit per `wiki-playbook.md`.
- `source:` — when a page is created from a dump, link back to `raw/dumps/<file>.md`.
- `tags:` — light, semantic. Don't over-tag.

## Style (from wiki-playbook.md)

- Concise, technical prose. No filler, no hedging beyond what's genuinely uncertain.
- Math in LaTeX: `$...$` inline, `$$...$$` display.
- Wikilink any concept that has or could have its own page.
- Never delete content — move deprecated material to `90-archive/` with a stub explaining where it went.
- Never invent citations.

## When dumps add to concepts

When `dumps-playbook.md` routes content here, your verbatim words go under a `## Notes — YYYY-MM-DD` section appended at the bottom (above `## References`). The canonical sections (Definition / Intuition / etc.) stay reserved for polished, concise wiki content.

## Example pages this vault will likely have

- `slip detection.md`
- `deformation gradient tensor.md`
- `Mindlin partial slip.md`
- `event-based vision sensor.md`
- `EtherCAT.md`
- `incipient slip.md`
- `visuotactile sensor.md`
- `BLDC anticogging.md`

## Filename convention

Lowercase, spaces allowed (matches Obsidian wikilink display). E.g., `slip detection.md` not `slip-detection.md` or `SlipDetection.md`. The wikilink `[[slip detection]]` then renders cleanly without aliasing.

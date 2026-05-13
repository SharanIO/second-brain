# WIKI.md — Schema Reference

This vault follows the Karpathy LLM Wiki pattern. Sources land in `raw/`, the compiled knowledge base lives in `wiki/`, and Claude maintains it as a side effect of conversations and processing routines.

**Active modes: D (Personal second brain) + E (Research synthesis)**

---

## Folder map

| Folder | Purpose |
|---|---|
| `wiki/concepts/` | Atomic concept pages — one idea per file, heavily linked |
| `wiki/entities/` | Named subjects: people, papers, tools, orgs, labs |
| `wiki/domains/` | High-level domain overviews (Robotics, NLP, Music, etc.) |
| `wiki/projects/` | Active projects with status, decisions, open questions |
| `wiki/courses/` | Structured course notes (sequential learning) |
| `wiki/sources/` | Wiki pages synthesized from ingested source documents |
| `wiki/questions/` | Open research questions and things to explore |
| `wiki/comparisons/` | Side-by-side comparisons of concepts, tools, approaches |
| `wiki/folds/` | Log rollup pages produced by `fold the log` |
| `wiki/meta/` | Obsidian Bases dashboards (dashboard.base, entities.base, sources.base) |
| `raw/incoming/` | Drop zone for all new external sources (PDFs, URLs, articles) |
| `raw/papers/` | Filed paper source files (PDFs + extracted markdown) |
| `raw/dumps/` | Stream-of-consciousness typed thoughts (lossless processing) |

---

## Note types and required frontmatter

### concept
```yaml
---
type: concept
status: stub | active | done
tags: []
last-updated: YYYY-MM-DD
---
```
Sections: Definition · Intuition · Mathematical statement · Why it matters · Connections · References

### entity
```yaml
---
type: entity
subtype: person | paper | tool | org | lab
tags: []
last-updated: YYYY-MM-DD
---
```

### domain
```yaml
---
type: domain
tags: []
last-updated: YYYY-MM-DD
---
```
Sections: Overview · Key concepts · Key entities · Open questions · Related domains

### project
```yaml
---
type: project
status: active | paused | done | blocked
started: YYYY-MM-DD
last-updated: YYYY-MM-DD
---
```
Sections: Goal · Status · Key decisions · Open questions · Artifacts · Notes — YYYY-MM-DD

### course
```yaml
---
type: course
course: <course-name>
chapter: <N>
tags: []
last-updated: YYYY-MM-DD
---
```

### source
```yaml
---
type: source
subtype: paper | book | article | transcript | handwriting
source: raw/papers/<filename>
status: integrated | partial | stub
tags: []
last-updated: YYYY-MM-DD
---
```
Sections: Contribution · Method · Key results · Critique · Concept pages created · Connections

### question
```yaml
---
type: question
status: open | answered | abandoned
tags: []
last-updated: YYYY-MM-DD
---
```

### comparison
```yaml
---
type: comparison
tags: []
last-updated: YYYY-MM-DD
---
```

---

## Core operations

| You say | Claude does |
|---|---|
| `ingest [file or url]` | Reads source → creates 8–15 wiki pages → updates index + log |
| `ingest all of these` | Batch parallel ingestion with cross-referencing |
| `what do you know about X?` | Reads hot.md → index.md → relevant pages → synthesized answer with citations |
| `/save` or `/save [name]` | Files current conversation as a wiki note |
| `/autoresearch [topic]` | 3-round web research loop → synthesized pages filed into wiki |
| `lint the wiki` | Health check: orphans, dead links, contradictions, stale claims |
| `fold the log` | Rolls up recent log.md entries into a compact page in wiki/folds/ |
| `update hot cache` | Manually refreshes wiki/hot.md |
| `process my dumps` | Lossless reorganization of raw/dumps/ — see dumps-playbook.md |
| `integrate this paper` | wiki-playbook flow for raw/incoming/ source — see wiki-playbook.md |

---

## Critical rules

- **Never modify `raw/`** — sources are immutable. Only wiki/ pages get updated.
- **Use `[[wikilinks]]`** for all internal references, never file paths.
- **One concept per page** — atomic notes. Split if a page covers two independent ideas.
- **Every page needs frontmatter** — type, status (if applicable), last-updated, tags.
- **Append-only log** — `wiki/log.md` is never edited, only prepended to. Newest entries at top.
- **Flag contradictions** — when a new source contradicts an existing wiki page, add a `[!contradiction]` callout to the concept page noting both sources.
- **Never invent citations** — if a claim needs a source not available, mark with `> [!todo] cite this`.
- **Preserve dumps verbatim** — raw/dumps/ content is losslessly reorganized, never summarized. See dumps-playbook.md.

---

## Session startup sequence

Every session in this vault, Claude reads in this order:
1. `CLAUDE.md` (auto-loaded)
2. `wiki/hot.md` (recent context cache)
3. `wiki/index.md` (master catalog)
4. The relevant playbook for the task at hand

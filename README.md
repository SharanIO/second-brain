# Second Brain

Personal learning vault — Karpathy-style LLM-wiki pattern. Organized so an agent (Claude) can maintain it as a side effect of conversations and processing routines.

## Playbooks

Two agent-facing playbooks at the root:

- **`wiki-playbook.md`** — how Claude integrates **external sources** (papers, books, articles, transcripts, handwriting) into the wiki. Style: concise, technical.
- **`dumps-playbook.md`** — how Claude processes the user's **own typed thoughts** in `raw/dumps/` into organized notes. Style: lossless — every word preserved.

The two playbooks have *opposite* style contracts. Make sure the right one applies to the right input.

Also:
- **`CLAUDE.md`** — auto-loaded by Claude Code. Contains trigger vocabulary, default behaviors, and session startup sequence.
- **`WIKI.md`** — schema reference: all note types, frontmatter conventions, folder purposes, and core operations.
- **`.cursorrules`** — original user/vault context block.

## Folder map

| Folder | Purpose |
|---|---|
| `wiki/concepts/` | Atomic concept pages (the synthesis layer) |
| `wiki/entities/` | Named subjects: people, papers, tools, orgs, labs |
| `wiki/domains/` | High-level domain overview pages |
| `wiki/projects/` | Active projects with status, decisions, artifacts |
| `wiki/courses/` | Structured course notes |
| `wiki/sources/` | Wiki pages synthesized from ingested source documents |
| `wiki/questions/` | Open research questions |
| `wiki/comparisons/` | Side-by-side concept/tool comparisons |
| `wiki/folds/` | Log rollup pages (wiki-fold) |
| `wiki/meta/` | Obsidian Bases dashboards |
| `wiki/index.md` | Master catalog — updated on every ingest |
| `wiki/hot.md` | Session context cache (~500 words) |
| `wiki/log.md` | Append-only operation log |
| `wiki/overview.md` | Executive summary + research gaps |
| `raw/incoming/` | Drop zone for all new external sources |
| `raw/papers/` | Filed paper source files (PDFs + extracted markdown) |
| `raw/dumps/` | Stream-of-consciousness typed thoughts |
| `daily/` | Daily notes (one per day) |
| `assets/` | Images and binaries embedded in notes |
| `code/` | Code implementations linked to wiki notes |
| `guides/` | Runbooks and setup how-tos |
| `archive/` | Deprecated content (never deleted) |
| `inbox/` | Unrouted files awaiting a decision |
| `_templates/` | Templater plugin templates |

## The two main routines

### 1. Source integration (wiki-playbook)

Drop a paper / article / book / talk → into `raw/incoming/` → ask Claude to integrate it → relevant `wiki/concepts/` pages get updated, a `wiki/sources/` page is created, cross-links are added, `wiki/index.md` and `wiki/log.md` are updated.

### 2. Dump processing (dumps-playbook)

Type a stream-of-consciousness note into `raw/dumps/` → ask Claude to "process the inbox" → Claude splits it by topic into organized notes **without dropping a word** → original stays in `raw/dumps/` verbatim with `processed: YYYY-MM-DD` added to frontmatter.

## Commit conventions

- `vault-auto: ...` — auto-commits from Obsidian Git plugin or PostToolUse hook.
- `manual-edit: ...` — direct human edits.
- `agent-edit: ...` — edits by Claude.

One commit per logical unit (one paper integrated = one commit; one dump processed = one commit).

## Frontmatter conventions

Standard YAML fields used across the vault:

```yaml
type: daily | concept | entity | domain | project | course | source | question | comparison
status: stub | active | done | paused | blocked | open | answered
last-updated: YYYY-MM-DD
source: <relative path to raw source>
processed: YYYY-MM-DD
tags: [list]
```

Frontmatter is **preserved on every agent edit** — never silently dropped.

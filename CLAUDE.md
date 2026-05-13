# Vault context for Claude

This file is loaded automatically by Claude Code and any other Claude-based agent operating in this vault. It defines the user's context and the vault's working agreements so that triggers and contracts are persistent locally — not dependent on any remote memory layer.

## Who

Sharan — robotics engineer (contractor at Meta Reality Labs via Unify Technologies, working on tactile sensing, slip detection, and actuator characterization). Preparing PhD applications. Plays drums; runs a music project called "The Seventh Bar."

Personal context relevant to vault content lives across `wiki/projects/` (PhD apps, music, apartment lease, H-1B sponsorship, resume rebuild, etc.). See `.cursorrules` for the original full context block.

## Working agreements

This vault follows the Karpathy LLM-wiki pattern. Two agent playbooks govern processing:

- **`wiki-playbook.md`** — for **external sources** (papers, books, articles, transcripts, handwriting under `raw/`). Style: concise, technical integration into the wiki.
- **`dumps-playbook.md`** — for **the user's own typed thoughts** in `raw/dumps/`. Style: **lossless reorganization** — every word preserved, never summarized.

The two playbooks have **opposite** style contracts. Apply the right one to the right input.

For the full schema, folder map, note types, and frontmatter conventions, see `WIKI.md`.

## Trigger vocabulary

When the user uses any of the phrases below, follow the linked playbook flow.

| User says | What to do |
|---|---|
| "process my dumps", "process the inbox", "run the dumps routine", "go through dumps" | Read `dumps-playbook.md`, list unprocessed files in `raw/dumps/` (those without a `processed:` frontmatter field, ignoring `README.md`), execute the flow per the lossless contract. |
| "integrate this paper", "process the source in incoming", "file this paper" | Read `wiki-playbook.md`, process the relevant file in `raw/incoming/` (or named source), follow the concise-integration flow. |
| "ingest [file or url]" | wiki-ingest flow: read source → extract entities and concepts → create 8–15 wiki pages → update `wiki/index.md` and `wiki/log.md`. Use defuddle for URLs. |
| "ingest all of these" | Batch ingest with parallel processing and cross-referencing. |
| "what do you know about X?", "what's in the wiki on X?" | Read `wiki/hot.md` → `wiki/index.md` → drill into 3–5 relevant pages → synthesize answer with inline citations to specific wiki pages (not training data). |
| "/save", "save this conversation" | File the current conversation as a wiki note in the appropriate subfolder. Prompt for a name if not provided. |
| "/autoresearch [topic]" | Run a 3-round autonomous web research loop: (1) search, (2) fetch top results, (3) gap-filling searches. Synthesize findings and file into wiki. Update index and hot cache. |
| "lint the wiki" | Health check: orphaned pages, dead links, stale claims, missing cross-references, content gaps, duplicate concepts. Report issues and fix if instructed. |
| "fold the log" | Roll up the top 2^k entries from `wiki/log.md` into a compact extractive summary page in `wiki/folds/`. Dry-run first, commit only after confirmation. |
| "update hot cache" | Manually refresh `wiki/hot.md` with a summary of recent changes and active threads. Keep under 500 words. |
| "what's in my inbox", "what hasn't been processed" | List unprocessed dumps in `raw/dumps/` and unfiled sources in `raw/incoming/`. Report only — do not process. |
| "archive X" | Move `X` to `archive/` mirroring its original path. Leave a stub-redirect at the old path per archive conventions. |
| "what do I have on Y" / "summarize project Y" | Read the relevant `wiki/projects/Y.md` and report — no edits without instruction. |

## Default behaviors

- **Session startup**: Read `wiki/hot.md` first (recent context cache), then `wiki/index.md` (master catalog). This happens automatically via the SessionStart hook but verify on first message if needed.
- Always preserve YAML frontmatter on edits. Add `last-updated: YYYY-MM-DD` on substantive changes.
- Never delete content — archive per `archive/` conventions.
- Commits: prefix `agent-edit: ` for any agent-driven changes. One commit per logical unit (one dump processed = one commit; one paper integrated = one commit). See `wiki-playbook.md` for full commit conventions.
- When in doubt about destination or which playbook contract applies, ask the user before acting.
- Never invent citations. If a claim needs a source not available, mark with `> [!todo] cite this`.
- Flag contradictions with `[!contradiction]` callouts on concept pages when a new source disagrees with existing content.

## Files an agent should read at session start

In order:

1. This file (`CLAUDE.md`) — auto-loaded.
2. `wiki/hot.md` — recent context cache (~500 words, updated each session).
3. `wiki/index.md` — master catalog of all wiki pages.
4. `.cursorrules` — original user/vault context (authoritative for user background and Python/C++/ROS 2 fluency).
5. `README.md` — folder map.
6. The relevant playbook for the task at hand (`wiki-playbook.md` or `dumps-playbook.md`).

Reading these in order takes under 2 minutes and prevents almost all routing mistakes.

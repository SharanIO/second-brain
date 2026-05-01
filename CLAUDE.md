# Vault context for Claude

This file is loaded automatically by Claude Code and any other Claude-based agent operating in this vault. It defines the user's context and the vault's working agreements so that triggers and contracts are persistent locally — not dependent on any remote memory layer.

## Who

Sharan — robotics engineer (contractor at Meta Reality Labs via Unify Technologies, working on tactile sensing, slip detection, and actuator characterization). Preparing PhD applications. Plays drums; runs a music project called "The Seventh Bar."

Personal context relevant to vault content lives across `40-projects/` (PhD apps, music, apartment lease, H-1B sponsorship, resume rebuild, etc.). See `.cursorrules` for the original full context block.

## Working agreements

This vault follows the Karpathy LLM-wiki pattern. Two agent playbooks govern processing:

- **`wiki-playbook.md`** — for **external sources** (papers, books, articles, transcripts, handwriting under `raw/`). Style: concise, technical integration into the wiki.
- **`dumps-playbook.md`** — for **the user's own typed thoughts** in `raw/dumps/`. Style: **lossless reorganization** — every word preserved, never summarized.

The two playbooks have **opposite** style contracts. Apply the right one to the right input.

For folder-by-folder layout, see `README.md` (and the per-folder `README.md` files). For style and commit conventions, see `wiki-playbook.md`.

## Trigger vocabulary

When the user uses any of the phrases below, follow the linked playbook flow.

| User says | What to do |
|---|---|
| "process my dumps", "process the inbox", "run the dumps routine", "go through dumps" | Read `dumps-playbook.md`, list unprocessed files in `raw/dumps/` (those without a `processed:` frontmatter field, ignoring `README.md`), execute the flow per the lossless contract. |
| "integrate this paper", "process the source in incoming", "file this paper" | Read `wiki-playbook.md`, process the relevant file in `raw/incoming/` (or named source), follow the concise-integration flow. |
| "what's in my inbox", "what hasn't been processed" | List unprocessed dumps in `raw/dumps/` and unfiled sources in `raw/incoming/`. Report only — do not process. |
| "archive X" | Move `X` to `90-archive/` mirroring its original path. Leave a stub-redirect at the old path per `90-archive/README.md`. |
| "what do I have on Y" / "summarize project Y" | Read the relevant `40-projects/Y.md` and report — no edits without instruction. |

## Default behaviors

- Always preserve YAML frontmatter on edits. Add `last-updated: YYYY-MM-DD` on substantive changes.
- Never delete content — archive per `90-archive/README.md`.
- Commits: prefix `agent-edit: ` for any agent-driven changes. One commit per logical unit (one dump processed = one commit; one paper integrated = one commit). See `wiki-playbook.md` for full commit conventions.
- When in doubt about destination or which playbook contract applies, ask the user before acting.
- Never invent citations. If a claim needs a source not available, mark with `> [!todo] cite this`.

## Files an agent should read at session start

In order:

1. This file (`CLAUDE.md`).
2. `.cursorrules` — original user/vault context (still authoritative for the user's background and Python/C++/ROS 2 fluency).
3. `README.md` — folder map.
4. The relevant playbook for the task at hand (`wiki-playbook.md` or `dumps-playbook.md`).

Reading these in this order takes <1 minute and prevents almost all routing mistakes.

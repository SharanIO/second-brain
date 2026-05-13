---
type: hot-cache
last-updated: 2026-05-13
---

## Last Updated
2026-05-13 02:10

## Key Recent Facts
- Vault was fully restructured this session: numbered folders → `wiki/` hierarchy (Agrici Daniel claude-obsidian pattern)
- 15 NLP concept pages live in `wiki/concepts/` — all at `status: complete` or `status: active`
- 4 session hooks active: SessionStart (reads hot.md), PostCompact (re-reads hot.md), PostToolUse Write|Edit (auto-commits wiki/ and raw/), Stop (signals wiki changed)
- defuddle v0.18.1 installed globally — use for URL ingestion to cut token usage
- Two playbooks: `wiki-playbook.md` (external sources, concise) vs `dumps-playbook.md` (user thoughts, lossless)

## Recent Changes
- Created `wiki/domains/nlp.md` — full NLP domain map: 6 clusters, concept dependency graph, historical arc, missing areas
- Expanded `wiki/overview.md` — Sharan's full context, all domains (NLP detailed, Robotics gap, Projects), 7 unprocessed robotics papers listed
- Expanded `wiki/concepts/positional-encoding.md` — sinusoidal math, RoPE rotation derivation, ALiBi, 4-method comparison table
- Expanded `wiki/concepts/regularization.md` — L2/L1 geometry, Adam/AdamW divergence with full math, dropout, label smoothing, early stopping
- Fixed frontmatter: resume project pages corrected from `type: concept` → `type: project`
- Fixed stale critique in `wiki/sources/adv-nlp-course-notes.md` (removed outdated raw/papers/ reference)
- Added NLP domain to `wiki/index.md`

## Active Threads
- **Robotics domain** (highest priority gap): 7 papers in `raw/papers/`, zero wiki pages — ready to integrate when instructed
- **PhD applications**: no project page yet — `wiki/projects/phd-applications.md` is high priority
- **Attention Is All You Need**: full paper extracted in `raw/papers/attention-is-all-you-need/`, not yet processed through wiki-playbook
- **LeetCode**: user is interested; `code/leetcode/` + `wiki/projects/leetcode.md` not yet created
- **Resume rebuild**: active in `wiki/projects/resume/` — prompt and writing rules are ready

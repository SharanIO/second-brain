---
type: log
---

# Operation Log

Append-only. Newest entries at top.

Format per entry:
```
## YYYY-MM-DD HH:MM — <operation>
Pages created: N | Pages updated: N | Source: <filename>
- Summary of what was done
```

---

## 2026-05-13 02:10 — Content expansion
Pages created: 2 | Pages updated: 6 | Source: session

- Expanded `wiki/domains/nlp.md` from scratch: 6 concept clusters with historical motivation, full dependency graph, missing areas section
- Expanded `wiki/overview.md` to full depth: Sharan's context, NLP coverage map, robotics gap (7 papers), projects status, active threads
- Expanded `wiki/concepts/positional-encoding.md`: permutation-invariance proof, sinusoidal math + base-10000 intuition, RoPE rotation derivation, ALiBi, 4-method comparison table
- Expanded `wiki/concepts/regularization.md`: bias-variance framing, L2/L1 geometry + Bayesian priors, Adam vs AdamW divergence with full math, label smoothing, early stopping, LayerNorm as implicit regularization
- Fixed frontmatter: `resume-meta-ai-prompt.md` and `resume-writing-rules.md` corrected to `type: project`
- Fixed stale critique in `adv-nlp-course-notes.md` (raw/papers/ reference removed)
- Updated `wiki/index.md` Domains section
- Updated `wiki/hot.md` (first population)

---

## 2026-05-13 — Vault restructure

Migrated from numbered folder scheme to wiki/ structure. Adopted Agrici Daniel's claude-obsidian pattern.

- Moved 14 concept pages → `wiki/concepts/`
- Moved 2 project pages → `wiki/projects/`
- Moved 12 adv-nlp course chapters → `wiki/courses/adv-nlp/`
- Moved adv-nlp-course-notes.md → `wiki/sources/`
- Moved 7 robotics PDFs → `raw/papers/`
- Created `wiki/hot.md`, `wiki/index.md`, `wiki/log.md`, `wiki/overview.md`
- Created `wiki/meta/` dashboards
- Created `WIKI.md` schema reference
- Added 4 session hooks (SessionStart, PostCompact, PostToolUse, Stop)
- Removed empty raw subfolders (articles, books, transcripts, handwriting)

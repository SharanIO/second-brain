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

# 10-daily/

**Daily notes** — one file per day. Use the `templates/daily.md.md` template (Templater plugin).

## Template structure

```markdown
---
date: 2026-05-01
type: daily
---

# Friday, May 1st 2026

## Focus today
-

## Captured
-

## Reflection
```

## What goes in each section

- **Focus today** — 1–3 things you intend to make progress on. Written at the start of the day.
- **Captured** — quick notes you jotted during the day. Meeting takeaways, ideas, links, reminders. Light-touch capture, not deep thinking.
- **Reflection** — end-of-day recap. What happened, what you learned, what's stuck.

## Daily vs dumps — when to use which

| | `10-daily/` | `raw/dumps/` |
|---|---|---|
| Cadence | One per day | One per *thought* (multiple per day OK) |
| Style | Structured (focus / captured / reflection) | Freeform stream-of-consciousness |
| Length | Short — single line per `Captured` bullet | Whatever you want |
| Processed by agent? | Optional, when explicitly asked | Yes, via the dumps-playbook routine |

Rule of thumb: if it's "today I had a meeting and decided X" → `10-daily/`. If it's "let me think through this technical problem out loud for 20 minutes" → `raw/dumps/`.

## Filename convention

`YYYY-MM-DD.md` (e.g., `2026-05-01.md`). Templater handles this automatically if configured.

## Example

`10-daily/2026-05-01.md`:

```markdown
---
date: 2026-05-01
type: daily
---

# Friday, May 1st 2026

## Focus today
- Finalize lease signing for Beaumont
- 2hr block on slip detection literature review

## Captured
- Manager mentioned Q3 demo pushed to August
- [[event-based vision sensor]] could be relevant — followup
- Recruiter from Zoox emailed, follow up Monday

## Reflection
Made progress on lease but lost the afternoon to the contract extension email
draft. Tomorrow start with the literature review block first.
```

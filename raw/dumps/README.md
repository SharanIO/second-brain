# raw/dumps/

**Stream-of-consciousness inbox.** Write whatever's on your mind here, no structure required.

## How to use

1. Create a new file in this folder. Filename can be anything — `YYYY-MM-DD-HHMM.md` is a good default but don't think about it.
2. Write. Bullets, paragraphs, fragments, mid-sentence pivots, contradictions — all fine. No frontmatter needed.
3. Save. That's it. Don't organize, don't tag, don't link.

When you're ready, ask Claude to **"process the inbox"** (or "process dumps"). Claude will:

- Read each unprocessed dump.
- Split it by topic into organized notes that go to the right home (`20-concepts/`, `40-projects/`, or stub concept pages).
- Mark this file as processed by adding `processed: YYYY-MM-DD` to the frontmatter — content is left **untouched**.
- Report what went where.

## The contract

Claude will **not** drop words, summarize, or "tighten" your prose. Every distinct idea, caveat, hedge, and tangent appears somewhere in the organized output. Full rules in `dumps-playbook.md` at the vault root.

If anything ever feels lost, this folder is your audit trail — the original is preserved verbatim forever.

## What unprocessed looks like

Files in this folder **without** a `processed:` field in their frontmatter. To find them in Obsidian: search `path:raw/dumps -["processed"]` or just look at file metadata.

## Example

`raw/dumps/2026-05-01-1430.md`:

```markdown
been thinking about EVS cameras for slip detection again. async pixel firing on
intensity change might be perfect for slip events bc slip is inherently
transient — current frame cameras at 30Hz alias the actual slip signal which
happens way faster. but data is sparse and async so the whole tracker arch
has to change. how does this interact with the deformation gradient
computation? gradients assume dense field. maybe a hybrid: EVS for trigger,
RGB for gradient.

also — need to email manager about contract extension by friday. don't
mention immigration stuff in same thread, keep separate.

random: PhD reading list — should add Calandra 2018 to the slip papers list?
```

After processing this would split into (roughly):
- `20-concepts/event-based vision sensor.md` — stub, with the EVS thinking under `## Notes — 2026-05-01`
- `40-projects/contract-extension.md` (or wherever appropriate) — the manager email line
- `40-projects/phd-applications.md` — the Calandra 2018 question

Original stays here verbatim with `processed: 2026-05-01` added.

## Distinct from

- `raw/handwriting/` — OCR'd handwritten notes (different input modality).
- `raw/transcripts/` — transcribed audio (different input modality).
- `raw/incoming/` — external sources (PDFs, articles) awaiting filing.
- `00-inbox/` — generic unfiled stuff that doesn't have a workflow.
- `10-daily/` — daily journal entries (per-day cadence; dumps are per-thought).

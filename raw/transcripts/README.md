# raw/transcripts/

**Transcribed audio or video.** Talks, podcasts, interviews, lectures, your own voice memos — anything where the original modality was speech.

## What goes here

- Conference talk transcripts (e.g., NeurIPS / RSS / ICRA talks).
- Podcast episode transcripts.
- Lecture transcripts (course videos).
- Interview transcripts (e.g., Lex Fridman, Dwarkesh, MLST).
- Your own voice memo transcripts (Whisper'd from a `.m4a`).

## File format

`.md` files with the transcript as the body. Recommended frontmatter:

```markdown
---
type: transcript
source: <URL or recording filename>
speaker(s): <names>
date-recorded: YYYY-MM-DD
date-transcribed: YYYY-MM-DD
duration: ~Nh Mm
---

# Title of talk / episode

[Transcript body, paragraph form. Speaker tags if multi-speaker.]
```

## Workflow

Same wiki-playbook flow as papers and articles:

1. Claude reads the transcript in full.
2. Updates relevant `20-concepts/` pages with new examples or framings.
3. Creates concept stubs as needed.
4. Optionally creates a `30-papers/<slug>.md` review-style note (only if the talk is substantial enough — a 90-minute Hinton interview yes, a 5-minute clip probably not).
5. Cross-links concepts.

## Naming convention

`<speaker>-<venue-or-podcast>-<YYYY-MM-DD>.md`. E.g.:
- `hinton-fridman-2024-05-12.md`
- `karpathy-rss-keynote-2025-07-15.md`
- `voice-memo-2026-04-30-slip-pipeline-thoughts.md`

## Voice memos vs dumps

| | `raw/transcripts/` (voice memo) | `raw/dumps/` (typed) |
|---|---|---|
| Modality | Speech → transcribed | Direct typing |
| Contract | wiki-playbook (concise integration) | dumps-playbook (lossless reorganization) |
| Original artifact | Audio file + transcript | The `.md` file itself |

If you specifically want lossless treatment of a voice memo (e.g., it's meandering personal thinking), you can drop the transcript into `raw/dumps/` instead — that flips the contract from concise-integration to lossless-reorganization.

## Distinct from

- Live conversation notes (e.g., a meeting you took notes in) — those are `10-daily/` captures or a `40-projects/` project page entry, not a transcript.

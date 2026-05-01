# raw/

**Source material.** Anything that gets *integrated into* the wiki but isn't itself a wiki page. Sources live here forever, even after they've been processed — they're the citation trail.

Per `wiki-playbook.md`:

> Sources land in `raw/` (subdivided into `papers/`, `books/`, `articles/`, `transcripts/`, `handwriting/`, with `incoming/` as the staging area).

`raw/dumps/` was added later as a sibling for the user's own typed stream-of-consciousness — see `dumps-playbook.md`.

## Subfolders

| Folder | Contains | Processed by |
|---|---|---|
| `incoming/` | External sources (PDFs, articles) awaiting categorization | wiki-playbook |
| `papers/` | Filed academic papers (PDFs) | wiki-playbook |
| `books/` | Books, textbooks (PDFs/epubs) | wiki-playbook |
| `articles/` | Web articles, blog posts saved offline | wiki-playbook |
| `transcripts/` | Transcribed audio (interviews, talks, your own voice memos) | wiki-playbook |
| `handwriting/` | OCR'd handwritten notes | wiki-playbook |
| `dumps/` | Your own typed stream-of-consciousness | dumps-playbook (lossless) |

## Two distinct processing contracts

- **wiki-playbook.md** applies to `papers/`, `books/`, `articles/`, `transcripts/`, `handwriting/`, `incoming/`. The source still exists; the agent produces a *concise* wiki integration.
- **dumps-playbook.md** applies to `dumps/`. The source IS the user's words; the agent produces a *lossless* reorganization that preserves every word.

The two contracts are opposites — make sure you (and any agent) apply the right one to the right subfolder.

## Lifecycle of a source

1. Lands in `raw/incoming/` (external sources) or `raw/dumps/` (your typing) or directly in a typed subfolder (`raw/papers/`, etc.) if you know where it goes.
2. Gets read and integrated by the appropriate playbook flow.
3. Original stays where it is, **forever**. Marked processed via frontmatter (for dumps) or by the existence of derivative wiki pages (for external sources).
4. The wiki pages that resulted from it cite it back — the source is the audit trail.

## Sources are never deleted

If a source is wrong / superseded / no longer useful, the *wiki integration* gets corrected — the source itself stays. This is the citation chain working as intended.

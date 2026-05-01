# raw/handwriting/

**OCR'd handwritten notes.** Scans or photos of paper notebooks, whiteboard photos, handwritten margin notes — converted to text via OCR.

## What goes here

- iPad/Notability/GoodNotes exports → text.
- Phone photos of a paper notebook page, run through OCR.
- Whiteboard photos from meetings, OCR'd.
- Anything where the original was handwritten and you want the text content searchable and integrable.

## File format

`.md` files with the OCR'd text as the body. Keep the source image too if you have it.

Recommended frontmatter:

```markdown
---
type: handwriting
source-image: assets/handwriting/2026-04-15-page-1.jpg
date-written: YYYY-MM-DD
date-ocrd: YYYY-MM-DD
---

# Notebook page — 2026-04-15

[OCR'd text. Clean up obvious OCR errors but otherwise preserve.]
```

## Workflow

Same wiki-playbook flow:

1. Claude reads the OCR'd text.
2. Updates relevant `20-concepts/` pages.
3. Creates concept stubs as needed.
4. Cross-links.

If your handwritten note is itself stream-of-consciousness thinking (not a structured derivation or note), consider routing it through `raw/dumps/` instead — that flips the contract to lossless. The `raw/handwriting/` flow assumes the original was already organized.

## Naming convention

`<YYYY-MM-DD>-<short-context>.md`. E.g.:
- `2026-04-15-slip-derivation-page-1.md`
- `2026-04-22-whiteboard-pipeline-arch.md`

## Source images

Keep originals. OCR is lossy; the image is the truth. Store under `assets/handwriting/` or alongside the `.md` file with the same name.

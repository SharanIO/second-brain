# raw/incoming/

**Staging area for external sources** awaiting filing into the right subfolder of `raw/` (`papers/`, `books/`, `articles/`, etc.).

This is *not* the dumps inbox — that's `raw/dumps/`, with a different contract.

## What goes here

External sources you've collected but haven't yet:
- Categorized by type (paper vs article vs book), or
- Read and integrated into the wiki.

Examples:
- A PDF you downloaded but haven't classified yet.
- An article saved offline that might be worth integrating.
- A book chapter scan awaiting OCR.

## Workflow

When Claude processes `raw/incoming/`:

1. Reads each file in full.
2. Classifies it: paper? article? book? transcript?
3. Moves it to the appropriate sibling folder (`raw/papers/`, etc.). May ask if ambiguous.
4. Runs the wiki-playbook integration:
   - Updates relevant `20-concepts/` pages.
   - Creates new concept stubs as needed.
   - Creates a `30-papers/` review note (if it's a paper).
   - Cross-links aggressively.
5. Commits per source: `agent-edit: integrate <source title>`.

## Distinct from `raw/dumps/`

| | `raw/incoming/` | `raw/dumps/` |
|---|---|---|
| Source | External (collected from elsewhere) | Internal (your typing) |
| Contract | Concise wiki integration | Lossless reorganization |
| Playbook | wiki-playbook.md | dumps-playbook.md |
| Source persists post-processing? | Yes (PDF stays) | Yes (your words stay) |
| Output style | Distilled, technical | Faithful, every word preserved |

## Current contents

- `attention-is-all-you-need.pdf` — Vaswani et al. 2017, the transformer paper. Awaiting integration.
- `NLP.pdf` — origin/identity unclear; needs review.

## What does NOT go here

- Your own typed thoughts → `raw/dumps/`.
- Random unfiled junk that has no workflow → `00-inbox/`.
- Sources you've already classified — file them directly into `raw/papers/`, `raw/books/`, etc.

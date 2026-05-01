# raw/books/

**Books and book-length sources** — full books, textbooks, monographs, long technical reports. PDFs or epubs.

## Naming convention

`<author-lastname>-<short-title>.pdf`. E.g.:
- `siciliano-robotics-modeling-planning-control.pdf`
- `sutton-barto-rl-2nd-ed.pdf`

## Workflow

Unlike papers (one review note per paper), books are usually too large for a single review note. The integration pattern:

1. Claude reads the book section by section as you work through it.
2. **Multiple `20-concepts/` pages** get updated as concepts come up — books are concept-heavy and one book typically contributes to many concept pages.
3. **Optionally**, create a `30-papers/<book-slug>.md` review note for the book overall (your meta-take, when you've finished it). This is optional — for a textbook you reference but don't read end-to-end, no review note is needed.
4. Cross-link from concept pages to specific chapters: `See [[siciliano-robotics-modeling-planning-control|Siciliano §3.4]] for the standard derivation.`

## What does NOT go here

- Academic papers → `raw/papers/`.
- A book chapter someone sent as a standalone PDF → `raw/articles/` if it stands alone, or `raw/books/` if it's part of a book you'll read more of.

## Lifecycle

Books stay forever. A book you no longer endorse still gets to live here — your concept pages capture your evolved view.

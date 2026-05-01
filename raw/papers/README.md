# raw/papers/

**Filed academic papers (PDFs).** Each paper here typically has a corresponding review note in `30-papers/`.

## Naming convention

`<first-author-lastname><year>-<short-slug>.pdf`. E.g.:
- `calandra2018-more-than-feeling.pdf`
- `dong2017-gelslim.pdf`
- `vaswani2017-attention-is-all-you-need.pdf`

This makes citation, searching, and linking easy.

## Workflow

A paper arrives via `raw/incoming/` → after classification it moves here → wiki-playbook flow:

1. Claude reads the PDF in full.
2. Updates relevant `20-concepts/` pages with new examples / refinements / contradictions.
3. Creates concept stubs for new ideas the paper introduces.
4. Creates a `30-papers/<paper-slug>.md` review note.
5. Cross-links: every concept mentioned in the review wikilinks to its concept page.

## What does NOT go here

- Books → `raw/books/`.
- Blog posts and web articles → `raw/articles/`.
- Preprints you haven't read or filed yet → keep in `raw/incoming/` until ready.

## Linking from notes

From a concept page or paper review, link by relative path:

```markdown
See [the original paper](raw/papers/calandra2018-more-than-feeling.pdf).
```

Or use Obsidian's PDF embed:

```markdown
![[calandra2018-more-than-feeling.pdf]]
```

## Lifecycle

PDFs here stay forever. Even if you decide a paper's central claim is wrong, the PDF stays — your `30-papers/` review and `20-concepts/` pages capture the correction.

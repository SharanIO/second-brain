# raw/articles/

**Web articles, blog posts, technical write-ups** saved offline. Anything that's prose-form on the internet but you want to integrate into your wiki.

## What goes here

- Karpathy / Lilian Weng / Chris Olah-style technical blog posts.
- Industry write-ups (engineering blogs from robotics or AI labs).
- News or feature articles relevant to your work or projects.
- Substack essays, Medium posts, etc.

## What does NOT go here

- Academic papers (even on arXiv) → `raw/papers/`.
- Books → `raw/books/`.
- YouTube transcripts → `raw/transcripts/`.

The line: if it has a `[paper-id]` and looks like a PDF preprint, it's a paper. If it's prose styled as a blog post or article, it's an article.

## File format

- Web pages: save as `.md` (clean) or `.pdf` (faithful) — your call. Markdown is searchable; PDF preserves layout.
- Naming: `<author-or-publication>-<short-slug>.md`. E.g., `karpathy-software-2-0.md`, `anthropic-constitutional-ai.md`.

## Workflow

Same as papers in spirit, lighter in practice:

1. Claude reads the article.
2. Updates relevant `20-concepts/` pages.
3. Creates concept stubs as needed.
4. Optionally creates a `30-papers/<slug>.md` review note — only if the article is substantial enough to warrant a standalone review. Most blog posts don't need one; the integration into concept pages is the artifact.

## Citing

From a concept page:

```markdown
See [[karpathy-software-2-0]] for the original framing of "Software 2.0".
```

If the article links to a primary source (paper, talk), prefer citing the primary source; the article is contextual scaffolding.

# 30-papers/

**One paper review note per academic paper.** Each note summarizes the paper's contribution, method, key result, and your critique. Linked from the underlying PDF in `raw/papers/`.

## Template

`templates/paper.md.md` (check the file for current canonical structure).

Typical sections:

```markdown
---
type: paper
authors:
year:
venue:
pdf: raw/papers/<filename>.pdf
tags:
---

# Paper title

## Contribution

## Method

## Key result

## Critique

## Connections
- [[related concept]]

## Quotes / figures worth keeping
```

## Workflow (from wiki-playbook.md)

When you add a new PDF to `raw/papers/` (or `raw/incoming/`), Claude:

1. Reads the PDF in full.
2. Updates relevant existing concept pages in `20-concepts/` with new examples, refinements, contradictions.
3. Creates stub concept pages for any new concepts the paper introduces.
4. Creates this paper review note here.
5. Cross-links aggressively — every concept mentioned should `[[wikilink]]` to its concept page.

## Style

- Concise. Technical. No hedging beyond what's genuinely uncertain.
- The review is **your understanding**, not a re-summary of the abstract. What did you take from it? What's wrong with it?
- Critique is allowed and encouraged. Disagreement with the paper goes in `## Critique` and may also surface as a `## Contradictions` section on the relevant concept page.

## Filename convention

`<first-author-lastname><year>-<short-slug>.md` works well. E.g., `calandra2018-more-than-feeling.md`.

## Example pages relevant to your work

- `calandra2018-more-than-feeling.md` (slip prediction)
- `dong2017-gelslim.md` (visuotactile sensors)
- Etc.

## Distinct from

- `raw/papers/` — the PDFs themselves live there. This folder holds the *review notes about* those PDFs.
- `20-concepts/` — concepts get integrated knowledge from many papers. A paper review is the unit; concepts are the synthesis.

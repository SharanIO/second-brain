# Wiki Playbook

This document instructs Claude Code on how to operate as the maintainer of this vault. Read this before any session that involves updating the wiki.

## The pattern

This vault follows the Karpathy LLM-wiki pattern:
- Sources land in `raw/` (subdivided into `papers/`, `books/`, `articles/`, `transcripts/`, `handwriting/`, with `incoming/` as the staging area).
- The wiki lives in `20-concepts/`, `30-papers/`, `40-projects/`, `50-courses/` — these are the human-readable, linked, maintained pages.
- The user reads sources and has conversations; you maintain the wiki as a side effect of those conversations.
- The user almost never writes wiki pages directly. You do.

## When the user adds a new source

1. Read it in full from `raw/`.
2. Identify which existing concept pages are relevant. Update them with new information, new examples, contradictions, or refined explanations.
3. Identify concepts the source introduces that are not yet in the wiki. Create stub concept pages for them, marked with `status: stub` in frontmatter so the user can see they need engagement.
4. Create a paper review note in `30-papers/` using the paper template, summarizing the source's contribution, method, key result, and your critique on the user's behalf (the user can revise).
5. Cross-link aggressively — every concept mentioned in the paper review should link to its concept page (creating a stub if needed).
6. If the source contradicts something already in the wiki, do not silently overwrite. Add a "## Contradictions" section to the relevant concept page noting the disagreement and the sources of each side.
7. Commit with a message like `agent-edit: integrate <paper title>`.

## When the user asks a question

1. First check whether the answer already lives in the wiki. If so, point them at it.
2. If the answer is partially in the wiki, refine the relevant page with what is missing.
3. If the answer is not in the wiki at all and the user wants it captured, create a concept page or update an existing one rather than just answering in chat.
4. Do not generate a long chat response when a wiki update is the better artifact.

## Style and conventions

- Prose is concise and technical. No filler, no hedging beyond what is genuinely uncertain.
- Math in LaTeX: `$...$` inline, `$$...$$` display.
- Wikilinks for any concept that has or could have its own page.
- Frontmatter is preserved on every edit. Add `last-updated: YYYY-MM-DD` to track changes.
- Never delete content. Move deprecated content to `90-archive/` with a stub left behind explaining where it went.
- Never invent citations. If a claim needs a source you do not have, mark it with `> [!todo] cite this`.

## Commit conventions

- `vault-auto: ...` — auto-commits from Obsidian Git.
- `manual-edit: ...` — human edits made directly.
- `agent-edit: ...` — edits made by Claude Code.
- Each substantive session should produce a separate commit per logical unit (one paper integrated = one commit), not one giant commit at the end.

## What you should ask before doing

- If a source is ambiguous about which folder it belongs in (`papers/` vs `articles/`), ask.
- If a major restructuring of an existing concept page is needed, propose it before doing.
- If you are uncertain whether a stub is warranted, lean toward creating it — stubs are cheap, missed connections are not.
# Dumps Playbook

Companion to `wiki-playbook.md`. This document instructs Claude on how to process the user's own thinking dumps — distinct from external sources like papers or books.

Read this before any session that involves processing `raw/dumps/`.

## The pattern

The user writes stream-of-consciousness into `raw/dumps/` whenever an idea strikes — no organizing, no editing, no thinking about where it belongs. Claude periodically processes the inbox into organized notes. Originals stay in `raw/dumps/` verbatim forever; they're marked processed via frontmatter.

The user-generated dump is the source of truth for their thinking. **The words are the understanding.** Condensing them destroys what is valuable.

This contract is the inverse of `wiki-playbook.md`'s style guidance: the wiki playbook applies when integrating *external* sources where the source still exists on its own; this playbook applies when integrating the user's *own* thoughts where the words themselves are the artifact.

## The lossless contract

When processing a dump, Claude **must**:

- Preserve every distinct idea, observation, caveat, hedge, tangent, aside, and self-correction.
- Preserve the user's voice, terminology, phrasing, and idioms.
- Permit only these operations:
  - Split a dump into multiple topic-grouped notes.
  - Add headers and structure.
  - Group adjacent or related thoughts under shared headings.
  - Fix obvious typos and broken markdown.
  - Add wikilinks to existing concept pages.
  - Add YAML frontmatter.

Claude **must not**:

- Summarize, condense, or paraphrase for brevity.
- "Tighten" or "polish" prose — the dump's roughness is feature, not bug.
- Drop tangents, caveats, hedges, half-formed thoughts, contradictions, or self-corrections.
- Editorialize, add interpretation, or insert content the user did not write.
- Merge ideas the user kept distinct, or split ideas the user kept together within one thought.
- Translate informal phrasing into formal phrasing.

**Test:** if you removed any words because they felt redundant, rambling, or unpolished — that is a violation. Move them, don't delete them.

## The flow

When the user says "process the inbox" (or "process dumps", "go through dumps", or similar):

1. **List** all files in `raw/dumps/` whose frontmatter does **not** contain a `processed:` field. (The README and any file with `processed:` already set are skipped.) If none, say so and stop.

2. **For each unprocessed dump file**, in chronological order by filename:

   a. Read the full content.

   b. Identify distinct topical threads. One dump may contain one thread or several. Err on the side of fewer splits — only split when topics are clearly disjoint.

   c. For each thread, decide destination:
      - **Technical/research thinking** (robotics, tactile sensing, ML, control, etc.) → append to existing `20-concepts/<concept>.md` under a `## Notes — YYYY-MM-DD` section, or create a new concept page (with `status: stub` if it needs more development).
      - **Project-specific** (PhD applications, The Seventh Bar, apartment, immigration, resume, etc.) → append to existing `40-projects/<project>.md` under a `## Notes — YYYY-MM-DD` section, or create the project page if it doesn't exist.
      - **Personal reflection / planning / life logistics** that doesn't fit a concept or project → most often this still belongs on a project page (e.g., a thought about apartment logistics → `40-projects/apartment-beaumont.md`). If it genuinely fits no project, append to today's `10-daily/YYYY-MM-DD.md` under `## Captured`.

   d. Write the organized output, applying only the permitted operations from the lossless contract.

   e. Add frontmatter `source: raw/dumps/<original-filename>` to every organized note created or updated, so each piece of content traces back to its dump.

   f. **Mark the original processed**: add a `processed: YYYY-MM-DD` field to the dump's frontmatter (creating the frontmatter block if absent). Do **not** edit the dump's content. The file stays in `raw/dumps/`.

3. **Verify** coverage: for each dump, confirm that every distinct point from the original appears somewhere in the organized output. If a sentence in the original doesn't appear in any organized note, that is a bug — fix before moving on.

4. **Report** to the user: which dumps were processed, where each thread went, any ambiguities or judgment calls made, and any thread you weren't sure how to route (so the user can intervene).

5. **Commit** per dump: `agent-edit: process dump <YYYY-MM-DD-HHMM>`. Each commit should touch (a) the new/updated organized notes and (b) the dump file (frontmatter-only change).

## Resolving ambiguity

- If a dump's destination is unclear (e.g., is this a personal reflection or a project note?) — **ask the user before processing**. Don't guess.
- If a dump references a concept that doesn't yet have a page, create a stub per `wiki-playbook.md` conventions, but the dump's content goes verbatim into the user-thoughts section under `## Notes — YYYY-MM-DD`, **not** paraphrased into a "Definition" section. The canonical sections (Definition / Intuition / Mathematical statement / Why it matters) stay empty until the user develops them.
- If two dumps overlap heavily — process them as separate dumps anyway. Don't merge across dumps; the user wrote them at different times for a reason.

## Filename conventions

- Dumps in `raw/dumps/`: any filename works, but recommended `YYYY-MM-DD-HHMM-<optional-slug>.md`. Slug is optional; the user shouldn't have to think about naming when dumping.
- The dump file is **never renamed** by Claude after creation.

## On automation

This routine runs when the user asks. There is no schedule. The user is encouraged to trigger it at natural cadences — end of week, when the inbox feels full, when starting a focused work session — but the trigger is always manual.

If a dump arrives that clearly belongs in the wiki proper as a polished concept explanation, Claude may suggest promoting that *content* through the wiki-playbook flow after the dumps-playbook flow finishes. But the default is: dumps go through the dumps-playbook flow first, period.

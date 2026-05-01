# templates/

**Templater plugin templates.** Files here are inserted into new notes by Obsidian's Templater plugin, which substitutes `<% ... %>` expressions at insertion time.

## Current templates

- `daily.md.md` — daily note (used by `10-daily/`).
- `concept.md` — concept page (used by `20-concepts/`).
- `paper.md.md` — paper review (used by `30-papers/`).

## Why some files end in `.md.md`

Templater convention: the template file is itself a `.md` file (so Obsidian shows it), and when applied to create a new note, the resulting note is also `.md`. The double extension is a quirk of how the plugin names templates — keep it as-is on existing templates.

## Templater syntax in this vault

Used variables:
- `<% tp.date.now("YYYY-MM-DD") %>` — current date in ISO format.
- `<% tp.date.now("dddd, MMMM Do YYYY") %>` — current date in long format.
- `<% tp.file.title %>` — the new file's title.

## Adding a new template

1. Create a new `.md` (or `.md.md`) in this folder.
2. Use Templater syntax for any dynamic substitution.
3. Configure Obsidian's Templater plugin to recognize this folder (likely already configured).
4. Use Cmd/Ctrl-P → "Templater: Insert template" to apply.

## Suggested templates to add later

When the workflow demands them:
- `project.md` — for `40-projects/` (would include the suggested project structure from that folder's README).
- `transcript.md` — for `raw/transcripts/`.
- `course.md` — for `50-courses/`.

## Frontmatter fields used across templates

Standard fields (preserved by all agent edits per `wiki-playbook.md`):

| Field | Used by | Purpose |
|---|---|---|
| `type` | all | `daily`, `concept`, `paper`, `project`, `transcript`, `handwriting` |
| `tags` | concept | semantic tags |
| `date` | daily | date of the daily note |
| `status` | concept, project | `stub`, `active`, `done`, etc. |
| `last-updated` | concept | date of last edit |
| `source` | concept (when from dumps) | links back to `raw/dumps/<file>.md` |
| `processed` | dumps | date the dump was processed |
| `archived` | archived files | date archived |

# 90-archive/

**Deprecated content.** When something in the wiki is no longer accurate, no longer useful, or has been superseded — it moves here, not into the trash.

## The rule (from wiki-playbook.md)

> Never delete content. Move deprecated content to `90-archive/` with a stub left behind explaining where it went.

## Why archive instead of delete

- You may want to reference *what you used to think* later.
- A deprecated note may still have backlinks pointing to it; deletion breaks them.
- Sometimes you discover the deprecation was wrong and want to restore.

## How to archive

1. Move the file from its current location to `90-archive/<original-path>/<filename>.md`. Mirror the original folder structure inside `90-archive/` so the provenance is clear.
2. Add to its frontmatter:
   ```yaml
   archived: YYYY-MM-DD
   archived-reason: superseded-by-X | obsolete | merged-into-Y
   ```
3. In the original location, leave a stub file with the same name:
   ```markdown
   ---
   type: stub-redirect
   archived: YYYY-MM-DD
   ---

   # <original title>

   Archived to `90-archive/<path>/<filename>.md`. Reason: <reason>.

   Replaced by: [[new note]] (if applicable).
   ```

This preserves backlinks — anyone clicking `[[old note]]` lands on the redirect stub and follows it.

## Example

Suppose `20-concepts/old-slip-model.md` was your initial Mindlin-based formulation, but you've since adopted a new framing.

- Move file → `90-archive/20-concepts/old-slip-model.md`, add `archived: 2026-05-01` to frontmatter.
- Leave stub at `20-concepts/old-slip-model.md` pointing to `[[mindlin partial slip]]` (the new canonical page).

## What does NOT belong here

- Files you've decided are junk and never want again — those can be deleted (this is fine; not everything needs to be archived).
- Drafts that are just rough — those stay where they are with `status: stub` or `status: draft`.

Archiving is for content that *was* canonical and no longer is. Not for trash, not for in-progress work.

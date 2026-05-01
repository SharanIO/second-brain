# assets/

**Binary attachments referenced from notes** — images, diagrams, PDFs that are part of a note's content rather than a standalone source.

## What goes here

- Diagrams and figures embedded in concept pages (e.g., `slip-detection-diagram.png`).
- Screenshots used in project notes.
- Charts you generated from your own data.
- Logos, icons, or other graphic elements.

## What does NOT go here

- **External paper PDFs** → `raw/papers/`.
- **External book PDFs** → `raw/books/`.
- **Code that produces these assets** → `code/`.
- **One-off attachments not yet linked from anywhere** → `00-inbox/`.

The distinguishing question: is this asset *referenced from a note*? If yes, it belongs here. If it's a standalone source you're integrating, it belongs in `raw/`.

## Naming convention

`<descriptor>-<context>.<ext>`. E.g., `slip-detection-pipeline-arch.png`, not `Screen Shot 2026-05-01 at 3.42.13 PM.png`. Rename on intake.

## Linking from notes

Obsidian: `![[image.png]]` for inline embeds, `[[image.png]]` for a link.

## Optional subfolders

If `assets/` gets large, organize by note or project:

```
assets/
├── slip-detection/
│   ├── pipeline-arch.png
│   └── deformation-tensor-fig.svg
├── apartment/
│   └── floor-plan.png
└── general/
    └── ...
```

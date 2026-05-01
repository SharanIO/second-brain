# code/

**Code implementations linked to notes.** From `.cursorrules`:

> When implementing papers, place code in `code/` subfolder with a clear README linking back to the relevant paper note.

## Structure

One subfolder per implementation or project:

```
code/
├── slip-detection-pipeline/
│   ├── README.md         ← links to [[slip detection]] and relevant paper notes
│   ├── deformation.py
│   ├── tracker.py
│   └── tests/
├── orca3-actuator-logger/
│   ├── README.md         ← links to [[40-projects/orca3-characterization]]
│   └── ...
└── calandra2018-reproduction/
    ├── README.md         ← links to [[30-papers/calandra2018-more-than-feeling]]
    └── ...
```

## README contents per subfolder

Each implementation folder's README should answer:

1. **What is this?** One-line purpose.
2. **What note(s) does this implement / support?** Link via `[[paper note]]` or `[[concept page]]` or `[[project page]]`.
3. **How to run.** Setup, dependencies, entry point.
4. **Status.** Working / WIP / abandoned.

Example:

```markdown
# slip-detection-pipeline

Implementation of the deformation-gradient → contact-mechanical-features →
incipient-slip-prediction pipeline described in [[slip detection]].

Related papers: [[calandra2018-more-than-feeling]], [[dong2017-gelslim]].

## Run

```bash
python -m slip_detection.run --sensor-config configs/gelslim.yaml
```

## Status

Working on `np.gradient` deformation tensor; Delaunay triangulation perf fix
applied 2026-04-22.
```

## What does NOT go here

- Standalone scripts unrelated to wiki content → put them in their own repo.
- Notebooks for one-off exploration → can live here in a `scratch/` subfolder, but ideally promote findings to a concept or project note.

## Git

This folder is part of the vault's git repo, so code commits ride along with note commits. If a code project gets large enough to warrant its own repo, factor it out and link from the note.

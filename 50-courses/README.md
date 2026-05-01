# 50-courses/

**Course notes** — one folder or page per course you're taking (online, university, self-study with structured curriculum).

Courses are a structured subset of projects: they have modules, assignments, and a fixed end. Separating them from `40-projects/` keeps the project list focused on open-ended initiatives.

## Suggested structure (single-file)

For lighter courses, one file is enough:

```markdown
---
type: course
status: active
started: YYYY-MM-DD
provider:
---

# Course name

## Goal

Why you're taking it. What you want at the end.

## Schedule

- Week 1: topic
- Week 2: topic
- ...

## Module notes

### Module 1
- key idea → [[concept]]

### Module 2
- ...

## Assignments
- [ ] HW1 — due YYYY-MM-DD
- [x] HW2 — done

## Reflection
```

## Suggested structure (folder per course)

For heavier courses, use a subfolder:

```
50-courses/
└── cs231n-spring2026/
    ├── README.md          ← overview, schedule, status
    ├── module-01.md
    ├── module-02.md
    ├── assignment-01.md
    └── final-project.md
```

## Concepts vs courses

When a course teaches a concept that already has (or deserves) a `20-concepts/` page, integrate the concept knowledge there — don't duplicate it in the course folder. The course page links out: `Module 4 covers [[backpropagation]] in depth.`

## Example courses you might track

- A robotics PhD prep course (e.g., a focused Berkeley CS287 self-study).
- An online ML course (Karpathy's zero-to-hero series, fast.ai, etc.).
- A drumming course or syllabus from your teacher at Bootstrap Music.

## When to use a course page vs a project page

- Has fixed modules / weekly cadence / external curriculum → `50-courses/`.
- Open-ended, no fixed structure → `40-projects/`.

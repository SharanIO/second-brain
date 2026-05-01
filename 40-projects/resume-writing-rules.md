---
type: concept
tags: [career, resume, writing, robotics]
last-updated: 2026-04-30
---

# Resume Writing Rules

Rules for writing and maintaining resume bullet points. Derived from studying how robotics engineers at Boston Dynamics, NASA JPL, CMU, and similar organizations write their work.

## The Formula

```
[Strong verb]  +  [specific artifact]  +  [technology]  +  [quantified result or concrete technical claim]
```

Every bullet must have a trajectory: something was built, and something changed because of it.

## Rules

- **15–25 words per bullet.** One line. Scannable.
- **Lead with what you built**, not the problem it solved.
- **Include technology naturally** — woven into the sentence, not bolded keyword lists detached from outcome.
- **End with what changed**: a metric, a precision, a time saved, a capability unlocked, a failure mode eliminated.
- **Jargon is fine** but must be grounded in a concrete outcome — never jargon alone.
- **No em dashes with explanations** ("X — which means Y"). If the first half needs explaining, rewrite it.
- **No passive voice.** No "responsible for", "involved in", "assisted with".
- **No multi-clause HOW explanations.** Focus on WHAT was built and WHAT changed.

## Weight-Adding Concepts

Use these where accurate — they signal real engineering, not coursework:

| Concept | Example |
|---|---|
| Precision / tolerance | `0.1N`, `sub-centimeter`, `sub-pixel`, `millimeter-accurate` |
| Speed / throughput | `60 Hz to 3 kHz`, `days to hours`, exact latency numbers |
| Safety-critical grounding | `derived from tissue damage models`, `within mechanical compliance limits` |
| Ownership signals | `custom`, `from scratch`, `replaced the default` |
| Production signals | `deployed`, `fine-tuned`, `cross-validated` |
| Durability signals | `across months of testing`, `across N experiments`, `in unstructured environments` |
| Sophistication signals | `model-free`, `real-time`, `multi-hypothesis`, `surface-agnostic` |

## Strong Action Verbs

Built · Engineered · Designed · Implemented · Deployed · Developed · Optimized · Rebuilt · Integrated · Extracted · Analyzed · Benchmarked · Replaced · Tracked · Standardized

Avoid: Assisted · Contributed to · Was responsible for · Helped · Worked on

## What Makes a Bullet Feel Like a Story vs. a Keyword List

**Keyword list (weak):**
> Implemented PRM and RRT* for motion planning in ROS and C++

**Story (strong):**
> Optimized PRM and RRT* motion planners in C++/ROS, reducing path computation time for real-time dynamic obstacle avoidance on a high-DOF manipulator.

The difference: the strong version has an artifact (the planners), a technology (C++/ROS), and an outcome (real-time obstacle avoidance). The weak version just names things.

## Goal

The bullet should make the reader curious enough to ask *"how did you do that?"* in an interview — not explain everything upfront. If you cannot say what you did in one line, you do not yet know what you did.

## Connections

- [[resume-meta-ai-prompt]] — prompt to run against internal Meta docs to extract metrics and strengthen bullets

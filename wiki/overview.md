---
type: overview
last-updated: 2026-05-13
---

# Vault Overview

This document is the executive briefing for the vault. It answers three questions: who owns this vault and what is their context, what knowledge actually lives here, and where the gaps are. It is written for Claude at session start and for Sharan when orienting after time away. It should be updated whenever the vault's coverage changes substantially.

---

## Who

**Sharan** (Sai Sharan Thirunagari) — robotics engineer working as a contractor at Meta Reality Labs via Unify Technologies. The work sits at the intersection of robotic tactile sensing, data infrastructure, and contact physics. Specifically: building the hardware and software systems that generate training data for robotic e-skin — tactile sensors that detect how objects grip and slip. Day-to-day work involves EtherCAT pipelines, force-controlled testbeds, optical flow deformation tracking, real-time C++ watchdogs, and multi-modal sensor fusion across NI-DAQ and force sensor streams.

Parallel tracks running simultaneously:
- **PhD applications** — targeting robotics programs with a focus on tactile sensing, dexterous manipulation, or contact-rich learning. No applications submitted yet as of May 2026; research and lab targeting are in early stages.
- **Music** — drums, runs a project called "The Seventh Bar."
- **Immigration** — H-1B sponsorship under active monitoring; contract extension at Meta is a related thread.
- **Resume rebuild** — active, with a structured prompt-based workflow for extracting internal metrics from Meta's systems.

Technical background: Python and C++ are primary languages; ROS 2 fluency; strong hardware-software integration experience. Currently learning ML/NLP deeply through coursework and self-study. See `.cursorrules` for the full original context block.

---

## What is in this vault

### NLP / Machine Learning — *most developed area*

The NLP concept layer is the most populated part of the wiki. All pages came from integrating a single source: ~130 pages of handwritten Advanced NLP course notes (OCR'd), which covered the full arc from statistical language models to RLHF alignment. The 12-chapter structured course notes in `wiki/courses/adv-nlp/` are the primary learning resource; the 15 concept pages in `wiki/concepts/` are the synthesized wiki layer derived from them.

**Concept pages (15):**
- Statistical: [[n-gram-models]]
- Representations: [[word-embeddings]]
- Training machinery: [[activation-functions]], [[weight-initialization]], [[dropout]], [[regularization]], [[optimizers]]
- Sequence modeling: [[recurrent-neural-networks]]
- Attention & architecture: [[attention-mechanism]], [[positional-encoding]], [[transformers]]
- Scale: [[tokenization]], [[pretraining-and-finetuning]], [[lora]]
- Alignment: [[rlhf]]

**Domain page:** [[nlp]] — maps how all 15 concepts relate to each other, traces the historical arc from n-grams to alignment, and documents what is missing.

**Course notes:** `wiki/courses/adv-nlp/` — 12 chapters, ~2,300 lines, covering the same arc in structured sequential form with PyTorch code, tensor shape walkthroughs, and optimizer internals.

**Source page:** [[adv-nlp-course-notes]] — synthesis note for the original course notes PDF, including structure, key insights, and critique.

**Papers not yet integrated:** `raw/papers/attention-is-all-you-need/` contains the full extracted markdown of Vaswani et al. (2017) "Attention Is All You Need." This is the foundational paper for everything in the transformers cluster. It has not yet been processed through the wiki-playbook.

---

### Robotics — *papers present, wiki layer empty*

Seven academic papers on dexterous manipulation and tactile sensing are in `raw/papers/` as extracted markdown files with associated figures. None of them have been processed through the wiki-playbook yet. There are no concept pages, no entity pages, and no domain page for robotics. This is the largest gap between what Sharan works on daily and what the vault actually knows.

**Papers available for integration:**

| Paper | Topic |
|---|---|
| Crossing the Human-Robot Embodiment Gap with Sim-to-Real RL | Sim-to-real transfer, imitation learning |
| Everyday Finger — A Robotic Finger for Everyday Manipulation | Dexterous manipulation hardware |
| Gentle Object Retraction in Dense Clutter (Multimodal Force Sensing + IL) | Force sensing, imitation learning |
| In-Hand Object Rotation via Rapid Motor Adaptation | Dexterous in-hand manipulation |
| MUSIC — Learning Muscle-Driven Dexterous Hand Control | Dexterous hand control, muscle actuation |
| SimToolReal — Object-Centric Policy for Zero-Shot Tool Manipulation | Zero-shot generalization, tool use |
| Where to Touch, How to Contact — Hierarchical RL MPC Framework | Contact planning, dexterous manipulation |

When these are integrated, the target concept pages to create would include: `tactile-sensing`, `slip-detection`, `actuator-characterization`, `sim-to-real`, `dexterous-manipulation`, `contact-rich-learning`, `imitation-learning`, `force-control`.

---

### Projects — *partially populated*

`wiki/projects/` tracks active work with sustained context. Currently:

**Resume rebuild** (`wiki/projects/resume/`) — the most developed project area. Contains:
- `resume-writing-rules.md` — the formula and rules for writing robotics engineer resume bullets: strong verb + specific artifact + technology + quantified result. Includes weight-adding concepts, strong action verbs, and examples of weak vs. strong bullets.
- `resume-meta-ai-prompt.md` — a structured 3-part prompt designed to run against Meta's internal AI systems to extract concrete metrics from design docs, code review history, metrics dashboards, and incident reports. Covers 8 work areas (EtherCAT pipeline, Python concurrent pipeline, multi-modal sensor fusion, slip characterization testbed, optical flow deformation, slip prediction analysis, C++ safety watchdog, sensor evaluation protocol) with specific questions about before/after numbers for each.

**Not yet created** (high priority):
- `phd-applications.md` — target schools, lab research, SOP status, deadlines, advisor contacts
- `the-seventh-bar.md` — music project: practice log, set ideas, recording plans
- `h1b-sponsorship.md` — petition status, timeline, AC21 considerations
- `slip-detection-pipeline.md` — research work at Meta: pipeline architecture, current state, open problems

---

### Guides — *operational reference*

`guides/git-cleanup-runbook.md` — step-by-step instructions for removing large binary files from git history using `git-filter-repo`. Written when NLP.pdf (~211 MB) was accidentally committed. Steps cover: backup, confirming history, installing git-filter-repo, running the rewrite, force-pushing, and Obsidian Git plugin reconfiguration.

---

## What is not in this vault

The gaps are extensive because the vault is new. The most important missing areas, ordered by priority:

**1. Robotics domain (highest priority)** — Sharan's daily work has zero wiki representation. Seven unprocessed papers exist in `raw/papers/`. Running `integrate this paper` on each would bootstrap the robotics concept and entity layers.

**2. Project pages for active life threads** — PhD applications, music, H-1B, and slip detection research all have no project pages. Any thought dumps about these topics have nowhere to land when processed.

**3. NLP gaps within the existing domain** — Several important NLP topics are missing even though the coverage is otherwise strong:
- Evaluation metrics (BLEU, ROUGE, BERTScore, perplexity)
- Dense retrieval and sentence transformers
- Inference optimization (quantization, speculative decoding, vLLM)
- Prompting techniques (CoT, RAG, few-shot)
- Multimodal models (CLIP, LLaVA) — especially relevant for robotics VLA models

**4. Entity pages** — No pages exist for any person (potential PhD advisors, collaborators, key researchers), any tool (GelSight, ROS 2, PyTorch, FAISS), or any organization (Meta Reality Labs, target PhD programs). Entity pages form the "who and what" layer of the wiki.

**5. Algorithms domain** — If LeetCode practice is starting, concept pages for algorithm patterns (dynamic programming, sliding window, binary search, graph traversal) will be needed. No pages exist yet.

**6. Comparisons** — No side-by-side comparison pages exist yet. The most natural ones to create: AdamW vs SGD, LoRA vs full finetuning, encoder-only vs decoder-only vs encoder-decoder, RoPE vs sinusoidal vs ALiBi.

---

## Active threads

As of May 2026:

| Thread | Status | Location |
|---|---|---|
| Vault restructure | Complete | This session |
| NLP concept layer | 15 pages, active | `wiki/concepts/` |
| Attention Is All You Need integration | Not started | `raw/papers/attention-is-all-you-need/` |
| Robotics paper integration (7 papers) | Not started | `raw/papers/` |
| PhD applications project page | Not started | — |
| LeetCode setup | Planned | `code/leetcode/`, `wiki/projects/leetcode.md` |
| Resume rebuild | Active | `wiki/projects/resume/` |

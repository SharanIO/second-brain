---
type: concept
tags: [career, resume, meta, prompt]
last-updated: 2026-04-30
---

# Resume — Meta Internal AI Prompt

A prompt to run against your internal Meta AI (Workplace AI or similar) with access to internal docs, metrics dashboards, design docs, and code history. The goal is to find concrete numbers and missing accomplishments that strengthen the Meta Reality Labs bullets on the resume.

Run this after any significant project milestone, before performance reviews, or whenever updating the resume.

## Connections

- [[resume-writing-rules]] — the formula and rules this prompt is built around

---

## THE PROMPT

> Paste everything below this line into your internal AI and give it access to your Workplace docs, internal wikis, code history, and metrics before running.

---

You are helping me improve the bullet points on my resume for my role as **Robotics Engineer at Meta Reality Labs** (March 2025 – Present). I have been in this role for over a year. My work sits at the intersection of robotic tactile sensing, data infrastructure, and contact physics — specifically building the hardware and software systems that generate training data for robotic e-skin, which are tactile sensors that detect how objects grip and slip.

I need you to do three things in sequence. Do not skip to Part 2 until you have completed Part 1.

---

### PART 1 — Search my internal record and extract every concrete metric you can find

Search through all resources available to you — including but not limited to:

- **Design docs and PRDs** I authored or was tagged in, related to: EtherCAT pipeline, data acquisition systems, sensor fusion, slip characterization, the RMUC cluster, tactile sensing, e-skin, optical flow deformation, safety watchdog, sensor evaluation protocols
- **Code review history** — diffs I submitted, reviewed, or was the primary author of; look for comments that mention performance numbers, latency, throughput, or before/after comparisons
- **Incident reports and oncall runbooks** where my systems were involved — especially any that reference the watchdog, pipeline failures, or sensor anomalies
- **Metrics dashboards and data quality reports** — any tracked metric for pipeline throughput, sync accuracy, sensor noise floor, prediction accuracy, or dataset size
- **Project wikis, team pages, and internal launch docs** for the e-skin or tactile sensing project that reference my contributions
- **Retrospectives, H1/H2 goal trackers, or launch reviews** — any document where my work was cited as a deliverable, dependency, or achievement
- **1:1 notes, perf prep documents, or self-assessments** I have written — these often contain accomplishment summaries in plain language
- **Task trackers or engineering tickets** I owned, closed, or was assigned — look for resolution notes, linked metrics, or impact statements
- **Slack threads or Workplace posts** where I shared results, shipped a feature, or presented findings to the team

For each of the following areas of my work, return the most specific quantitative facts you can find — exact numbers are always better than approximations:

1. **EtherCAT pipeline rebuild**
   - What was the exact throughput before and after the rebuild? (I recall 60 Hz → 3 kHz but verify)
   - What was the measured timing jitter before and after?
   - How many ML experiments or training runs have used data from this pipeline?
   - How large is the dataset it has produced (hours, GB, number of samples)?
   - Has any model been trained on this data and achieved a measurable result?

2. **Python concurrent data pipeline**
   - What is the exact sensor integration time before and after? (I recall "days to hours" — get the actual numbers)
   - How many distinct sensor types or data streams does it currently handle simultaneously?
   - What is the measured sync accuracy between streams?
   - Any uptime, reliability, or error rate metrics?
   - How many engineers or teams depend on this pipeline today?

3. **Multi-modal sensor fusion (NI-DAQ + force sensors)**
   - What is the measured timestamp alignment accuracy between digital and analog streams?
   - How many sensor modalities are currently fused?
   - Was there a before/after comparison on data usability or downstream analysis quality?

4. **Slip characterization testbed**
   - How many total experiments have been run on this testbed?
   - What range of applied loads, surface materials, or object types has been tested?
   - Have any design specifications been formally adopted based on its output?
   - Has it been referenced in any PRD, design review, or hardware spec as a source of truth?

5. **Optical flow deformation pipeline**
   - What spatial resolution does it achieve?
   - What is the slip speed range it covers robustly?
   - Has it been used to generate a labeled dataset? If so, how large?
   - What is the frame rate or latency of the pipeline end-to-end?

6. **Slip prediction analysis (MAD filtering, spatial deformation, FFT signatures)**
   - Is there a measured prediction accuracy, precision, recall, or F1 for slip prediction?
   - What is the lead time of the pre-slip signature — how many milliseconds before visible slip does it appear?
   - Has the frequency signature been validated on held-out hardware data or in simulation with a quantified match?
   - Has this analysis been cited in any paper, internal report, or design doc?

7. **Real-time C++ safety watchdog**
   - How many anomaly events has it detected and acted on?
   - What is its measured response latency in milliseconds?
   - Over what time period has "zero equipment damage" been achieved?
   - Has it been tested against injected fault conditions? If so, what was the detection rate?

8. **Sensor evaluation protocol**
   - How many sensors have been formally qualified using this protocol?
   - Has the protocol been adopted by the team or formalized in a shared doc?
   - What was the evaluation time before vs. after standardization?
   - Has it been used by engineers other than me?

---

### PART 2 — Rewrite the bullets using what you found

Once you have gathered the facts from Part 1, rewrite each of the 10 bullets below using the following formula and rules. Do not paraphrase the current bullets — improve them using the actual metrics you found.

**The formula:**
```
[Strong verb] + [specific artifact] + [technology] + [quantified result or concrete technical claim]
```

**Hard rules — every bullet must follow all of these:**

1. **One line, 15–25 words.** If it runs longer, cut methodology, not the outcome.
2. **Lead with the artifact** — what was built or done. Not the problem. Not the context. Not "in order to."
3. **End with a concrete outcome.** A number beats a description. A capability beats a process. Something must have changed.
4. **No em dashes followed by explanations.** The sentence must stand without an appended footnote.
5. **No passive voice.** Every bullet starts with an active verb that belongs to the engineer.
6. **Jargon must be grounded.** "EtherCAT" is fine. "EtherCAT for better data" is not. "EtherCAT pipeline scaled to 3 kHz" is fine.
7. **Avoid vague endings** like "enabling better results", "improving overall performance", "for use in downstream tasks." Be specific.

**Weight-adding phrases — use where accurate:**
- Exact numbers always beat ranges: `3 kHz` beats `higher frequency`
- Precision language: `sub-pixel`, `sub-centimeter`, `0.1N`, `millimeter-accurate`
- Ownership language: `custom`, `from scratch`, `replaced the default`
- Safety-grounded numbers: `derived from tissue damage models`, `within mechanical compliance limits`
- Real-world durability: `across X months of testing`, `across N experiments`
- Sophistication markers: `model-free`, `real-time`, `surface-agnostic`, `multi-hypothesis`

**Strong verbs to prefer:** Built · Rebuilt · Engineered · Designed · Implemented · Deployed · Optimized · Integrated · Extracted · Analyzed · Tracked · Replaced · Standardized · Benchmarked

**Current bullets (improve these — do not just rephrase):**

1. Built a force-controlled slip testbed using ATI Nano17 and Futek sensors, extracting contact signatures that defined hardware specifications for robotic e-skin design.
2. Rebuilt the EtherCAT data pipeline in C++ from 60 Hz to 3 kHz, eliminating timing jitter and producing ML-ready synchronized sensor data.
3. Designed a concurrent Python data pipeline with lock-free shared memory and async I/O, cutting new sensor integration time from days to hours.
4. Built a multi-modal fusion system aligning force sensors and NI-DAQ analog streams via hardware timing markers, enabling synchronized multi-sensor capture across the cluster.
5. Tracked sensor surface deformation at sub-pixel resolution using pyramidal optical flow in OpenCV, revealing contact behavior that force sensors alone cannot capture.
6. Applied median absolute deviation filtering to raw force trajectories, isolating the elastic stick-phase deformation signature that precedes slip.
7. Analyzed spatial strain fields to track contact area evolution as a model-free slip indicator, removing dependence on surface-specific friction models.
8. Extracted pre-slip frequency signatures from force data using FFT analysis, cross-validating experimental thresholds against simulation to bridge the real-to-sim gap.
9. Engineered a real-time C++ sliding-window anomaly detection watchdog, achieving zero equipment damage across months of high-force experimentation.
10. Standardized sensor evaluation protocols measuring minimum detectable force, noise floor, and temporal lag, establishing qualification criteria for novel tactile sensors prior to integration.

---

### PART 3 — Flag what I am underselling or missing entirely

After reviewing my internal record, return a short list covering:

**A. Bullets where the current claim is weaker than the record shows**
For example: I say "days to hours" but the actual numbers are "4 days to 90 minutes." List the bullet, what I said, and what the record shows.

**B. Contributions not captured in the current 10 bullets**
Any project, system, or tool I owned or co-owned that does not appear above. Prioritize things that:
- Other engineers depend on (platform-level impact)
- Were cited in a design review, PRD, or retrospective
- Were recognized by a manager or peer in writing
- Shipped to production or were used in a real experiment

**C. Impact that is broader than my immediate team**
Any work that was reused, referenced, or depended on by engineers outside my immediate team. Cross-team impact always reads better on a resume.

---

### OUTPUT FORMAT

Return your response in three clearly labeled sections:

**Section 1 — Metrics Table**
A table with columns: `Bullet #` | `Metric found` | `Source doc or location` | `Suggested replacement phrasing`

**Section 2 — Rewritten Bullets**
The full list of 10 rewritten bullets. For each one, note in parentheses what changed and why.

**Section 3 — Missing and Undersold Work**
Bullet-point list of flagged items from Part 3, organized by A / B / C above.

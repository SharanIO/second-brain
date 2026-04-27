---
title: RLHF (Reinforcement Learning from Human Feedback)
status: complete
last-updated: 2026-04-26
tags: [nlp, alignment, llm, rl]
---

# RLHF (Reinforcement Learning from Human Feedback)

A training pipeline for aligning LLMs with human intent — making them produce outputs that are helpful, harmless, and honest rather than just statistically likely continuations of text.

## Motivation: SFT limitations

Supervised Finetuning (SFT) on instruction data has key weaknesses:
1. Doesn't learn from **negative feedback** — only trains on good examples.
2. Many prompts have multiple acceptable outputs but many unacceptable ones; training on just one acceptable output is insufficient.
3. Hard to teach the model when to **abstain** (leading to hallucinations).

## Three-step pipeline

### Step 1: Supervised Finetuning (SFT)
Start from a pretrained LM. Finetune on a high-quality dataset of (instruction, response) pairs to produce a model capable of following instructions. See [[pretraining-and-finetuning]].

### Step 2: Reward Model Training
Collect **preference data**: for each prompt $x$, generate multiple responses and have human annotators rank them ($y_w \succ y_l$, meaning $y_w$ is preferred over $y_l$).

Train a reward model $r(x, y) \in \mathbb{R}$ using the **Bradley-Terry pairwise preference model**:

$$P(y_w \succ y_l \mid x) = \frac{\exp(r(x, y_w))}{\exp(r(x, y_w)) + \exp(r(x, y_l))}$$

Loss (negative log-likelihood):

$$L = -\log \sigma\!\left(r(x, y_w) - r(x, y_l)\right)$$

Intuition: the good sample $y_w$ should have higher reward than $y_l$. Initialize from the SFT model (it already knows language; the task is classification of quality).

Collecting human preference data is **extremely expensive**. Alternative: **RLAIF** (RL from AI Feedback / Constitutional AI) — use judgments from a capable LLM (e.g., GPT-4) instead.

### Step 3: RL Optimization (PPO)
Use the reward model as an environment signal to further optimize the SFT model with reinforcement learning.

**Best-of-n (rejection) sampling**: generate $n$ responses, score each with the reward model, keep the best. Simple but slow.

**RAFT** (Reward rAnked Fine Tuning): gather best-of-n samples in bulk; use the best responses to finetune the LM. Cannot learn from negative responses.

**PPO** (Proximal Policy Optimization): the standard RL approach. Distributes the reward signal across all tokens in a sequence so the model can learn which specific tokens made the response better or worse. Can learn from both positive and negative samples.

Challenge: the reward is observed only at the end of a full generated sequence (sparse reward), making credit assignment hard.

## Key hyperparameters / design choices

- KL penalty against the reference SFT model (prevents reward hacking / degenerate outputs)
- Reward model size — should be at least as large as the policy
- Number of comparison labels per prompt

## Sources

- [[adv-nlp-course-notes]]

## See also

- [[pretraining-and-finetuning]]
- [[lora]]
- [[transformers]]

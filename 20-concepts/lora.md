---
title: LoRA (Low-Rank Adaptation)
status: complete
last-updated: 2026-04-26
tags: [nlp, peft, finetuning, llm]
---

# LoRA (Low-Rank Adaptation)

A parameter-efficient finetuning method from Microsoft Research. Instead of updating the full weight matrix $W \in \mathbb{R}^{m \times n}$ during finetuning, LoRA decomposes the update $\Delta W$ into two low-rank matrices:

$$\Delta W = AB^\top, \quad A \in \mathbb{R}^{m \times r},\ B \in \mathbb{R}^{n \times r}$$

where $r \ll \min(m, n)$ is the **rank** hyperparameter.

The modified forward pass:

$$h = f\!\left((W_{\text{pre}} + AB^\top)x\right)$$

During finetuning, $W_{\text{pre}}$ is frozen; only $A$ and $B$ are updated.

## Why it works

Neural network weight matrices tend to have low intrinsic rank — the meaningful update $\Delta W$ during task adaptation lies in a much lower-dimensional subspace than the full matrix. LoRA exploits this.

## Parameter savings

Full finetuning requires updating $mn$ parameters. LoRA updates $r(m + n)$ parameters.

$$\text{savings} = \frac{mn}{r(m+n)}$$

For a $4096 \times 4096$ matrix with $r = 8$: from ~16M parameters down to ~65k.

## Hyperparameter: rank $r$

Typical values: $r \in \{4, 8, 16, 64\}$. Higher rank = more expressive but more parameters. $r = 8$ is a common default.

## Where LoRA is applied

In transformers, LoRA is typically applied to:
- $W_Q, W_K, W_V$ (attention projection matrices) per head
- Occasionally the feed-forward projection layers

Not usually applied to embedding matrices or bias terms.

## QLoRA

Quantized LoRA. Reduces GPU memory further by quantizing the frozen pretrained weights to 4-bit or 8-bit integers before applying LoRA adapters on top.

- FP32 → INT8 or INT4 via quantization (some precision loss)
- Libraries: `bitsandbytes`, `unsloth`
- Enables finetuning of very large models (70B+) on consumer GPUs

## Limitation

LoRA adapters are task-specific. Serving multiple tasks simultaneously (batched inference with different adapters) is non-trivial — a known bottleneck in production deployment.

## Sources

- [[adv-nlp-course-notes]]

## See also

- [[pretraining-and-finetuning]]
- [[rlhf]]
- [[optimizers]]

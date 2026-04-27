---
title: "Chapter 12: Finetuning, PEFT, and Alignment"
course: Advanced NLP
chapter: 12
last-updated: 2026-04-26
tags: [nlp, finetuning, lora, rlhf, alignment, peft]
---

# Chapter 12: Finetuning, PEFT, and Alignment

## Learning objectives

By the end of this chapter you should be able to:
- Describe full finetuning and its memory cost
- Explain prompt tuning and prefix tuning as PEFT methods
- Explain LoRA: the rank decomposition idea, its memory savings, and where it is applied
- Describe RLHF: why SFT alone is insufficient, how the reward model is trained, and how PPO uses it
- Explain QLoRA and why it is needed for very large models

---

## 12.1 From pretraining to usefulness

A pretrained language model is a remarkable artifact: it has read trillions of words and has compressed knowledge of grammar, facts, reasoning patterns, and style into its weights. But it is not yet *useful* in the way a product like ChatGPT is useful.

Directly prompting a pretrained (but not finetuned) model:

> **Prompt**: "What is the capital of France?"
> **Completion**: "What is the capital of Germany? What is the capital of Spain? What is the capital of..."

The model is trying to continue the text in a plausible way — it has seen lots of list-of-questions in its training data and it imitates that pattern. It does not know it should answer the question.

**Finetuning** is the process of adapting a pretrained model's behavior for a specific task or for instruction-following in general. There is a spectrum of methods, from expensive but thorough to cheap but limited.

---

## 12.2 Full finetuning: update everything

The simplest approach: take the pretrained model and continue training it on a smaller labeled dataset, updating all parameters.

**Advantages**: maximally expressive — the entire model can adapt to the new task.

**Disadvantages**:
- **Cost**: training a 70B-parameter model requires hundreds of GPUs and days of compute.
- **Storage**: you need a separate full copy of the model's weights for every task you want to support.
- **Catastrophic forgetting**: if finetuned aggressively on a small dataset, the model can "forget" its general capabilities, becoming good at the specific task but losing its broader knowledge.

For small models (< 1B parameters), full finetuning is practical. For large models, it is prohibitive.

---

## 12.3 Parameter-efficient finetuning (PEFT)

The core insight of PEFT: **most of the adaptation needed for a specific task lies in a small subspace of the parameter space**. We do not need to update all parameters; we can freeze most of the pretrained model and update only a small number of new parameters.

### Prompt tuning

The simplest PEFT method: prepend a small number of learned "virtual" token embeddings to the input sequence. Only these embeddings are updated during finetuning; the rest of the model is frozen.

```
Input:  [<v1>] [<v2>] [<v3>] [actual] [input] [tokens]
          ↑       ↑       ↑
       learned virtual tokens (updated)    (frozen)
```

The virtual tokens `<v1>, <v2>, <v3>` are not in the vocabulary — they are free-floating vectors initialized randomly and updated by gradient descent. The model learns to interpret them as "you are now in task X mode".

**Memory savings**: instead of updating billions of parameters, we update only $k \times d_{model}$ parameters (where $k$ is the number of virtual tokens, typically 5–100). Sweet spot is $k \approx 5$–10.

**Limitation**: works best with very large models (> 10B). For smaller models, full finetuning consistently outperforms prompt tuning. Also: the virtual tokens cannot provide per-layer guidance — they are only added at the input.

### Prefix tuning

An extension of prompt tuning: prepend learned virtual tokens to the keys and values in **every layer's attention**, not just the input embeddings. This allows the "soft prompt" to influence the model's representations at every depth.

```
Layer k attention: K = [<prefix_k>; K_real],   V = [<prefix_v>; K_real]
                        ↑                              ↑
                  learned (updated)              pretrained (frozen)
```

Prefix tuning is more expressive than prompt tuning but uses more memory (one prefix per layer per task).

---

## 12.4 LoRA: Low-Rank Adaptation

LoRA (Hu et al., 2021, Microsoft Research) is the dominant PEFT method for large language models. It is based on one key observation:

**The weight updates during finetuning have low intrinsic rank.** When you finetune a model for a specific task, the change in the weight matrices $\Delta W = W_{finetuned} - W_{pretrained}$ tends to lie in a much lower-dimensional subspace than the full $m \times n$ matrix.

### The idea

Instead of learning $\Delta W \in \mathbb{R}^{m \times n}$ directly (too expensive), factorize it into two small matrices:

$$\Delta W = A B^\top, \quad A \in \mathbb{R}^{m \times r},\ B \in \mathbb{R}^{n \times r}$$

where $r \ll \min(m, n)$ is the **rank** — a hyperparameter you choose.

The modified forward pass becomes:

$$\mathbf{h} = W_{\text{pretrained}} \mathbf{x} + \Delta W \mathbf{x} = W_{\text{pretrained}} \mathbf{x} + A B^\top \mathbf{x}$$

During finetuning, $W_{\text{pretrained}}$ is frozen. Only $A$ and $B$ are updated.

### Memory savings

| Method | Parameters updated |
|---|---|
| Full finetuning | $m \times n$ |
| LoRA (rank $r$) | $r(m + n)$ |
| Savings | $\frac{mn}{r(m+n)} \approx \frac{\min(m,n)}{2r}$ |

For a $4096 \times 4096$ weight matrix with $r = 8$: from 16.7M parameters down to 65k — a **256× reduction**.

### Initialization

$A$ is initialized from a Gaussian distribution; $B$ is initialized to zero. This ensures $\Delta W = AB^\top = 0$ at the start — the LoRA adapter starts as a no-op, and the pretrained model's behavior is exactly preserved at the beginning of finetuning.

### Where LoRA is applied

In practice, LoRA is applied to the attention projection matrices $W_Q, W_K, W_V, W_O$ and sometimes the FFN matrices. It is not typically applied to embeddings or layer norm parameters.

The rank $r$ is the critical hyperparameter. Typical values: 4, 8, 16, 64. Larger rank = more expressive but more parameters. $r = 8$ is a common default.

### After finetuning: merging

Once finetuning is complete, we can merge $\Delta W$ back into the pretrained weights:

$$W_{\text{final}} = W_{\text{pretrained}} + AB^\top$$

This produces a single weight matrix — no runtime overhead compared to the original model. The LoRA adapter is invisible at inference time.

### Limitation: batched multi-task inference

If you want to serve multiple different LoRA adapters simultaneously (different adapters for different tasks in the same server), inference becomes complex. Each request in the batch might need a different adapter applied to the same base model. This is an active area of research.

---

## 12.5 QLoRA: finetuning 70B+ models on a single GPU

LoRA reduces the number of trainable parameters. But during training, we still need to store the frozen pretrained weights in GPU memory — and a 70B-parameter model in FP32 needs 280 GB of VRAM. That is more than a cluster of A100 GPUs combined.

**QLoRA** (Dettmers et al., 2023) solves this by **quantizing** the frozen pretrained weights to 4-bit integers before applying LoRA on top.

### Quantization basics

Quantization converts floating-point weights to lower-precision integers:
- FP32: 4 bytes per parameter (32 bits)
- FP16: 2 bytes per parameter (16 bits) — standard for training
- INT8: 1 byte per parameter (8 bits) — 4× compression vs FP32
- INT4: 0.5 bytes per parameter (4 bits) — 8× compression vs FP32

A 70B model in INT4 fits in ~35 GB — a single A100 80GB.

The tradeoff: quantization introduces rounding error. QLoRA uses **NF4** (Normal Float 4-bit), a 4-bit data type designed to minimize error for normally distributed weights (which neural network weights approximately are).

### How QLoRA works

1. Load the pretrained model weights in NF4 (4-bit quantized).
2. During the forward pass, dequantize to BF16 for the computation (just-in-time).
3. Compute gradients through the LoRA adapter ($A$ and $B$, stored in FP32).
4. Update only $A$ and $B$.

The frozen model weights are never updated and never stored in full precision — only the small LoRA adapter is trained in full precision.

QLoRA enabled finetuning Llama 65B on a single 48 GB GPU, democratizing LLM finetuning significantly. Libraries: `bitsandbytes`, `unsloth`, `peft`.

---

## 12.6 Supervised finetuning (SFT) and instruction tuning

Before RLHF, the standard approach for making models follow instructions is **supervised finetuning (SFT)**: collect a dataset of (instruction, response) pairs written by humans, and finetune the model on them with a standard cross-entropy loss.

**Example training pair**:

> **Instruction**: "Explain gradient descent to a 10-year-old."
> **Response**: "Imagine you're lost in the mountains in the dark and you want to get to the valley below..."

The model learns to recognize instruction-response format and to produce helpful outputs. This is often called **instruction tuning**.

**Limitations of SFT alone**:
1. The model only sees positive examples — there are no negative training signals. It cannot learn "this is an unhelpful or harmful response" vs "this is a good one".
2. For a given prompt, there are many acceptable responses and many unacceptable ones. SFT trains on just one example per prompt.
3. It does not teach the model to refuse harmful requests or to express uncertainty. The model may confidently generate plausible-sounding but false information (**hallucination**).

---

## 12.7 RLHF: learning from human preferences

**Reinforcement Learning from Human Feedback** (RLHF) addresses SFT's limitations by explicitly training the model to produce outputs that humans prefer.

### The three-step pipeline

**Step 1: Supervised Finetuning (SFT)**
Start with a pretrained model. Finetune it on a curated instruction dataset to get a model capable of following instructions. This is the starting point, not the end.

**Step 2: Train a reward model**
The reward model $r_\phi(x, y)$ takes a prompt $x$ and a response $y$ and outputs a scalar reward — a measure of how good the response is.

To train it:
1. For each prompt $x$, use the SFT model to generate several different responses.
2. Have human annotators rank the responses: $y_w \succ y_l$ (response $y_w$ is preferred over $y_l$).
3. Train the reward model to predict this preference using the **Bradley-Terry model**:

$$P(y_w \succ y_l \mid x) = \sigma\!\left(r_\phi(x, y_w) - r_\phi(x, y_l)\right) = \frac{e^{r_\phi(x, y_w)}}{e^{r_\phi(x, y_w)} + e^{r_\phi(x, y_l)}}$$

Loss:
$$L_{RM} = -\log \sigma\!\left(r_\phi(x, y_w) - r_\phi(x, y_l)\right)$$

Intuition: the model should score preferred responses higher than dispreferred ones. The loss penalizes it whenever it gets this wrong.

**Step 3: RL fine-tuning with PPO**
Use the reward model as a signal to further optimize the language model (now called the **policy**) via reinforcement learning.

The policy generates a response $y$ to a prompt $x$. The reward model scores it: $r = r_\phi(x, y)$. The RL algorithm (PPO — Proximal Policy Optimization) updates the policy to increase the probability of responses that get high reward.

A critical addition: a **KL divergence penalty** against the original SFT model prevents the policy from "reward hacking" — finding degenerate responses that trick the reward model while being useless to humans.

$$\text{Objective} = \mathbb{E}\!\left[r_\phi(x, y)\right] - \beta \cdot \text{KL}\!\left[\pi_{\text{RL}} \| \pi_{\text{SFT}}\right]$$

### Alternatives to human feedback

Collecting human preference data is extremely expensive. Alternatives:

**RLAIF** (RL from AI Feedback): replace human annotators with a capable AI (e.g., GPT-4). The AI ranks responses according to a set of principles. Used in Anthropic's Constitutional AI.

**DPO** (Direct Preference Optimization): a mathematically equivalent alternative to PPO that skips the explicit reward model. Given preference pairs $(y_w, y_l)$, DPO directly optimizes the policy with a classification-style loss. Simpler to implement, more stable to train. Has become the practical standard over PPO for most applications.

---

## Summary

- **Full finetuning** updates all parameters — most expressive but expensive; one model copy per task.
- **Prompt tuning**: learn a few soft tokens prepended to the input; only those tokens are updated.
- **Prefix tuning**: extend soft prompts to every attention layer for more per-layer control.
- **LoRA**: freeze the pretrained weights; learn low-rank decomposition $AB^\top$ of the weight update. Typical savings: 100–1000× in trainable parameters.
- **QLoRA**: quantize frozen weights to 4-bit (NF4) + LoRA on top. Enables finetuning 70B models on a single GPU.
- **SFT** (instruction tuning): finetuning on (instruction, response) pairs. Teaches instruction-following but cannot learn from negative feedback.
- **RLHF**: three-stage pipeline (SFT → reward model → PPO) that trains on human preference judgments. Produces models that are helpful, harmless, and honest.
- **DPO**: a simpler alternative to RLHF that directly optimizes on preference pairs without an explicit reward model.

**Previous**: [[11-tokenization|Chapter 11 — Tokenization]]

---

*End of course. You now have the conceptual foundation to read current NLP research papers, understand model documentation, and experiment with LLMs at a technical level.*

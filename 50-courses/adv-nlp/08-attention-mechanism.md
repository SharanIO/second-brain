---
title: "Chapter 8: The Attention Mechanism"
course: Advanced NLP
chapter: 8
last-updated: 2026-04-26
tags: [nlp, attention, transformers, self-attention]
---

# Chapter 8: The Attention Mechanism

## Learning objectives

By the end of this chapter you should be able to:
- Explain the intuition behind attention using the Query-Key-Value analogy
- Write out the scaled dot-product attention formula and explain each component
- Describe self-attention and how it differs from cross-attention
- Explain why self-attention is parallelizable during training but sequential at inference
- Describe what the KV-cache is and why it matters for efficient inference
- Explain what masking is and why different transformer variants use different masks

---

## 8.1 The core insight

Recall the seq-to-seq bottleneck from Chapter 7: we forced the encoder to compress an entire sentence into a single fixed-size vector. The decoder then tried to reconstruct the full output from this compressed representation — losing information along the way.

The insight that breaks this bottleneck: **instead of asking the encoder to summarize everything upfront, let the decoder look at all encoder hidden states simultaneously and learn which ones to attend to at each decoding step** [Bahdanau et al., 2015].

More concretely: when generating the English word "fish" in the translation of "Les chats mangent du poisson", the decoder should attend strongly to the encoder's representation of "poisson" and weakly to "Les" and "chats". The model learns this attention distribution from data — it does not need to be programmed by a human.

This is the core idea. Now let us formalize it.

---

## 8.2 Query, Key, Value: the formal framework

Attention is cleanly described using three matrices derived from the input: **Queries**, **Keys**, and **Values**.

The analogy to a database or search engine is exact:
- When you search Google, the text you type is your **Query**.
- The indexed page titles are the **Keys** — they describe what each document is about.
- The document contents are the **Values** — the actual information you retrieve.
- Relevance is determined by matching your Query to the Keys.

In a neural attention layer, for each token in the sequence:

$$\mathbf{q} = W_Q \mathbf{x}, \quad \mathbf{k} = W_K \mathbf{x}, \quad \mathbf{v} = W_V \mathbf{x}$$

where $W_Q, W_K, W_V \in \mathbb{R}^{d_k \times d_{model}}$ are learned projection matrices. Every token is simultaneously a Query (asking about others), a Key (answering others' queries), and a Value (the information it contributes when attended to).

---

## 8.3 Scaled dot-product attention

Given a set of Queries $Q \in \mathbb{R}^{T \times d_k}$, Keys $K \in \mathbb{R}^{T \times d_k}$, and Values $V \in \mathbb{R}^{T \times d_v}$ (one row per token), the attention output is:

$$\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right) V \quad \text{[Vaswani et al., 2017]}$$

Let us unpack this step by step.

**Step 1 — Compute scores**: $QK^\top \in \mathbb{R}^{T \times T}$. Entry $(i, j)$ is the dot product between query $i$ and key $j$ — a scalar measuring how relevant token $j$ is to token $i$. A high score means token $i$ should pay a lot of attention to token $j$.

**Step 2 — Scale**: divide by $\sqrt{d_k}$. Without this, dot products in high dimensions become very large, pushing softmax into regions where its gradient is nearly zero. The scale factor keeps dot products in a reasonable range.

**Step 3 — Softmax**: convert each row of scores into a probability distribution. Now entry $(i, j)$ is the attention weight — how much of token $j$'s value should be included in token $i$'s output. Each row sums to 1.

**Step 4 — Weighted sum of Values**: multiply by $V$. For each token $i$, the output is a weighted sum of all value vectors, where the weights are the attention probabilities. Tokens that are highly relevant contribute more.

The result is an output matrix of shape $[T \times d_v]$ — one output vector per token, enriched by attending to all other tokens.

---

## 8.4 Self-attention

When $Q$, $K$, and $V$ are all derived from the **same** sequence, the operation is called **self-attention**. Each token looks at all other tokens in the same sequence and asks: *"which of these tokens should I be paying attention to?"*

This is a profound capability. In the sentence *"The animal didn't cross the street because it was too tired"*, the pronoun "it" can attend to "animal" — which is 8 tokens back — with a high weight, resolving the coreference. No recurrence is needed. The attention weight from "it" to "animal" is learned from training data and emerges naturally.

Compare this to an RNN, where this information would need to survive 8 sequential hidden state updates, with the vanishing gradient problem threatening to erase it.

---

## 8.5 Parallelism: why transformers train fast

One of the most important practical advantages of self-attention over RNNs is **parallelism during training**.

In an RNN, hidden state $\mathbf{h}_t$ cannot be computed until $\mathbf{h}_{t-1}$ is ready. The computation is fundamentally sequential — a chain of dependencies that cannot be broken.

In self-attention, the output for every token depends on the keys and values of all other tokens — but **the entire computation can be expressed as matrix multiplications**:

$$\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right) V$$

All token outputs are computed simultaneously. On modern GPUs, which are optimized for large matrix operations, this means a 512-token sequence can be processed as a single operation rather than 512 sequential steps. This is the primary reason transformers train so much faster than RNNs.

> [!warning] Inference is still sequential for generation
> At training time, the full sequence is available and self-attention is fully parallel. At inference time, when generating text token-by-token, each new token depends on all previous tokens — so generation is still sequential. However, thanks to the **KV-cache** (Section 8.7), we avoid recomputing keys and values for the prefix at each step.

---

## 8.6 Masking: controlling what each token can see

The attention score matrix $QK^\top$ allows every token to attend to every other token by default. But sometimes we need to restrict this:

### Causal mask (for language modeling)

When training a language model, we must not allow the model to "see the future" — if token $t$ could attend to tokens $t+1, t+2, \ldots$, the model would simply copy the correct answer rather than learning to predict it.

A **causal mask** sets all scores in the upper triangle of $QK^\top$ to $-\infty$ before the softmax, forcing those attention weights to zero:

$$\text{score}(i, j) = \begin{cases} q_i \cdot k_j & \text{if } j \leq i \\ -\infty & \text{if } j > i \end{cases}$$

After softmax, a score of $-\infty$ becomes a weight of exactly 0. Token $i$ can only attend to tokens at positions $\leq i$.

This is why GPT-style models use a "lower triangular" attention pattern — each position sees only itself and its past.

### Padding mask

In a batch of sequences, shorter sequences are padded with a special `[PAD]` token. The padding mask prevents real tokens from attending to pad positions (and vice versa).

### Local/sliding window mask

A variant where each token can only attend to the $w$ most recent tokens (window size $w$). This reduces the quadratic memory cost of full self-attention and is used in very long-context models.

---

## 8.7 The KV-cache: efficient inference

During text generation, we generate token by token. At step $t$, we have the prefix $[w_1, \ldots, w_{t-1}]$ and want to generate $w_t$.

Without any optimization, we would need to recompute the keys $K$ and values $V$ for the entire prefix at every generation step — even though $[w_1, \ldots, w_{t-2}]$ were computed at the previous step and have not changed.

The **KV-cache** stores the key and value vectors computed at each step and reuses them. At step $t$, we only compute $\mathbf{k}_t$ and $\mathbf{v}_t$ for the new token, then append them to the cached $K$ and $V$ matrices. The attention for the new token attends to all cached keys and values.

This reduces the per-step computational cost from $O(t^2)$ to $O(t)$ and is essential for making large language models fast enough to use in practice. The tradeoff: the KV-cache grows linearly with context length and consumes significant GPU memory.

---

## 8.8 Cross-attention

When the Queries come from one sequence (the decoder) and the Keys and Values come from a different sequence (the encoder), the operation is called **cross-attention**.

$$\text{CrossAttn}(Q_{dec}, K_{enc}, V_{enc})$$

This is the mechanism that connects the encoder and decoder in seq-to-seq models like T5. The decoder's query at each step asks: *"which part of the encoded source sequence is most relevant right now?"* The encoder's keys and values provide the database to look up.

Cross-attention completely replaces the bottleneck of the original seq-to-seq architecture: instead of one vector, the decoder has access to the full set of encoder representations at every decoding step.

---

## Summary

- Attention allows each token to compute a **weighted sum over all other tokens' values**, where the weights are determined by query-key similarity.
- The formula: $\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right) V$. Scaling by $\sqrt{d_k}$ prevents gradient saturation.
- **Self-attention**: all of Q, K, V come from the same sequence. Enables direct long-range dependencies.
- **Self-attention is fully parallel during training** (matrix ops); sequential during autoregressive inference.
- **KV-cache**: store computed keys/values so we only compute new tokens at inference time.
- **Causal masking** prevents tokens from attending to future positions during language model training.
- **Cross-attention**: Q from decoder, K and V from encoder — breaks the seq-to-seq bottleneck.

**Previous**: [[07-recurrent-neural-networks|Chapter 7 — Recurrent Neural Networks]]
**Next**: [[09-transformers|Chapter 9 — The Transformer Architecture]]

---

## References

- **ADV NLP Course Notes** — source for the QKV formulation, self-attention parallelism analysis, KV-cache description, and masking. See [[adv-nlp-course-notes]].
- Bahdanau, D., Cho, K., & Bengio, Y. (2015). Neural Machine Translation by Jointly Learning to Align and Translate. *ICLR*. — the original attention mechanism for seq-to-seq; broke the bottleneck.
- Vaswani, A. et al. (2017). Attention is All You Need. *NeurIPS*. — introduced scaled dot-product attention and multi-head attention; the Transformer paper.
- Luong, M.-T., Pham, H., & Manning, C. D. (2015). Effective Approaches to Attention-based Neural Machine Translation. *EMNLP*. — simplified attention variants (dot-product, general, concat).
- Dao, T. et al. (2022). FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness. *NeurIPS*. — efficient implementation of self-attention critical for long contexts.

> [!todo] cite this — add a reference for the KV-cache as a specific named technique; it is standard practice but was not introduced in a single named paper.

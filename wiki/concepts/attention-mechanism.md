---
title: Attention Mechanism
status: complete
last-updated: 2026-04-26
tags: [nlp, deep-learning, transformers, attention]
---

# Attention Mechanism

Allows a model to weight which parts of its input are most relevant when producing each output. Introduced for seq-to-seq RNNs to break the "bottleneck" problem; later became the core of [[transformers]] (Vaswani et al., 2017), which dropped recurrence entirely.

## Motivation: the RNN bottleneck

In [[recurrent-neural-networks|seq-to-seq RNNs]], the encoder compresses an entire source sentence into a single fixed vector. Ran Mooney (2014): *"you can't represent the meaning of a sentence in a [single] vector."* Attention solves this by letting the decoder look back at all encoder hidden states, weighted by relevance.

## Query, Key, Value (QKV)

Each token is projected into three vectors:

- **Query** ($Q = f(W_Q G)$): "what am I looking for?"
- **Key** ($K = f(W_K G)$): "what do I advertise about myself?"
- **Value** ($V = f(W_V G)$): "what information do I provide if selected?"

Analogy: searching Google — the typed query is $Q$, the article titles are $K$, and the article content is $V$. Relevance is cosine similarity between $Q$ and $K$.

## Scaled dot-product attention

$$\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right) V$$

The $\sqrt{d_k}$ scaling prevents dot products from growing large in high dimensions, which would push softmax into saturated (near-zero gradient) regions.

## Self-attention

Each token attends to all other tokens in the *same* sequence. This lets the model capture long-range dependencies without the sequential bottleneck of RNNs.

### Parallelism

| Phase | Parallelizable? | Why |
|---|---|---|
| Training | Yes | Full sequence available; compute all attention scores simultaneously |
| Inference | No | Autoregressive generation — each token depends on all previous |

**KV-cache**: at inference time, store previously computed $K$ and $V$ vectors so they don't need to be recomputed at each decoding step.

## Multi-head attention

Instead of a single attention operation over $d_{\text{model}}$-dimensional vectors, project Q, K, V into $h$ lower-dimensional subspaces and run attention in parallel. Each head has its own learned projections:

$$\text{MultiHead}(Q,K,V) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h) W^O$$
$$\text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, KW_i^V)$$

In the original Transformer: $h = 8$ heads, $d_k = d_v = d_{\text{model}}/h = 64$. Total compute equals single-head attention at $d = 512$ — but each head independently specializes in different aspects of the sequence (syntax, coreference, phrase structure). Ablations show single-head is 0.9 BLEU worse; too many heads (32) also degrades quality.

### Three application contexts in the Transformer

1. **Encoder self-attention**: Q, K, V all from the encoder's previous layer. Bidirectional — every position attends to every other.
2. **Decoder masked self-attention**: causal mask sets future positions to $-\infty$ before softmax, so position $i$ only attends to positions $\leq i$. Required for autoregressive generation.
3. **Encoder-decoder cross-attention**: Q from the decoder's previous layer; K, V from the encoder's final output. The only path the decoder uses to "read" the source.

### Efficiency vs. recurrence

| Layer type | Complexity per layer | Sequential ops | Max path length |
|---|---|---|---|
| Self-attention | $O(n^2 \cdot d)$ | $O(1)$ | $O(1)$ |
| Recurrent | $O(n \cdot d^2)$ | $O(n)$ | $O(n)$ |
| Convolutional | $O(k \cdot n \cdot d^2)$ | $O(1)$ | $O(\log_k n)$ |

Self-attention is faster than recurrence when $n < d$ (typical in NLP). The $O(1)$ path length is the decisive advantage — any two positions interact in one step regardless of distance.

## Masking

- **Causal mask** (decoder): prevents each position from attending to future tokens. Required for autoregressive LMs to prevent "cheating."
- **Padding mask**: ignores padding tokens in batched sequences.
- **Local mask**: restrict attention to the $n$ most recent tokens (sliding window attention).

## Filter intuition

Attention acts as a learned filter — it removes irrelevant information and focuses on what matters for the current prediction.

## Sources

- [[adv-nlp-course-notes]]
- [[attention-is-all-you-need]] (Vaswani et al. 2017)

## See also

- [[transformers]]
- [[recurrent-neural-networks]]
- [[word-embeddings]]
- [[positional-encoding]]

---
title: Word Embeddings
status: complete
last-updated: 2026-04-26
tags: [nlp, representations, deep-learning]
---

# Word Embeddings

Dense, low-dimensional vector representations of words learned during training. They replace the sparse [[n-gram-models|one-hot vectors]] of classical LMs.

## Motivation

One-hot vectors have two key flaws: they are huge ($|V|$-dimensional) and they treat all words as equally dissimilar (all dot products are zero). Embeddings fix both:

- **Size**: much lower dimension than $|V|$ (e.g., 512 or 768)
- **Similarity**: semantically similar words end up close in embedding space

The trade-off: each dimension has no interpretable meaning. The right dimensionality is found empirically.

## Mechanics

A weight matrix $E \in \mathbb{R}^{|V| \times d}$ maps a token id (integer index) to a $d$-dimensional vector. This matrix is a learned parameter updated during training.

In a transformer, token embeddings are combined with [[positional-encoding]] before being fed to attention layers:

$$x_i = E[w_i] + \text{pos}(i)$$

## Role in transformers

In a transformer processing a 3-token sequence with $d=512$ and 8 heads:

| Step | Shape | What happened |
|---|---|---|
| Input IDs | [3] | token indices |
| Embedding | [3 × 512] | each token → 512-dim vector |
| + Positional encoding | [3 × 512] | order info added |
| Split into heads | [8, 3, 64] | 8 heads × 64 dims each |

Weight decay is rarely applied to word embeddings (unlike attention weight matrices).

## Sources

- [[adv-nlp-course-notes]]

## See also

- [[n-gram-models]] — the one-hot predecessor
- [[attention-mechanism]] — uses embeddings as Q, K, V inputs
- [[transformers]]
- [[positional-encoding]]

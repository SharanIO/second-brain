---
title: Positional Encoding
status: stub
last-updated: 2026-04-26
tags: [nlp, transformers, architecture]
---

# Positional Encoding

[[attention-mechanism|Self-attention]] is permutation-invariant: it produces the same output regardless of the order of the input tokens. Positional encodings inject position information into token representations so the model knows which token comes first.

## Approaches

**Learned positional embeddings**: a lookup table $P \in \mathbb{R}^{L_{\max} \times d}$ maps each position index to a $d$-dimensional vector. These are model parameters, updated during training. Limitation: the model has no representation for positions beyond $L_{\max}$.

**Sinusoidal (deterministic)**: Vaswani et al. 2017 used fixed sine and cosine functions of different frequencies. No learned parameters; generalizes better to longer sequences.

**Relative position encodings (RoPE, ALiBi)**: encode the *distance* between tokens rather than absolute positions. Improves length generalization.

## Where applied

Positional encodings are added to token embeddings at the input to each (or only the first) transformer layer:

$$x_i = E[w_i] + \text{pos}(i)$$

They share the same dimensionality $d$ as the embeddings. The position embeddings are model parameters (for learned variants) but are typically not subject to weight decay.

The number of learnable position embeddings equals the maximum sequence length the model was trained on.

## Sources

- [[adv-nlp-course-notes]]

## See also

- [[attention-mechanism]]
- [[transformers]]
- [[word-embeddings]]

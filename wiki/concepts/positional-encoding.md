---
title: Positional Encoding
status: active
last-updated: 2026-05-13
tags: [nlp, transformers, architecture]
---

# Positional Encoding

Positional encoding is the mechanism by which a transformer is made aware of the order of tokens in a sequence. Without it, the model would be completely permutation-invariant — "the dog bit the man" and "the man bit the dog" would produce identical representations for every token, since [[attention-mechanism|self-attention]] operates purely on set membership and weighted similarity, not position.

This is a fundamental problem unique to transformers. [[recurrent-neural-networks|RNNs]] have positional awareness baked in by construction — the hidden state $h_t$ is computed from $h_{t-1}$, so earlier tokens are literally processed before later ones. Transformers process all tokens in parallel and therefore must have order information injected explicitly.

---

## Why self-attention is permutation-invariant

In self-attention, every token is mapped to a query $Q$, key $K$, and value $V$ vector. The attention output for token $i$ is:

$$\text{Attention}(Q, K, V)_i = \sum_j \frac{\exp(Q_i \cdot K_j / \sqrt{d_k})}{\sum_{j'} \exp(Q_i \cdot K_{j'} / \sqrt{d_k})} V_j$$

Notice that the computation for token $i$ involves dot products with all $K_j$ and a weighted sum of all $V_j$. There is no reference to position $i$ or $j$ anywhere in the formula. If you shuffle the input sequence, each token still attends to the same set of tokens with the same weights — only the output order changes to match the shuffled input. The *relationships* computed by attention are entirely position-agnostic.

This means without positional encoding, the transformer has no way to distinguish "subject verb object" from "object verb subject."

---

## Approach 1: Sinusoidal (Vaswani et al. 2017)

The original transformer paper used fixed, deterministic positional encodings based on sine and cosine functions of different frequencies. The encoding for position $pos$ and dimension $i$ is:

$$PE_{(pos, 2i)} = \sin\!\left(\frac{pos}{10000^{2i/d}}\right)$$

$$PE_{(pos, 2i+1)} = \cos\!\left(\frac{pos}{10000^{2i/d}}\right)$$

where $d$ is the model dimension and $i$ ranges from $0$ to $d/2 - 1$. The encoding for a given position is a vector of length $d$ where even dimensions use sine and odd dimensions use cosine.

**The key intuition** is that different dimensions oscillate at different frequencies. Low-indexed dimensions ($i$ near 0) oscillate quickly — they change significantly even between adjacent positions, encoding fine-grained local order. High-indexed dimensions ($i$ near $d/2$) oscillate very slowly — they barely change between adjacent positions but change substantially over long ranges, encoding coarse global position. Together, the full $d$-dimensional vector gives each position a unique fingerprint, analogous to a binary number where each bit position has a different frequency.

**Why this works at inference time beyond training lengths**: the sine and cosine functions are defined for all real numbers, so the model can compute a positional encoding for position 5000 even if it only trained on sequences up to length 512. Whether the model can *use* that representation is a different question (it often struggles due to distributional shift), but the encoding itself is well-defined. This is the main advantage over learned positional embeddings.

**Why 10000?**: the base 10000 is a design choice that spaces the frequencies so that adjacent positions are distinguishable across all $d$ dimensions without any dimension oscillating so slowly that it is effectively constant.

The sinusoidal encoding is added to the token embedding before the first transformer layer:

$$x_i = E[w_i] + PE_i$$

Both terms have dimension $d$, so the sum is directly usable as the transformer input.

---

## Approach 2: Learned Positional Embeddings

Instead of a fixed function, learned positional embeddings are a matrix $P \in \mathbb{R}^{L_{\max} \times d}$ where row $i$ is a learned $d$-dimensional vector for position $i$. This matrix is a model parameter updated by backpropagation alongside the token embedding matrix. The input representation is identical to sinusoidal:

$$x_i = E[w_i] + P[i]$$

**Advantages**: the model can learn the positional representation that works best for its architecture and data distribution, rather than being constrained to a fixed sinusoidal pattern.

**Disadvantages**: the model has no representation for positions beyond $L_{\max}$. At inference, if the sequence is longer than the training context window, the model simply has no positional encoding to use. This is a hard ceiling on context length. GPT-2 used learned positional embeddings with $L_{\max} = 1024$; early BERT used $L_{\max} = 512$.

---

## Approach 3: Relative Positional Encodings

Absolute positional encodings assign each position a fixed vector independent of what else is in the sequence. Relative positional encodings instead encode the *relationship* between two positions — specifically, the distance $i - j$ between token $i$ and token $j$ — and incorporate this into the attention score.

The key insight is that what often matters for language is not "this token is at position 5" but rather "this token is 2 positions after that one." The same grammatical relationship (subject → verb, modifier → noun) tends to hold regardless of where in the document it appears.

### RoPE (Rotary Position Embedding)

RoPE, introduced by Su et al. (2021) and now standard in Llama, Mistral, Falcon, and most modern LLMs, encodes position by rotating the query and key vectors in 2D subspaces before the dot product is computed. The rotation angle for a given position and dimension pair is:

$$\theta_{pos, i} = pos \cdot 10000^{-2i/d}$$

The query and key vectors are split into pairs of dimensions, and each pair is rotated by $\theta_{pos}$:

$$\begin{pmatrix} q_{2i}' \\ q_{2i+1}' \end{pmatrix} = \begin{pmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{pmatrix} \begin{pmatrix} q_{2i} \\ q_{2i+1} \end{pmatrix}$$

The crucial property of RoPE is that when you compute the dot product $Q_m \cdot K_n$ after applying these rotations, the result depends only on $m - n$ (the relative distance), not on $m$ and $n$ individually. This is because the rotation matrices satisfy $R_m^\top R_n = R_{m-n}$. The attention score automatically encodes relative position.

**Why RoPE enables longer context generalization**: because the model only ever sees relative distances during training, there is no hard cutoff at $L_{\max}$. The representations for distance-10, distance-100, and distance-1000 are all extrapolations of the same rotation scheme. In practice, RoPE generalizes to 2–4× the training context length with minimal degradation, and with techniques like YaRN (Yet another RoPE extensioN) or rope scaling, models can be extended to much longer contexts.

### ALiBi (Attention with Linear Biases)

ALiBi (Press et al., 2021) takes a simpler approach: instead of modifying the query and key representations, it adds a fixed linear bias to the attention scores before the softmax:

$$\text{Attention score}(i, j) = Q_i \cdot K_j - m \cdot |i - j|$$

where $m$ is a head-specific slope that decreases for higher-numbered attention heads. The effect is that tokens attending to distant positions pay a linear penalty, and this penalty is steeper for some heads than others. Crucially, there are no additional learned parameters. ALiBi was designed to generalize at inference to contexts longer than seen during training, and it works well for this purpose, though RoPE has largely superseded it in large models.

---

## Where positional encoding is applied in the transformer

For absolute encodings (sinusoidal, learned), the encoding is added once to the token embeddings at the very beginning of the network, before the first attention layer:

$$\text{Input to layer 1} = E[w_i] + \text{pos}(i)$$

For relative encodings (RoPE, ALiBi), the position information is applied inside each attention layer, modifying the Q and K vectors or the attention scores directly. This means every layer is aware of relative position, not just the first.

The position embeddings (for learned variants) are parameters like any others, but they are typically **not** subject to weight decay — the same convention as token embeddings.

---

## Summary comparison

| Method | Parameters | Context length | Generalizes beyond training | Used by |
|---|---|---|---|---|
| Sinusoidal (absolute) | None | Unlimited | Poorly | Original Transformer |
| Learned (absolute) | $L_{\max} \times d$ | Hard cap at $L_{\max}$ | No | BERT, GPT-2 |
| RoPE (relative) | None | Unlimited | Well | Llama, Mistral, Falcon |
| ALiBi (relative) | None | Unlimited | Well | MPT, BLOOM |

---

## Sources

- [[adv-nlp-course-notes]]
- Vaswani et al. (2017) — "Attention Is All You Need" — [[attention-is-all-you-need]]
- Su et al. (2021) — "RoFormer: Enhanced Transformer with Rotary Position Embedding"
- Press et al. (2021) — "Train Short, Test Long: Attention with Linear Biases Enables Input Length Extrapolation"

## See also

- [[attention-mechanism]]
- [[transformers]]
- [[word-embeddings]]

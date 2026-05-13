---
title: Recurrent Neural Networks (RNNs)
status: complete
last-updated: 2026-04-26
tags: [nlp, deep-learning, sequence-models]
---

# Recurrent Neural Networks (RNNs)

A class of neural networks designed for sequential data. Each timestep takes the current input and the previous hidden state to produce a new hidden state:

$$h_t = f(W_h h_{t-1} + W_e e_t)$$

where $e_t$ is the word embedding at position $t$ and $h_0$ is initialized to zero.

## Advantages over n-gram models

- Can process sequences of any length
- Model size is $O(n)$, not $O(e^n)$
- "Memory" of previous words is encoded in the hidden state
- Weights are **shared across all time steps** — the same $W_h$ is used at every position

## Limitations

1. **Bottleneck**: the entire sequence meaning must be compressed into a fixed-size hidden vector for seq-to-seq tasks. Ran Mooney (2014): *"you can't represent the meaning of a sentence in a [single] vector."*
2. **Recency bias**: more recent words have stronger influence on the hidden state.
3. **Vanishing gradients**: gradients from many steps back shrink toward zero; the network struggles to learn long-range dependencies.
4. **Sequential computation**: $h_t$ cannot be computed until $h_{t-1}$ is done — no parallelism across timesteps.

## Training objective

At each timestep $t$, predict the next word. Loss is averaged across all positions:

$$L = \frac{1}{T}\sum_{t=1}^{T} -\log P(w_t \mid w_{<t})$$

## Variants

- **LSTM** (Long Short-Term Memory): uses gating mechanisms to selectively remember/forget information; mitigates vanishing gradients.
- **GRU** (Gated Recurrent Unit): simplified gating; fewer parameters than LSTM.
- **Attention-based RNN**: augments the decoder with an attention mechanism over encoder hidden states, breaking the bottleneck.

## Historical note

The seq-to-seq bottleneck and the lack of parallelism motivated the [[attention-mechanism]], which Vaswani et al. (2017) then used to build the [[transformers|Transformer]] — dropping recurrence entirely.

## Sources

- [[adv-nlp-course-notes]]

## See also

- [[attention-mechanism]]
- [[transformers]]
- [[n-gram-models]]

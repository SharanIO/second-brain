---
title: "Chapter 7: Recurrent Neural Networks"
course: Advanced NLP
chapter: 7
last-updated: 2026-04-26
tags: [nlp, rnn, sequence-models, lstm]
---

# Chapter 7: Recurrent Neural Networks

## Learning objectives

By the end of this chapter you should be able to:
- Explain the core idea of an RNN and why it differs from an MLP for sequences
- Write out the RNN recurrence equation and describe what each term means
- Describe the vanishing gradient problem and why it limits long-range memory
- Explain the seq-to-seq architecture and its bottleneck limitation
- Describe how LSTMs address the vanishing gradient problem at a high level

---

## 7.1 The sequence problem

The neural language model from Chapter 4 takes a fixed-size input and produces a fixed-size output. But language is not fixed-length. A sentence might be 5 words or 500 words. More fundamentally, the meaning of word at position $t$ depends on what came before it — often on context that is many positions back.

We need a neural architecture that:
1. Can process sequences of **any length**
2. Has some notion of **memory** — what has been seen so far
3. Uses the **same parameters** for every position (otherwise the number of parameters scales with sequence length)

Recurrent neural networks provide all three.

---

## 7.2 The core idea: hidden state as memory

An RNN processes a sequence one step at a time. At each step $t$, it takes:
- The current input $\mathbf{x}_t$ (the embedding of the current word)
- The previous **hidden state** $\mathbf{h}_{t-1}$ (the network's "memory" of what it has seen so far)

And produces:
- A new hidden state $\mathbf{h}_t$ (updated memory)
- Optionally, an output $\mathbf{y}_t$ (e.g. a prediction of the next word)

The recurrence equation:

$$\mathbf{h}_t = f(W_h \mathbf{h}_{t-1} + W_e \mathbf{x}_t + \mathbf{b})$$

where $W_h$ is the **recurrent weight matrix** (connects hidden state to hidden state), $W_e$ is the **input weight matrix** (connects input to hidden state), and $f$ is a nonlinear activation (typically tanh).

Three things to notice:
1. The same weights $W_h$, $W_e$, and $\mathbf{b}$ are used at every timestep. This is called **weight sharing** and is what makes RNNs scalable to sequences of any length.
2. The hidden state $\mathbf{h}_t$ is a compressed summary of everything seen up to position $t$.
3. $\mathbf{h}_0$ is initialized to zeros (the model starts with an "empty memory").

For language modeling, the output at each step is a prediction of the next word:

$$\mathbf{y}_t = \text{softmax}(W_o \mathbf{h}_t)$$

The loss is the average cross-entropy across all timesteps:

$$L = -\frac{1}{T} \sum_{t=1}^{T} \log P(w_t \mid w_{<t})$$

---

## 7.3 Advantages over n-gram models and fixed-window MLPs

RNNs improve on both previous approaches:

| Property | N-gram | Fixed-window MLP | RNN |
|---|---|---|---|
| Handles arbitrary length? | Yes | No (fixed window) | Yes |
| Model size scales with length? | $O(e^n)$ | $O(k \cdot d)$ fixed | $O(d^2)$ fixed |
| Shares parameters across positions? | No | No | Yes |
| Can access info from far back? | No (limited by $n$) | No (limited by window) | Theoretically yes |

The theoretical "unlimited memory" is the key promise of RNNs. The hidden state is a learned, compressed representation of the entire prefix.

---

## 7.4 The vanishing gradient problem

Unfortunately, the "unlimited memory" promise breaks down in practice. Training an RNN requires computing gradients not just through the current step, but backward through all previous steps — **backpropagation through time** (BPTT).

When the gradient flows backward from step $T$ to step $1$, it is multiplied by $W_h$ at each step. If the largest singular value of $W_h$ is less than 1, the gradient shrinks exponentially:

$$\frac{\partial L}{\partial \mathbf{h}_1} = \left(\frac{\partial L}{\partial \mathbf{h}_T}\right) \cdot W_h^{T-1}$$

After 10–20 steps, the gradient is essentially zero. The optimizer receives no useful signal about how parameters at early steps affect the final loss. This is the **vanishing gradient problem**.

The practical consequence: RNNs struggle to learn dependencies that span more than ~10–20 words. For the sentence *"The cat that the dog chased finally caught the bird"*, the RNN may fail to connect "cat" to the "caught" at the end.

The reverse is also possible: if the singular values of $W_h$ are greater than 1, the gradient **explodes** exponentially. This is the **exploding gradient problem**, which is addressed by gradient clipping (Chapter 5).

---

## 7.5 The seq-to-seq architecture and the bottleneck

Many NLP tasks require mapping a variable-length input sequence to a variable-length output sequence — for example, machine translation. This gave rise to the **encoder-decoder** or **seq-to-seq** architecture [Sutskever et al., 2014; Cho et al., 2014].

**Encoder**: an RNN reads the source sequence word by word and produces a final hidden state $\mathbf{h}_T$ — a single fixed-size vector that is supposed to capture the entire meaning of the source sentence.

**Decoder**: a separate RNN is initialized with $\mathbf{h}_T$ as its starting hidden state and generates the target sequence word by word.

```
Source: "Les chats mangent du poisson"
   ↓ encoder RNN
h_T = [0.2, -0.7, 0.4, ...]  ← entire sentence compressed here
   ↓ decoder RNN
Target: "Cats eat fish"
```

This architecture works surprisingly well for short sequences, but it has a fundamental flaw: **the bottleneck**. The entire meaning of the source sentence — every word, every grammatical relationship — must be crammed into a single fixed-size vector $\mathbf{h}_T$. For long sentences, this is simply too much information for one vector to hold.

Ran Mooney (2014) famously put it: *"You can't represent the meaning of a sentence in a [single] vector."* [attributed in ADV NLP Course Notes; original source unverified — > [!todo] locate primary citation]

The bottleneck manifests as degraded translation quality as sentence length increases — a well-documented empirical finding in early NLP systems.

---

## 7.6 LSTMs and GRUs: fighting vanishing gradients

Two architectural improvements were developed to partially address the vanishing gradient problem.

### LSTM (Long Short-Term Memory)

An LSTM augments the basic RNN with a **cell state** [Hochreiter & Schmidhuber, 1997] $\mathbf{c}_t$ — a "highway" for information to flow across many timesteps with minimal transformation — and three **gates** that control what information is written to, read from, or forgotten from the cell state.

- **Forget gate**: decides what to erase from the cell state: $\mathbf{f}_t = \sigma(W_f [\mathbf{h}_{t-1}, \mathbf{x}_t])$
- **Input gate**: decides what new information to write: $\mathbf{i}_t = \sigma(W_i [\mathbf{h}_{t-1}, \mathbf{x}_t])$
- **Output gate**: decides what to read from the cell state: $\mathbf{o}_t = \sigma(W_o [\mathbf{h}_{t-1}, \mathbf{x}_t])$

The cell state update:
$$\mathbf{c}_t = \mathbf{f}_t \odot \mathbf{c}_{t-1} + \mathbf{i}_t \odot \tilde{\mathbf{c}}_t$$

Because the forget gate can be close to 1 (keep everything), information from early steps can flow directly to late steps with little attenuation, reducing the vanishing gradient problem.

### GRU (Gated Recurrent Unit) [Cho et al., 2014]

A simplified alternative with two gates instead of three. Fewer parameters than LSTM, comparable performance in practice.

> [!note] LSTMs are still limited
> LSTMs extend the effective memory range from ~10 to perhaps ~50–100 timesteps, but they do not solve the fundamental problem. For truly long-range dependencies — a pronoun referring to an antecedent 200 words earlier — LSTMs still struggle. And they are inherently sequential: $\mathbf{h}_t$ cannot be computed until $\mathbf{h}_{t-1}$ is ready, which means no parallelism across timesteps.

---

## 7.7 The motivation for attention

By 2014–2015, it was clear that the seq-to-seq bottleneck was a critical limitation. The solution was to let the decoder look at *all* the encoder hidden states, not just the last one, and learn to select which ones are relevant at each decoding step. This is the **attention mechanism**.

Rather than: *"compress the entire source into one vector, then decode"*

We ask: *"at each decoding step, which parts of the source sequence should I focus on?"*

This insight — that the decoder should be allowed to selectively access all encoder states — is the foundation of the attention mechanism we study in the next chapter. And when researchers generalized this idea to allow *every* position to attend to *every other* position, removing the recurrence entirely, the Transformer was born.

---

## Summary

- An RNN processes sequences with a **hidden state** that carries memory from one step to the next: $\mathbf{h}_t = f(W_h \mathbf{h}_{t-1} + W_e \mathbf{x}_t)$.
- The same weights are used at every timestep (**weight sharing**), making RNNs parameter-efficient and length-agnostic.
- The **vanishing gradient problem** means RNNs cannot effectively learn dependencies spanning more than ~20 words.
- The **seq-to-seq bottleneck**: compressing an entire sentence into one vector loses too much information for long sentences.
- **LSTMs** and **GRUs** use gating mechanisms to partially mitigate vanishing gradients, but remain fundamentally sequential.
- The bottleneck limitation directly motivated the **attention mechanism**, which we study next.

**Previous**: [[06-building-blocks|Chapter 6 — Building Blocks of Modern Neural Networks]]
**Next**: [[08-attention-mechanism|Chapter 8 — The Attention Mechanism]]

---

## References

- **ADV NLP Course Notes** — source for the RNN formulation, seq-to-seq bottleneck discussion, and the Ran Mooney quote. See [[adv-nlp-course-notes]].
- Elman, J. L. (1990). Finding Structure in Time. *Cognitive Science*, 14(2). — foundational RNN paper.
- Hochreiter, S. & Schmidhuber, J. (1997). Long Short-Term Memory. *Neural Computation*, 9(8). — LSTM; the solution to vanishing gradients in RNNs.
- Bengio, Y., Simard, P., & Frasconi, P. (1994). Learning Long-Term Dependencies with Gradient Descent is Difficult. *IEEE Transactions on Neural Networks*. — formal analysis of the vanishing gradient problem.
- Cho, K. et al. (2014). Learning Phrase Representations using RNN Encoder–Decoder for Statistical Machine Translation. *EMNLP*. — GRU and the encoder-decoder architecture.
- Sutskever, I., Vinyals, O., & Le, Q. V. (2014). Sequence to Sequence Learning with Neural Networks. *NeurIPS*. — seq-to-seq with LSTM; established the encoder-decoder paradigm.
- Pascanu, R., Mikolov, T., & Bengio, Y. (2013). On the difficulty of training recurrent neural networks. *ICML*. — gradient explosion and clipping in RNNs.

> [!todo] cite this — locate the primary source for the Ran Mooney (2014) "can't represent meaning in a single vector" quote; the ADV NLP notes attribute it but do not provide a paper title.

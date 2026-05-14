---
title: "Attention Is All You Need"
type: source
source: raw/papers/attention-is-all-you-need/attention-is-all-you-need.md
status: integrated
last-updated: 2026-05-13
tags: [nlp, transformers, attention, architecture]
authors: [Vaswani, Shazeer, Parmar, Uszkoreit, Jones, Gomez, Kaiser, Polosukhin]
year: 2017
venue: NeurIPS
---

# Attention Is All You Need — Vaswani et al. (2017)

The paper that introduced the Transformer architecture. The core claim: you can build a competitive sequence transduction model using attention alone — no recurrence, no convolution. At the time, recurrent models (LSTM, GRU) were the unquestioned standard for sequence modeling. This paper replaced them entirely.

---

## What problem it solves

[[recurrent-neural-networks|RNNs]] are fundamentally sequential: computing hidden state $h_t$ requires $h_{t-1}$. This prevents parallelization across time steps during training, which is the main bottleneck at long sequence lengths. Several attempts had tried to reduce sequential computation using convolutional architectures (ByteNet, ConvS2S), but those required stacks of layers to capture long-range dependencies — $O(n/k)$ layers for contiguous convolutions, $O(\log_k n)$ for dilated — meaning the path length between distant tokens remained long.

The Transformer collapses path length to $O(1)$: any two positions can exchange information in a single attention operation, regardless of distance in the sequence.

---

## Architecture

### Overview

The Transformer is an encoder-decoder model. The encoder maps an input sequence $(x_1, \ldots, x_n)$ to continuous representations $z = (z_1, \ldots, z_n)$. The decoder generates output $(y_1, \ldots, y_m)$ autoregressively, one token at a time, consuming its own previous output.

### Base model hyperparameters

| Parameter | Value |
|---|---|
| Layers $N$ | 6 (encoder) + 6 (decoder) |
| Model dimension $d_{\text{model}}$ | 512 |
| Feed-forward dimension $d_{\text{ff}}$ | 2048 |
| Attention heads $h$ | 8 |
| Key/value dimension $d_k = d_v$ | 64 (= 512/8) |
| Dropout $P_{\text{drop}}$ | 0.1 |
| Label smoothing $\varepsilon_{ls}$ | 0.1 |
| Training steps | 100K (base), 300K (big) |
| Parameters | 65M (base), 213M (big) |

### Scaled dot-product attention

$$\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right) V$$

The $\sqrt{d_k}$ scaling factor is motivated in the paper explicitly: for large $d_k$, the dot products $q \cdot k = \sum_{i=1}^{d_k} q_i k_i$ grow in variance (variance = $d_k$ if components have variance 1), pushing the softmax into regions with near-zero gradient. Scaling by $1/\sqrt{d_k}$ returns the variance to 1.

### Multi-head attention

Instead of a single attention operation over $d_{\text{model}}$-dimensional vectors, project Q, K, V into $h$ lower-dimensional subspaces and run attention in parallel:

$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h) W^O$$
$$\text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, KW_i^V)$$

With $h = 8$ heads and $d_k = d_v = 64$, total computational cost equals single-head attention at $d_{\text{model}} = 512$ — you get richer representations for the same compute. Each head can independently learn to attend to different aspects of the sequence.

### Three application contexts of attention in the model

1. **Encoder self-attention**: all queries, keys, values from the encoder's previous layer. Every encoder position can attend to every other encoder position. Bidirectional.
2. **Decoder self-attention (masked)**: same as encoder self-attention, but with a causal mask that sets attention weights for future positions to $-\infty$ before softmax, ensuring position $i$ can only attend to positions $\leq i$. Required for autoregressive generation.
3. **Encoder-decoder cross-attention**: queries come from the decoder's previous layer; keys and values come from the encoder's final representation. This is the only place the decoder "reads" the source sequence.

### Position-wise feed-forward network

Applied identically and independently to each position:

$$\text{FFN}(x) = \max(0, xW_1 + b_1)W_2 + b_2$$

$d_{\text{model}} = 512$ → $d_{\text{ff}} = 2048$ → $d_{\text{model}} = 512$. The inner dimension is 4× the model dimension. Uses ReLU (later models switch to GELU, SiLU).

### Residual connections and layer normalization

Every sublayer (attention, FFN) uses:

$$\text{output} = \text{LayerNorm}(x + \text{Sublayer}(x))$$

All sublayers and embedding layers output the same dimension $d_{\text{model}} = 512$ to make residual addition dimensionally consistent.

### Positional encoding

Sinusoidal encoding added to input embeddings at the bottom of both encoder and decoder stacks. Full treatment in [[positional-encoding]]. Ablation (Table 3, row E) showed learned positional embeddings produce nearly identical results.

### Weight tying

The model shares the same weight matrix across: (1) source token embeddings, (2) target token embeddings, and (3) the pre-softmax linear transformation. The embedding weights are scaled by $\sqrt{d_{\text{model}}}$ to prevent the embeddings from being too small in magnitude relative to the positional encoding.

---

## Training

### Optimizer

Adam with $\beta_1 = 0.9$, $\beta_2 = 0.98$, $\epsilon = 10^{-9}$ and a custom learning rate schedule:

$$\text{lr} = d_{\text{model}}^{-0.5} \cdot \min(\text{step}^{-0.5},\ \text{step} \cdot \text{warmup}^{-1.5})$$

This increases linearly for `warmup_steps = 4000` steps, then decays proportionally to the inverse square root of the step number. The warmup prevents early instability when the gradient estimates are poor. This schedule has become the default for transformer pretraining.

### Regularization

Three techniques:
- **Residual dropout** ($P_{\text{drop}} = 0.1$): applied to each sublayer output before the residual addition, and to the embedding + positional encoding sums.
- **Label smoothing** ($\varepsilon_{ls} = 0.1$): softens the one-hot targets. Hurts perplexity (model is less confident) but improves BLEU and accuracy. Full treatment in [[regularization]].

---

## Results

| Model | EN-DE BLEU | EN-FR BLEU | Training cost (FLOPs) |
|---|---|---|---|
| ConvS2S (best prior single model) | 25.16 | 40.46 | $9.6 \times 10^{18}$ |
| GNMT + RL Ensemble | 26.30 | 41.16 | $1.8 \times 10^{20}$ |
| **Transformer (base)** | 27.3 | 38.1 | $3.3 \times 10^{18}$ |
| **Transformer (big)** | **28.4** | **41.0** | $2.3 \times 10^{19}$ |

The big model beats all prior single models and ensembles on EN-DE by 2+ BLEU. The base model alone outperforms all prior single models at roughly 1/3 the training compute of the nearest competitor. EN-FR big model trained for 3.5 days on 8 P100 GPUs.

### Ablations (Table 3)

Key findings from varying the base model:
- **Heads**: single-head attention is 0.9 BLEU worse than 8 heads; too many heads (32) also degrades quality. 8 is the sweet spot for this scale.
- **Key size $d_k$**: reducing $d_k$ (16 instead of 64) hurts quality, suggesting the dot-product compatibility function is demanding and benefits from higher-dimensional keys.
- **Model size**: bigger models ($d_{\text{model}} = 1024$, $d_{\text{ff}} = 4096$) consistently help.
- **Dropout**: removing dropout causes significant overfitting (5.77 vs 4.92 perplexity).
- **Label smoothing**: $\varepsilon = 0.2$ slightly hurts; $\varepsilon = 0.1$ is the reported best.

---

## Why self-attention over recurrence (Section 4)

The paper provides a formal comparison (Table 1) of self-attention, recurrent, and convolutional layers on three metrics:

| Layer type | Complexity per layer | Sequential ops | Max path length |
|---|---|---|---|
| Self-attention | $O(n^2 \cdot d)$ | $O(1)$ | $O(1)$ |
| Recurrent | $O(n \cdot d^2)$ | $O(n)$ | $O(n)$ |
| Convolutional | $O(k \cdot n \cdot d^2)$ | $O(1)$ | $O(\log_k n)$ |
| Restricted self-attn (window $r$) | $O(r \cdot n \cdot d)$ | $O(1)$ | $O(n/r)$ |

Self-attention is faster than recurrence when $n < d$ (true for most NLP tasks at the time — typical sequences were shorter than the model dimension). The constant path length is the decisive advantage for learning long-range dependencies.

---

## Contribution to existing wiki pages

- [[attention-mechanism]] — multi-head attention detail, three attention contexts, complexity table
- [[transformers]] — exact hyperparameters, warmup LR schedule, weight tying, ablation results
- [[positional-encoding]] — original sinusoidal PE; ablation showing learned PE is equivalent
- [[regularization]] — label smoothing usage, residual dropout details

---

## Critique

- The translation benchmark (WMT 2014) is now considered modest — modern models train on orders of magnitude more data.
- The paper's claim that sinusoidal PE "may allow extrapolation to longer sequences" is not well supported empirically; the model still degrades significantly beyond training length. RoPE (2021) solved this properly.
- The warmup LR schedule is influential but not well-theoretically motivated in the paper — later work (e.g., Chinchilla) provides better principled schedules.
- The "attention is interpretable" claim in Section 4 and the appendix is controversial — attention weights do not reliably correspond to importance in the way the paper implies.

## See also

- [[attention-mechanism]]
- [[transformers]]
- [[positional-encoding]]
- [[regularization]]
- [[recurrent-neural-networks]]

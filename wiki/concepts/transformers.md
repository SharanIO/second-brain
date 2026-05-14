---
title: Transformers
status: complete
last-updated: 2026-04-26
tags: [nlp, deep-learning, architecture]
---

# Transformers

A neural architecture built entirely around [[attention-mechanism|multi-head self-attention]], introduced by Vaswani et al. (2017, "Attention is All You Need"). Replaced [[recurrent-neural-networks|RNNs]] as the dominant architecture for sequence modeling.

## Architecture overview

Each transformer layer contains two sublayers:

1. **Multi-head self-attention** (MH-SA)
2. **Feed-forward network** (FFN / MLP) — position-wise, applied independently to each token

Each sublayer uses a **residual connection** (Add & Norm):

$$\text{output} = \text{LayerNorm}(x + \text{sublayer}(x))$$

Residual connections prevent vanishing gradients by providing a highway for information from early layers.

## Multi-head attention

Instead of a single attention function, use $h$ parallel attention heads, each with its own $W_Q, W_K, W_V$ projections. Each head may specialize in different linguistic properties (syntax, coreference, phrase structure). Outputs are concatenated and projected:

$$\text{MultiHead}(Q,K,V) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h) W^O$$

In the original Transformer: $h = 8$, $d_k = d_v = 64$, $d_{\text{model}} = 512$. Full treatment in [[attention-mechanism]].

## Positional encoding

Self-attention is permutation-invariant — it has no built-in notion of order. [[positional-encoding|Positional encodings]] are added to the token embeddings to inject position information. Can be:
- **Learned**: embedding lookup table (limited to the training context length)
- **Deterministic**: sinusoidal functions of position (generalizes to longer sequences)

## Model variants

### Decoder-only (GPT family)
- Uses **causal (masked) self-attention**: each token can only attend to previous tokens
- Trained with causal LM objective: predict next token
- Best for: text generation
- Examples: GPT-2, GPT-3, Llama

### Encoder-only (BERT family)
- Uses **unmasked (bidirectional) self-attention**: each token sees the full sequence
- Trained with **masked LM** objective: predict masked tokens
- Cannot generate text autoregressively
- Best for: classification, encoding, representation tasks
- Examples: BERT, RoBERTa, ELECTRA

### Encoder-decoder (T5 family)
- Encoder: bidirectional (unmasked)
- Decoder: causal (masked) + **cross-attention** to encoder representations
- Cross-attention: decoder queries attend to encoder keys/values (the only bridge between encoder and decoder)
- Best for: seq-to-seq tasks — translation, summarization, Q&A
- Examples: T5, BART, mT5

## Original Transformer hyperparameters (Vaswani et al. 2017)

The base model configuration that established the defaults still used by most modern architectures:

| Parameter | Base | Big |
|---|---|---|
| Layers $N$ | 6 | 6 |
| $d_{\text{model}}$ | 512 | 1024 |
| $d_{\text{ff}}$ | 2048 | 4096 |
| Heads $h$ | 8 | 16 |
| $d_k = d_v$ | 64 | 64 |
| Dropout | 0.1 | 0.3 |
| Parameters | 65M | 213M |
| Training steps | 100K | 300K |

**Results**: Transformer (big) achieved 28.4 BLEU on WMT 2014 EN-DE and 41.0 BLEU on EN-FR — beating all prior single models and ensembles at a fraction of the compute cost.

### Warmup learning rate schedule

$$\text{lr} = d_{\text{model}}^{-0.5} \cdot \min(\text{step}^{-0.5},\ \text{step} \cdot \text{warmup}^{-1.5})$$

Linear warmup for `warmup_steps = 4000`, then inverse square root decay. The warmup prevents early instability when gradient estimates are unreliable. This schedule has become the default for transformer pretraining.

### Weight tying

The source embedding matrix, target embedding matrix, and pre-softmax projection share the same weights. Embedding weights are scaled by $\sqrt{d_{\text{model}}}$. This reduces parameters and enforces consistency between how tokens are encoded and decoded.

## Tensor shape walkthrough (3-token sequence, $d=512$, 8 heads)

| Step | Shape | What happened |
|---|---|---|
| Input IDs | [3] | token indices |
| Embedding | [3 × 512] | token → dense vector |
| + Positional encoding | [3 × 512] | order info added |
| Split into heads | [8, 3, 64] | 8 heads, each 64-dim |
| Self-attention | [3 × 512] | heads concatenated |
| Add & Norm | [3 × 512] | residual + LayerNorm |
| Feed-forward | [3 × 512] | GELU/linear processing |
| End of Layer 1 | [3 × 512] | ready for Layer 2 |

## Sources

- [[adv-nlp-course-notes]]
- [[attention-is-all-you-need]] (Vaswani et al. 2017)

## See also

- [[attention-mechanism]]
- [[positional-encoding]]
- [[pretraining-and-finetuning]]
- [[recurrent-neural-networks]]

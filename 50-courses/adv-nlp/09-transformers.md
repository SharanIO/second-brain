---
title: "Chapter 9: The Transformer Architecture"
course: Advanced NLP
chapter: 9
last-updated: 2026-04-26
tags: [nlp, transformers, architecture, attention]
---

# Chapter 9: The Transformer Architecture

## Learning objectives

By the end of this chapter you should be able to:
- Describe the full transformer layer and the role of each component
- Explain multi-head attention and why multiple heads are useful
- Describe positional encoding and explain why it is necessary
- Distinguish between decoder-only, encoder-only, and encoder-decoder transformers
- Trace a sequence of tokens through a complete transformer forward pass with correct shapes

---

## 9.1 From attention to architecture

Chapter 8 gave us the attention mechanism: a way for each token to selectively gather information from all other tokens in the sequence. But a single attention layer is not a complete language model. We need to:

1. Process multiple positions simultaneously in parallel (we have this)
2. Apply non-linear transformations to refine representations (we need MLP layers)
3. Allow information from lower layers to reach higher layers without degradation (we need residual connections)
4. Stabilize training for deep networks (we need layer normalization)
5. Encode positional information (attention is order-agnostic)
6. Stack many layers to build depth (we need a repeatable block structure)

The **Transformer** (Vaswani et al., *"Attention is All You Need"*, 2017) combines all of these into one architecture that has dominated NLP ever since.

---

## 9.2 The transformer layer

One transformer layer contains exactly two sublayers, each with a residual connection:

**Sublayer 1: Multi-head self-attention**
$$\mathbf{x} \leftarrow \text{LayerNorm}(\mathbf{x} + \text{MultiHeadAttn}(\mathbf{x}))$$

**Sublayer 2: Feed-forward network (MLP)**
$$\mathbf{x} \leftarrow \text{LayerNorm}(\mathbf{x} + \text{FFN}(\mathbf{x}))$$

The pattern "sublayer → add residual → normalize" repeats. This is the **Add & Norm** pattern.

### Why residual connections?

A residual connection adds the input of a sublayer directly to its output: $\mathbf{y} = \mathbf{x} + \text{sublayer}(\mathbf{x})$.

The key property: gradients can flow directly from deep layers to shallow layers through the addition operation, without passing through the potentially complicated sublayer computation. This prevents vanishing gradients in very deep networks (transformers commonly have 12–96 layers).

Think of it as adding a "shortcut" — the information from the previous layer does not have to be completely relearned; the sublayer only needs to compute what *changes*, not the entire representation from scratch.

### Feed-forward network (FFN)

The FFN in each transformer layer is a simple two-layer MLP applied **independently to each token**:

$$\text{FFN}(\mathbf{x}) = W_2 \cdot \text{GELU}(W_1 \mathbf{x} + \mathbf{b}_1) + \mathbf{b}_2$$

The expansion ratio is typically 4: if $d_{model} = 512$, then $W_1 \in \mathbb{R}^{2048 \times 512}$ (expands to 2048) and $W_2 \in \mathbb{R}^{512 \times 2048}$ (projects back to 512).

The FFN acts on each token independently, without any cross-token interaction. Its role is to process and refine each token's representation *after* it has already gathered context from self-attention.

---

## 9.3 Multi-head attention

Instead of performing attention once, we perform it $h$ times in parallel with different learned projections — these are the **heads**.

$$\text{head}_i = \text{Attention}(QW_i^Q,\ KW_i^K,\ VW_i^V)$$
$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h) W^O$$

Each head operates in a lower-dimensional space: if $d_{model} = 512$ and $h = 8$, each head has dimension $d_k = d_v = 512/8 = 64$.

**Why multiple heads?** Different heads can specialize in different types of relationships:
- One head might track syntactic dependencies (verb and its subject)
- Another might track coreference (pronoun and its antecedent)
- Another might track semantic similarity (synonyms)

The concatenation of all heads is projected back to $d_{model}$ by $W^O$. This is the final multi-head attention output.

The number of heads $h$ is a model-level hyperparameter — the same number of heads is used in every layer.

---

## 9.4 Positional encoding: giving the model a sense of order

Self-attention is **permutation-invariant**: if you shuffle the input tokens, the attention mechanism produces the same output vectors (just in a different order). It has no inherent notion of which token came first.

This is a problem: word order is critical for meaning. "Dog bites man" and "Man bites dog" have the same tokens but entirely different meanings.

**Positional encodings** solve this by adding position-dependent information to each token's embedding before it enters the transformer:

$$\mathbf{x}_t \leftarrow \mathbf{e}_{w_t} + \mathbf{p}_t$$

where $\mathbf{p}_t$ encodes the position $t$.

### Learned positional embeddings

A lookup table $P \in \mathbb{R}^{L_{max} \times d_{model}}$ stores a learned embedding for each position up to a maximum length. Simple and effective. Limitation: positions beyond $L_{max}$ have no representation.

### Sinusoidal positional encoding (original Transformer paper)

$$\mathbf{p}_{t, 2i} = \sin\!\left(\frac{t}{10000^{2i/d_{model}}}\right), \quad \mathbf{p}_{t, 2i+1} = \cos\!\left(\frac{t}{10000^{2i/d_{model}}}\right)$$

Different dimensions of $\mathbf{p}_t$ oscillate at different frequencies — high dimensions oscillate slowly (capturing global position), low dimensions oscillate quickly (capturing local position). This allows the model to generalize to longer sequences than seen during training.

### Modern alternatives: RoPE, ALiBi

More recent models use **Rotary Position Embedding (RoPE)** or **ALiBi**, which encode *relative* positions rather than absolute ones. These generalize much better to long sequences and are standard in Llama, Mistral, and GPT-4-class models.

> [!todo] cite this — expand RoPE/ALiBi section when relevant papers are added to the vault

---

## 9.5 Three transformer variants

All modern transformer-based language models are variants of one of three architectures. Each is characterized by what kind of attention mask the self-attention uses and what pretraining objective it is trained with.

### Decoder-only (GPT family)

Uses **causal (masked) self-attention**: each token can only attend to itself and all previous tokens.

```
Input:  [The] [cat] [sat] [on]
          ↓     ↓     ↓    ↓
Mask:   token can see: ↙
          The   The,cat   The,cat,sat   The,cat,sat,on
```

Trained with the **causal language modeling** objective: predict the next token. Because the model is trained on every prefix simultaneously (all positions are predicted in parallel), training is highly efficient.

Best for: text generation, chat, completion.
Examples: GPT-2, GPT-3, Llama, Mistral, Claude, Gemini.

Why decoder-only for most modern LLMs? Simplicity and efficiency. The causal mask means the architecture can be applied to any task by framing it as text-to-text completion. The encoder is not needed.

### Encoder-only (BERT family)

Uses **bidirectional (unmasked) self-attention**: each token can attend to all other tokens, including future ones.

Trained with **masked language modeling** (MLM): randomly mask ~15% of input tokens and predict them. Because the model sees both left and right context for each masked token, it builds richer contextual representations.

Best for: classification, named entity recognition, sentence similarity, reading comprehension.
Cannot generate text autoregressively (no causal mask means it sees the future during training, so it cannot be used to predict unseen next tokens).

Examples: BERT, RoBERTa, ELECTRA, DeBERTa.

The special `[CLS]` token is prepended to every input; its final representation aggregates information from the whole sequence and is used for classification. The loss is computed only on `[MASK]` positions.

### Encoder-decoder (T5 / BART family)

Combines a bidirectional encoder with a causally-masked decoder, connected by **cross-attention**:

- **Encoder**: reads the full input with unmasked self-attention; produces contextual representations.
- **Decoder**: generates output autoregressively with causal self-attention, plus cross-attention to the encoder's output at every layer.

The cross-attention uses encoder representations as keys and values; the decoder's current hidden state as the query. This is the bridge between encoder and decoder.

Best for: seq-to-seq tasks — translation, summarization, question answering with a given context.
Examples: T5, mT5, BART, mBART.

---

## 9.6 Full forward pass: tracing a sequence through a transformer

Let us trace a 3-token input *"cats eat fish"* through one transformer layer with $d_{model} = 512$, $h = 8$ heads, and $d_{ff} = 2048$.

| Step | Operation | Shape |
|---|---|---|
| Input token IDs | $[42, 11, 302]$ | $[3]$ |
| Embedding lookup | $E[\text{token\_ids}]$ | $[3 \times 512]$ |
| + Positional encoding | $\mathbf{x} + \mathbf{p}$ | $[3 \times 512]$ |
| Project to Q, K, V | $xW_Q$, $xW_K$, $xW_V$ | $3 \times [3 \times 64]$ per head |
| Scaled dot-product attn | $\text{softmax}(QK^\top/8)V$ | $[3 \times 64]$ per head |
| Concatenate 8 heads | $\text{Concat}(\ldots)$ | $[3 \times 512]$ |
| Project with $W^O$ | multiply | $[3 \times 512]$ |
| Residual + LayerNorm | $\text{LN}(x + \text{attn\_out})$ | $[3 \times 512]$ |
| FFN: expand | $W_1 x$ + GELU | $[3 \times 2048]$ |
| FFN: contract | $W_2 x$ | $[3 \times 512]$ |
| Residual + LayerNorm | $\text{LN}(x + \text{ffn\_out})$ | $[3 \times 512]$ |

After $N$ such layers, a final linear projection (to vocabulary size) and softmax produces token probabilities.

Notice that the shape $[3 \times 512]$ is preserved throughout — every token starts and ends with the same dimensionality. The transformer is a sequence-to-sequence function that preserves shape.

---

## Summary

- A **transformer layer** = multi-head self-attention + FFN, each wrapped in a residual + LayerNorm.
- **Residual connections** allow gradients to flow directly through deep networks without vanishing.
- **Multi-head attention** runs attention $h$ times in parallel, each head learning different relational patterns.
- **Positional encoding** adds order information to embeddings, since self-attention is otherwise permutation-invariant.
- **Decoder-only** (causal mask, next-token prediction): text generation. **Encoder-only** (bidirectional, MLM): understanding. **Encoder-decoder** (both): seq-to-seq.
- Shapes are preserved through all transformer layers: $[T \times d_{model}]$ in, $[T \times d_{model}]$ out.

**Previous**: [[08-attention-mechanism|Chapter 8 — The Attention Mechanism]]
**Next**: [[10-pretraining-objectives|Chapter 10 — Pretraining: Self-Supervised Learning at Scale]]

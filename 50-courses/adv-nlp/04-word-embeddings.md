---
title: "Chapter 4: Word Representations and Embeddings"
course: Advanced NLP
chapter: 4
last-updated: 2026-04-26
tags: [nlp, embeddings, representations, word2vec]
---

# Chapter 4: Word Representations and Embeddings

## Learning objectives

By the end of this chapter you should be able to:
- Explain why one-hot vectors are insufficient and what properties a good word representation should have
- Describe what a word embedding is and how it is learned
- Explain how an embedding lookup table works mechanically
- Describe what it means for embeddings to capture semantic similarity

---

## 4.1 The problem with one-hot vectors

In Chapter 2, we represented each word as a one-hot vector: a vector of zeros with a single 1 at the word's position in the vocabulary. For a vocabulary of 50,000 words, each word is a vector of length 50,000 with exactly one non-zero entry.

This representation has three crippling problems:

**Problem 1: Memory.** Each word is a 50,000-dimensional vector. The weight matrix connecting the input layer to the first hidden layer would be of shape $[hidden\_size \times 50{,}000]$. Even with a small hidden size of 256, that is 12.8 million parameters just for the first layer — and most of them are multiplied by zeros.

**Problem 2: No similarity.** For any two distinct words $u$ and $v$, their dot product is zero: $\mathbf{u} \cdot \mathbf{v} = 0$. The representation cannot capture the fact that "cat" and "kitten" are more similar to each other than either is to "democracy". From the model's perspective, all words are equally different.

**Problem 3: No generalization.** If the model learns something useful from the sentence "The cat sat on the mat", it cannot apply that knowledge to "The kitten sat on the mat" because "cat" and "kitten" are represented as completely unrelated vectors. Each word must be learned from scratch.

These are not minor engineering problems — they are fundamental flaws in the representation itself. They motivate a completely different approach.

---

## 4.2 What should a good word representation look like?

Think about the properties we actually want:

1. **Dense**: a few hundred dimensions, not fifty thousand. Computationally efficient.
2. **Similarity-preserving**: words with similar meanings should have similar representations. The dot product (or cosine similarity) between "cat" and "kitten" should be high; between "cat" and "democracy", low.
3. **Learnable**: the representations should emerge from training on text data, not be hand-coded by a linguist.
4. **Fixed-size**: every word should map to a vector of the same length, regardless of whether it is short or long.

A **word embedding** satisfies all four properties. It is a dense, low-dimensional, learned vector representation of a word.

---

## 4.3 The embedding lookup table

Mechanically, word embeddings live in a matrix $E \in \mathbb{R}^{|V| \times d}$, where $|V|$ is the vocabulary size and $d$ is the embedding dimension (a hyperparameter, typically 128–1024).

Each row of $E$ is the embedding vector for one word. To get the embedding for word $w$, we simply look up the row corresponding to $w$'s integer index:

$$\mathbf{e}_w = E[w]$$

This is identical to multiplying the one-hot vector $\mathbf{v}_w$ by $E$:

$$E^\top \mathbf{v}_w = \mathbf{e}_w$$

But the lookup is much more efficient — we never actually construct the one-hot vector.

In PyTorch:

```python
import torch.nn as nn

vocab_size = 50000
embed_dim  = 512

embedding = nn.Embedding(vocab_size, embed_dim)

# Given a batch of token IDs:
token_ids = torch.tensor([42, 7, 1839])   # shape: [3]
embeddings = embedding(token_ids)          # shape: [3, 512]
```

The matrix $E$ is a learned parameter — it is randomly initialized at the start of training and updated by backpropagation, exactly like any other weight matrix.

> [!note] Embeddings as a compression
> Instead of a $[hidden \times |V|]$ weight matrix applied to sparse one-hot vectors, we have a $[|V| \times d]$ embedding matrix followed by regular dense operations. The embedding is essentially a learned compression: it maps a discrete symbol to a point in a continuous $d$-dimensional space.

---

## 4.4 How embeddings learn similarity

The remarkable thing about word embeddings is that, trained on a large corpus with a language modeling objective, they spontaneously learn to encode semantic relationships.

The intuition comes from the **distributional hypothesis**: *words that occur in similar contexts tend to have similar meanings*. "Cat" and "kitten" both appear near words like "fur", "meow", "litter", and "purr". The language modeling objective pushes the model to give similar predictions for similar contexts — and therefore to give similar representations to words that appear in similar contexts.

After training, you tend to find:
- $\text{sim}(\text{"cat"}, \text{"kitten"}) \gg \text{sim}(\text{"cat"}, \text{"democracy"})$
- Arithmetic relationships: $\mathbf{e}_{\text{king}} - \mathbf{e}_{\text{man}} + \mathbf{e}_{\text{woman}} \approx \mathbf{e}_{\text{queen}}$
- Clusters of semantically related words sitting close together in the embedding space

> [!warning] Interpretability
> Individual dimensions of an embedding vector do not have clean interpretable meanings. You cannot say "dimension 42 measures 'animateness'". The geometric relationships between embeddings are meaningful, but the individual numbers are not. This is unlike one-hot vectors, where every dimension means something specific.

---

## 4.5 From word embeddings to sequence representations

A language model processes a sequence of words. Given token ids $[w_1, w_2, \ldots, w_T]$, we look up each word's embedding to get a sequence of vectors:

$$[\mathbf{e}_{w_1},\ \mathbf{e}_{w_2},\ \ldots,\ \mathbf{e}_{w_T}]$$

But we immediately face a new problem: the model needs to know the *order* of the words, not just their identities. The embeddings themselves carry no positional information — $\mathbf{e}_{\text{cat}}$ is the same vector regardless of whether "cat" appears at position 1 or position 100.

### Early approaches to sequence composition

Early neural language models handled this by:

**Summing**: add all the word embeddings together. Fast and simple, but throws away word order entirely — "dog bites man" and "man bites dog" get the same representation.

**Concatenating in order**: take the embeddings of the last $k$ words and concatenate them into a single vector. This preserves local order but fixes the context window size to $k$.

Neither approach is satisfactory for long sequences. The proper solution — how transformers handle position — comes in Chapter 9. For now, the key point is that embeddings give us rich, dense, learnable word representations that can be fed into any subsequent neural architecture.

---

## 4.6 Embedding dimensionality

How large should $d$ be? There is no universal answer — it is a hyperparameter you tune. Some intuitions:

- **Too small**: the embedding space is too cramped. Words that should be distinguishable get pushed together. Typical minimum: 64–128 for simple tasks.
- **Too large**: more parameters to train, slower computation, more data needed to avoid overfitting. Diminishing returns beyond $d \approx 1024$ for most tasks.
- **Modern LLMs**: GPT-3 uses $d = 12{,}288$; Llama 3 (8B) uses $d = 4{,}096$.

The embedding dimension is one of the most important architectural choices in a language model.

---

## Summary

- One-hot vectors are memory-inefficient, encode no similarity, and cannot generalize across words.
- A **word embedding** is a dense, low-dimensional, learned vector representation of a word.
- Embeddings live in a lookup table $E \in \mathbb{R}^{|V| \times d}$; fetching an embedding is just indexing a row.
- Embeddings learn to encode semantic similarity via the distributional hypothesis: similar context → similar meaning → similar vector.
- Individual embedding dimensions are not interpretable; geometric relationships between embeddings are.
- Sequences of token IDs become sequences of embedding vectors, which can be fed into further neural layers.

**Previous**: [[03-neural-networks-primer|Chapter 3 — Neural Networks: A Primer]]
**Next**: [[05-training-mechanics|Chapter 5 — Training Neural Networks: Backpropagation and Optimizers]]

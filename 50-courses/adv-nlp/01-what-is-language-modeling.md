---
title: "Chapter 1: What is Language Modeling?"
course: Advanced NLP
chapter: 1
last-updated: 2026-04-26
tags: [nlp, language-models, probability, foundations]
---

# Chapter 1: What is Language Modeling?

## Learning objectives

By the end of this chapter you should be able to:
- Explain what a language model is and why it is useful
- Write down the probability of a sentence using the chain rule
- Explain what perplexity measures and how to interpret it
- Describe the core challenge that makes language modeling hard

---

## 1.1 What do we mean by "language"?

Before we build a model of language, we need to agree on what language *is* from a computational point of view. For our purposes, language is a sequence of discrete symbols — words, or later, subword pieces — drawn from a fixed vocabulary $V$.

A sentence like *"The cat sat on the mat"* is just a sequence of tokens:

$$[\text{The},\ \text{cat},\ \text{sat},\ \text{on},\ \text{the},\ \text{mat}]$$

Everything in NLP starts here. The vocabulary $V$ is the set of all symbols the model knows about. Its size $|V|$ is typically in the tens of thousands.

---

## 1.2 What is a language model?

A **language model** is a probability distribution over sequences of tokens. Given any sequence of words $w_1, w_2, \ldots, w_T$, a language model assigns it a probability:

$$P(w_1, w_2, \ldots, w_T)$$

That's it. A language model is a function that takes a sequence and returns a number between 0 and 1, telling you how likely that sequence is to occur in natural language.

For example, a good language model should assign:

- $P(\text{"The cat sat on the mat"})$ → relatively high
- $P(\text{"Mat the on sat cat the"})$ → close to zero
- $P(\text{"The cat sat on the moon"})$ → moderate (unusual but plausible)

### Why is this useful?

This deceptively simple definition turns out to be extraordinarily powerful. If you can model the probability of text well, you can:

- **Generate text**: sample the next word by asking *"what word is most probable given what came before?"*
- **Score hypotheses**: pick the most likely translation, transcription, or completion
- **Compress data**: a model that assigns high probability to real text is implicitly a good compressor
- **Power downstream tasks**: with enough scale, language models develop emergent capabilities — reasoning, coding, question answering — as a side effect of predicting text well

Modern systems like GPT and Llama are "just" language models trained at extreme scale. The fundamental objective never changes.

---

## 1.3 The chain rule: factoring the joint probability

Computing $P(w_1, w_2, \ldots, w_T)$ directly is intractable — there are $|V|^T$ possible sequences and we cannot store a separate probability for each one. Instead, we use the **chain rule of probability** to factor the joint probability into a product of conditional probabilities:

$$P(w_1, w_2, \ldots, w_T) = P(w_1) \cdot P(w_2 \mid w_1) \cdot P(w_3 \mid w_1, w_2) \cdots P(w_T \mid w_1, \ldots, w_{T-1})$$

Or more compactly:

$$P(w_1, \ldots, w_T) = \prod_{t=1}^{T} P(w_t \mid w_1, \ldots, w_{t-1})$$

This is not an approximation — it is an exact identity from probability theory. What it tells us is: **the probability of a sequence is the product of the probability of each word given everything that came before it.**

This reframes the language modeling problem: instead of estimating the probability of whole sentences, we just need to estimate the probability of the *next word* given a *prefix*. This is the core task.

> [!note] The autoregressive view
> Writing $P(w_t \mid w_1, \ldots, w_{t-1})$ for every $t$ means we are modeling language *left to right*, one word at a time. This is called **autoregressive** language modeling, and it is the basis of GPT-style models.

---

## 1.4 What makes this hard?

The conditional probability $P(w_t \mid w_1, \ldots, w_{t-1})$ seems simple, but there are three compounding difficulties.

**The vocabulary is large.** At each step, the model must assign a probability to every word in a vocabulary of ~50,000 items. These probabilities must sum to 1.

**The context is long.** The prefix $w_1, \ldots, w_{t-1}$ can be hundreds or thousands of words long. The model must somehow use all of it, but most of the relevant signal may come from just a few key words far back in the context.

**Data sparsity.** Most specific sequences of words never appear in training data, even in very large corpora. A model that only memorizes frequencies of observed sequences will fail catastrophically on any novel input.

These three challenges — large output space, long context, and data sparsity — drive almost every architectural decision we will study in this course.

---

## 1.5 Evaluating a language model: perplexity

Once we have a language model, how do we know if it is good? The standard metric is **perplexity**.

Intuitively, perplexity measures how "surprised" the model is by a held-out test set. A model that assigns high probability to the actual next words is less surprised — and has lower perplexity.

Formally, perplexity on a test sequence of $T$ tokens is:

$$\text{PPL} = \exp\!\left(-\frac{1}{T} \sum_{t=1}^{T} \log P(w_t \mid w_{<t})\right)$$

This is just the exponentiated average negative log-probability per token. Lower perplexity is better.

**Intuition**: perplexity is roughly the "effective vocabulary size" the model is confused between at each step. A perplexity of 100 means the model is, on average, as confused as if it had to choose uniformly among 100 words. A perplexity of 10 means it has narrowed it down to ~10 plausible next words.

> [!tip] Perplexity in practice
> On standard English benchmarks, n-gram models achieve perplexity in the hundreds. Good neural LMs achieve tens. GPT-4-class models achieve single digits on many datasets. A perplexity of 1 would mean the model perfectly predicts every next word — impossible on real data.

---

## 1.6 Types of language models

We will study language models in roughly historical order, each one solving the problems left by its predecessor:

| Model type | Core idea | Main limitation |
|---|---|---|
| N-gram models | Count word co-occurrences | Exponential parameter count; no generalization |
| Neural LMs (MLP) | Learn dense word representations | Fixed, small context window |
| RNNs | Process sequences with shared weights | Sequential; vanishing gradients |
| Transformers | Attend to the full context in parallel | Quadratic memory in sequence length |

Each chapter will introduce one row of this table, explain the key ideas from first principles, and show exactly where it breaks down — motivating the next chapter.

---

## Summary

- A **language model** is a probability distribution over token sequences.
- The **chain rule** lets us factor this into next-word predictions: $P(w_t \mid w_1, \ldots, w_{t-1})$.
- The core challenge is estimating these conditional probabilities despite a large vocabulary, long context, and sparse data.
- **Perplexity** is the standard evaluation metric — lower is better.
- The history of NLP is a series of increasingly powerful solutions to the three core challenges.

**Next**: [[02-n-gram-language-models|Chapter 2 — N-gram Language Models]]

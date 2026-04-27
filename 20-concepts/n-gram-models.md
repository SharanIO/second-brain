---
title: N-gram Models
status: complete
last-updated: 2026-04-26
tags: [nlp, language-models, statistical]
---

# N-gram Models

A statistical approach to language modeling. An n-gram model estimates the probability of a word given the $n-1$ preceding words (the "prefix"):

$$P(w_n \mid w_1, w_2, \ldots, w_{n-1}) \approx P(w_n \mid w_{n-2}, w_{n-1})$$

## Variants

| Model | Condition on |
|---|---|
| Unigram | nothing — $\prod_i P(w_i)$ |
| Bigram | previous 1 word |
| Trigram | previous 2 words |
| n-gram | previous $n-1$ words |

The **Markov assumption** underlies all of these: the probability of the next word depends only on the last $k$ words, not the full history.

Large models use n up to ~20; longer prefixes cause exponential growth in possible combinations ($O(e^n)$ storage).

**Infini-gram** — a variant that dynamically chooses $n$ based on the available training context.

## Representation: one-hot encoding

Each word in the vocabulary is represented as a binary vector with a single 1 at its index. Problems:
- Memory: $O(|V|)$ per word
- Dot product of any two distinct words is zero — no notion of similarity
- Words absent from training data get zero probability

Words that do exist are counted and converted to probabilities directly from corpus frequency.

## Perplexity

The standard evaluation metric. A better model assigns higher probability to the actual next word, so lower perplexity = better model. A model with equal probability over all words has maximum perplexity; the model is "confused."

## Limitations

1. **Exponential storage**: $O(e^n)$ combinations.
2. **Independence**: all words treated as independent; no shared representations for semantically similar words.
3. **Sparsity**: many word sequences never appear in training, giving zero probability.
4. **No semantic similarity**: similar words have different one-hot codes (dot product = 0), so they cannot help each other's probability estimates.
5. **Not explainable**: relies purely on counts.

These limitations motivate [[word-embeddings]] and [[neural-language-models]].

## Sources

- [[adv-nlp-course-notes]]

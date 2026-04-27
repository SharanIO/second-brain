---
title: "Chapter 2: N-gram Language Models"
course: Advanced NLP
chapter: 2
last-updated: 2026-04-26
tags: [nlp, language-models, n-grams, statistical]
---

# Chapter 2: N-gram Language Models

## Learning objectives

By the end of this chapter you should be able to:
- Define an n-gram model and write down its probability formula
- Explain the Markov assumption and why it is necessary
- Describe the sparsity problem and how smoothing addresses it
- List the fundamental limitations of n-gram models that motivated neural approaches

---

## 2.1 The core idea: approximate the context

Recall from Chapter 1 that a language model must estimate $P(w_t \mid w_1, \ldots, w_{t-1})$ — the probability of the next word given the entire preceding context. The problem is that this context grows without bound. We need to approximate it.

The n-gram model's approximation is the **Markov assumption**: assume the next word depends only on the last $n-1$ words, not the entire history.

$$P(w_t \mid w_1, \ldots, w_{t-1}) \approx P(w_t \mid w_{t-n+1}, \ldots, w_{t-1})$$

This is a lossy approximation — the word "it" in the sentence *"The cat that the dog chased caught it"* refers to "the cat", which is five words back. An n-gram model with $n = 2$ would have no access to this. But the approximation makes the problem computationally tractable.

---

## 2.2 The n-gram family

The parameter $n$ controls how much context the model sees. The special cases have names:

**Unigram** ($n=1$): each word is treated as independent of everything else.
$$P(w_1, w_2, \ldots, w_T) = \prod_{t=1}^{T} P(w_t)$$
This is simply the word frequency distribution. It knows that "the" is common and "xylophone" is rare, but it cannot model any word-order patterns at all.

**Bigram** ($n=2$): the next word depends on the immediately preceding word.
$$P(w_t \mid w_{t-1})$$
Now the model knows that "black" frequently precedes "cat" but rarely precedes "democracy". Already much more expressive than unigram.

**Trigram** ($n=3$): the next word depends on the two preceding words.
$$P(w_t \mid w_{t-2}, w_{t-1})$$

In practice, large language models trained before the deep learning era used $n$ up to around 5. Going higher makes the sparsity problem (Section 2.4) worse faster than it improves accuracy.

> [!note] Infini-gram
> A recent variant called **infini-gram** sidesteps the fixed-$n$ limitation by dynamically choosing the longest matching prefix found in the training corpus. It is essentially a very large suffix array over text.

---

## 2.3 Estimating probabilities from data

Given a large text corpus, how do we actually compute $P(w_t \mid w_{t-n+1}, \ldots, w_{t-1})$?

We count. The **maximum likelihood estimate** is:

$$P(w_t \mid w_{t-n+1}, \ldots, w_{t-1}) = \frac{C(w_{t-n+1}, \ldots, w_{t-1}, w_t)}{C(w_{t-n+1}, \ldots, w_{t-1})}$$

where $C(\cdot)$ denotes the count of that n-gram in the training corpus.

**Example** (bigram): To estimate $P(\text{"sat"} \mid \text{"cat"})$, count how many times "cat sat" appears in training data, then divide by how many times "cat" appears in total.

$$P(\text{sat} \mid \text{cat}) = \frac{C(\text{cat sat})}{C(\text{cat})}$$

If "cat sat" appears 37 times and "cat" appears 200 times, then $P(\text{sat} \mid \text{cat}) = 0.185$.

This is simple, fast, and interpretable. The entire model can be stored as a large lookup table.

---

## 2.4 The sparsity problem

The count-based approach has a devastating flaw: any n-gram that does not appear in the training data gets a count of zero, and therefore a probability of zero. In a bigram model, if "elephant sneezed" never appeared in training, the model will assign $P(\text{sneezed} \mid \text{elephant}) = 0$.

Multiplying zero probabilities along the chain rule makes the entire sentence probability zero. A model that assigns zero probability to a perfectly grammatical sentence is clearly broken.

Sparsity gets exponentially worse as $n$ increases. The number of possible distinct n-grams is $|V|^n$. With $|V| = 50{,}000$ and $n = 3$, there are $1.25 \times 10^{14}$ possible trigrams — vastly more than any training corpus contains.

### Smoothing

The standard fix is **smoothing**: steal a small amount of probability mass from seen events and redistribute it to unseen ones.

**Laplace (add-1) smoothing**: add 1 to every count, including zeros.
$$P_{\text{smooth}}(w_t \mid w_{t-1}) = \frac{C(w_{t-1}, w_t) + 1}{C(w_{t-1}) + |V|}$$

**Kneser-Ney smoothing** (the practical standard): a more sophisticated method that discounts seen counts and redistributes the mass in a way that respects word frequency patterns. A word like "Francisco" appears frequently, but almost always after "San" — Kneser-Ney can capture this.

Smoothing helps, but it is fundamentally a patch on a broken design.

---

## 2.5 Word representations: one-hot vectors

In an n-gram model, each word in the vocabulary is represented as a **one-hot vector**: a vector of length $|V|$ with a single 1 at the index corresponding to that word, and 0 everywhere else.

For a vocabulary of size 5:
- "cat" → $[0, 1, 0, 0, 0]$
- "dog" → $[0, 0, 1, 0, 0]$
- "sat" → $[0, 0, 0, 1, 0]$

One-hot vectors are simple but deeply limited. Notice that the dot product of any two distinct one-hot vectors is exactly zero. This means the representation treats all words as *equally dissimilar* to each other. The model has no way to know that "cat" and "dog" are more similar to each other than either is to "democracy". Every word starts as a stranger.

This is a fundamental limitation, not just a technical inconvenience. It means:
- The model cannot generalize across synonyms
- Training data for "cat" teaches the model nothing about "feline"
- The model cannot borrow probability mass from similar words to fill in sparse counts

---

## 2.6 The full picture: limitations of n-gram models

Let us summarize everything n-gram models get wrong. Each limitation becomes a design goal for the neural models in later chapters.

**1. Exponential storage.** The number of possible n-grams is $O(|V|^n)$. Even with sparse storage, large n-gram models consume hundreds of gigabytes.

**2. No semantic similarity.** One-hot representations give zero similarity between any two words. "Cat" and "kitten" are indistinguishable to the model.

**3. Sparsity.** Most n-grams are never seen in training. Any reasonable text will contain novel combinations.

**4. Fixed, small context.** Even with $n = 5$, the model can only see the last 4 words. Long-range dependencies — a pronoun referring to a noun 20 words earlier — are invisible.

**5. No compositionality.** The model treats each n-gram as a primitive. It has no way to understand that "the black cat" and "the dark feline" mean similar things, because it cannot decompose the phrases.

**6. Not explainable.** The model is a lookup table of counts. There is no internal structure to inspect or reason about.

These six limitations are exactly what neural language models are designed to overcome. The next chapter introduces the mathematical machinery — neural networks — that makes this possible.

---

## Summary

- N-gram models approximate the full context with the last $n-1$ words (the **Markov assumption**).
- Probabilities are estimated by counting n-gram frequencies in a corpus.
- The **sparsity problem** means any unseen n-gram gets zero probability; smoothing is a partial fix.
- Words are represented as **one-hot vectors**, which cannot capture semantic similarity.
- N-gram models have six fundamental limitations that motivate the neural approaches in subsequent chapters.

**Previous**: [[01-what-is-language-modeling|Chapter 1 — What is Language Modeling?]]
**Next**: [[03-neural-networks-primer|Chapter 3 — Neural Networks: A Primer]]

---
title: "Chapter 10: Pretraining — Self-Supervised Learning at Scale"
course: Advanced NLP
chapter: 10
last-updated: 2026-04-26
tags: [nlp, pretraining, self-supervised, bert, gpt, t5]
---

# Chapter 10: Pretraining — Self-Supervised Learning at Scale

## Learning objectives

By the end of this chapter you should be able to:
- Explain what self-supervised learning is and why it is powerful
- Describe causal language modeling (CLM) and which models use it
- Describe masked language modeling (MLM) and which models use it
- Explain the T5 span-masking objective
- Describe the concept of emergent properties in large models
- Explain what finetuning is and why it is needed after pretraining

---

## 10.1 The core idea: learn from the data itself

All the models we have discussed so far require labeled data — for each input, a human must provide the correct output. Labeling data is expensive, slow, and limited in scale. The internet contains trillions of words of text, but humans cannot feasibly label all of it.

**Self-supervised learning** solves this by constructing labels automatically from the data itself. We do not need a human to tell the model what a sentence means — we just ask it to predict parts of the sentence from other parts. The correct answer is already in the data.

The power of this approach comes from scale: given an objective that is simple to construct from any raw text, we can train on effectively unlimited data. And as models grow larger and training data grows larger, something remarkable happens: the models spontaneously develop capabilities that were never explicitly trained for — the ability to reason, to write code, to solve math problems, to follow instructions.

---

## 10.2 Causal language modeling (CLM): predicting the next token

The simplest self-supervised objective: predict the next token given all previous tokens.

$$L_{CLM} = -\frac{1}{T} \sum_{t=1}^{T} \log P(w_t \mid w_1, w_2, \ldots, w_{t-1})$$

This is the same objective we discussed in Chapter 1, implemented with a decoder-only transformer (Chapter 9) and a causal attention mask. At each position $t$, the model predicts word $t$ using only positions $1, \ldots, t-1$.

The elegance of this objective is that **a single forward pass over a sequence of $T$ tokens generates $T$ training examples** — one for each position. This makes training extremely efficient.

**Which models**: GPT, GPT-2, GPT-3, GPT-4, Llama, Mistral, Claude, Gemini.

### Emergent properties

When CLM models are trained at sufficient scale, they develop capabilities that emerge suddenly and were not explicitly optimized for:

- **Few-shot learning**: show the model 3 examples of a task in the prompt, and it generalizes to new examples.
- **Chain-of-thought reasoning**: with the right prompt, the model reasons step-by-step.
- **Code generation**: despite training only on next-token prediction of mixed text-code data.
- **Instruction following**: with some finetuning (Chapter 12).

These emergent properties are not understood theoretically, but they are empirically robust and scale roughly predictably with model and data size.

---

## 10.3 Masked language modeling (MLM): BERT

Rather than predicting the next word, MLM randomly masks some tokens in the input and asks the model to predict the original tokens from their context — using both left and right context simultaneously.

### The BERT procedure

1. Take a sentence: `The cat sat on the mat.`
2. Randomly select ~15% of tokens. Of those:
   - 80% of the time: replace with `[MASK]`: `The [MASK] sat on the mat.`
   - 10% of the time: replace with a random word: `The pizza sat on the mat.`
   - 10% of the time: keep unchanged (but still predict): `The cat sat on the mat.`
3. Run the full sequence (with substitutions) through a bidirectional encoder.
4. Compute the loss only at the masked positions.

The random word and keep-unchanged variants prevent the model from learning that `[MASK]` tokens are always the "important" ones — it must treat all tokens as potentially significant.

$$L_{MLM} = -\sum_{i \in \text{masked}} \log P(w_i \mid \mathbf{x}_{\text{corrupted}})$$

### Why bidirectional?

MLM allows every token to attend to every other token (no causal mask). This means the model has access to both left and right context when predicting a masked word — far more information than a left-to-right model.

For the sentence *"The cat sat on the [MASK]"*, a causal model can only use "The cat sat on the" to predict "mat". A bidirectional model also sees what comes after — if there were a period then another sentence starting "The dog ran off the...", that context might help.

### The [CLS] token

BERT prepends a special `[CLS]` (classification) token to every input. At the end of the forward pass, the `[CLS]` token's representation has, through self-attention, aggregated information from the entire sequence. This representation is used as the input to classification heads during finetuning.

### BERT also uses Next Sentence Prediction (NSP)

The original BERT paper added a second pretraining objective: given two segments of text, predict whether the second follows the first in the original document. Later analysis (RoBERTa) found NSP was not helpful and dropped it.

**Which models**: BERT, RoBERTa (no NSP, more data), ELECTRA (predict replaced tokens), DeBERTa.

---

## 10.4 T5: everything is text-to-text

T5 (Text-to-Text Transfer Transformer) is an encoder-decoder model that frames **every NLP task as a text-to-text problem**. Translation is text-to-text. Summarization is text-to-text. Classification is text-to-text (the output is the label name, e.g. "positive"). Question answering is text-to-text.

### T5 pretraining: span masking

T5 uses a variant of MLM called **span corruption**:
1. Sample contiguous spans of tokens to mask (rather than individual tokens).
2. Replace each span with a single sentinel token: `<extra_id_0>`, `<extra_id_1>`, etc.
3. The encoder sees the corrupted input with sentinel tokens.
4. The decoder generates the sentinel tokens followed by the original masked text.

**Input**: `The cat <extra_id_0> the mat.`
**Target**: `<extra_id_0> sat on <extra_id_1>`

The sentinel tokens act as placeholders. The decoder generates a compact sequence — only the missing spans, not the full original sentence — which is more efficient than predicting individual tokens one by one.

**Which models**: T5, mT5 (multilingual), FLAN-T5.

---

## 10.5 The two-stage pipeline: pretrain then finetune

The full recipe for building a capable NLP model is:

**Stage 1 — Pretraining**:
- Architecture: a large transformer (decoder-only, encoder-only, or encoder-decoder)
- Data: as much raw text as possible (trillions of tokens)
- Objective: self-supervised (CLM, MLM, or span masking)
- Goal: learn broad linguistic and world knowledge

**Stage 2 — Finetuning**:
- Start from the pretrained model
- Data: a much smaller labeled dataset specific to the target task
- Objective: task-specific supervised loss
- Goal: adapt the general model to a specific task or domain

Why is pretraining so valuable? Because the model learns from billions of examples to build rich internal representations of language, grammar, facts about the world, and reasoning patterns. Finetuning on a small task-specific dataset then leverages this knowledge — the model does not need to relearn language from scratch for every new task.

This is called **transfer learning**: knowledge learned in one setting (language modeling on the internet) transfers to another (sentiment analysis, translation, question answering).

---

## 10.6 What gets learned during pretraining?

A large pretrained language model implicitly encodes:

- **Syntax**: subject-verb agreement, clause boundaries, nested structures
- **Semantics**: word meanings, entity types, relationships between concepts
- **World knowledge**: facts about history, science, geography, people — everything that appears in the training data
- **Reasoning patterns**: simple logical inferences that appear in text
- **Style and register**: formal vs. informal writing, different genres

None of these were explicitly taught. They are side effects of learning to predict the next token (or a masked token) extremely well across a huge variety of text.

> [!note] The limits of pretraining
> Pretraining on raw text does not teach models to be helpful, honest, or to follow instructions. A model trained only with CLM will generate text that continues whatever prompt it receives — it will happily continue harmful content, write code to attack systems, or generate nonsense. Alignment (Chapter 12) is needed to make models that are useful for human interaction.

---

## Summary

- **Self-supervised learning** constructs training labels automatically from raw data — no human annotation needed.
- **CLM** (causal LM): predict the next token; requires causal masking; the basis of all GPT-style models.
- **MLM** (masked LM): predict randomly masked tokens from bidirectional context; the basis of BERT.
- **Span masking** (T5): mask contiguous spans; decoder generates the missing text; enables text-to-text for all tasks.
- At sufficient scale, CLM models develop **emergent properties** — capabilities not explicitly trained for.
- The **two-stage pipeline** (pretrain → finetune) enables knowledge transfer from massive unlabeled data to specific tasks.

**Previous**: [[09-transformers|Chapter 9 — The Transformer Architecture]]
**Next**: [[11-tokenization|Chapter 11 — Tokenization]]

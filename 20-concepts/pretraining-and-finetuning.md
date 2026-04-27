---
title: Pretraining and Finetuning
status: complete
last-updated: 2026-04-26
tags: [nlp, deep-learning, transfer-learning, llm]
---

# Pretraining and Finetuning

The dominant paradigm for building NLP systems. Train a large model on massive unlabeled data (pretraining), then adapt it to a specific task with a smaller labeled dataset (finetuning).

## Step 1: Pretraining

- **Objective**: self-supervised — the labels are derived from the data itself, no human annotation needed.
- **Scale**: as much data as possible; as large a model as you can afford.
- **Goal**: a model that learns broad linguistic properties — grammar, world knowledge, emergent reasoning — without targeting any specific task.

### Pretraining objectives

**Causal LM (decoder-only, GPT-style)**
Predict the next token given all previous tokens. Straightforward autoregressive objective. Good for text generation.

**Masked LM (encoder-only, BERT-style)**
Randomly mask ~15–30% of tokens; predict the masked tokens given the full bidirectional context. Better for understanding tasks; cannot generate text autoregressively. Masking rate is a hyperparameter — too high loses context for the model to learn from.

**Seq-to-seq (encoder-decoder, T5-style)**
Mask contiguous spans in the encoder input; the decoder generates the missing spans (sentinel tokens). Unlike BERT, the decoder generates token-by-token rather than classifying over the full vocabulary at each masked position.

## Step 2: Finetuning

Adapt the pretrained model to a downstream task/domain using a smaller labeled dataset. Several approaches:

### Full finetuning
Update all model parameters. Most effective but expensive; requires storing a full model copy per task.

### Supervised Finetuning (SFT) / Instruction Tuning
Finetune on a dataset of (instruction, response) pairs. Goal: make the model follow instructions. Not self-supervised — requires labeled data. Limitation: doesn't learn from negative feedback.

### Parameter-Efficient Finetuning (PEFT)
See [[lora]] and [[prompt-tuning]]. Keep pretrained parameters frozen; update only a small set of new parameters.

## BERT pretraining detail

- `[CLS]` token prepended to every input sequence; its final representation is used for classification tasks.
- `[MASK]` token replaces selected tokens; loss is computed only on masked positions.
- Uses unmasked (bidirectional) attention — the model can see the full sequence.

## GPT finetuning detail

- Decoder-only: uses causal (masked) attention.
- The prefix (prompt) is passed unmasked; the model generates continuations autoregressively.
- Used for text generation; can be finetuned for classification by predicting a label token.

## T5 detail

- Encoder-decoder.
- Frames all NLP tasks as text-to-text: input is text, output is text.
- Sentinel tokens represent masked spans in the encoder input.
- Capable of being finetuned for any task without architectural changes.

## Sources

- [[adv-nlp-course-notes]]

## See also

- [[transformers]]
- [[lora]]
- [[rlhf]]
- [[tokenization]]

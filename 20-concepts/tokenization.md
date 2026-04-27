---
title: Tokenization
status: complete
last-updated: 2026-04-26
tags: [nlp, preprocessing, language-models]
---

# Tokenization

The process of converting raw text into a sequence of integer indices (token IDs) that a model can process.

$$\text{"Students opened their books"} \xrightarrow{\text{tokenizer}} [11, 298, 34, 567, \ldots]$$

## Approaches

### Word-level
Split on whitespace. Simple, but:
- Unknown words at test time → `<UNK>` token (loses information)
- Treats morphological variants (open, opened, opening) as entirely separate types
- Vocabulary size = number of unique words → large

### Character-level
Each character is a token. No unknown words. Problems:
- Sequences become very long → quadratic attention cost
- Inference is slow (many more tokens per sequence)

### Subword tokenization (the standard)
Balance between word-level and character-level. Splits rare/unknown words into known subword units while keeping frequent words intact.

**BPE (Byte-Pair Encoding)**
Originally introduced for machine translation; later used in GPT, BERT, T5.

Algorithm:
1. Start with a character-level vocabulary.
2. Count all character-pair frequencies in the corpus.
3. Merge the most frequent pair into a new subword unit and add to vocabulary.
4. Repeat for a fixed number of merge steps.
5. Stop. The resulting vocabulary balances coverage vs. size.

As long as all individual characters appear in training data, no `<UNK>` tokens are needed.

**WordPiece**
Merges pairs by likelihood under a language model, not raw frequency. Used in BERT.

**SentencePiece**
A library for subword tokenization that does not require pre-tokenization (whitespace splitting). Useful for languages that don't use spaces between words (Thai, Chinese, Japanese).

## Multilingual tokenization

Large LLMs use multilingual vocabularies. A language's share in the training corpus roughly determines its share of vocabulary slots. Languages with agglutinative or non-concatenative morphology (Turkish, Arabic) are harder to tokenize with BPE.

**ByT5**: T5 trained on byte-level tokenization — each byte is a token. Vocabulary size = 256. Sequences are very long; uses a heavy encoder and light decoder to compensate.

## Typical vocabulary sizes

| Model | Vocabulary size |
|---|---|
| GPT-2 | 50,257 |
| BERT | 30,522 |
| T5 | 32,128 |
| General multilingual | 32k–64k |

Unicode has ~138k symbols; BPE normalizes and merges to reach a practical size.

## Sources

- [[adv-nlp-course-notes]]

## See also

- [[pretraining-and-finetuning]]
- [[word-embeddings]]

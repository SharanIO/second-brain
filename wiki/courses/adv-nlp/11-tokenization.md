---
title: "Chapter 11: Tokenization"
course: Advanced NLP
chapter: 11
last-updated: 2026-04-26
tags: [nlp, tokenization, bpe, preprocessing]
---

# Chapter 11: Tokenization

## Learning objectives

By the end of this chapter you should be able to:
- Explain why tokenization is necessary and what it produces
- Describe the limitations of word-level and character-level tokenization
- Explain the BPE algorithm step by step
- Describe WordPiece, SentencePiece, and when each is used
- Understand the challenges of multilingual tokenization

---

## 11.1 What is tokenization and why do we need it?

A neural language model operates on **integers**, not text. Before any processing can begin, we must convert a raw string like *"The cat sat."* into a sequence of integer indices: $[464, 3797, 3332, 13]$.

Each integer is a **token ID** — it indexes into the vocabulary $V$, a fixed lookup table mapping integers to string fragments. The inverse mapping (integer → string) is also stored, so we can convert model outputs back to readable text.

Tokenization is deceptively consequential. A poor tokenization scheme can:
- Explode sequence length (making attention expensive)
- Create unknown tokens at inference time
- Treat morphologically related words as completely unrelated
- Fail entirely for languages the scheme was not designed for

Getting tokenization right is one of the practical engineering challenges of building LLMs.

---

## 11.2 Word-level tokenization: the naive approach

The most intuitive approach: split text on whitespace and punctuation, and treat each resulting word as one token.

**Vocabulary**: the set of all unique words in the training corpus. For a typical English corpus, this is 100,000–500,000 words.

**Problems**:

1. **Unknown words ($<$UNK$>$)**: any word not seen during training gets mapped to a special `<UNK>` token. At test time, proper nouns, technical terms, and rare words all collapse to the same representation. The model loses all information about what the word actually was.

2. **Morphological blindness**: "run", "runs", "running", "runner" are treated as four completely separate vocabulary entries, each requiring its own row in the embedding matrix. The model cannot generalize from training data about "run" to help predict "running".

3. **Vocabulary explosion**: in morphologically rich languages (Finnish, Turkish, Arabic), the number of distinct word forms can be enormous. A million-word vocabulary is still insufficient.

4. **Untreated OOV at inference**: a word like "ChatGPT" that did not exist when the model was trained gets turned into `<UNK>`, losing its meaning entirely.

---

## 11.3 Character-level tokenization: too fine-grained

At the other extreme, treat every individual character (a, b, c, ..., space, punctuation) as a token. Vocabulary size becomes tiny — 256 bytes for pure byte-level, or a few hundred for Unicode character-level.

**Advantages**: no unknown tokens (every input can be represented), handles all languages, handles arbitrary new words.

**Problems**:

1. **Very long sequences**: a 100-word sentence might become 500 characters. Since self-attention is $O(T^2)$ in sequence length, this makes training and inference much more expensive.

2. **Learning burden**: the model must learn to associate character sequences with meaning from scratch, rather than starting from meaningful word-level units. This requires more training and more parameters.

3. **Not how humans read**: humans read words as units, not individual letters. Character-level models effectively ask the model to learn word boundaries on top of everything else.

---

## 11.4 Subword tokenization: the practical solution

The key insight: **common words should be one token; rare words should be split into subword pieces**. This balances vocabulary size against sequence length and eliminates unknown words.

"running" → one token (common word)
"antidisestablishmentarianism" → ["anti", "dis", "establish", "ment", "arian", "ism"] (rare word, split into common pieces)

As long as all individual characters (or bytes) are in the vocabulary, any string can be represented — never `<UNK>`.

---

## 11.5 BPE: Byte-Pair Encoding

BPE is the most widely used subword tokenization algorithm, used in GPT-2, GPT-3, Llama, Mistral, and many others. Originally developed for machine translation [Sennrich et al., 2016].

### The algorithm

1. **Initialize** the vocabulary with all individual characters (or bytes) in the training corpus.

2. **Count** the frequency of all adjacent token pairs in the corpus.

3. **Merge** the most frequent pair into a new token. Add this new token to the vocabulary.

4. **Update** the corpus by replacing all occurrences of that pair with the new merged token.

5. **Repeat** steps 2–4 for a fixed number of merge operations (a hyperparameter).

### Worked example

Suppose the training corpus contains these words with frequencies:

| Word | Frequency |
|---|---|
| hug | 10 |
| pug | 5 |
| pun | 12 |
| bun | 4 |
| huge | 5 |

Initial vocabulary (characters): `{h, u, g, p, n, b, e}`

Corpus as character sequences:
- `h u g` (10 times), `p u g` (5 times), `p u n` (12 times), `b u n` (4 times), `h u g e` (5 times)

Count pair frequencies:
- `(u, g)`: 10 + 5 + 5 = 20 ← most frequent
- `(p, u)`: 5 + 12 = 17
- `(u, n)`: 12 + 4 = 16
- `(h, u)`: 10 + 5 = 15

Merge `(u, g)` → `ug`. Vocabulary now includes `ug`.

New corpus: `h ug` (10), `p ug` (5), `p u n` (12), `b u n` (4), `h ug e` (5)

Repeat. Eventually `hug` becomes a single token (high frequency), while `antidisestablishmentarianism` is represented as a sequence of smaller subwords.

### Key properties

- **No unknown tokens**: the base vocabulary includes all characters/bytes. Any string is encodable.
- **Frequency-driven**: common words end up as single tokens; rare words are split.
- **Merge count is the main hyperparameter**: more merges → larger vocabulary → shorter sequences. Typical vocabulary sizes: 32k–100k.
- **Pretokenization**: before BPE is applied, the text is typically split on whitespace so that tokens do not cross word boundaries. "cat" and " cat" (with leading space) become separate tokens in GPT-style tokenization.

---

## 11.6 WordPiece and SentencePiece

### WordPiece (BERT) [Schuster & Nakamura, 2012; Wu et al., 2016]

Similar to BPE but the merge criterion is different: instead of merging the *most frequent* pair, it merges the pair that **maximizes the language model likelihood** — i.e., the pair whose merging increases the probability of the training data the most.

This tends to produce slightly different subword splits than BPE, but the practical difference is small. Subwords that are part of a larger word (but not at the start) are marked with `##`: "running" → `["run", "##ning"]`.

### SentencePiece [Kudo & Richardson, 2018]

A library that implements both BPE and WordPiece, with one important difference: it operates directly on the raw text string **without any pretokenization**. It treats the space character as a regular character, and uses a special `▁` marker to indicate the beginning of a word.

This is critical for languages that do not use spaces to separate words — Japanese, Chinese, Thai — where whitespace-based pretokenization would fail entirely. SentencePiece is used in T5, Llama (via its own variant), and many multilingual models.

---

## 11.7 The multilingual challenge

Most tokenizers are designed around English, which uses spaces between words and has a limited morphological system. Many of the world's languages have features that break these assumptions:

- **No spaces**: Chinese, Japanese, Thai write words without whitespace separators.
- **Agglutinative morphology**: Turkish, Finnish attach many grammatical suffixes to a root, creating enormous numbers of distinct forms. BPE applied to Turkish needs many more merges to handle this well.
- **Non-concatenative morphology**: Arabic and Hebrew encode meaning by inserting vowels into consonantal roots, which BPE's linear merging cannot model naturally.
- **Different scripts**: Greek, Arabic, Chinese, Devanagari all require their own character inventory.

In multilingual models (mBERT, mT5, BLOOM), the vocabulary must accommodate all languages simultaneously. Languages with large training corpora get more vocabulary entries; low-resource languages are underrepresented and their texts are tokenized into many more pieces — making them disproportionately expensive to process.

**ByT5** [Xue et al., 2022] takes the opposite approach: train directly on **bytes** (vocabulary size = 256). No tokenization design decisions required; every language is handled uniformly. The tradeoff is very long sequences and a heavier encoder, but the model generalizes well across languages and handles novel inputs robustly.

---

## 11.8 Practical consequences of tokenization choices

Tokenization has subtle effects on model behavior:

- Numbers are often split character-by-character: "12345" might become `["1", "2", "3", "4", "5"]`. Arithmetic is harder when numbers are fragmented.
- Whitespace and capitalization are significant: "Cat" and "cat" may be different tokens.
- Code tokenization: programming languages have different statistical properties than prose. GPT-4 uses a large vocabulary specifically designed to handle code more efficiently.
- Context length: GPT-4 with a 128k token context window is not 128k words — it is ~100k words (since most words are 1–2 tokens in English, but more in other languages).

---

## Summary

- Tokenization converts raw text to integer IDs; the vocabulary maps integers back to string fragments.
- **Word-level**: simple but causes unknown words, morphological blindness, and vocabulary explosion.
- **Character-level**: no unknowns but very long sequences and heavy learning burden.
- **BPE** merges the most frequent character pairs iteratively until the vocabulary reaches a target size. Common words → one token; rare words → subword pieces. No unknowns.
- **WordPiece** (BERT) merges by LM likelihood; **SentencePiece** avoids pretokenization for languages without spaces.
- Multilingual models must balance vocabulary allocation across languages; low-resource languages get fewer tokens and longer sequences.

**Previous**: [[10-pretraining-objectives|Chapter 10 — Pretraining]]
**Next**: [[12-finetuning-peft-alignment|Chapter 12 — Finetuning, PEFT, and Alignment]]

---

## References

- **ADV NLP Course Notes** — primary source for the tokenization survey, the hug/pug BPE example, BPE algorithm steps, and multilingual challenges. See [[adv-nlp-course-notes]].
- Sennrich, R., Haddow, B., & Birch, A. (2016). Neural Machine Translation of Rare Words with Subword Units. *ACL*. — BPE for NMT; the paper that popularized subword tokenization.
- Schuster, M. & Nakamura, K. (2012). Japanese and Korean Voice Search. *ICASSP*. — original WordPiece paper.
- Wu, Y. et al. (2016). Google's Neural Machine Translation System: Bridging the Gap between Human and Machine Translation. *arXiv:1609.08144*. — WordPiece as used in production at Google; later adopted for BERT.
- Kudo, T. & Richardson, J. (2018). SentencePiece: A simple and language independent subword tokenizer and detokenizer for Neural Text Processing. *EMNLP*. — SentencePiece library.
- Kudo, T. (2018). Subword Regularization: Improving Neural Network Translation Models with Multiple Subword Candidates. *ACL*. — unigram language model tokenization (alternative to BPE within SentencePiece).
- Xue, L. et al. (2022). ByT5: Towards a Token-Free Future with Pre-trained Byte-to-Byte Models. *TACL*. — byte-level tokenization with T5.
- Jurafsky, D. & Martin, J. H. (2023). *Speech and Language Processing* (3rd ed. draft), Ch. 2. — comprehensive tokenization coverage.

> [!todo] cite this — add a reference for the exact Unicode character count (~138k symbols) and standard multilingual vocab size conventions (32k–64k).

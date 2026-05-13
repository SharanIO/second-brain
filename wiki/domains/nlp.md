---
type: domain
title: Natural Language Processing (NLP)
status: active
last-updated: 2026-05-13
tags: [nlp, deep-learning, transformers, alignment]
---

# Natural Language Processing (NLP)

The field of enabling machines to understand, generate, and reason over human language. The modern NLP stack is almost entirely transformer-based, built on top of large-scale self-supervised pretraining.

## The arc of the field

```
Statistical LMs         Neural LMs           Transformers          Alignment
(n-grams, sparse)  →  (embeddings, MLP)  →  (attention-based)  →  (RLHF, PEFT)
     1990s                  2013                  2017                 2022+
```

---

## Concept map

### 1. Statistical foundations
The baseline before neural methods. Still useful for understanding perplexity, smoothing, and the sparsity problem.

- [[n-gram-models]] — Markov assumption, perplexity, one-hot encoding, sparsity

---

### 2. Neural foundations
The building blocks shared across all modern architectures. Understanding these deeply means you can read any paper.

- [[word-embeddings]] — dense representations, Word2Vec, GloVe, semantic geometry
- [[activation-functions]] — ReLU, GELU, SiLU; why GELU dominates in transformers
- [[weight-initialization]] — Xavier, Kaiming; why initialization determines training stability
- [[dropout]] — regularization via stochastic deactivation
- [[regularization]] — L2 vs weight decay, and why they diverge under Adam
- [[optimizers]] — SGD → Momentum → AdaGrad → RMSProp → Adam → AdamW → Adafactor

---

### 3. Sequence modeling
How models handle order and dependencies before transformers.

- [[recurrent-neural-networks]] — hidden states, vanishing gradient, seq-to-seq bottleneck

---

### 4. Attention and transformers
The paradigm shift. Everything modern is built on this.

- [[attention-mechanism]] — QKV formulation, self-attention, KV-cache, masking
- [[positional-encoding]] — sinusoidal vs learned, why attention needs explicit position info
- [[transformers]] — full architecture, decoder-only vs encoder-only vs encoder-decoder, tensor shapes

---

### 5. Training at scale
How large models are actually built and adapted.

- [[tokenization]] — BPE, WordPiece, SentencePiece, ByT5; why tokenization choices matter
- [[pretraining-and-finetuning]] — self-supervised objectives, masked LM, causal LM, SFT
- [[lora]] — low-rank adaptation, QLoRA; parameter-efficient finetuning

---

### 6. Alignment
Making models do what humans actually want.

- [[rlhf]] — SFT → reward model (Bradley-Terry) → PPO; Constitutional AI; RLAIF

---

## Learning resource

- `courses/adv-nlp/` — 12-chapter structured notes covering this full arc (n-gram models → RLHF)
- [[adv-nlp-course-notes]] — source synthesis note with key insights and critiques

---

## What's missing from this wiki

- **Evaluation** — BLEU, ROUGE, BERTScore, perplexity benchmarks; no concept page yet
- **Embeddings at scale** — sentence transformers, dense retrieval, FAISS; no concept page yet
- **Inference optimization** — quantization, speculative decoding, KV-cache management; no concept page yet
- **Multimodal** — vision-language models (CLIP, LLaVA); no concept page yet
- **Prompting** — chain-of-thought, few-shot, RAG; no concept page yet
- **Specific models** — GPT, BERT, LLaMA, Mistral; no entity pages yet

---

## Connections to other domains

- [[robotics]] — language-conditioned manipulation, VLMs for robot policies
- `domains/algorithms.md` — attention is O(n²) in sequence length; algorithmic efficiency matters at scale

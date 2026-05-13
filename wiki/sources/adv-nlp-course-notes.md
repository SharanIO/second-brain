---
title: "Advanced NLP — Course Notes"
type: course-notes
source: raw/papers/NLP/NLP.md
status: integrated
last-updated: 2026-04-26
tags: [nlp, deep-learning, transformers, language-models]
---

# Advanced NLP — Course Notes

Handwritten lecture notes from an Advanced NLP course, OCR'd into markdown (~130 pages). Not a research paper — personal study notes spanning the full arc from classical n-gram models to RLHF alignment.

> [!warning] Source anomaly
> Pages 125–130 of the original PDF contain an unrelated lease contract document, likely a scanning accident. Ignore that content.

## Contribution

A dense but cohesive tour of the NLP stack: statistical LMs → neural LMs → RNNs → attention → transformers → pretraining/finetuning → PEFT → alignment. Strong on implementation intuition (PyTorch code, tensor shapes, optimizer internals). Weaker on formal derivations and citations.

## Structure

The notes progress roughly chronologically through the field:

1. **Statistical language models** — n-grams, Markov assumption, perplexity, one-hot encoding, sparsity
2. **Neural language models** — word embeddings, MLP for LM, softmax, loss functions
3. **Training mechanics** — forward pass, backprop (chain rule worked examples), SGD variants
4. **Architectural components** — activation functions (ReLU, GELU, SiLU), weight initialization (Xavier, Kaiming), dropout, layer normalization
5. **Optimizers** — SGD → Momentum → AdaGrad → RMSProp → Adam → AdamW → Adafactor
6. **Regularization** — L2 penalty vs. weight decay (and why they differ under Adam)
7. **RNNs** — hidden states, vanishing gradient, seq-to-seq bottleneck problem
8. **Attention mechanism** — QKV formulation, self-attention, parallelism during training vs. sequential at test time, KV-cache
9. **Transformers** — multi-head attention (Vaswani et al. 2017), residual connections, positional encoding, decoder-only vs. encoder-only vs. encoder-decoder
10. **Pretraining & finetuning** — self-supervised objectives, masked LM (BERT), causal LM (GPT), T5, SFT
11. **Tokenization** — word-level, character-level, BPE (Byte-Pair Encoding), WordPiece, SentencePiece, ByT5
12. **PEFT** — prompt tuning, prefix tuning, LoRA, QLoRA
13. **Alignment** — RLHF, reward modeling (Bradley-Terry), PPO, RAFT, Constitutional AI

## Key results / insights worth keeping

- Infini-gram dynamically chooses n for n-gram models.
- GELU is preferred over ReLU in transformers because it produces smooth, curved gradients; it approximates a "grading curve" on the input distribution.
- Kaiming init compensates for the half-distribution killed by ReLU/GELU; Xavier assumes linear activation near zero.
- AdamW decouples weight decay from gradient moving averages; plain Adam conflates them, degrading regularization.
- Self-attention is parallelizable during training (full sequence available) but sequential at inference (autoregressively generates token by token).
- KV-cache stores previously computed key/value vectors to avoid recomputation at each decoding step.
- LoRA decomposes the weight update $\Delta W$ into two low-rank matrices $A$ ($m \times r$) and $B$ ($r \times n$), reducing trainable params from $mn$ to $r(m+n)$.
- RLHF limitation: SFT doesn't learn from negative feedback; reward model + PPO addresses this but is expensive.

## Critique

The notes are genuinely useful as an implementation-oriented survey. The main weaknesses:
- Heavy OCR noise in later sections (repeated "Virnasked" artifacts from page 119's table cell extraction).
- Citation practice is inconsistent — Vaswani et al. 2017 and "Ran Mooney 2014" (on the seq-to-seq bottleneck) are named but not fully cited.
- The RLHF section is abbreviated; the PPO details are cut off.
- Misclassified as `raw/papers/` — these are course notes and would fit better in `raw/transcripts/` or a dedicated `50-courses/adv-nlp/` folder.

## Concept pages created

- [[n-gram-models]]
- [[word-embeddings]]
- [[activation-functions]]
- [[weight-initialization]]
- [[dropout]]
- [[optimizers]]
- [[attention-mechanism]]
- [[transformers]]
- [[recurrent-neural-networks]]
- [[pretraining-and-finetuning]]
- [[tokenization]]
- [[lora]]
- [[rlhf]]

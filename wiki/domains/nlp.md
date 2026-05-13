---
type: domain
title: Natural Language Processing (NLP)
status: active
last-updated: 2026-05-13
tags: [nlp, deep-learning, transformers, alignment, language-models]
---

# Natural Language Processing (NLP)

Natural Language Processing is the subfield of machine learning concerned with enabling machines to understand, generate, and reason over human language. Over the past decade, NLP has undergone one of the most dramatic paradigm shifts in all of science: from hand-engineered features and statistical counting methods, to learned dense representations, to recurrent networks, to the transformer architecture that now underlies almost every state-of-the-art system. The current generation of large language models — GPT-4, Claude, Llama, Gemini — are all transformers trained on internet-scale text with self-supervised objectives and refined with human feedback.

Understanding NLP at depth means understanding *why* each development was necessary — what broke in the previous paradigm, what the new approach fixed, and what it introduced in turn. This domain page maps that arc and links to every concept page in the vault.

---

## The arc of the field

The history of NLP can be told as a sequence of bottlenecks and the solutions invented to overcome them:

```
Statistical LMs         Neural LMs           Sequence Models      Transformers          Alignment
(n-grams, sparse)  →  (embeddings, MLP)  →  (RNNs, attention) →  (full attention)  →  (RLHF, PEFT)
     1990s                2013–2015             2015–2017              2017+               2022+
```

Each step was motivated by a concrete failure mode in the previous approach:

- **N-grams** captured local context but suffered from exponential sparsity. You can't store all 5-grams in English.
- **Embeddings** replaced sparse one-hot vectors with dense representations, giving semantically similar words similar vectors — but early neural LMs still treated sequences as fixed-length windows.
- **RNNs** processed sequences of arbitrary length with shared weights, but compressed all prior context into a fixed-size hidden vector. Long-range dependencies were lost; training was sequential and slow.
- **Attention** let the decoder look back at *all* encoder states simultaneously, breaking the bottleneck. Vaswani et al. (2017) then asked: if attention can replace the recurrent bottleneck, why keep the recurrence at all?
- **Transformers** dropped recurrence entirely and built purely on attention. This enabled full parallelism during training and scaled to billions of parameters.
- **RLHF and PEFT** addressed the alignment problem: a model trained to predict text doesn't automatically do what humans want. LoRA and instruction tuning made adaptation cheap; RLHF made it steerable.

---

## Concept map

### 1. Statistical foundations

The starting point of the field and the baseline that motivated everything else. N-gram models are worth understanding not because they're used in practice, but because their failure modes are precisely what every subsequent development was designed to fix.

[[n-gram-models]] — An n-gram model estimates the probability of the next word by conditioning on the $n-1$ preceding words, making the **Markov assumption** that only recent history matters. Every word in the vocabulary is represented as a one-hot vector — a binary vector with a single 1 at the word's index. This representation has a fatal flaw: the dot product between any two distinct words is zero, meaning the model has no notion of semantic similarity. "Cat" and "kitten" are as dissimilar as "cat" and "refrigerator." Evaluating these models uses **perplexity**: the geometric mean inverse probability assigned to a test set. Lower is better — a model that is very confident about the right next word is less "perplexed." The core limitation is **sparsity**: as $n$ grows, the number of possible n-gram combinations grows exponentially ($O(e^n)$), and most of them never appear in training data. This zero-probability problem persists unless smoothing is applied, and no amount of smoothing fixes the underlying representational poverty of one-hot vectors.

---

### 2. Neural foundations

Once you replace one-hot vectors with learned dense representations, you need a training infrastructure: activations that introduce nonlinearity, weight initialization that prevents signal collapse, regularization that prevents overfitting, and optimization algorithms that update parameters efficiently. These are not NLP-specific — they're the shared machinery of all deep learning — but understanding them precisely is what separates someone who can read a paper from someone who can debug a training run.

[[word-embeddings]] — The core fix for one-hot sparsity. A weight matrix $E \in \mathbb{R}^{|V| \times d}$ maps each token index to a dense $d$-dimensional vector. The key properties are that (1) similar words end up close in embedding space — "king" and "queen" are nearby, "cat" and "dog" are nearby — and (2) the space has geometric structure: the classic example is that $\text{king} - \text{man} + \text{woman} \approx \text{queen}$. In a transformer, token embeddings are summed with positional encodings before being passed to attention layers: $x_i = E[w_i] + \text{pos}(i)$. The embedding matrix is a learned parameter updated by backpropagation, but notably weight decay is rarely applied to it.

[[activation-functions]] — Nonlinear functions applied after linear transformations. Without them, stacked linear layers collapse to a single linear transformation and the network loses all expressive power. **ReLU** ($\max(0, x)$) is the classic workhorse but suffers from dead neurons: if a neuron's preactivation is always negative, its gradient is always zero and it never recovers. **GELU** (Gaussian Error Linear Unit, $x \cdot \Phi(x)$) is the modern standard in transformers — it produces smooth, curved gradients by weighting each input by the probability that it would survive a Gaussian gating. Think of it as a probabilistic version of ReLU: large positive inputs pass through fully, small or negative inputs are smoothly attenuated. GELU is used in BERT, GPT, and virtually all modern architectures because its smooth gradient landscape makes training more stable. The choice of activation directly determines the right initialization strategy.

[[weight-initialization]] — Weights must be initialized so that activations neither vanish nor explode as the signal propagates through layers. The goal is variance preservation: if each layer multiplies the signal by a random matrix, the variance of the activations should remain approximately constant. **Xavier (Glorot) initialization** achieves this for symmetric activations like sigmoid and tanh by sampling from $\mathcal{N}(0, \frac{2}{n_{in} + n_{out}})$. **Kaiming (He) initialization** compensates for the fact that ReLU and GELU zero out the negative half of the input distribution, halving the signal energy — so it doubles the variance: $\mathcal{N}(0, \frac{2}{n_{in}})$. Using the wrong initialization for a given activation can cause training to fail entirely, even for a correctly designed architecture.

[[dropout]] — A regularization technique that randomly deactivates neurons during training. At each forward pass, each neuron is kept with probability $(1-p)$ and zeroed with probability $p$, and the surviving activations are scaled up by $\frac{1}{1-p}$ to preserve expected magnitude. At inference, all neurons are active with no scaling needed. Dropout works because it forces the network to build redundant representations — no neuron can rely on any specific other neuron always being present, so each must learn to carry information independently. The ensemble interpretation: dropout is equivalent to training and averaging an exponentially large ensemble of subnetworks, each sharing parameters. Typical values are $p = 0.1$–$0.2$ in transformer attention layers.

[[regularization]] — Techniques that penalize complexity to prevent overfitting. **L2 regularization** adds a penalty term $\frac{\lambda}{2}\sum w^2$ to the loss, encouraging weights to stay small. **Weight decay** applies the shrinkage directly: $w_{\text{new}} = w_{\text{old}} - \eta \nabla L - \lambda w_{\text{old}}$. For SGD, these two formulations are mathematically identical. For **Adam**, they are not — standard Adam folds the $\lambda w$ term into the adaptive gradient moving averages, which distorts the regularization. **AdamW** corrects this by decoupling weight decay from the gradient update, applying it as a direct multiplicative shrinkage after the gradient step. This is why every modern LLM uses AdamW rather than Adam.

[[optimizers]] — The full progression from vanilla SGD to the modern standard: **SGD with Momentum** adds a velocity term to smooth oscillations; **AdaGrad** uses per-parameter adaptive learning rates scaled by accumulated gradient history, but the accumulation only grows, eventually stopping learning; **RMSProp** fixes this with an exponential moving average of squared gradients; **Adam** combines momentum (first moment) and RMSProp (second moment) with bias correction for early steps; **AdamW** decouples weight decay; **Adafactor** factorizes the moment matrices for memory efficiency at 70B+ scale. The progression is not arbitrary — each step addresses a concrete failure mode of the previous one. The single most important hyperparameter throughout is the learning rate.

---

### 3. Sequence modeling

With embeddings and training machinery in place, the next question is architecture: how do you actually process a sequence of variable length?

[[recurrent-neural-networks]] — RNNs process sequences one token at a time, maintaining a hidden state $h_t = f(W_h h_{t-1} + W_e e_t)$ that accumulates information from all previous tokens. This solved the fixed-window problem of early neural LMs, but introduced new ones. The **vanishing gradient** problem: backpropagating through many timesteps causes gradients to shrink exponentially, so the network cannot learn dependencies spanning more than ~20 tokens. The **bottleneck problem**: in seq-to-seq tasks (translation, summarization), the encoder must compress the entire source sentence into a single fixed-size vector, which Ran Mooney described as fundamentally impossible for complex sentences. The **sequential computation** problem: $h_t$ depends on $h_{t-1}$, so all timesteps must be computed in order — no parallelism across positions during training. LSTMs and GRUs addressed the vanishing gradient with gating mechanisms but not the bottleneck or parallelism issues. The RNN bottleneck directly motivated the attention mechanism.

---

### 4. Attention and transformers

The paradigm shift. Attention was first introduced as an augmentation to RNNs to break the encoder bottleneck; Vaswani et al. (2017) then used it as the sole architectural primitive, eliminating recurrence entirely.

[[attention-mechanism]] — Attention lets the model dynamically weight which parts of the input are most relevant when producing each output token. Every token is projected into three vectors: a **Query** $Q$ ("what am I looking for?"), a **Key** $K$ ("what do I advertise?"), and a **Value** $V$ ("what information do I provide?"). Relevance is computed as scaled dot product between Q and K: $\text{Attention}(Q,K,V) = \text{softmax}(\frac{QK^\top}{\sqrt{d_k}})V$. The $\sqrt{d_k}$ scaling is critical — without it, dot products in high dimensions push the softmax into near-zero gradient saturation. **Self-attention** allows every token to attend to every other token in the same sequence, capturing arbitrary long-range dependencies in a single layer. Training is fully parallel since the entire sequence is available; inference is sequential because each token must be generated before the next. **KV-cache** avoids recomputing key/value vectors at each decoding step. **Causal masking** prevents a position from attending to future tokens during autoregressive generation.

[[positional-encoding]] — Self-attention is permutation-invariant: the output is the same regardless of input order. Positional encodings inject order information by adding a position-dependent vector to each token embedding: $x_i = E[w_i] + \text{pos}(i)$. Vaswani et al. used fixed **sinusoidal encodings** — sine and cosine functions of different frequencies for each dimension — which generalize to sequence lengths beyond training. **Learned positional embeddings** are a lookup table trained end-to-end but limited to the training context length. **Relative encodings** (RoPE, ALiBi) encode the *distance* between tokens rather than absolute positions, significantly improving length generalization and now standard in modern LLMs like Llama and Mistral.

[[transformers]] — The full architecture stacks layers, each containing two sublayers: multi-head self-attention and a position-wise feed-forward network (FFN). Each sublayer is wrapped with a residual connection and layer normalization: $\text{output} = \text{LayerNorm}(x + \text{sublayer}(x))$. **Multi-head attention** runs $h$ parallel attention heads, each with its own projections, allowing the model to simultaneously attend to different aspects of the sequence (syntax, coreference, semantics). The three standard variants are: **decoder-only** (GPT family, causal masking, trained with next-token prediction, best for generation); **encoder-only** (BERT family, bidirectional attention, trained with masked LM, best for classification and representation); **encoder-decoder** (T5 family, encoder reads the full input, decoder generates output autoregressively with cross-attention to encoder representations, best for seq-to-seq tasks). A concrete tensor walkthrough for a 3-token sequence with $d=512$ and 8 heads: input IDs $[3]$ → embedding $[3 \times 512]$ → add positional encoding $[3 \times 512]$ → split into heads $[8, 3, 64]$ → self-attention $[3 \times 512]$ → residual + LayerNorm $[3 \times 512]$ → FFN $[3 \times 512]$.

---

### 5. Training at scale

Building a transformer and training it to predict text is one thing; building a large language model involves additional engineering decisions at the input level (tokenization) and the learning paradigm level (pretraining + finetuning).

[[tokenization]] — Raw text must be converted to integer token IDs before it can be fed to a model. Word-level tokenization produces unknown tokens for anything outside the training vocabulary. Character-level avoids unknowns but produces very long sequences with quadratic attention cost. **Subword tokenization** is the modern standard: frequent words stay intact; rare or novel words are split into known subword units. **BPE (Byte-Pair Encoding)** starts with individual characters and iteratively merges the most frequent adjacent pair into a new token. This guarantees no unknown tokens as long as all individual characters appear in training. **WordPiece** (BERT) merges pairs by language model likelihood rather than raw frequency. **SentencePiece** operates directly on raw text without requiring pre-tokenization, making it suitable for languages that don't use spaces. **ByT5** goes to the extreme of byte-level tokenization — vocabulary size 256 — with very long sequences offset by a heavy encoder. Tokenization choices have deep consequences: a language's share of the training corpus determines how many vocabulary slots it gets, and morphologically complex languages (Turkish, Arabic) are systematically under-resourced.

[[pretraining-and-finetuning]] — The dominant paradigm: train a general-purpose model on massive unlabeled data with a self-supervised objective, then adapt it to a specific task. **Causal LM** (GPT-style): predict the next token given all previous tokens. Straightforward autoregressive objective, naturally suited to generation. **Masked LM** (BERT-style): randomly mask 15–30% of tokens and predict them from full bidirectional context. Better for understanding tasks; cannot generate. **Seq-to-seq** (T5-style): mask contiguous spans in the encoder input; the decoder generates the masked spans using sentinel tokens. **Supervised Finetuning (SFT)** adapts a pretrained model to follow instructions using (prompt, response) pairs, but has a key limitation: it learns only from positive examples and cannot learn from negative feedback. This limitation is what RLHF was designed to address.

[[lora]] — Full finetuning of a large model requires updating all parameters and storing a full model copy per task — prohibitively expensive. LoRA (Low-Rank Adaptation) decomposes the weight update $\Delta W$ into two low-rank matrices: $\Delta W = AB^\top$ where $A \in \mathbb{R}^{m \times r}$, $B \in \mathbb{R}^{n \times r}$, and $r \ll \min(m, n)$. The forward pass becomes $h = f((W_{\text{pre}} + AB^\top)x)$, with $W_{\text{pre}}$ frozen and only $A, B$ updated. The motivation is that neural networks' weight updates during task adaptation tend to be low-rank — the meaningful change lies in a much lower-dimensional subspace than the full matrix. For a $4096 \times 4096$ matrix with $r=8$, LoRA reduces trainable parameters from ~16M to ~65k. **QLoRA** combines quantization (compressing the frozen weights to 4-bit integers) with LoRA adapters on top, enabling finetuning of 70B+ models on consumer GPUs.

---

### 6. Alignment

A model trained to predict text is not the same as a model trained to be helpful. It may produce toxic, false, or unhelpful outputs because statistical text prediction rewards plausibility, not usefulness. Alignment is the problem of making models do what humans actually want.

[[rlhf]] — Reinforcement Learning from Human Feedback is the three-step pipeline that transformed raw pretrained LMs into instruction-following assistants. **Step 1**: Supervised Finetuning on (instruction, response) pairs to produce a model capable of following instructions at all. **Step 2**: Reward model training — collect preference data by generating multiple responses per prompt and having humans rank them; train a reward model $r(x, y)$ to predict which response is preferred using the Bradley-Terry pairwise model: $P(y_w \succ y_l \mid x) = \sigma(r(x, y_w) - r(x, y_l))$; the loss is $-\log \sigma(r(x, y_w) - r(x, y_l))$. **Step 3**: RL optimization — use PPO (Proximal Policy Optimization) to optimize the SFT model against the reward signal, with a KL penalty against the reference SFT model to prevent reward hacking. **RAFT** (Reward rAnked Fine Tuning) is a simpler alternative: generate best-of-n samples, keep the highest-scoring ones, finetune on those. **RLAIF** (RL from AI Feedback, Constitutional AI) replaces expensive human annotators with a capable LLM as the judge, making the pipeline dramatically cheaper. The key limitation throughout: the reward is sparse (observed only at the end of a full response), making credit assignment hard.

---

## Learning resources in this vault

- `wiki/courses/adv-nlp/` — 12 structured chapters covering this full arc from n-gram models to RLHF, with PyTorch code, tensor shapes, and optimizer internals. The best entry point if coming to any of these topics cold.
- [[adv-nlp-course-notes]] — the source synthesis note for these course notes: full structural overview, key insights, critiques, and a list of all concept pages the integration produced.

---

## Concept dependency graph

Understanding this domain well means knowing the dependencies — you cannot understand transformers without attention, you cannot understand attention without word embeddings, and so on.

```
n-gram-models
    ↓ (failure: sparsity, no similarity)
word-embeddings ←─────────────────────────────┐
    ↓                                          │
activation-functions ← weight-initialization  │
    ↓                                          │
dropout, regularization, optimizers            │
    ↓                                          │
recurrent-neural-networks                      │
    ↓ (failure: bottleneck, no parallelism)    │
attention-mechanism + positional-encoding ─────┘
    ↓
transformers
    ↓
tokenization + pretraining-and-finetuning
    ↓
lora → rlhf
```

---

## What is not yet in this wiki

The pages above cover the full arc from classical NLP to alignment, but several important areas have no concept pages yet. These represent open work items:

**Evaluation** — How do you measure whether an NLP model is actually good? BLEU and ROUGE for generation; BERTScore and MoverScore for semantic similarity; perplexity on held-out text; human evaluation protocols; the Elo-based chatbot arena paradigm. No concept page exists.

**Embeddings at scale** — Sentence transformers (contrastive training), dense retrieval (DPR), FAISS for approximate nearest-neighbor search, the distinction between token-level and sentence-level embeddings. These are foundational for RAG systems and semantic search. No concept page exists.

**Inference optimization** — Quantization (INT8, INT4, GPTQ, AWQ), speculative decoding, KV-cache management and eviction policies, continuous batching, PagedAttention (vLLM). These determine whether a model is actually deployable. No concept page exists.

**Prompting techniques** — Chain-of-thought, few-shot prompting, zero-shot chain-of-thought, self-consistency, tree-of-thought, ReAct, RAG (Retrieval-Augmented Generation). No concept page exists.

**Multimodal** — Vision-language models (CLIP contrastive pretraining, LLaVA instruction tuning on image-text pairs, Flamingo cross-attention architecture). Directly relevant for robotics (language-conditioned manipulation policies). No concept page exists.

**Specific model entities** — GPT (architecture + training details), BERT (masked LM, [CLS] token), LLaMA/Llama 2 (grouped-query attention, RoPE, SwiGLU), Mistral (sliding window attention, mixture of experts). These belong in `wiki/entities/` rather than `wiki/concepts/`. None exist yet.

---

## Connections to other domains

**Robotics** — Language-conditioned manipulation is an active research frontier. Vision-language-action models (RT-2, π0) use transformer architectures pretrained on internet-scale text and images, then finetuned on robot demonstrations. Understanding the NLP stack is prerequisite for reading this literature. See `wiki/domains/robotics.md` when it exists.

**Algorithms** — Self-attention is $O(n^2)$ in sequence length in both time and memory. For long documents, this is the binding constraint. Linear attention variants, sparse attention, and sliding window attention are all attempts to break this quadratic scaling — all motivated by the same algorithmic question of how to compute pairwise similarity efficiently.

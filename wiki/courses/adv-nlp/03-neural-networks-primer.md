---
title: "Chapter 3: Neural Networks — A Primer"
course: Advanced NLP
chapter: 3
last-updated: 2026-04-26
tags: [deep-learning, neural-networks, foundations, mlp]
---

# Chapter 3: Neural Networks — A Primer

## Learning objectives

By the end of this chapter you should be able to:
- Describe what a neuron computes and why nonlinearity is essential
- Explain the structure of a feedforward network (MLP)
- Write out the full forward pass of a simple network in matrix notation
- Understand what a loss function is and why we need one
- Describe the training loop at a high level

---

## 3.1 The basic unit: a neuron

A single **neuron** takes a vector of inputs $\mathbf{x} = [x_1, x_2, \ldots, x_n]$, computes a weighted sum, adds a bias, and passes the result through a nonlinear function:

$$z = w_1 x_1 + w_2 x_2 + \cdots + w_n x_n + b = \mathbf{w}^\top \mathbf{x} + b$$

$$a = f(z)$$

where $\mathbf{w}$ is a vector of **weights**, $b$ is a **bias**, and $f$ is a nonlinear **activation function**. The output $a$ is the neuron's activation.

The weights and bias are the **parameters** of the neuron — the numbers the model learns during training.

### Why do we need the nonlinearity $f$?

If we removed $f$ and stacked multiple linear transformations, the entire stack would collapse to a single linear transformation. No matter how many layers you add, a composition of linear functions is still linear. Nonlinear activations are what give neural networks their power to represent complex, curved decision boundaries. We will study activation functions in depth in Chapter 6.

---

## 3.2 Layers and networks

In practice, we do not work with one neuron at a time. We group neurons into **layers**, where every neuron in a layer takes the same input vector and produces one scalar output. Stacking multiple layers gives us a **neural network** (or **multilayer perceptron**, MLP).

### The weight matrix

If a layer has $n_{in}$ inputs and $n_{out}$ neurons, we can represent all the weights as a single matrix $W \in \mathbb{R}^{n_{out} \times n_{in}}$ and all the biases as a vector $\mathbf{b} \in \mathbb{R}^{n_{out}}$.

The entire layer's computation becomes:

$$\mathbf{z} = W\mathbf{x} + \mathbf{b}$$
$$\mathbf{a} = f(\mathbf{z})$$

where $f$ is applied elementwise. This is a single matrix multiplication and a vectorized nonlinearity — very fast on modern hardware.

**Dimensions**: if the input is $[n_{in}]$ and the weight matrix is $[n_{out} \times n_{in}]$, then $W\mathbf{x}$ produces a vector of shape $[n_{out}]$. You can think of each row of $W$ as the weights of one neuron.

---

## 3.3 A complete feedforward network

A network with one hidden layer processes input through three stages:

**Layer 1 (hidden layer)**:
$$\mathbf{z}^{(1)} = W^{(1)}\mathbf{x} + \mathbf{b}^{(1)}$$
$$\mathbf{h} = f(\mathbf{z}^{(1)})$$

**Layer 2 (output layer)**:
$$\mathbf{z}^{(2)} = W^{(2)}\mathbf{h} + \mathbf{b}^{(2)}$$
$$\hat{\mathbf{y}} = g(\mathbf{z}^{(2)})$$

where $f$ is a hidden-layer activation (e.g. GELU) and $g$ is the output activation appropriate to the task (e.g. softmax for classification).

### Concrete example in PyTorch

```python
import torch
import torch.nn as nn

class SimpleNet(nn.Module):
    def __init__(self, in_size, hidden_size, out_size):
        super().__init__()
        self.layer1 = nn.Linear(in_size, hidden_size)   # W^(1), b^(1)
        self.act    = nn.GELU()
        self.layer2 = nn.Linear(hidden_size, out_size)  # W^(2), b^(2)

    def forward(self, x):
        h = self.act(self.layer1(x))   # hidden representation
        return self.layer2(h)          # logits (raw scores)

model = SimpleNet(in_size=10, hidden_size=64, out_size=5)
```

`nn.Linear(in, out)` automatically creates $W \in \mathbb{R}^{out \times in}$ and $\mathbf{b} \in \mathbb{R}^{out}$ as learnable parameters.

---

## 3.4 The output layer and softmax

For language modeling and classification, the output layer produces a vector of **logits** — one raw score per class (or per vocabulary word). These are arbitrary real numbers that we convert to a probability distribution using the **softmax function**:

$$\text{softmax}(\mathbf{z})_i = \frac{e^{z_i}}{\sum_j e^{z_j}}$$

Softmax does two things: it makes all outputs positive (via the exponential) and normalizes them to sum to 1. The result is a valid probability distribution over all classes.

> [!note] Why not just normalize?
> We could divide each score by the sum, but using $e^z$ rather than $z$ has two benefits: it always produces positive values (even for negative logits), and it amplifies differences between scores — the highest logit gets a disproportionately large probability, which makes decisions sharper.

---

## 3.5 The loss function

A neural network's parameters start random. We need a way to measure how wrong the model is and a signal to improve it. That is the **loss function** $L$.

### Cross-entropy loss (for classification/language modeling)

When the correct answer is class $y$ and the model predicts probability $P(y)$, the cross-entropy loss is:

$$L = -\log P(y)$$

This is the negative log-probability of the correct answer. When the model is confident and correct — $P(y) \approx 1$ — the loss is near zero. When the model is confident and wrong — $P(y) \approx 0$ — the loss becomes very large (approaches infinity). This is exactly the behavior we want: penalize mistakes harshly.

For a batch of $N$ examples:

$$L = -\frac{1}{N} \sum_{i=1}^{N} \log P(y_i \mid \mathbf{x}_i)$$

### Mean squared error (for regression)

When predicting a continuous number rather than a class label, we use MSE:

$$L = \frac{1}{N} \sum_{i=1}^{N} (y_i - \hat{y}_i)^2$$

Cross-entropy is standard for language modeling; MSE is standard for regression tasks.

---

## 3.6 The training loop

Neural network training is a loop of four steps, repeated thousands or millions of times:

**Step 1 — Forward pass**: feed a batch of training examples through the network and compute predictions $\hat{\mathbf{y}}$.

**Step 2 — Compute loss**: compare $\hat{\mathbf{y}}$ to the true labels $\mathbf{y}$ using the loss function.

**Step 3 — Backward pass (backpropagation)**: compute the gradient of the loss with respect to every parameter in the network. The gradient tells us: *"if I increase this weight by a tiny amount, does the loss go up or down, and by how much?"*

**Step 4 — Parameter update**: move the parameters in the direction that reduces the loss:

$$\theta_{\text{new}} = \theta_{\text{old}} - \eta \cdot \nabla_\theta L$$

where $\eta$ (the **learning rate**) controls the step size.

Repeat until the loss stops decreasing on a held-out validation set.

```python
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3)
loss_fn   = nn.CrossEntropyLoss()

for batch_x, batch_y in dataloader:
    # Step 1: forward
    logits = model(batch_x)
    # Step 2: loss
    loss = loss_fn(logits, batch_y)
    # Step 3: backward
    optimizer.zero_grad()
    loss.backward()
    # Step 4: update
    optimizer.step()
```

> [!tip] Epochs and batches
> One **epoch** = one full pass over the entire training dataset. Because the dataset is too large to fit in GPU memory at once, we process it in **mini-batches** of typically 32–512 examples. Each mini-batch produces one gradient estimate and one parameter update.

---

## 3.7 What do hidden layers learn?

After training, the hidden representations $\mathbf{h}$ in a neural network are not random noise — they are *meaningful* compressed representations of the input. In a language model, these representations capture things like:

- Whether the current word is a noun or verb
- The sentiment of the surrounding context
- Whether the sentence is a question or statement

This is the key insight that makes neural networks so powerful: the network learns its own internal representation of the data, rather than relying on hand-crafted features. The weight matrices $W^{(1)}, W^{(2)}, \ldots$ define these learned transformations, and the activation functions ensure they are nonlinear and expressive.

We will deepen this understanding considerably as we study embeddings (Chapter 4), the full MLP architecture with modern components (Chapter 6), and eventually the transformer (Chapter 9).

---

## Summary

- A **neuron** computes a weighted sum of its inputs plus a bias, then applies a nonlinear activation.
- Neurons are grouped into **layers**; the full computation is a matrix multiplication: $\mathbf{z} = W\mathbf{x} + \mathbf{b}$.
- **Softmax** converts raw output scores (logits) into a probability distribution.
- The **loss function** measures how wrong the model is; **cross-entropy** is standard for language modeling.
- Training is a loop: forward pass → compute loss → backpropagation → parameter update.
- Hidden layers learn compressed, meaningful representations of the input.

**Previous**: [[02-n-gram-language-models|Chapter 2 — N-gram Language Models]]
**Next**: [[04-word-embeddings|Chapter 4 — Word Representations and Embeddings]]

---

## References

- **ADV NLP Course Notes** — source for the MLP forward-pass walkthrough, softmax, and training loop structure. See [[adv-nlp-course-notes]].
- Goodfellow, I., Bengio, Y., & Courville, A. (2016). *Deep Learning*. MIT Press. — standard reference for Chapters 6–8 (feedforward networks, regularization, optimization).
- Paszke, A. et al. (2019). PyTorch: An Imperative Style, High-Performance Deep Learning Library. *NeurIPS*. — the PyTorch framework used in all code examples.
- Rosenblatt, F. (1958). The Perceptron: A probabilistic model for information storage and organization in the brain. *Psychological Review*. — origin of the neuron/perceptron concept.
- Rumelhart, D. E., Hinton, G. E., & Williams, R. J. (1986). Learning representations by back-propagating errors. *Nature*. — original backpropagation paper.

> [!todo] cite this — add a canonical reference for the cross-entropy loss as a maximum likelihood objective (Bishop 2006 *PRML* Ch. 4 is standard).

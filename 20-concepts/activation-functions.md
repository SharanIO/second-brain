---
title: Activation Functions
status: complete
last-updated: 2026-04-26
tags: [deep-learning, neural-networks]
---

# Activation Functions

Nonlinear functions applied after linear transformations in a neural network layer. Without them, stacked linear layers collapse to a single linear transformation.

## Common functions

### ReLU (Rectified Linear Unit)

$$f(x) = \max(0, x)$$

Simple and fast. Problem: **dead neurons** — if a neuron's input is always negative, its gradient is always zero and it never updates.

### Leaky ReLU

$$f(x) = \max(\alpha x, x)$$

Fixes dead neurons by allowing a small slope $\alpha$ for negative inputs.

### PReLU (Parametric ReLU)

$$f(x) = \max(\alpha x, x)$$

Same as Leaky ReLU but $\alpha$ is learned during training.

### GELU (Gaussian Error Linear Unit)

$$f(x) = x \cdot \Phi(x)$$

where $\Phi(x)$ is the CDF of the standard normal distribution. Intuition: scales the input by the probability that it would be kept under a Gaussian "grading curve." If $x$ is large, high probability of keeping it; if small, low probability.

- Produces smooth, curved gradients (no hard zero)
- Used in BERT, GPT, and most modern transformers
- Pairs with [[weight-initialization|Kaiming initialization]]

### SiLU (Sigmoid Linear Unit)

$$f(x) = \frac{x}{1 + e^{-\beta x}}$$

A smooth approximation of ReLU. Related to GELU.

## Choosing an activation

| Use case | Recommended |
|---|---|
| Modern transformer | GELU |
| Old-school classification | Sigmoid / Tanh |
| Simple fast network | ReLU |

The choice also drives [[weight-initialization]] strategy: GELU/ReLU → Kaiming; Sigmoid/Tanh → Xavier.

## Sources

- [[adv-nlp-course-notes]]

## See also

- [[weight-initialization]]
- [[transformers]]

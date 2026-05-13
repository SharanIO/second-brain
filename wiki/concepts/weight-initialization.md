---
title: Weight Initialization
status: complete
last-updated: 2026-04-26
tags: [deep-learning, training, neural-networks]
---

# Weight Initialization

The goal of initialization is to keep the **variance of activations constant** across all layers. If the signal weakens every layer, gradients vanish and the network stops learning. If it amplifies, gradients explode.

## Xavier (Glorot) Initialization

$$W \sim \mathcal{N}\!\left(0,\ \frac{2}{n_{in} + n_{out}}\right)$$

- **Best for**: symmetric activations — Sigmoid, Tanh
- **Logic**: assumes the activation is approximately linear near zero; balances variance across both the forward and backward pass

## Kaiming (He) Initialization

$$W \sim \mathcal{N}\!\left(0,\ \frac{2}{n_{in}}\right)$$

- **Best for**: asymmetric activations — ReLU, [[activation-functions|GELU]]
- **Logic**: ReLU/GELU zeroes out the negative half of the distribution, halving the signal energy at each layer. Kaiming compensates by doubling the initial weight variance.

## Decision rule

| Activation | Use |
|---|---|
| GELU (modern transformer) | Kaiming |
| ReLU | Kaiming |
| Sigmoid | Xavier |
| Tanh | Xavier |

In PyTorch, `nn.init.kaiming_normal_(m.weight, nonlinearity='relu')` is used even for GELU because their curves are similar enough that the same setting works well.

## Sources

- [[adv-nlp-course-notes]]

## See also

- [[activation-functions]]
- [[dropout]]

---
title: Optimizers
status: complete
last-updated: 2026-04-26
tags: [deep-learning, training, optimization]
---

# Optimizers

Algorithms that update model parameters to minimize a loss function. The basic update rule:

$$\theta_{\text{new}} = \theta_{\text{old}} - \eta \cdot \nabla_\theta L$$

where $\eta$ is the learning rate.

## Gradient descent variants

| Variant | Update uses |
|---|---|
| Gradient Descent | full dataset |
| SGD | single sample |
| Mini-batch SGD | batch of samples |

One **epoch** = all training samples seen once.

## Optimizer progression

### SGD with Momentum
Accumulates a velocity vector in the direction of persistent gradients; smooths oscillations and speeds up convergence in flat directions.

### AdaGrad (Adaptive Gradient)
Maintains a per-parameter running sum of squared gradients. Rare parameters (small gradients) get larger effective learning rates. Problem: the sum only grows, so the learning rate eventually shrinks to zero — the model stops learning.

### RMSProp (Root Mean Square Propagation)
Fixes AdaGrad by replacing the cumulative sum with an **exponential moving average** of squared gradients (decay rate $\rho \approx 0.9$). Cares about *recent* gradient magnitude, not full history. Lacks directional smoothing.

### Adam (Adaptive Moment Estimation)
Combines Momentum (first moment — direction) and RMSProp (second moment — per-parameter scaling). Applies bias correction for the first steps.

$$m_t = \beta_1 m_{t-1} + (1-\beta_1) g_t \quad \text{(first moment)}$$
$$v_t = \beta_2 v_{t-1} + (1-\beta_2) g_t^2 \quad \text{(second moment)}$$
$$\theta_{t+1} = \theta_t - \frac{\eta}{\sqrt{\hat{v}_t} + \epsilon} \hat{m}_t$$

Typical hyperparameters: $\beta_1 = 0.9$, $\beta_2 = 0.999$, $\eta = 10^{-3}$.

### AdamW (Adam with Weight Decay)
Standard Adam mixes [[regularization|weight decay]] into the gradient moving averages, which mathematically distorts the regularization. AdamW applies weight decay **directly to the weights after the gradient step**, decoupling the two:

$$w_{\text{new}} = w_{\text{old}} - \alpha \frac{\partial L}{\partial w} - \lambda w_{\text{old}}$$

This is the standard optimizer for almost all LLMs (GPT, Llama, etc.).

### Adafactor
Memory-efficient variant for giant models. Instead of storing two full $m \times n$ moment matrices per parameter (expensive for 70B+ models), it factorizes them into smaller chunks. Used in Google's T5 and PaLM.

## Key hyperparameters

- **Learning rate $\eta$**: step size. Most important hyperparameter.
- **Batch size**: larger = better gradient estimate, but more compute. Larger batches often require learning rate scaling.
- **Early stopping**: halt when validation loss starts increasing to prevent overfitting.

## Where weight decay is applied

In transformers, weight decay is applied to:
1. $W_Q, W_K, W_V$ in attention layers
2. Feed-forward expansion/contraction layers

It is rarely applied to word embeddings or bias terms.

## Sources

- [[adv-nlp-course-notes]]

## See also

- [[dropout]]
- [[regularization]]
- [[attention-mechanism]]

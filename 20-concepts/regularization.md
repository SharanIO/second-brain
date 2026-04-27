---
title: Regularization
status: stub
last-updated: 2026-04-26
tags: [deep-learning, training, overfitting]
---

# Regularization

Techniques that reduce overfitting by discouraging the model from memorizing training data. Large weights tend to overfit; regularization penalizes them.

## L2 Regularization (Weight Penalty)

Add a penalty term to the loss function:

$$L_{\text{total}} = L_{\text{original}} + \frac{\lambda}{2} \sum w^2$$

$\lambda$ (lambda) is the weight decay coefficient. The optimizer now minimizes both prediction error and weight magnitude simultaneously.

## Weight Decay (Direct)

Shrink weights directly at each update step:

$$w_{\text{new}} = w_{\text{old}} - \alpha \frac{\partial L}{\partial w} - \lambda w_{\text{old}}$$

The $-\lambda w_{\text{old}}$ term decays the weight toward zero regardless of the gradient.

## L2 vs. Weight Decay: are they the same?

For **SGD**: mathematically identical.

For **Adam / RMSProp**: they differ. Standard Adam mixes the $\lambda w$ decay term into the adaptive gradient moving averages, distorting the regularization. **AdamW** decouples the weight decay from the gradient update, applying it directly — this is the correct formulation and the standard for LLMs.

## Benefits

1. Reduces overfitting — large weights tend to memorize training data
2. Mitigates exploding gradients
3. Ensures no single neuron or connection dominates

## Other regularization techniques

- [[dropout]]
- Early stopping
- Data augmentation

## Sources

- [[adv-nlp-course-notes]]

## See also

- [[optimizers]] (AdamW)
- [[dropout]]

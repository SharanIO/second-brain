---
title: Dropout
status: complete
last-updated: 2026-04-26
tags: [deep-learning, regularization, training]
---

# Dropout

A regularization technique that randomly "turns off" a fraction of neurons during each training step. Introduced by Srivastava et al.

## Mechanism

For a layer output $h$ and dropout probability $p$:

1. Sample a binary mask $r$ of the same shape as $h$, where each entry is 1 with probability $(1-p)$ and 0 with probability $p$.
2. Apply: $h_{\text{dropped}} = h \cdot r$
3. Scale to preserve expected signal magnitude:

$$h_{\text{final}} = \frac{h \cdot r}{1 - p}$$

**Crucial**: dropout is active only during training. At inference, all neurons are on and no scaling is needed (the training-time scaling already corrects for this).

## Why it works

1. **Prevents co-adaptation**: neurons cannot rely on specific other neurons always being present, so they learn more independent features.
2. **Forces redundancy**: the network builds multiple independent ways to represent the same information.
3. **Ensemble effect**: equivalent to training and averaging an exponentially large number of subnetworks.

## Typical values

$p = 0.1$–$0.2$ in transformer attention layers; $p = 0.5$ is common in fully connected layers for image classification.

## Sources

- [[adv-nlp-course-notes]]

## See also

- [[weight-initialization]]
- [[optimizers]]

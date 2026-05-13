---
title: "Chapter 5: Training Neural Networks — Backpropagation and Optimizers"
course: Advanced NLP
chapter: 5
last-updated: 2026-04-26
tags: [deep-learning, backpropagation, optimizers, training]
---

# Chapter 5: Training Neural Networks — Backpropagation and Optimizers

## Learning objectives

By the end of this chapter you should be able to:
- Explain what a gradient is and what it tells you about a parameter
- Describe the chain rule and how it enables backpropagation through a network
- Distinguish between SGD, mini-batch SGD, Adam, and AdamW
- Explain why AdamW is the standard optimizer for modern LLMs
- Define learning rate, batch size, and epoch

---

## 5.1 The goal of training

At the start of training, all the weights and biases in a neural network are initialized randomly (Chapter 6 will explain how we choose the initialization). The network produces garbage predictions, and the loss is high.

Training is the process of adjusting the parameters so that the loss decreases. But a neural network might have hundreds of millions of parameters — we cannot possibly adjust each one by hand. We need an algorithm that computes, for every parameter, exactly how changing it would affect the loss. That algorithm is **backpropagation**.

---

## 5.2 Gradients: what they are and what they tell you

The **gradient** of the loss $L$ with respect to a parameter $\theta$ is written $\frac{\partial L}{\partial \theta}$. It is a single number that answers the question:

*"If I increase $\theta$ by a tiny amount $\epsilon$, approximately how much does $L$ change?"*

The answer is: $L$ changes by approximately $\epsilon \cdot \frac{\partial L}{\partial \theta}$.

- If $\frac{\partial L}{\partial \theta} > 0$: increasing $\theta$ increases the loss. We should *decrease* $\theta$.
- If $\frac{\partial L}{\partial \theta} < 0$: increasing $\theta$ decreases the loss. We should *increase* $\theta$.
- If $\frac{\partial L}{\partial \theta} = 0$: changing $\theta$ (slightly) has no effect on the loss. We are at a local optimum with respect to this parameter.

The **gradient vector** $\nabla_\theta L$ stacks the gradients of all parameters into a single vector. It points in the direction of steepest *increase* in the loss. To minimize the loss, we step in the *negative* gradient direction.

---

## 5.3 Backpropagation: gradients through a network

Computing $\nabla_\theta L$ for a deep network requires the **chain rule** from calculus. If $L$ depends on $z$, which depends on $w$, then:

$$\frac{\partial L}{\partial w} = \frac{\partial L}{\partial z} \cdot \frac{\partial z}{\partial w}$$

For a two-layer network with output $\hat{y} = \tanh(W_2 \cdot \tanh(W_1 x))$ and loss $L = \frac{1}{2}(y - \hat{y})^2$:

$$\frac{\partial L}{\partial W_2} = \frac{\partial L}{\partial \hat{y}} \cdot \frac{\partial \hat{y}}{\partial z_2} \cdot \frac{\partial z_2}{\partial W_2}$$

Each term has a simple closed form:

$$\frac{\partial L}{\partial \hat{y}} = -(y - \hat{y})$$
$$\frac{\partial \hat{y}}{\partial z_2} = 1 - \tanh^2(z_2) = 1 - \hat{y}^2$$
$$\frac{\partial z_2}{\partial W_2} = h \quad (\text{the hidden activation})$$

So: $\frac{\partial L}{\partial W_2} = -(y - \hat{y})(1 - \hat{y}^2) \cdot h$

The chain rule chains backward through every operation in the network — hence **back**propagation. The key insight is that each intermediate gradient is computed once and **cached**, then reused by all the layers below it. This makes backpropagation an $O(n\_params)$ algorithm rather than an exponential one.

In practice, you never implement backpropagation by hand. PyTorch's autograd system does it automatically whenever you call `loss.backward()`.

---

## 5.4 Gradient descent: the basic update rule

Once we have the gradient, the parameter update is:

$$\theta_{\text{new}} = \theta_{\text{old}} - \eta \cdot \nabla_\theta L$$

where $\eta$ is the **learning rate** — a hyperparameter that controls how large a step we take. This is called **gradient descent**.

**Learning rate intuition**:
- Too large: the steps overshoot the minimum; the loss oscillates or diverges.
- Too small: training is extremely slow; we make negligible progress each step.
- Just right: smooth, steady decrease in loss.

Choosing a good learning rate is one of the most practically important skills in training neural networks.

### Variants: how much data per update?

| Variant | Update computed from | Notes |
|---|---|---|
| Gradient Descent | Entire training dataset | Accurate but impractical for large datasets |
| Stochastic GD (SGD) | Single example | Noisy but fast; can escape local minima |
| Mini-batch SGD | A batch of $B$ examples | The practical standard |

**Mini-batch SGD** is the workhorse of deep learning. Using a batch of $B$ examples gives a gradient estimate that is both reasonably accurate (averaging over $B$ examples reduces noise) and computationally efficient (GPU parallelism amortizes the cost).

**Epoch**: one complete pass over the training dataset. With a batch size of 32 and a training set of 32,000 examples, one epoch = 1,000 gradient updates.

---

## 5.5 The optimizer zoo: from SGD to AdamW

Plain SGD works, but it is slow and sensitive to learning rate choice. A series of improved optimizers has been developed, each addressing a specific weakness.

### SGD with Momentum

Plain SGD can oscillate in ravine-shaped loss landscapes — taking large steps across the ravine and small steps along it. Momentum fixes this by accumulating a velocity vector:

$$v_t = \beta v_{t-1} + \nabla_\theta L$$
$$\theta_{t+1} = \theta_t - \eta v_t$$

The velocity $v_t$ is an exponentially weighted average of past gradients. Directions with consistent gradients build up momentum; oscillating directions cancel out. Typical $\beta = 0.9$.

### AdaGrad (Adaptive Gradient) [Duchi et al., 2011]

Different parameters need different learning rates. A rare word embedding should get large updates when it appears; a common word's embedding is already well-trained. AdaGrad keeps a per-parameter sum of squared gradients $G$ and scales the learning rate accordingly:

$$G_t = G_{t-1} + (\nabla_\theta L)^2$$
$$\theta_{t+1} = \theta_t - \frac{\eta}{\sqrt{G_t + \epsilon}} \nabla_\theta L$$

Parameters with large gradients (common features) get smaller effective learning rates; parameters with small gradients (rare features) get larger ones.

**Problem**: $G_t$ only ever grows. After enough training, every learning rate shrinks to zero and the model stops learning. This is called the "dying learning rate" problem.

### RMSProp (Root Mean Square Propagation)

RMSProp fixes AdaGrad's dying learning rate by using an **exponential moving average** of squared gradients instead of a cumulative sum:

$$v_t = \rho \cdot v_{t-1} + (1 - \rho) \cdot (\nabla_\theta L)^2$$
$$\theta_{t+1} = \theta_t - \frac{\eta}{\sqrt{v_t + \epsilon}} \nabla_\theta L$$

The decay rate $\rho \approx 0.9$ means old gradient information fades away. The model now cares about *recent* gradient magnitude, not the full history. The learning rate no longer dies.

**Remaining problem**: RMSProp scales learning rates per-parameter but does not use directional information — it can still oscillate in curved loss landscapes.

### Adam (Adaptive Moment Estimation) [Kingma & Ba, 2015]

Adam is the combination of Momentum and RMSProp. It tracks two quantities:

- **First moment** (mean of gradients — direction): $m_t = \beta_1 m_{t-1} + (1-\beta_1) g_t$
- **Second moment** (mean of squared gradients — magnitude): $v_t = \beta_2 v_{t-1} + (1-\beta_2) g_t^2$

Because $m_0 = v_0 = 0$, the estimates are biased toward zero early in training. Adam corrects for this:

$$\hat{m}_t = \frac{m_t}{1 - \beta_1^t}, \qquad \hat{v}_t = \frac{v_t}{1 - \beta_2^t}$$

The update rule:

$$\theta_{t+1} = \theta_t - \frac{\eta}{\sqrt{\hat{v}_t} + \epsilon} \hat{m}_t$$

Typical hyperparameters: $\beta_1 = 0.9$, $\beta_2 = 0.999$, $\eta = 10^{-3}$, $\epsilon = 10^{-8}$.

Adam is the most widely used optimizer. It is robust to learning rate choice and works well across a wide range of architectures and tasks.

### AdamW: the LLM standard [Loshchilov & Hutter, 2019]

Standard Adam has a subtle bug with **weight decay** (L2 regularization). Weight decay is supposed to penalize large weights by adding a term $\lambda \|\theta\|^2$ to the loss. In standard Adam, this term enters the gradient and gets processed by the adaptive scaling — which mathematically distorts the regularization effect.

**AdamW** decouples weight decay from the gradient update, applying it directly to the weights after the gradient step:

$$\theta_{t+1} = \theta_t - \frac{\eta}{\sqrt{\hat{v}_t} + \epsilon} \hat{m}_t \quad - \quad \eta \lambda \theta_t$$

The $-\eta \lambda \theta_t$ term shrinks every weight toward zero by a fixed fraction at every step, independent of the gradient. This is the mathematically correct form of weight decay.

AdamW is the standard optimizer for virtually all modern LLMs — GPT, Llama, Mistral, and so on.

### Adafactor: memory-efficient at scale

Adam stores two extra values per parameter (first and second moments). For a model with 70 billion parameters, that is 140 billion extra floating-point values — hundreds of gigabytes just for the optimizer state.

**Adafactor** addresses this by factorizing the second-moment matrix into two smaller matrices, drastically reducing memory usage. Used in Google's T5 and PaLM models.

---

## 5.6 Practical training details

### Early stopping

Train until the validation loss starts increasing. At that point, the model is beginning to overfit the training data — it is memorizing rather than learning. Save the model at the point of lowest validation loss.

### Learning rate scheduling

A fixed learning rate is rarely optimal. Common practice:
- **Warmup**: start with a very small learning rate for a few thousand steps, then increase to the target rate. Prevents large, destabilizing updates at the start when the model is far from any local optimum.
- **Decay**: gradually reduce the learning rate toward the end of training. Common schedules: cosine decay, linear decay.

### Gradient clipping

If gradients become very large (gradient explosion), the update step can destroy the model. **Gradient clipping** caps the gradient norm at a threshold (typically 1.0) before the update [Pascanu et al., 2013]:

$$g \leftarrow g \cdot \min\!\left(1,\ \frac{1.0}{\|g\|}\right)$$

---

## Summary

- The **gradient** $\frac{\partial L}{\partial \theta}$ tells us how to change $\theta$ to reduce the loss.
- **Backpropagation** computes gradients efficiently using the chain rule; PyTorch does this automatically.
- **Mini-batch SGD** is the practical training algorithm: process small batches, compute gradients, update.
- **Adam** combines momentum (direction) and RMSProp (per-parameter scaling) with bias correction.
- **AdamW** decouples weight decay from the gradient update — this is the correct formulation and the standard for LLMs.
- **Early stopping**, **learning rate scheduling**, and **gradient clipping** are essential practical tools.

**Previous**: [[04-word-embeddings|Chapter 4 — Word Representations and Embeddings]]
**Next**: [[06-building-blocks|Chapter 6 — Building Blocks of Modern Neural Networks]]

---

## References

- **ADV NLP Course Notes** — source for the optimizer progression (AdaGrad → Adam → AdamW → Adafactor), L2 vs. weight decay distinction. See [[adv-nlp-course-notes]].
- Rumelhart, D. E., Hinton, G. E., & Williams, R. J. (1986). Learning representations by back-propagating errors. *Nature*, 323. — original backpropagation paper.
- Duchi, J., Hazan, E., & Singer, Y. (2011). Adaptive Subgradient Methods for Online Learning and Stochastic Optimization. *JMLR*, 12. — AdaGrad.
- Hinton, G. (2012). *Neural Networks for Machine Learning*, Lecture 6e (Coursera). — RMSProp (unpublished; this lecture is the canonical reference).
- Kingma, D. P. & Ba, J. (2015). Adam: A Method for Stochastic Optimization. *ICLR*. — Adam optimizer.
- Loshchilov, I. & Hutter, F. (2019). Decoupled Weight Decay Regularization. *ICLR*. — AdamW; explains why plain Adam handles weight decay incorrectly.
- Shazeer, N. & Stern, M. (2018). Adafactor: Adaptive Learning Rates with Sublinear Memory Cost. *ICML*. — Adafactor optimizer.
- Pascanu, R., Mikolov, T., & Bengio, Y. (2013). On the difficulty of training recurrent neural networks. *ICML*. — gradient clipping.
- Goodfellow, I., Bengio, Y., & Courville, A. (2016). *Deep Learning*. MIT Press, Ch. 8. — optimization for neural networks.

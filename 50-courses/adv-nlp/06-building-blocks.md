---
title: "Chapter 6: Building Blocks of Modern Neural Networks"
course: Advanced NLP
chapter: 6
last-updated: 2026-04-26
tags: [deep-learning, activation-functions, dropout, layer-norm, weight-initialization]
---

# Chapter 6: Building Blocks of Modern Neural Networks

## Learning objectives

By the end of this chapter you should be able to:
- Explain ReLU, GELU, and why GELU is preferred in transformers
- Describe the problem of vanishing/exploding gradients and how initialization addresses it
- Explain Xavier and Kaiming initialization and when to use each
- Describe dropout and explain how it prevents overfitting
- Explain what layer normalization does and why it stabilizes training
- Assemble all these components into a complete MLP layer

---

## 6.1 Activation functions in depth

In Chapter 3 we said that activation functions must be nonlinear. Here we look at the specific choices and their consequences.

### ReLU (Rectified Linear Unit)

$$f(x) = \max(0, x)$$

ReLU is the workhorse of deep learning. It is:
- Computationally trivial (just a comparison)
- Non-saturating for positive inputs (gradient = 1, no vanishing)
- Biologically motivated (neurons either fire or don't)

**The dead neuron problem**: if a neuron's input is consistently negative — perhaps because its weights were poorly initialized — its output is always zero and its gradient is always zero. The optimizer cannot update it. The neuron is "dead". Once dead, it stays dead.

### Leaky ReLU and PReLU

$$f(x) = \max(\alpha x, x) \quad (\alpha \ll 1)$$

Leaky ReLU allows a small nonzero slope $\alpha$ (e.g. 0.01) for negative inputs, preventing dead neurons. **PReLU** (Parametric ReLU) makes $\alpha$ a learned parameter rather than a fixed constant.

### GELU (Gaussian Error Linear Unit) [Hendrycks & Gimpel, 2016]

$$f(x) = x \cdot \Phi(x)$$

where $\Phi(x)$ is the cumulative distribution function of the standard normal distribution. Intuitively: GELU weights the input by the probability that a standard normal random variable is less than $x$.

For large positive $x$: $\Phi(x) \approx 1$, so $f(x) \approx x$ — the input passes through nearly unchanged.
For large negative $x$: $\Phi(x) \approx 0$, so $f(x) \approx 0$ — the input is suppressed.

The key difference from ReLU is that the transition is **smooth** — GELU has a continuous gradient everywhere, unlike ReLU's sharp kink at zero. This makes optimization slightly smoother.

GELU is used in BERT, GPT-2, GPT-3, and most modern transformers. If you see "GELU" in a transformer description, this is what it means.

> [!note] Practical approximation
> $\Phi(x)$ requires computing the error function, which is slow. In practice, a fast approximation is used:
> $f(x) \approx 0.5x\left(1 + \tanh\!\left(\sqrt{2/\pi}(x + 0.044715x^3)\right)\right)$
> PyTorch's `nn.GELU()` uses this by default.

### SiLU (Sigmoid Linear Unit)

$$f(x) = x \cdot \sigma(x) = \frac{x}{1 + e^{-x}}$$

Closely related to GELU; used in the LLaMA family of models.

---

## 6.2 Weight initialization: starting in the right place

Before any training occurs, all parameters must be initialized to some starting values. The choice of initialization matters enormously for deep networks.

### The problem: signal degradation

Consider a network with 10 layers. At each layer, the output is roughly:

$$\mathbf{h}^{(l)} = f(W^{(l)} \mathbf{h}^{(l-1)})$$

If the weights $W^{(l)}$ have values that are slightly too large, the activations grow exponentially layer by layer — **exploding gradients**. If they are slightly too small, the activations shrink exponentially — **vanishing gradients**. In both cases, the network cannot be trained.

The goal of initialization is to keep the **variance of activations approximately constant** across all layers.

### Xavier (Glorot) initialization [Glorot & Bengio, 2010]

Best for: sigmoid and tanh activations.

$$W \sim \mathcal{N}\!\left(0,\ \frac{2}{n_{in} + n_{out}}\right)$$

Derivation sketch: for a linear layer with $n_{in}$ inputs and $n_{out}$ outputs, if inputs have variance 1 and weights are i.i.d., the output variance scales as $n_{in} \cdot \text{Var}(W)$. Setting $\text{Var}(W) = 2/(n_{in} + n_{out})$ balances variance preservation in both the forward and backward pass.

### Kaiming (He) initialization [He et al., 2015]

Best for: ReLU and GELU activations.

$$W \sim \mathcal{N}\!\left(0,\ \frac{2}{n_{in}}\right)$$

ReLU zeroes out roughly half its inputs (all negative values). This halves the variance of the signal at each layer. Kaiming compensates by using a variance twice as large as Xavier, effectively doubling the signal to counteract ReLU's attenuation.

The factor of 2 accounts for the expected variance of a half-rectified Gaussian.

```python
for m in model.modules():
    if isinstance(m, nn.Linear):
        nn.init.kaiming_normal_(m.weight, nonlinearity='relu')
        nn.init.constant_(m.bias, 0)
```

> [!tip] Kaiming for GELU too?
> PyTorch's `nonlinearity='relu'` is also appropriate for GELU, because the two activation functions have very similar variance behavior near zero. This is the standard practice even for transformer models.

---

## 6.3 Dropout: regularization by randomness

Even with good initialization, deep networks with many parameters tend to **overfit** — they memorize the training data rather than learning general patterns. The model performs well on training examples but poorly on new, unseen examples.

**Dropout** is a regularization technique that forces the network to learn more robust features [Srivastava et al., 2014].

### Mechanism

During each forward pass in training:
1. For each neuron in a dropout layer, independently sample a Bernoulli random variable: keep the neuron active with probability $1 - p$, zero it out with probability $p$.
2. Scale the surviving neurons by $\frac{1}{1-p}$ to preserve the expected sum.

$$h_{\text{dropped}} = \frac{h \cdot r}{1 - p}, \quad r_i \sim \text{Bernoulli}(1-p)$$

At inference time, dropout is turned off — all neurons are active. No scaling is needed because training already compensated.

### Why it works

By randomly silencing neurons, dropout:
- **Prevents co-adaptation**: a neuron cannot learn to rely on specific other neurons always being present, since those neurons may be dropped.
- **Forces redundancy**: the network must learn multiple independent pathways to represent the same information.
- **Ensemble effect**: training with dropout is approximately equivalent to training and averaging an exponential number of different subnetworks.

Typical dropout rates: $p = 0.1$ in transformer attention layers, $p = 0.5$ in fully connected layers for vision tasks.

```python
self.drop = nn.Dropout(p=0.1)
# In forward():
x = self.drop(x)  # active during training, identity during eval
model.eval()  # disables dropout for inference
model.train() # re-enables dropout for training
```

---

## 6.4 Layer normalization: stabilizing activations

As a network trains, the distribution of activations in each layer shifts as the parameters change — a phenomenon called **internal covariate shift**. Later layers must constantly adapt to these distribution changes, slowing training.

**Layer normalization** (LayerNorm) normalizes the activations within each example, independently across examples in the batch [Ba et al., 2016]:

$$\text{LayerNorm}(\mathbf{x}) = \frac{\mathbf{x} - \mu}{\sigma + \epsilon} \cdot \gamma + \beta$$

where:
- $\mu$ and $\sigma$ are the mean and standard deviation computed across the feature dimensions for a single example
- $\gamma$ and $\beta$ are learned scale and shift parameters (same shape as $\mathbf{x}$)
- $\epsilon$ is a small constant for numerical stability

After LayerNorm, the activations have approximately zero mean and unit variance — but the network can learn to rescale and shift them via $\gamma$ and $\beta$.

LayerNorm is the standard normalization in transformers (as opposed to BatchNorm, which normalizes across examples in a batch and behaves poorly for variable-length sequences).

---

## 6.5 Putting it all together: a complete MLP layer

A modern MLP layer in a transformer combines all of the above:

```
Input → Linear → LayerNorm → GELU → Dropout → Linear → Output
```

In PyTorch:

```python
class MLPLayer(nn.Module):
    def __init__(self, d_model, d_ff, dropout=0.1):
        super().__init__()
        self.fc1  = nn.Linear(d_model, d_ff)
        self.norm = nn.LayerNorm(d_ff)
        self.act  = nn.GELU()
        self.drop = nn.Dropout(dropout)
        self.fc2  = nn.Linear(d_ff, d_model)

        # Kaiming initialization for GELU layers
        for m in [self.fc1, self.fc2]:
            nn.init.kaiming_normal_(m.weight, nonlinearity='relu')
            nn.init.zeros_(m.bias)

    def forward(self, x):
        x = self.drop(self.act(self.norm(self.fc1(x))))
        return self.fc2(x)
```

A few conventions worth noting:
- The "expansion ratio" $d_{ff} / d_{model}$ is typically 4 (so a model with $d_{model} = 512$ has an FFN with $d_{ff} = 2048$).
- The second linear layer projects back to $d_{model}$, preserving the residual connection interface (Chapter 9).
- Kaiming initialization is applied to all linear weights; biases are initialized to zero.

---

## Summary

- **GELU** is the standard activation for transformers: smooth, differentiable everywhere, suppresses small or negative inputs probabilistically.
- **Kaiming initialization** compensates for ReLU/GELU's variance attenuation: use for asymmetric activations; use Xavier for sigmoid/tanh.
- **Dropout** prevents overfitting by randomly zeroing neurons during training, forcing the network to learn redundant representations.
- **Layer normalization** stabilizes training by normalizing activations within each example.
- A full transformer MLP layer: `Linear → LayerNorm → GELU → Dropout → Linear`.

**Previous**: [[05-training-mechanics|Chapter 5 — Training Neural Networks]]
**Next**: [[07-recurrent-neural-networks|Chapter 7 — Recurrent Neural Networks]]

---

## References

- **ADV NLP Course Notes** — source for the activation function survey (ReLU, GELU, SiLU), Xavier/Kaiming comparison, dropout mechanics, and the full MLP architecture with PyTorch code. See [[adv-nlp-course-notes]].
- Nair, V. & Hinton, G. E. (2010). Rectified Linear Units Improve Restricted Boltzmann Machines. *ICML*. — ReLU activation.
- Hendrycks, D. & Gimpel, K. (2016). Gaussian Error Linear Units (GELUs). *arXiv:1606.08415*. — GELU activation; used in BERT, GPT-2, and most modern transformers.
- Ramachandran, P., Zoph, B., & Le, Q. V. (2017). Swish: A Self-Gated Activation Function. *arXiv:1710.05941*. — SiLU/Swish activation (used in LLaMA family).
- Glorot, X. & Bengio, Y. (2010). Understanding the difficulty of training deep feedforward neural networks. *AISTATS*. — Xavier initialization.
- He, K. et al. (2015). Delving Deep into Rectifiers: Surpassing Human-Level Performance on ImageNet Classification. *ICCV*. — Kaiming initialization.
- Srivastava, N. et al. (2014). Dropout: A Simple Way to Prevent Neural Networks from Overfitting. *JMLR*, 15. — dropout regularization.
- Ba, J. L., Kiros, J. R., & Hinton, G. E. (2016). Layer Normalization. *arXiv:1607.06450*. — LayerNorm; preferred over BatchNorm for variable-length sequences.

> [!todo] cite this — add a reference for the 4× FFN expansion ratio convention (common in transformer literature but no single canonical source; see Vaswani et al. 2017).

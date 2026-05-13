---
title: Regularization
status: active
last-updated: 2026-05-13
tags: [deep-learning, training, overfitting]
---

# Regularization

Regularization is the collection of techniques that reduce a model's tendency to overfit — to memorize training data rather than learning generalizable patterns. Every regularization technique works by constraining the hypothesis space the model searches over: either by penalizing complexity, randomly masking structure, or stopping training before the model has time to memorize.

The fundamental tension is the **bias-variance tradeoff**. A model with no regularization can achieve near-zero training loss but has high variance — it fits the noise specific to the training set and fails on unseen data. A heavily regularized model has lower variance but higher bias — it underfits because the constraints are too strong. The goal is to find the regime where generalization error is minimized.

In deep learning, overfitting manifests differently than in classical ML. Modern neural networks are dramatically overparameterized — they have far more parameters than training examples, yet they generalize remarkably well. This is the "benign overfitting" or "double descent" phenomenon. Regularization in deep learning is still important, but its role has evolved from "prevent the model from memorizing" (the classical framing) to "guide optimization toward weight configurations that generalize" (the modern framing). Small flat minima — weight configurations where the loss is low and doesn't rise steeply in any direction — generalize better than sharp narrow minima. Many regularization techniques steer the optimizer toward flat minima.

---

## The core problem: what overfitting looks like in practice

A model overfitting shows:
- Training loss continues to decrease
- Validation loss starts increasing (the divergence point)
- The model's learned weights become very large in magnitude

The last point is diagnostic. Large weights create sharp, narrow decision boundaries. A slight shift in the input distribution can move examples across boundaries. Mathematically, large weights amplify small perturbations in the input: if $f(x) = w^\top x$ and $w$ is large, then $f(x + \epsilon)$ deviates substantially from $f(x)$ for small $\epsilon$. Regularization forces weights to stay small, making the function smoother and more robust.

---

## L2 regularization (weight penalty)

L2 regularization adds a penalty to the loss function proportional to the sum of squared weights:

$$L_{\text{total}} = L_{\text{task}} + \frac{\lambda}{2} \sum_i w_i^2$$

The $\frac{1}{2}$ factor is a notational convenience — it makes the derivative clean. The gradient of the penalty with respect to weight $w_i$ is:

$$\frac{\partial}{\partial w_i} \left( \frac{\lambda}{2} w_i^2 \right) = \lambda w_i$$

So the full gradient update becomes:

$$w_i \leftarrow w_i - \alpha \left( \frac{\partial L_{\text{task}}}{\partial w_i} + \lambda w_i \right)$$

This is equivalent to:

$$w_i \leftarrow w_i (1 - \alpha \lambda) - \alpha \frac{\partial L_{\text{task}}}{\partial w_i}$$

The term $(1 - \alpha \lambda)$ is a multiplicative shrinkage applied to the weight every step, independent of the gradient. This is why L2 regularization is also called **weight decay** in the SGD setting — weights are decayed toward zero at every update.

**Geometric interpretation**: the penalty $\frac{\lambda}{2} \sum w_i^2$ is the squared L2 norm of the weight vector, which is a sphere in weight space. The unconstrained optimum lives somewhere in weight space; the L2 penalty pulls the solution toward the origin. The optimal weight vector under L2 regularization is the point where the ellipsoid (contours of the task loss) is tangent to a sphere (contours of the L2 penalty). Because spheres are smooth with no corners, the tangent point can be anywhere — L2 solutions are generally dense (all weights are small but nonzero).

**Bayesian interpretation**: minimizing $L_{\text{task}} + \frac{\lambda}{2} \|w\|^2$ is equivalent to MAP estimation under a Gaussian prior $p(w) = \mathcal{N}(0, \sigma^2 I)$ on the weights, where $\lambda = 1/\sigma^2$. This is why L2 regularization has a principled probabilistic interpretation: it encodes a prior belief that weights are small.

---

## L1 regularization (Lasso)

L1 regularization penalizes the sum of absolute values:

$$L_{\text{total}} = L_{\text{task}} + \lambda \sum_i |w_i|$$

The subgradient of $|w_i|$ is $\text{sign}(w_i)$, so the update becomes:

$$w_i \leftarrow w_i - \alpha \left( \frac{\partial L_{\text{task}}}{\partial w_i} + \lambda \cdot \text{sign}(w_i) \right)$$

The key difference from L2: the penalty pulls each weight toward zero by a fixed amount $\alpha \lambda$ regardless of the weight's current magnitude. For small weights, this fixed subtraction drives them exactly to zero. L1 regularization therefore induces **sparse** solutions — many weights are exactly zero, and the model depends on only a few features.

**Geometric interpretation**: the L1 unit ball in $d$ dimensions is a cross-polytope (a diamond in 2D, an octahedron in 3D). The corners of this polytope lie on coordinate axes. When the task-loss ellipsoid is tangent to the L1 ball, it is most likely to be tangent at a corner (because corners are where the polytope is "pointy"). Corners correspond to solutions where many weights are zero. This is why L1 naturally produces sparse solutions while L2 does not.

**In deep learning**: L1 regularization is rarely used directly on neural network weights. The optimizer dynamics (especially Adam with momentum) interact awkwardly with the non-smooth $|w|$ penalty. L1-style sparsity is achieved instead via structured pruning or learned masks. L1 remains common in linear models (Lasso regression) and in regularizing embedding tables.

---

## Weight decay vs. L2 regularization: the Adam divergence

For SGD, L2 regularization and direct weight decay are mathematically identical — both produce the update $w \leftarrow w(1-\alpha\lambda) - \alpha g$ where $g$ is the gradient. The two formulations differ when using adaptive optimizers like Adam.

### How standard Adam handles L2 regularization (the wrong way)

Adam maintains first-moment (mean) and second-moment (variance) estimates of the gradient:

$$m_t = \beta_1 m_{t-1} + (1 - \beta_1) g_t$$
$$v_t = \beta_2 v_{t-1} + (1 - \beta_2) g_t^2$$

The parameter update is:

$$w \leftarrow w - \alpha \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon}$$

where $\hat{m}_t$ and $\hat{v}_t$ are bias-corrected estimates.

When you add an L2 penalty $\frac{\lambda}{2}\|w\|^2$ to the loss, the effective gradient becomes $g_t \leftarrow g_t + \lambda w$. This augmented gradient flows through the same adaptive mechanism — Adam scales it by $1/(\sqrt{\hat{v}_t} + \epsilon)$ just like any other gradient. The problem: $\hat{v}_t$ accumulates the squared values of the *full gradient including the penalty*. For weights where the task gradient is small but the weight itself is large, $\hat{v}_t$ becomes large due to the $\lambda w$ contribution, and the effective learning rate for those weights shrinks. The regularization adapts to the weight magnitude in a way that distorts the intended effect — weights that are already large get stronger effective regularization, weights that are small get weaker regularization. This is not uniform shrinkage toward zero; it is a correlated shrinkage that depends on the optimization history.

### AdamW: the correct formulation

AdamW (Loshchilov & Hutter, 2019) decouples the weight decay from the gradient estimate by applying it directly to the parameter, *after* the Adam update:

$$w \leftarrow w - \alpha \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon} - \alpha \lambda w$$

The weight decay term $\alpha \lambda w$ is never added to the gradient and never enters $\hat{m}_t$ or $\hat{v}_t$. It is applied multiplicatively — the weight shrinks by $(1 - \alpha \lambda)$ every step regardless of the adaptive scaling. This achieves uniform regularization: every weight decays toward zero at the same proportional rate, independent of what the adaptive optimizer is doing.

**The practical implication**: L2 regularization in PyTorch using `weight_decay` in `torch.optim.Adam` does the wrong thing — it adds the L2 penalty to the gradient and routes it through the adaptive machinery. `torch.optim.AdamW` does it correctly. For all LLM training (GPT, BERT, T5, and all their descendants), AdamW is the correct optimizer. The difference in generalization is measurable, especially at large learning rates.

**Typical values**: $\lambda = 0.01$ to $0.1$ for LLMs. GPT-3 used $\lambda = 0.1$; BERT used $\lambda = 0.01$. Biases and layer norm parameters are conventionally excluded from weight decay — they do not contribute to the kinds of sharp minima that weight decay is meant to avoid.

---

## Dropout

Dropout is covered in full in [[dropout]]. In brief: during training, each neuron is randomly zeroed with probability $p$ (typically $p = 0.1$ to $0.5$). At inference, all neurons are active but their outputs are scaled by $(1 - p)$ to maintain the same expected activation magnitude (or equivalently, training scales by $1/(1-p)$ — the "inverted dropout" convention used by PyTorch).

**Why it regularizes**: by randomly removing neurons, dropout prevents any single neuron or small group of neurons from being critical. The network cannot rely on the co-adaptation of specific units — it must develop multiple redundant representations of the same feature. This forces a more distributed, robust internal representation.

**Dropout and transformers**: dropout in transformers is applied after the attention output, after each feed-forward sublayer, and to the attention weights themselves (attention dropout). In modern large models, dropout rates are often very small ($p = 0.0$ to $p = 0.1$) or omitted entirely, because at scale the data diversity provides sufficient regularization and dropout slows training by adding noise to otherwise well-estimated gradients.

---

## Label smoothing

Label smoothing is a regularization technique specific to classification. Instead of training with hard one-hot targets:

$$y_i = \begin{cases} 1 & \text{if } i = \text{correct class} \\ 0 & \text{otherwise} \end{cases}$$

label smoothing uses soft targets:

$$y_i^{\text{smooth}} = \begin{cases} 1 - \varepsilon & \text{if } i = \text{correct class} \\ \varepsilon / (K - 1) & \text{otherwise} \end{cases}$$

where $K$ is the number of classes and $\varepsilon$ is a small constant (typically $0.1$). The mass $\varepsilon$ is redistributed uniformly from the true class to all other classes.

**Why this regularizes**: hard targets drive the model to push the logit for the correct class to $+\infty$ and all others to $-\infty$. This is impossible to achieve with finite weights, so the optimizer keeps pushing forever, growing weights without bound. More practically, a model trained with hard targets becomes overconfident — it assigns near-zero probability to everything except its top prediction, even on ambiguous examples where multiple classes are plausible. Label smoothing prevents the logits from growing too large and produces calibrated probability estimates.

**In transformers and LLMs**: label smoothing with $\varepsilon = 0.1$ was used in the original Vaswani et al. transformer and in BERT. For autoregressive language models like GPT, the "classes" are vocabulary tokens, so $K$ can be 50,000+. Label smoothing on next-token prediction softens the target distribution slightly, preventing the model from becoming overconfident about any particular next token.

---

## Early stopping

Early stopping monitors validation loss during training and halts training when validation loss stops improving (with some patience parameter). It is an implicit form of regularization: the model is stopped before it has accumulated enough gradient steps to fit the noise in the training set.

From an optimization perspective, early stopping is equivalent to constraining the solution to lie within a ball in parameter space around the initialization point — specifically, the L2 ball whose radius is proportional to the number of gradient steps taken. This gives early stopping a formal equivalence to L2 regularization for certain settings (quadratic loss, gradient descent).

**Practical use**: in pretraining large models, early stopping is rarely applied — the compute budget determines the number of steps, and models do not visibly overfit because datasets are enormous and each example is seen at most once (or a very small number of times). Early stopping is more relevant for fine-tuning on small datasets, where a few hundred examples can be memorized quickly.

---

## Layer normalization as implicit regularization

Layer normalization is discussed primarily as an architectural component in [[transformers]], but it has a regularization effect worth noting here. LayerNorm normalizes the activations of each layer to have zero mean and unit variance across the feature dimension:

$$\text{LayerNorm}(x)_i = \gamma \frac{x_i - \mu}{\sigma + \epsilon} + \beta$$

where $\mu$ and $\sigma$ are the mean and standard deviation of $x$ across its feature dimension. By renormalizing activations, LayerNorm prevents individual activations from growing to extreme magnitudes that would saturate nonlinearities or amplify small perturbations. The learned scale $\gamma$ and shift $\beta$ restore expressivity. The practical effect is that gradients flow more cleanly, training is more stable, and the model is less sensitive to learning rate choice — all of which contribute to regularization in the sense of "preventing pathological training dynamics."

---

## Data augmentation

Data augmentation generates additional training examples by applying transformations to existing ones. It regularizes by exposing the model to a larger effective dataset and by training invariances directly. For vision: random crops, flips, color jitter, rotation. For NLP: random deletion of words, synonym replacement, backtranslation, token masking (which is the basis of BERT's masked language model pretraining).

In terms of weight decay vs. data augmentation as competing regularization strategies: data augmentation trains the model to be invariant to specific transformations (which is a desirable inductive bias), while weight decay constrains weight magnitude without specifying what structure the model should learn. At scale, data augmentation often contributes more than explicit weight penalties because it encodes domain knowledge.

---

## Choosing $\lambda$: practical considerations

**Too small $\lambda$**: the model overfits. Validation loss diverges from training loss. In the weight space perspective, the model is in a sharp narrow minimum.

**Too large $\lambda$**: the model underfits. Weights are forced too close to zero, destroying the model's capacity to fit even the true signal. Validation and training loss are both high.

**Standard values by context**:

| Setting | Typical $\lambda$ |
|---|---|
| LLM pretraining (AdamW) | 0.01 – 0.1 |
| Fine-tuning | 0.001 – 0.01 |
| Vision transformers | 0.05 – 0.1 |
| Linear/logistic regression | Cross-validate |

**What to exclude from weight decay**: biases, layer norm parameters ($\gamma$, $\beta$), and embedding matrices are conventionally excluded. The reason: weight decay on biases shifts the mean of activations in ways that interact poorly with normalization. Weight decay on embeddings can disproportionately shrink representations for rare tokens (which have small gradient signal but large assigned weight decay). Most implementations of AdamW include a `no_decay` parameter group.

---

## Summary comparison

| Technique | Where applied | Mechanism | Effect |
|---|---|---|---|
| L2 / AdamW weight decay | All weights | Shrink $w$ toward 0 each step | Penalizes large weights; Gaussian prior |
| L1 | Linear layer weights | Fixed subtraction toward 0 | Sparse weights; Laplace prior |
| Dropout | Activations | Random zero-masking during training | Prevents co-adaptation of neurons |
| Label smoothing | Loss function | Soften one-hot targets | Prevents overconfident logits |
| Early stopping | Training loop | Stop before overfitting | Implicit L2 ball constraint |
| Layer normalization | Activations | Normalize to zero mean, unit variance | Stabilizes dynamics; smooths gradients |
| Data augmentation | Training data | Expand via transformations | Trains in invariances directly |

---

## Sources

- [[adv-nlp-course-notes]]
- Loshchilov & Hutter (2019) — "Decoupled Weight Decay Regularization" (AdamW)
- Müller et al. (2019) — "When Does Label Smoothing Help?"

## See also

- [[optimizers]] — AdamW is the standard for LLMs; the weight decay formulation is covered there
- [[dropout]] — full treatment of dropout as a regularization technique
- [[transformers]] — where layer normalization is used in architecture

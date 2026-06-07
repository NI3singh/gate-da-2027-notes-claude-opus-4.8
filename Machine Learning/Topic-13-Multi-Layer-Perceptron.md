# Topic 13 — Multi-Layer Perceptron (MLP)

**Subject:** Machine Learning &nbsp;|&nbsp; **Section:** Neural Networks (this is the *first* topic of the section)

---

## 0. Topic header & syllabus map

- **Topic:** Multi-Layer Perceptron (MLP)
- **Subject:** Machine Learning
- **Where it sits:** This **opens the Neural Networks block**. It comes right after the *Trees & model-evaluation* cluster (Decision Trees → Bias–Variance → Cross-validation). Everything before it — linear/ridge/logistic regression, kNN, naive Bayes, LDA, SVM — was either a **linear model** or relied on **hand-chosen structure** (a kernel, a distance metric, a tree-splitting rule). The MLP is the first model that **learns its own nonlinear features from data**. It leads directly into **Feed-Forward Neural Network (Topic 14)**, which is the general family that the MLP is the classic member of.

---

## 1. Why this topic exists (motivation + connection)

Most real decision boundaries are **nonlinear and not linearly separable**, but logistic regression, LDA, and linear SVM can only draw a single straight boundary (or a boundary that is linear in whatever features you hand it). Decision trees *can* be nonlinear but only with **axis-aligned** cuts and they are unstable. We want a model that builds its **own curved boundary by learning intermediate features automatically.**

The single neuron that powered logistic regression — the **perceptron** — provably **cannot represent XOR** (no straight line separates it). That single failure is the entire reason this topic exists: by **stacking layers** of simple neurons with **nonlinear activations**, an MLP re-expresses the input in a new space where the problem *becomes* linearly separable, and it discovers that re-expression from the data itself.

> **Bridge (Logistic Regression → MLP):** Logistic regression *is* a one-neuron network with a sigmoid output. It fails when classes are not linearly separable. **The MLP fixes this by inserting hidden layers** of neurons between input and output, so the final layer sees learned nonlinear features instead of raw inputs.

---

## 2. Prerequisites

- **Logistic regression** — sigmoid output, log-loss, gradient descent (an MLP generalizes it).
- **Gradient descent** (from the regression notes) — backprop is just gradient descent powered by the chain rule.
- **Linear separability / decision hyperplane** (from the perceptron & SVM notes).
- **Calculus:** partial derivatives and the **chain rule** — non-negotiable for backpropagation.
- **Linear algebra:** matrix–vector products (a layer is one matrix multiply + a bias).

---

## 3. Core idea — intuition first

**A neuron** takes inputs, multiplies each by a weight, adds them up (plus a bias), and passes the sum through an **activation function**:

> "weighted vote → squash → output."

A **single neuron draws one straight line** (hyperplane) and says "one side = class 1, other side = class 0." That is all a perceptron / logistic unit can do.

**Why we need a hidden layer — the XOR intuition.** XOR is "exactly one of the two inputs is 1." You cannot separate the two XOR classes with a single line. But you *can* with **two lines**: one hidden neuron learns "is the point above line A?", another learns "is it above line B?". These two yes/no answers form a **new 2-D feature space**, and in that space XOR *is* linearly separable, so the output neuron finishes the job with one more line. **Each hidden layer carves the space into pieces; the next layer recombines those pieces into more complex regions.**

> **Analogy:** an assembly line of detectors. Layer 1 detects simple patterns, layer 2 combines them into richer patterns, and the output layer reads off the answer. The network *learns* what each detector should look for.

The activation function **must be nonlinear**. If every neuron were linear, stacking layers would still give one linear map ($W_2(W_1x) = (W_2W_1)x$) — no gain. **Nonlinearity is what makes depth worth anything.**

---

## 4. Formal treatment

### 4.1 The single neuron (perceptron unit)

For input vector $\mathbf{x} = (x_1,\dots,x_d)$, weights $\mathbf{w}$, bias $b$:

$$\boxed{\,z = \mathbf{w}^\top \mathbf{x} + b = \sum_{i=1}^{d} w_i x_i + b \qquad a = \phi(z)\,}$$

- $z$ = **net input / pre-activation** (a linear combination).
- $a$ = **activation / output**, after the nonlinearity $\phi$.
- The **decision boundary** of one neuron is the hyperplane $\mathbf{w}^\top\mathbf{x}+b = 0$ — *always linear in the inputs.*

The original **McCulloch–Pitts / Rosenblatt perceptron** uses the **step (Heaviside)** activation, giving a hard 0/1 output.

### 4.2 Activation functions (formulas + derivatives)

The derivative matters because backprop multiplies by $\phi'$ at every layer.

| Activation | $\phi(z)$ | $\phi'(z)$ | Range | Note |
|---|---|---|---|---|
| **Step (Heaviside)** | $1$ if $z\ge 0$ else $0$ | $0$ (undefined at $0$) | $\{0,1\}$ | Classic perceptron; **not usable with backprop** (zero/undefined gradient). |
| **Sigmoid (logistic)** | $\sigma(z)=\dfrac{1}{1+e^{-z}}$ | $\sigma(z)\,(1-\sigma(z))$ | $(0,1)$ | Smooth; saturates → **vanishing gradients**. |
| **Tanh** | $\dfrac{e^{z}-e^{-z}}{e^{z}+e^{-z}}$ | $1-\tanh^2(z)$ | $(-1,1)$ | Zero-centred; still saturates. |
| **ReLU** | $\max(0,z)$ | $1$ if $z>0$ else $0$ (take $0$ at $z=0$) | $[0,\infty)$ | Cheap, no saturation for $z>0$; can "die." |
| **Linear / Identity** | $z$ | $1$ | $\mathbb{R}$ | Used at the **output** for regression. |
| **Softmax** (vector) | $\dfrac{e^{z_k}}{\sum_j e^{z_j}}$ | (see §4.10) | probabilities summing to 1 | **Multiclass output** layer. |

> **GATE-favourite derivation — sigmoid derivative.** With $\sigma=\dfrac{1}{1+e^{-z}}$,
> $$\sigma'(z)=\frac{e^{-z}}{(1+e^{-z})^2}=\frac{1}{1+e^{-z}}\cdot\frac{e^{-z}}{1+e^{-z}}=\sigma(z)\big(1-\sigma(z)\big).$$
> Its **maximum is $0.25$ at $z=0$** — remember this number; it is the root cause of vanishing gradients in deep sigmoid networks.

### 4.3 The perceptron learning rule (single neuron, step activation)

For target $t\in\{0,1\}$ and prediction $\hat y \in \{0,1\}$, with learning rate $\eta>0$:

$$\boxed{\;\mathbf{w} \leftarrow \mathbf{w} + \eta\,(t-\hat y)\,\mathbf{x}, \qquad b \leftarrow b + \eta\,(t-\hat y)\;}$$

- If the prediction is correct, $t-\hat y = 0$ → **no change**.
- This is **not** gradient descent (the step function isn't differentiable); it is a separate rule.

> **Perceptron Convergence Theorem (state the condition!).** If the training data is **linearly separable**, the perceptron learning rule **converges to a separating hyperplane in a finite number of updates**. If the data is **not** linearly separable, it **never converges** (it keeps oscillating). The number of updates is bounded by $(R/\gamma)^2$, where $R=\max_i\|\mathbf{x}_i\|$ and $\gamma$ is the margin — knowing the *existence* of a finite bound is enough for GATE.

### 4.4 Linear separability and the XOR problem

A single neuron separates with one hyperplane, so it can represent **AND** and **OR** but **not XOR**:

| $x_1$ | $x_2$ | AND | OR | XOR |
|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 1 | **1** |
| 1 | 0 | 0 | 1 | **1** |
| 1 | 1 | 1 | 1 | 0 |

For XOR the two "1" points sit on a diagonal opposite to the two "0" points — **no single line separates them.** Hence a single perceptron fails. The smallest network that solves XOR is **2 inputs → 1 hidden layer of 2 neurons → 1 output** (i.e. **2-2-1**).

**Explicit XOR solution (step activation).** Build XOR $=$ (OR) AND (NOT AND):

$$
h_1=\text{step}(x_1+x_2-0.5)\ (\text{OR}),\quad
h_2=\text{step}(x_1+x_2-1.5)\ (\text{AND}),\quad
y=\text{step}(h_1-h_2-0.5).
$$

Check: $(0,1)\!\to\! h_1{=}1,h_2{=}0,\,y{=}\text{step}(0.5){=}1$; $\ (1,1)\!\to\! h_1{=}1,h_2{=}1,\,y{=}\text{step}(-0.5){=}0$. ✔ All four rows give XOR.

### 4.5 MLP architecture

An MLP has an **input layer**, one or more **hidden layers**, and an **output layer**, all **fully connected (dense)** and feeding strictly **forward**. For layer $l$ (with $\mathbf{a}^{(0)}=\mathbf{x}$):

$$\boxed{\;\mathbf{z}^{(l)} = W^{(l)}\,\mathbf{a}^{(l-1)} + \mathbf{b}^{(l)}, \qquad \mathbf{a}^{(l)} = \phi\big(\mathbf{z}^{(l)}\big)\;}$$

- $W^{(l)}$ has shape $(n_l \times n_{l-1})$: row $j$ holds the weights of neuron $j$ in layer $l$.
- $\phi$ is applied **element-wise**.
- The final layer $L$ produces $\hat{\mathbf{y}}=\mathbf{a}^{(L)}$.

### 4.6 Counting parameters (very common NAT)

A dense layer with $n_{\text{in}}$ inputs and $n_{\text{out}}$ neurons has

$$\boxed{\;\#\text{weights}=n_{\text{in}}\cdot n_{\text{out}},\qquad \#\text{biases}=n_{\text{out}},\qquad \#\text{params}=(n_{\text{in}}+1)\,n_{\text{out}}\;}$$

Sum this over all layers. **The "$+1$" is the bias — the single most common thing students forget.**

### 4.7 Forward propagation

Compute layer by layer, left to right: $\mathbf{x}=\mathbf{a}^{(0)} \to \mathbf{z}^{(1)}\to \mathbf{a}^{(1)} \to \cdots \to \mathbf{a}^{(L)}=\hat{\mathbf{y}}$. **Always: multiply by weights, add bias, then apply the activation.**

### 4.8 Backpropagation — the training engine

**Goal:** for a loss $L$, compute $\partial L/\partial W^{(l)}$ and $\partial L/\partial \mathbf{b}^{(l)}$ for *every* layer, then do gradient descent. Backprop is **just the chain rule applied right-to-left**, reusing computation.

Define the **error signal** of a layer as $\displaystyle \delta^{(l)} = \frac{\partial L}{\partial \mathbf{z}^{(l)}}$. The **four backprop equations**:

$$
\textbf{(1) Output:}\quad \delta^{(L)} = \nabla_{\mathbf a}L \;\odot\; \phi'\!\big(\mathbf z^{(L)}\big)
\;\;\xrightarrow{\;L=\frac12\|\mathbf a^{(L)}-\mathbf y\|^2\;}\;\; (\mathbf a^{(L)}-\mathbf y)\odot\phi'(\mathbf z^{(L)})
$$

$$
\textbf{(2) Recurse back:}\quad \delta^{(l)} = \Big( (W^{(l+1)})^{\top}\,\delta^{(l+1)} \Big)\;\odot\; \phi'\!\big(\mathbf z^{(l)}\big)
$$

$$
\textbf{(3) Bias gradient:}\quad \frac{\partial L}{\partial \mathbf b^{(l)}} = \delta^{(l)}
\qquad\qquad
\textbf{(4) Weight gradient:}\quad \frac{\partial L}{\partial W^{(l)}} = \delta^{(l)}\big(\mathbf a^{(l-1)}\big)^{\top}
$$

where $\odot$ is element-wise multiplication. **Update (gradient descent):**

$$\boxed{\;W^{(l)} \leftarrow W^{(l)} - \eta\,\frac{\partial L}{\partial W^{(l)}}, \qquad \mathbf b^{(l)} \leftarrow \mathbf b^{(l)} - \eta\,\frac{\partial L}{\partial \mathbf b^{(l)}}\;}$$

> **Why backprop is efficient:** it computes the gradient for *all* parameters in **one forward + one backward pass**, instead of perturbing each weight separately. The $\phi'(\mathbf z^{(l)})$ factor in equation (2) is exactly why **saturating activations cause vanishing gradients** — many factors $\le 0.25$ multiply together and the signal dies.

*(We skip the fully general index-by-index derivation of equations (1)–(4): GATE asks you to **apply** them on a tiny network, which Worked Problem 3 does in full, not to re-derive them abstractly.)*

### 4.9 Gradient-descent variants

- **Batch GD** — one update using *all* training examples; stable, slow, lots of memory.
- **Stochastic GD (SGD)** — one update per *single* example; noisy, fast, can escape shallow minima.
- **Mini-batch GD** — one update per small batch (e.g. 32, 64); the practical default.

### 4.10 Loss functions (and the clean-gradient trick)

$$
\text{Regression (MSE):}\quad L=\tfrac12\sum_k(\hat y_k-y_k)^2,\quad \text{output activation } = \text{linear}.
$$
$$
\text{Binary (BCE):}\quad L=-\big[y\log\hat y+(1-y)\log(1-\hat y)\big],\quad \text{output } = \text{sigmoid}.
$$
$$
\text{Multiclass (Cross-Entropy):}\quad L=-\sum_k y_k\log\hat y_k,\quad \text{output } = \text{softmax}.
$$

> **Beautiful simplification (worth memorising).** With **sigmoid + BCE**, or **softmax + cross-entropy**, the output error collapses to
> $$\delta^{(L)} = \hat{\mathbf y} - \mathbf y$$
> — the messy $\phi'$ term cancels exactly. This is *why* these pairings are standard. **Caution:** this clean form holds **only** for these matched output-loss pairs; with MSE on a sigmoid output you must keep the $\sigma'$ factor.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Forward pass (NAT)

A network has 2 inputs, one hidden layer of **2 ReLU** neurons, and a **linear** output neuron. Input $\mathbf{x}=(1,2)$.
Hidden neuron 1: weights $(1,-1)$, bias $0$. Hidden neuron 2: weights $(0.5,0.5)$, bias $-1$.
Output neuron: weights $(2,4)$ (from $h_1,h_2$), bias $1$. **Find the output.**

**Solution.**
$z_{h_1}=1(1)+(-1)(2)+0=-1 \Rightarrow a_{h_1}=\text{ReLU}(-1)=0.$
$z_{h_2}=0.5(1)+0.5(2)-1=0.5 \Rightarrow a_{h_2}=\text{ReLU}(0.5)=0.5.$
Output $=2(0)+4(0.5)+1=\boxed{3}.$

*Notice ReLU killed $h_1$ entirely — a "dead" contribution. That is normal ReLU behaviour and a common point GATE checks.*

### Problem 2 — Counting parameters (NAT)

A fully-connected MLP has architecture **10 → 20 → 15 → 5** (input, two hidden layers, output), each neuron with a bias. **How many trainable parameters?**

**Solution.** Per layer, params $=(n_{\text{in}}+1)\,n_{\text{out}}$:

$$(10+1)\cdot 20 + (20+1)\cdot 15 + (15+1)\cdot 5 = 220 + 315 + 80 = \boxed{615}.$$

> **Trap variant:** if the question says **"weights only / no bias,"** drop every $+1$: $200+300+75=575$. Read the wording carefully.

### Problem 3 — One step of backpropagation (NAT, the hard one)

A **2-2-1** network uses **sigmoid everywhere** and loss $L=\tfrac12(\hat y-y)^2$. Input $\mathbf{x}=(0.5,0.3)$, target $y=1$, learning rate $\eta=0.5$.

- Hidden neuron 1: weights $(0.2,0.4)$, bias $0.1$.
- Hidden neuron 2: weights $(0.3,0.1)$, bias $0.2$.
- Output neuron: weights $(0.5,0.6)$ (from $h_1,h_2$), bias $0.3$.

**Find $\hat y$, the loss, and the updated output weights.**

**Step 1 — Forward pass.**
$z_{h_1}=0.2(0.5)+0.4(0.3)+0.1=0.32,\quad a_{h_1}=\sigma(0.32)=0.5793.$
$z_{h_2}=0.3(0.5)+0.1(0.3)+0.2=0.38,\quad a_{h_2}=\sigma(0.38)=0.5939.$
$z_o=0.5(0.5793)+0.6(0.5939)+0.3=0.9460,\quad \hat y=\sigma(0.9460)=\mathbf{0.7203}.$
$L=\tfrac12(0.7203-1)^2=\mathbf{0.0391}.$

**Step 2 — Output error.** Using $\delta_o=(\hat y-y)\,\hat y(1-\hat y)$:
$$\delta_o=(0.7203-1)(0.7203)(0.2797)=-0.05635.$$

**Step 3 — Output-layer gradients** ($\partial L/\partial w = \delta_o\cdot a_{h_i}$):
$$\frac{\partial L}{\partial w_{o1}}=(-0.05635)(0.5793)=-0.03264,\quad \frac{\partial L}{\partial w_{o2}}=(-0.05635)(0.5939)=-0.03346.$$

**Step 4 — Update** ($w\leftarrow w-\eta\,\partial L/\partial w$, $\eta=0.5$):
$$w_{o1}\leftarrow 0.5-0.5(-0.03264)=\boxed{0.5163},\qquad w_{o2}\leftarrow 0.6-0.5(-0.03346)=\boxed{0.6167}.$$

**(If asked for the hidden layer too):** propagate back with $\delta_{h_i}=\delta_o\,w_{oi}\,a_{h_i}(1-a_{h_i})$, giving $\delta_{h_1}=-0.00687,\ \delta_{h_2}=-0.00815$; then $\partial L/\partial w = \delta_{h_i}\cdot x_j$, updating e.g. $w_{h1,1}:0.2\to 0.2017$.

**Sanity check:** after this single step $\hat y$ rises from $0.7203$ to $0.7301$ and the loss falls from $0.0391$ to $0.0364$ — gradient descent did move toward the target $y=1$. ✔

---

## 6. Common GATE question types on this topic

- **Forward pass / neuron output:** plug numbers through one or more layers (watch the activation).
- **Parameter counting:** total weights / biases in a stated architecture.
- **Linear separability:** "Can a single perceptron represent AND / OR / XOR / this truth table?" (XOR ⇒ no).
- **Minimum architecture:** fewest neurons/layers to solve a given problem (XOR ⇒ 2-2-1).
- **One backprop step:** compute a gradient $\partial L/\partial w$ or an updated weight on a small net.
- **Activation properties:** derivative of sigmoid, its max value $0.25$, ranges, which avoids saturation, ReLU at 0.
- **Conceptual:** why nonlinear activations are required; why backprop = chain rule; effect of $\eta$; vanishing gradients.

---

## 7. Traps, pitfalls & common mistakes

- **Forgetting the bias ($+1$)** when counting parameters — the #1 error.
- **Dropping the $\phi'$ factor** in backprop. $\delta=(\hat y-y)$ alone is correct **only** for softmax+CE or sigmoid+BCE; with **MSE on a sigmoid output you must keep $\hat y(1-\hat y)$**.
- **Wrong update sign:** it is $w-\eta\nabla$ (descent), never $w+\eta\nabla$.
- **Confusing the two learning rules:** the *perceptron rule* $w\leftarrow w+\eta(t-\hat y)x$ (no derivative, step activation) is **not** gradient descent / backprop (which needs a differentiable activation).
- **Believing a single perceptron can do XOR** — it cannot; you need a hidden layer.
- **Using the step function with backprop** — its derivative is 0 (or undefined), so no learning; this is *why* sigmoid/tanh/ReLU exist.
- **All-linear network:** stacking linear layers gives one linear map — no benefit from depth. Nonlinearity is mandatory.
- **Activation order:** weights and bias **first**, activation **after**; $\phi$ is applied element-wise, not to the whole vector at once.
- **Sigmoid saturation / vanishing gradient:** $\sigma'\le 0.25$, so deep sigmoid stacks lose gradient signal — a frequent conceptual trap.
- **Cross-entropy expects probabilities:** feed it the output *after* sigmoid/softmax, not raw logits.
- **Learning rate:** too large → divergence/oscillation; too small → painfully slow convergence.
- **Dead ReLU:** a neuron stuck at $z<0$ for all inputs outputs 0 forever (gradient 0) — it never recovers.

---

## 8. Connection forward

The **Feed-Forward Neural Network (Topic 14)** is the general family of networks where data flows strictly forward with no cycles — and **the MLP is its prototypical, fully-connected member.** Having built the model (perceptron → hidden layers → activations), the forward pass, and the **backpropagation** training engine here, the next topic **zooms out**: it formalises forward propagation for arbitrary depth/width, discusses **how to match output activation to loss**, and states the **Universal Approximation Theorem** — *why* even a single hidden layer can approximate any continuous function. **Backprop, learned here, is the training algorithm for every feed-forward net that follows.**

---

## 9. One-line cheat-sheet summary

> **MLP = layers of neurons (weighted sum → nonlinear activation); a single perceptron is linear and fails on XOR, hidden layers fix it by learning nonlinear features; forward pass computes $\hat y$, and backprop (chain rule + gradient descent, $w\leftarrow w-\eta\,\partial L/\partial w$) trains it — keep the $\phi'$ factor unless using softmax+CE / sigmoid+BCE, and don't forget the $+1$ bias when counting parameters.**

---

### ➡️ Next topic (in order): **Feed-Forward Neural Network** (Topic 14) — the general forward-flowing network family, universal approximation, and matching outputs to losses.

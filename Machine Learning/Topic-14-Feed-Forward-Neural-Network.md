# Topic 14 — Feed-Forward Neural Network (FFNN)

**Subject:** Machine Learning &nbsp;|&nbsp; **Section:** Neural Networks (this is the *last* topic of the section)

---

## 0. Topic header & syllabus map

- **Topic:** Feed-Forward Neural Network (FFNN)
- **Subject:** Machine Learning
- **Where it sits:** This **closes the Neural Networks block** and comes immediately after the **Multi-Layer Perceptron (Topic 13)**. The MLP gave you the *machinery* — neurons, activations, the forward pass, and backpropagation. The FFNN topic **zooms out** to the whole *family* these belong to: it formalises the network as a one-directional flow of computation, states **what this family can theoretically represent (Universal Approximation)**, and answers the practical question *"how do I design and train such a network for any task?"* It is the final supervised topic; the **next chat begins Unsupervised Learning with Clustering Algorithms (Topic 15)**, where labels disappear entirely.

---

## 1. Why this topic exists (motivation + connection)

The MLP showed *one* network solving XOR, but it left three questions unanswered, and those three questions are exactly why "Feed-Forward Neural Network" is its own topic:

1. **How powerful is this class in general?** Is the XOR fix a lucky special case, or can such networks represent *any* function? → the **Universal Approximation Theorem**.
2. **How do I build one for a task that isn't binary?** Regression, multiclass — what goes in the output layer and which loss matches it? → **output-layer design**.
3. **How do I actually train one reliably?** What breaks at depth (vanishing gradients), why random initialisation is mandatory, and how to stop it overfitting. → **practical training**.

> **Bridge (MLP → FFNN):** An **MLP is the prototypical, fully-connected feed-forward network.** "Feed-forward" is the *general* property — information flows strictly forward through a directed acyclic graph, with **no loops or feedback.** Everything you learned in Topic 13 *is* a feed-forward network; this topic names the family, proves what it can do, and equips you to design and train any member of it.

---

## 2. Prerequisites

- **Multi-Layer Perceptron (Topic 13)** — neurons, activation functions and their derivatives, forward propagation, and especially **backpropagation** (we *use* it here, we do not re-derive it).
- **Gradient descent** and its batch/SGD/mini-batch variants.
- **Logistic regression & softmax** — for output-layer design.
- **Ridge regression (Topic 4)** and **Bias–Variance trade-off (Topic 11)** — for the regularization section (weight decay = L2 = ridge; dropout/early-stopping = variance control).

---

## 3. Core idea — intuition first

A feed-forward network is just **a chain of simple functions applied one after another**, with the data only ever moving **forward**:

$$\text{input} \to \text{layer 1} \to \text{layer 2} \to \cdots \to \text{output}.$$

There are **no cycles** — a neuron's output never loops back to influence its own input (that would be a *recurrent* network, which has memory of past inputs). A pure FFNN has **no memory**: the same input always produces the same output.

**Why this family is so flexible — the universal-approximation intuition.** Each hidden neuron produces a soft "step" or "bump" in input space. Add enough of these bumps together with the right heights, and you can trace the shape of *any* smooth curve — the way many rectangular tiles can approximate any area. That is the informal picture behind the theorem that a feed-forward net with **one** hidden layer of enough neurons can approximate any continuous function as closely as you like.

**Output is chosen to match the question being asked.** Predicting a number? End with a plain linear neuron. Yes/no? End with a sigmoid (a probability). One-of-$K$ classes? End with softmax (a probability distribution). The body of the network learns features; the **head** converts those features into the right *kind* of answer.

> **Analogy:** the hidden layers are a factory floor turning raw materials (inputs) into refined parts (features); the output layer is the packaging station that ships the product in the format the customer ordered (a number, a yes/no, or a class label).

---

## 4. Formal treatment

### 4.1 Definition: a feed-forward network as function composition

An FFNN is a **directed acyclic** arrangement of neurons in layers. With $\mathbf a^{(0)}=\mathbf x$ and layers $l=1,\dots,L$:

$$\boxed{\;\mathbf z^{(l)}=W^{(l)}\mathbf a^{(l-1)}+\mathbf b^{(l)},\qquad \mathbf a^{(l)}=\phi^{(l)}\!\big(\mathbf z^{(l)}\big)\;}$$

so the whole network is one **composed function**

$$\hat{\mathbf y}=f(\mathbf x)=\phi^{(L)}\!\Big(W^{(L)}\phi^{(L-1)}\big(\cdots\phi^{(1)}(W^{(1)}\mathbf x+\mathbf b^{(1)})\cdots\big)+\mathbf b^{(L)}\Big).$$

**Defining property — no feedback:** the computation graph is acyclic, so $f$ depends only on the current input.

| | **Feed-forward (FFNN / MLP)** | **Recurrent (RNN)** |
|---|---|---|
| Connections | acyclic, forward only | contain cycles / feedback |
| Memory of past inputs | **none** | yes (hidden state) |
| Same input → | always same output | output can depend on history |

> The MLP is the **fully-connected** FFNN (every neuron in a layer connects to every neuron in the next). FFNN is the broader umbrella; for GATE-DA purposes treat MLP and (dense) FFNN as the same model and the names interchangeably, while remembering the FFNN-vs-RNN distinction.

### 4.2 General forward propagation

Run §4.1 left to right: $\mathbf x \to \mathbf z^{(1)}\to \mathbf a^{(1)}\to\cdots\to\mathbf a^{(L)}=\hat{\mathbf y}$. **Per layer: matrix-multiply by weights, add bias, apply activation element-wise.** (Training is backprop + gradient descent, already covered in Topic 13 §4.8 — reuse those four equations and the update $W\leftarrow W-\eta\,\partial L/\partial W$.)

### 4.3 Output-layer design (match activation to loss) — *the core "new" skill here*

The hidden layers can use any activation (usually ReLU); the **output layer + loss** are fixed by the **task**:

| Task | # output units | Output activation | Loss | Output error $\delta^{(L)}$ |
|---|---|---|---|---|
| **Regression** | 1 (or $m$) | **Linear (identity)** | MSE $\;\tfrac12\sum(\hat y-y)^2$ | $\hat y-y$ |
| **Binary classification** | 1 | **Sigmoid** | Binary cross-entropy | $\hat y-y$ |
| **Multiclass ($K$ classes)** | $K$ | **Softmax** | Categorical cross-entropy | $\hat{\mathbf y}-\mathbf y$ |

$$\text{Softmax: } \hat y_k=\frac{e^{z_k}}{\sum_{j=1}^{K}e^{z_j}}\quad(\text{outputs are positive and sum to }1).$$

> **The reason these pairings are standard:** for **sigmoid + BCE** and **softmax + cross-entropy**, the output error collapses to the clean $\delta^{(L)}=\hat{\mathbf y}-\mathbf y$ — the activation derivative cancels, so training is stable and gradients don't vanish at the output. **Mismatching** (e.g. MSE on a softmax output) reintroduces the derivative and slows learning. *(Practical note: softmax is computed after subtracting the max logit for numerical stability — same probabilities, no overflow.)*

### 4.4 Universal Approximation Theorem (UAT) — *the theoretical centrepiece*

> **Theorem (Cybenko 1989; Hornik 1991, informal).** A feed-forward network with a **single hidden layer** containing a **finite** number of neurons and a suitable activation (any non-constant, bounded, continuous "squashing" function such as sigmoid; later generalised to any **non-polynomial** activation) can **approximate any continuous function on a compact (closed and bounded) subset of $\mathbb R^n$ to arbitrary accuracy** $\varepsilon>0$.

**State the conditions and caveats precisely (GATE tests exactly these):**

- **Continuity & compact domain** are required — it approximates *continuous* functions on a *bounded* region, not arbitrary discontinuous functions on all of $\mathbb R^n$.
- **"Enough" neurons:** the theorem guarantees that *some* network exists; it gives **no bound** on the width. Achieving small error may need a **very large (possibly exponential)** number of hidden units.
- **Existence ≠ trainability ≠ generalization.** UAT says a good network *exists*; it does **not** say gradient descent will *find* it, nor that the found network will *generalize* to unseen data. (This is why we still need good training and regularization.)
- One hidden layer **suffices in principle** — depth is about *efficiency*, not *possibility* (next).

This theorem is *why* the MLP could represent XOR and far more: the feed-forward family is, in principle, **universal**.

### 4.5 Depth vs width

A single wide hidden layer is already a universal approximator (4.4), so **why go deep?** **Efficiency.** For certain functions, a **deep** network needs only **polynomially** many neurons where a **shallow** one would need **exponentially** many. Depth lets each layer reuse and recombine the features of the previous one (hierarchical features), giving the same expressive power with far fewer parameters — at the cost of harder training (vanishing gradients, §4.7).

### 4.6 The training loop & terminology (definitions GATE asks)

Repeat until convergence: **forward pass → compute loss → backward pass (backprop) → update weights.** Key terms:

- **Epoch** = one full pass over the *entire* training set.
- **Iteration / update / step** = one parameter update, using **one (mini-)batch**.
- **Batch size $B$** = number of examples per update.

$$\boxed{\;\text{updates per epoch}=\left\lceil \tfrac{N}{B}\right\rceil,\qquad \text{total updates}=E\cdot\left\lceil \tfrac{N}{B}\right\rceil\;}$$

for $N$ training examples and $E$ epochs. (The ceiling accounts for a final smaller partial batch.)

### 4.7 Vanishing & exploding gradients (deeper than the MLP note)

Backprop multiplies error signals layer by layer: $\delta^{(l)}=\big((W^{(l+1)})^\top\delta^{(l+1)}\big)\odot\phi'(\mathbf z^{(l)})$. Across $L$ layers the gradient at early layers contains a **product** of many $\phi'$ and weight factors:

- **Sigmoid/tanh saturate:** $|\sigma'|\le 0.25$, $|\tanh'|\le 1$. Multiplying many sub-1 factors makes the gradient **shrink exponentially with depth** → early layers barely update → **vanishing gradient.**
- **Large weights:** the product can instead **blow up** → **exploding gradient** (unstable, NaNs).
- **ReLU mitigates vanishing:** its derivative is exactly **1** on the active region ($z>0$), so signals pass through undiminished. (Cost: "dead" neurons where $z<0$ always, giving derivative 0.)

This is the practical reason ReLU largely replaced sigmoid/tanh in hidden layers, and why depth (4.5) is hard to exploit naively.

### 4.8 Weight initialisation & symmetry breaking (classic trap)

> **You cannot initialise all weights to the same value (e.g. 0).** If every neuron in a layer starts with identical weights, they compute **identical outputs** and therefore receive **identical gradients**, so they update identically and **remain identical forever** — the layer behaves like a single neuron. This "symmetry" is never broken.

Therefore weights are initialised **randomly** (to small, varied values; e.g. scaled by layer size). **Biases may be set to 0** — only the *weights* need symmetry-breaking.

### 4.9 Regularization & overfitting in FFNNs (cross-topic bridge)

A network with many parameters can **overfit** (low bias, high variance — recall **Bias–Variance, Topic 11**). Standard controls:

- **L2 / weight decay:** add $\lambda\sum w^2$ to the loss → shrinks weights. **This is exactly the ridge penalty (Topic 4) applied to network weights.** Gradient gains a $+2\lambda w$ term, so the update becomes $w\leftarrow (1-2\eta\lambda)w-\eta\,\partial L/\partial w$ (decay toward 0).
- **Dropout:** randomly "drop" a fraction of neurons each training step → prevents co-adaptation, acts like training an ensemble; all neurons are used (scaled) at test time.
- **Early stopping:** halt training when validation error starts rising → caps model complexity (a variance control).

> **Bridge:** regularization in NNs is the same bias–variance management you saw with ridge and cross-validation — bigger/over-trained nets lower bias but raise variance, and these tools trade a little bias for a large drop in variance.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Softmax output + cross-entropy loss (NAT)

A 3-class FFNN produces output-layer logits (pre-softmax) $\mathbf z=(2.0,\,1.0,\,0.1)$ for a sample whose **true class is class 1 (the first)**. Find (a) the softmax probabilities and (b) the cross-entropy loss.

**Solution.**
$e^{2}=7.389,\; e^{1}=2.718,\; e^{0.1}=1.105$; sum $=11.213$.

$$\hat{\mathbf y}=\Big(\tfrac{7.389}{11.213},\tfrac{2.718}{11.213},\tfrac{1.105}{11.213}\Big)=(0.659,\,0.242,\,0.099)\quad(\text{sums to }1).$$

Cross-entropy with one-hot true label $\mathbf y=(1,0,0)$: $L=-\sum_k y_k\log\hat y_k=-\ln(0.659)=\boxed{0.417}.$

> **Trap:** CE only uses the probability of the **true** class. Had the true class been the lowest-scoring one (prob $0.099$), the loss would be $-\ln(0.099)=2.317$ — far larger. A confident wrong prediction is punished heavily.

### Problem 2 — Number of weight updates (NAT)

A network is trained on $N=12000$ samples with **mini-batch size $B=200$** for $E=15$ **epochs**. How many weight updates occur in total?

**Solution.** Updates per epoch $=\lceil N/B\rceil=\lceil 12000/200\rceil=60$. Total $=E\cdot 60=15\times 60=\boxed{900}.$

> **Trap (partial batch):** if instead $N=1000,\,B=300$, then $\lceil 1000/300\rceil=\lceil 3.33\rceil=\mathbf{4}$ updates/epoch (the 4th batch holds the leftover 100 samples) — **not 3**. Always take the ceiling. (Don't confuse an *epoch* — one full data pass — with an *iteration/update* — one batch.)

### Problem 3 — Concept check (MSQ, trap-heavy)

Which of the following are **TRUE** for feed-forward neural networks?
**(A)** A network whose every neuron uses a linear activation can represent any continuous function if it is deep enough.
**(B)** By the Universal Approximation Theorem, a single hidden layer with enough neurons can approximate any continuous function on a compact set to arbitrary accuracy.
**(C)** Initialising all weights to zero still allows the hidden neurons to learn different features.
**(D)** Using ReLU instead of sigmoid in hidden layers helps mitigate the vanishing-gradient problem.

**Solution.**
- **(A) False.** A composition of linear maps is itself linear ($W_2W_1\mathbf x$), so an all-linear network — at any depth — can only represent linear functions. **Nonlinear activations are essential.**
- **(B) True.** This is precisely the UAT (one hidden layer of *sufficient* width; existence only, no width bound).
- **(C) False.** Identical initial weights ⇒ identical outputs ⇒ identical gradients ⇒ neurons stay identical (**symmetry not broken**). Random init is required.
- **(D) True.** ReLU's derivative is 1 on its active region, so gradients pass without the $\le 0.25$ shrinkage of sigmoid.

**Answer: (B) and (D).**

---

## 6. Common GATE question types on this topic

- **Identify feed-forward vs recurrent** networks from a description/diagram (cycles ⇒ recurrent).
- **Universal Approximation Theorem** statement and its **conditions/caveats** (continuous, compact, existence-only, ≠ trainability/generalization).
- **Output-layer / loss matching:** which activation + loss for regression / binary / multiclass.
- **Softmax & cross-entropy computation** (compute probabilities and/or loss from logits).
- **Epoch / iteration / update counting:** total updates $=E\lceil N/B\rceil$.
- **Vanishing/exploding gradient reasoning:** why deep sigmoid nets stall; why ReLU helps.
- **Weight-initialisation / symmetry-breaking** (why not all zeros).
- **Why depth over width** (parameter efficiency); **why nonlinear activations** (all-linear collapses).
- Forward pass / parameter counting (same mechanics as MLP, Topic 13).

---

## 7. Traps, pitfalls & common mistakes

- **All-linear network ≠ universal:** stacking linear layers gives one linear map; depth without nonlinearity is useless.
- **Misreading the UAT:** it guarantees *existence* of an approximating network and needs *continuity + a compact domain*; it does **not** bound the number of neurons, nor promise that training finds it or that it generalizes.
- **Zero / identical initialisation:** breaks learning by symmetry — neurons stay clones. Use random weights (biases may be 0).
- **Epoch vs iteration confusion:** one epoch = full pass; updates per epoch $=\lceil N/B\rceil$. Forgetting the **ceiling** for a partial last batch loses a count.
- **Output–loss mismatch:** use linear+MSE (regression), sigmoid+BCE (binary), softmax+CE (multiclass). Mismatching reintroduces the activation derivative and slows training; the clean $\delta=\hat y-y$ holds **only** for the matched pairs.
- **Cross-entropy uses only the true class's probability** — don't sum over all classes' probabilities as if they all contribute (the one-hot zeros them out).
- **Sigmoid saturation:** $\sigma'\le 0.25$ ⇒ deep sigmoid stacks vanish; this is a *gradient* problem, not a representation problem.
- **Exploding gradients** (large weights) are the flip side of vanishing — instability/NaNs.
- **Softmax overflow:** subtract the max logit before exponentiating (numerical stability) — probabilities are unchanged.
- **Dead ReLU:** a unit with $z<0$ for all inputs has gradient 0 and never recovers.
- **Confusing weight decay with anything exotic:** L2 weight decay *is* the ridge penalty (Topic 4) on the network's weights.

---

## 8. Connection forward

This finishes the **supervised learning journey**: you went from a single straight line (linear/logistic regression), to margins and kernels (SVM), to recursive splits (decision trees), to evaluation (bias–variance, cross-validation), and finally to networks that **learn their own nonlinear features and are universal approximators** (MLP → FFNN). Every model so far required **labelled** data $(\mathbf x,y)$.

The **next chat opens the Unsupervised Learning section with Clustering Algorithms (Topic 15)**, where the labels $y$ vanish: instead of predicting a target, we look for **structure hidden in the inputs themselves** — grouping similar points together. The mindset shifts from "fit a function to labels" to "discover organisation without supervision," setting up k-means/k-medoid, hierarchical clustering, and (later) PCA.

---

## 9. One-line cheat-sheet summary

> **FFNN = a one-directional (acyclic, no-memory) composition of nonlinear layers, trained by backprop; the Universal Approximation Theorem says one wide hidden layer can approximate any continuous function on a compact set (existence only, not trainability), depth buys parameter-efficiency; pick the output by task (linear+MSE / sigmoid+BCE / softmax+CE, all giving $\delta=\hat y-y$); random-init the weights (zero-init breaks symmetry), use ReLU to dodge vanishing gradients, and L2/dropout/early-stopping to fight overfitting; total updates $=E\lceil N/B\rceil$.**

---

### ➡️ Next topic (new chat — Unsupervised Learning begins): **Clustering Algorithms (overview)** (Topic 15) — finding structure in *unlabelled* data, leading into k-means/k-medoid and hierarchical clustering.

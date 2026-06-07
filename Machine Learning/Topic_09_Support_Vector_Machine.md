## Topic 9 — Support Vector Machine (SVM)

*GATE DA · Machine Learning · Classification Methods (Chat 2, Topic 9 — closes this cluster)*

---

## 0. Topic header & syllabus map

- **Topic:** Support Vector Machine (SVM)
- **Subject:** Machine Learning
- **Where it sits:** Final topic of the **Classification Methods** cluster. Comes right after **Linear Discriminant Analysis** and is the last classifier before the syllabus turns to **Decision Trees** (a new chat). SVM is a **discriminative, geometric, distribution-free** classifier: it finds the **maximum-margin** separating hyperplane, depends only on a few critical boundary points (**support vectors**), and—via the **kernel trick**—produces non-linear boundaries.

---

## 1. Why this topic exists (motivation + connection)

**The real problem it solves:** When many separating lines exist, *which one generalizes best?* SVM answers: the one that sits as **far as possible from both classes** (the widest "street"). A fat margin leaves the most room for unseen test points to fall on the correct side.

**What the previous topic (LDA) couldn't handle:** LDA is **generative** — it *assumes* each class is Gaussian with a **shared covariance** and lets **every** training point (through the class means and covariance) shape the boundary. Three weaknesses follow: (1) if the Gaussian / equal-covariance assumption is wrong, the boundary is suboptimal; (2) distant points and **outliers still move the means/covariance**, so they perturb the boundary; (3) LDA is **inherently linear** (no built-in way to bend the boundary).

**How SVM fixes that:** SVM makes **no distributional assumption**. It optimizes a purely geometric objective (margin width), so the boundary is determined **only by the borderline points** — remove a far-away point and nothing changes. It explicitly **maximizes the margin** for better generalization, and the **kernel trick** lets it carve **non-linear** boundaries without ever leaving the optimization. So **LDA → SVM = from "model the class distributions, use all data" to "make no assumptions, find the widest-margin boundary from the critical points, and kernelize for non-linearity."**

---

## 2. Prerequisites

- **Hyperplanes & vector geometry:** $\mathbf{w}^\top\mathbf{x}+b=0$, dot products, vector norms.
- **Distance from a point to a hyperplane.**
- **Linear separability** and the idea of a decision boundary.
- **Constrained optimization basics:** Lagrange multipliers, the idea of a dual (KKT conditions at a conceptual level).
- **Quadratic forms / inner products** (for kernels).
- **Bias–variance trade-off** (Topic 11, forward ref) — to read the role of $C$.
- **Feature scaling** (from k-NN) — SVMs, especially RBF, are scale-sensitive.

---

## 3. Core idea — intuition first

Picture two clusters of points and a road you want to pave between them. Infinitely many roads separate the clusters, but the **safest** road is the **widest** one — its centre line stays maximally clear of both sides. SVM paves exactly that road: the **maximum-margin hyperplane**, with the road's edges just touching the nearest points of each class. Those touching points are the **support vectors**; they alone "hold up" the boundary. Every other point could be deleted without moving the road.

If the clusters can't be separated by a straight road (they interlock), SVM does one of two things: (a) allow a few points to sit on the wrong side, paying a penalty (**soft margin**, knob $C$); or (b) **lift** the points into a higher-dimensional space where a flat road *does* separate them, using a **kernel** to do this implicitly and cheaply.

> Headline: **SVM = widest street between the classes, propped up by the support vectors; kernels bend the street.**

---

## 4. Formal treatment

### 4.1 Hyperplane, classification, and distance

$$\text{Hyperplane: } \mathbf{w}^\top\mathbf{x}+b=0,\qquad \hat{y}=\operatorname{sign}(\mathbf{w}^\top\mathbf{x}+b)$$
$$\text{Distance from } \mathbf{x}_0:\quad \frac{|\mathbf{w}^\top\mathbf{x}_0+b|}{\lVert\mathbf{w}\rVert}$$

**Functional vs geometric margin** (classic distinction):
- Functional margin of $(\mathbf{x}_i,y_i)$: $\hat\gamma_i=y_i(\mathbf{w}^\top\mathbf{x}_i+b)$ — *not* scale-invariant.
- Geometric margin: $\gamma_i=\dfrac{y_i(\mathbf{w}^\top\mathbf{x}_i+b)}{\lVert\mathbf{w}\rVert}$ — the true distance, scale-invariant.

SVM fixes the scale by **canonical normalization**: make the smallest functional margin equal to 1, i.e. $\min_i y_i(\mathbf{w}^\top\mathbf{x}_i+b)=1$. Then:

$$\boxed{\text{margin half-width}=\frac{1}{\lVert\mathbf{w}\rVert},\qquad \text{full margin (street width)}=\frac{2}{\lVert\mathbf{w}\rVert}}$$

### 4.2 Hard-margin SVM (linearly separable)

Maximizing $2/\lVert\mathbf{w}\rVert$ ⟺ minimizing $\tfrac12\lVert\mathbf{w}\rVert^2$:

$$\boxed{\min_{\mathbf{w},b}\ \tfrac12\lVert\mathbf{w}\rVert^2 \quad\text{s.t.}\quad y_i(\mathbf{w}^\top\mathbf{x}_i+b)\ge 1\ \ \forall i}$$

> A **convex quadratic program** → unique global optimum. Equality $y_i(\mathbf{w}^\top\mathbf{x}_i+b)=1$ holds exactly for the **support vectors**.

### 4.3 Soft-margin SVM (overlap / noise)

Allow violations via slack $\xi_i\ge0$, penalized by $C$:

$$\boxed{\min_{\mathbf{w},b,\boldsymbol\xi}\ \tfrac12\lVert\mathbf{w}\rVert^2+C\sum_i\xi_i \quad\text{s.t.}\quad y_i(\mathbf{w}^\top\mathbf{x}_i+b)\ge 1-\xi_i,\ \ \xi_i\ge0}$$

**Slack interpretation** ($\xi_i=\max(0,\,1-y_i(\mathbf{w}^\top\mathbf{x}_i+b))$):

| $\xi_i$ | meaning |
|---|---|
| $\xi_i=0$ | correct & **outside** the margin (or on it) |
| $0<\xi_i<1$ | correct side but **inside** the margin (margin violation) |
| $\xi_i=1$ | exactly **on** the decision boundary |
| $\xi_i>1$ | **misclassified** |

**Role of $C$ (regularization knob — watch the direction!):**
- **Large $C$** → violations heavily penalized → **narrow** margin, fits training data → **low bias, high variance** (toward hard margin, overfit risk).
- **Small $C$** → violations cheap → **wide** margin, more tolerant → **high bias, low variance** (underfit risk).

> Trap: $C$ behaves **opposite** to ridge's $\lambda$. Large $C$ = *less* regularization. (Forward link to Bias–Variance, Topic 11.)

### 4.4 Dual formulation (Lagrangian)

$$\boxed{\max_{\boldsymbol\alpha}\ \sum_i\alpha_i-\tfrac12\sum_{i,j}\alpha_i\alpha_j\,y_iy_j\,K(\mathbf{x}_i,\mathbf{x}_j)\quad\text{s.t.}\quad \sum_i\alpha_iy_i=0,\ \ 0\le\alpha_i\le C}$$

(For hard margin, drop the upper bound: $\alpha_i\ge0$.) Recover the primal weight and decision function:

$$\mathbf{w}=\sum_i\alpha_iy_i\mathbf{x}_i,\qquad f(\mathbf{x})=\operatorname{sign}\!\Big(\sum_i\alpha_iy_iK(\mathbf{x}_i,\mathbf{x})+b\Big)$$

> **Two crucial facts:** (1) the data enters **only through inner products** $K(\mathbf{x}_i,\mathbf{x}_j)$ → kernels (§4.6); (2) **most $\alpha_i=0$** → the solution is **sparse**, supported only by the support vectors.

**KKT → support-vector types:**

| $\alpha_i$ | location | $y_if(\mathbf{x}_i)$ |
|---|---|---|
| $\alpha_i=0$ | **non**-support vector, outside margin | $\ge 1$ |
| $0<\alpha_i<C$ | **free SV**, exactly on the margin ($\xi_i=0$) | $=1$ |
| $\alpha_i=C$ | **bounded SV**, inside margin or misclassified | $\le 1$ |

**Bias term** from any free SV $\mathbf{x}_s$ (with $0<\alpha_s<C$): $\;b=y_s-\sum_i\alpha_iy_iK(\mathbf{x}_i,\mathbf{x}_s)$.

*(Useful identity for hard margin: $\sum_i\alpha_i=\lVert\mathbf{w}\rVert^2$ at the optimum — handy check.)*

### 4.5 Hinge-loss (regularized-risk) view

SVM is equivalently **regularized empirical risk** with the **hinge loss** $\ell(y,f)=\max(0,\,1-yf)$:

$$\min_{\mathbf{w},b}\ \tfrac12\lVert\mathbf{w}\rVert^2+C\sum_i\max\!\big(0,\,1-y_i(\mathbf{w}^\top\mathbf{x}_i+b)\big)$$

The hinge loss is 0 once a point is correctly classified **beyond** the margin, and grows linearly inside/across it — this is what produces the sparse support-vector solution. (Contrast: logistic regression uses log-loss, which is never exactly 0.)

### 4.6 The kernel trick

Replace inner products by a **kernel** $K(\mathbf{x},\mathbf{z})=\phi(\mathbf{x})^\top\phi(\mathbf{z})$, computing a high-dimensional inner product **without** forming $\phi$:

| Kernel | $K(\mathbf{x},\mathbf{z})$ | Note |
|---|---|---|
| **Linear** | $\mathbf{x}^\top\mathbf{z}$ | no mapping |
| **Polynomial** | $(\mathbf{x}^\top\mathbf{z}+c)^d$ | degree-$d$ interactions |
| **RBF / Gaussian** | $\exp(-\gamma\lVert\mathbf{x}-\mathbf{z}\rVert^2)$, $\gamma=\tfrac{1}{2\sigma^2}$ | infinite-dim feature space |
| **Sigmoid** | $\tanh(\kappa\,\mathbf{x}^\top\mathbf{z}+\theta)$ | neural-net-like |

> **Mercer's condition:** $K$ is a valid kernel iff its **Gram matrix is positive semi-definite** for any set of points. Not every function is a kernel.
>
> **RBF $\gamma$ direction:** large $\gamma$ (small $\sigma$) → narrow bumps → **wiggly** boundary (overfit); small $\gamma$ → smooth boundary (underfit).

*(GATE doesn't ask the full Lagrangian/dual derivation; it tests the primal/dual statements, the geometry, kernel computations, and the roles of $C,\gamma$. Derivation skipped accordingly.)*

---

## 5. Worked problems (GATE-style)

### Problem 1 — Hard-margin SVM: hyperplane, margin, support vectors (NAT)

Data: $A=(3,3)\,[+1]$, $B=(5,2)\,[+1]$, $C=(1,1)\,[-1]$, $D=(0,1)\,[-1]$.
Find $\mathbf{w}$, $b$, the margin, and the support vectors.

**Step 1 — Find the closest opposing pair.** $A$ and $C$ are the nearest $+/-$ points; the optimal hyperplane is their **perpendicular bisector**. So $\mathbf{w}\parallel(A-C)=(2,2)$ ⟹ $\mathbf{w}=(a,a)$.

**Step 2 — Apply canonical conditions** ($y_i(\mathbf{w}^\top\mathbf{x}_i+b)=1$ for the SVs):
$$A:\ 3a+3a+b=+1\ \Rightarrow\ 6a+b=1,\qquad C:\ a+a+b=-1\ \Rightarrow\ 2a+b=-1$$
Subtract: $4a=2\Rightarrow a=0.5$, then $b=-1-2(0.5)=-2$.
$$\boxed{\mathbf{w}=(0.5,\,0.5),\quad b=-2}\qquad(\text{boundary: } x_1+x_2=4)$$

**Step 3 — Margin.** $\lVert\mathbf{w}\rVert=\sqrt{0.25+0.25}=\tfrac{1}{\sqrt2}$, so
$$\text{margin width}=\frac{2}{\lVert\mathbf{w}\rVert}=2\sqrt2\approx 2.83.$$

**Step 4 — Verify the others are non-SVs.**
$B=(5,2)$: $\mathbf{w}^\top B+b=2.5+1-2=1.5$, $y_Bf=1.5>1$ ✓. $\;D=(0,1)$: $0+0.5-2=-1.5$, $y_Df=1.5>1$ ✓.

**Support vectors: $A=(3,3)$ and $C=(1,1)$** (both lie exactly on the margin, $y_if=1$).

**(Dual check.)** With one SV per class, $\sum\alpha_iy_i=0\Rightarrow\alpha_A=\alpha_C=\alpha$. Then $\mathbf{w}=\alpha(A)-\alpha(C)=\alpha(2,2)=(0.5,0.5)\Rightarrow\alpha=0.25$. And $\sum\alpha_i=0.5=\lVert\mathbf{w}\rVert^2$ ✓.

---

### Problem 2 — Kernel trick: polynomial kernel = explicit feature map (NAT)

For 2-D inputs and kernel $K(\mathbf{x},\mathbf{z})=(\mathbf{x}^\top\mathbf{z})^2$:
**(a)** find the feature map $\phi$; **(b)** compute $K$ for $\mathbf{x}=(1,2),\ \mathbf{z}=(3,1)$ both ways.

**(a)** Expand:
$$(\mathbf{x}^\top\mathbf{z})^2=(x_1z_1+x_2z_2)^2=x_1^2z_1^2+2x_1x_2z_1z_2+x_2^2z_2^2$$
This is $\phi(\mathbf{x})^\top\phi(\mathbf{z})$ with
$$\boxed{\phi(\mathbf{x})=\big(x_1^2,\ \sqrt2\,x_1x_2,\ x_2^2\big)}$$
So a degree-2 kernel implicitly maps $\mathbb{R}^2\to\mathbb{R}^3$ — and we never compute $\phi$ in practice.

**(b)** Direct: $\mathbf{x}^\top\mathbf{z}=1(3)+2(1)=5\Rightarrow K=5^2=\mathbf{25}$.
Via $\phi$: $\phi(\mathbf{x})=(1,\,2\sqrt2,\,4)$, $\phi(\mathbf{z})=(9,\,3\sqrt2,\,1)$;
$$\phi(\mathbf{x})^\top\phi(\mathbf{z})=1(9)+2\sqrt2\cdot3\sqrt2+4(1)=9+12+4=\mathbf{25}\ \checkmark$$

> **Lesson:** the kernel computes a higher-dimensional inner product at the cost of the original-space dot product. *(Aside: a degree-$d$ polynomial kernel $(\mathbf{x}^\top\mathbf{z}+1)^d$ in $\mathbb{R}^n$ corresponds to a feature space of dimension $\binom{n+d}{d}$ — RBF corresponds to an **infinite**-dimensional space.)*

---

### Problem 3 — Soft margin: slacks, point status, and the role of $C$ (MSQ + NAT)

Given $\mathbf{w}=(1,1),\ b=-3$ (so $f(\mathbf{x})=x_1+x_2-3$). For each point compute $y_if$, the slack $\xi=\max(0,1-y_if)$, and its status:

| Point | $y$ | $f=x_1+x_2-3$ | $y_if$ | $\xi$ | Status |
|---|---|---|---|---|---|
| $(4,2)$ | $+1$ | $3$ | $3$ | $0$ | correct, beyond margin (**non-SV**) |
| $(2,2)$ | $+1$ | $1$ | $1$ | $0$ | **on margin** (free SV) |
| $(2,1.5)$ | $+1$ | $0.5$ | $0.5$ | $0.5$ | inside margin, correct (**bounded SV**) |
| $(1,1)$ | $+1$ | $-1$ | $-1$ | $2$ | **misclassified** (bounded SV) |
| $(0,1)$ | $-1$ | $-2$ | $2$ | $0$ | correct, beyond margin (**non-SV**) |

**Total slack** $\sum\xi=0+0+0.5+2+0=\mathbf{2.5}$. If $C=2$, the penalty term $C\sum\xi=2(2.5)=\mathbf{5}$.

**Which statements are TRUE?**
- **(A)** Points with $\xi>1$ are misclassified. → **True** (e.g. $(1,1)$).
- **(B)** Increasing $C$ widens the margin. → **False** — large $C$ *narrows* it (penalizes slack more).
- **(C)** Only support vectors (here the last three with $\xi$ or on-margin) affect the boundary. → **True**; the two points with $y_if>1$ are non-SVs.
- **(D)** A point with $0<\xi<1$ is on the wrong side of the decision boundary. → **False** — it's on the *correct* side but inside the margin.

**Answer: A, C.**

> **Lesson:** slack $\xi$ measures how far a point intrudes past its margin; $\xi>1$ ⟹ misclassified. $C$ trades margin width against violations (large $C$ → narrow margin, overfit risk).

---

## 6. Common GATE question types on this topic

- **Compute $\mathbf{w}$, $b$, margin width $=2/\lVert\mathbf{w}\rVert$, and identify support vectors** from a small dataset (Problem 1).
- **Distance from a point to the hyperplane** $|\mathbf{w}^\top\mathbf{x}+b|/\lVert\mathbf{w}\rVert$.
- **Kernel value computation** and **kernel ↔ feature map** equivalence; feature-space dimension (Problem 2).
- **Slack-variable interpretation** and **role of $C$** (and bias–variance) (Problem 3).
- **Dual facts:** the constraint $\sum_i\alpha_iy_i=0$, box constraint $0\le\alpha_i\le C$, and $\mathbf{w}=\sum_i\alpha_iy_i\mathbf{x}_i$.
- **Hinge loss** definition / which loss SVM minimizes.
- **Hard vs soft margin**, linear vs kernel (RBF/polynomial) choice; **why maximize the margin** (generalization).
- **Counting / identifying support vectors** from $\alpha_i$ values.

---

## 7. Traps, pitfalls & common mistakes

- **Margin is $2/\lVert\mathbf{w}\rVert$ (full), $1/\lVert\mathbf{w}\rVert$ (half).** Mixing these halves/doubles the answer.
- **Functional ≠ geometric margin.** $y(\mathbf{w}^\top\mathbf{x}+b)$ is functional; divide by $\lVert\mathbf{w}\rVert$ for the actual distance.
- **$C$ is the wrong way round from $\lambda$.** *Large* $C$ = *less* tolerance = *narrower* margin = overfit risk. (Easy to invert under pressure.)
- **Only support vectors matter.** Deleting a non-SV changes nothing; deleting an SV changes the boundary. Don't average over all points like LDA.
- **Slack thresholds:** $\xi>1$ ⟹ misclassified; $0<\xi<1$ ⟹ correct side but inside the margin; $\xi=0$ ⟹ on/beyond the margin.
- **$\mathbf{w}$ is not directly available with non-linear kernels** (it lives in feature space). Use $f(\mathbf{x})=\sum_i\alpha_iy_iK(\mathbf{x}_i,\mathbf{x})+b$.
- **Not every function is a kernel** — it must satisfy **Mercer's condition** (PSD Gram matrix).
- **RBF $\gamma$ vs $\sigma$:** large $\gamma$ = small $\sigma$ = wiggly/overfit; small $\gamma$ = smooth/underfit. Don't confuse the direction.
- **Forgetting feature scaling.** Like k-NN, SVM (esp. RBF) is scale-sensitive — unscaled large features dominate distances/inner products.
- **The dual constraint $\sum\alpha_iy_i=0$ must hold** — a common check that gets dropped.
- **SVM is natively binary.** Multiclass needs one-vs-rest or one-vs-one.
- **Maximizing margin = minimizing $\tfrac12\lVert\mathbf{w}\rVert^2$** (inverse relation) — not maximizing $\lVert\mathbf{w}\rVert$.

---

## 8. Connection forward

Every classifier so far — regression-based (logistic), distance-based (k-NN), probabilistic (Naive Bayes, LDA), and margin-based (SVM) — works in a **numeric vector space**: it needs features encoded as numbers, is usually **scale-sensitive**, and produces a single **global** decision surface (linear, or smooth-non-linear with a kernel) that is often hard to interpret (a kernel SVM is effectively a black box).

The next topic, **Decision Trees** (a new chat), is a clean break: it builds a hierarchy of **simple axis-aligned "if–then" splits**, one feature at a time, carving the space into **piecewise-constant rectangular regions**. This buys what SVM/LDA couldn't easily give: **human-readable rules** (a flowchart), **native handling of categorical features and mixed data**, **no feature scaling**, and **non-linear interactions without kernels**. So the move is **SVM → Decision Trees = from "one global (possibly kernelized) decision surface in a metric space" to "a hierarchy of interpretable, scale-free, axis-aligned rules."**

*(Sideways link: SVM's hinge loss + L2 penalty is the **regularized-risk** twin of logistic regression's log-loss + penalty — two members of the linear-classifier family seen through different loss functions.)*

---

## 9. One-line cheat-sheet summary

**SVM:** maximize the margin $=2/\lVert\mathbf{w}\rVert$ ⟺ $\min\tfrac12\lVert\mathbf{w}\rVert^2$ s.t. $y_i(\mathbf{w}^\top\mathbf{x}_i+b)\ge1$; boundary set only by **support vectors** ($\alpha_i>0$); **soft margin** adds slack $\xi_i$ with knob $C$ (large $C$ → narrow margin/overfit); **dual** depends on data via inner products $\Rightarrow$ **kernel trick** ($K=(\mathbf{x}^\top\mathbf{z}+c)^d$, RBF $e^{-\gamma\lVert\mathbf{x}-\mathbf{z}\rVert^2}$, Mercer/PSD); equivalently minimizes **hinge loss + L2**; distribution-free, sparse, scale-sensitive, natively binary.

---

### ➡️ Next topic
**Topic 10 — Decision Trees** — this **begins a new chat** (Chat 3: Trees & model evaluation). Start that chat when ready; this completes the Classification Methods cluster (Chat 2).

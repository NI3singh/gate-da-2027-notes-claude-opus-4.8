## Topic 8 — Linear Discriminant Analysis (LDA)

*GATE DA · Machine Learning · Classification Methods (Chat 2, Topic 8)*

---

## 0. Topic header & syllabus map

- **Topic:** Linear Discriminant Analysis (LDA)
- **Subject:** Machine Learning
- **Where it sits:** Third topic of the **Classification Methods** cluster. Comes right after **Naive Bayes** and leads into **Support Vector Machine (SVM)**. LDA has **two faces** that GATE tests: (1) a **generative classifier** (Bayes-optimal under Gaussian classes with a *shared* covariance) producing a **linear** boundary, and (2) a **supervised dimensionality-reduction / projection** method (Fisher's linear discriminant). It is the direct generalization of Gaussian Naive Bayes — it keeps the Gaussian/Bayes machinery but lets features be **correlated** (full shared covariance).

---

## 1. Why this topic exists (motivation + connection)

**The real problem it solves:** Separate classes with a **linear** rule that also accounts for the *shape and correlation* of each class's spread — and, as a bonus, **project** high-dimensional data onto the few directions that best separate the classes.

**What the previous topic (Naive Bayes) couldn't handle:** Gaussian Naive Bayes assumes features are **independent given the class** → a **diagonal** covariance. It is therefore *blind to feature correlations*: if two features are strongly correlated within a class (an elongated, tilted cluster), NB models it as an axis-aligned blob and can misclassify near the boundary.

**How LDA fixes that:** LDA models each class as a **multivariate Gaussian with a full covariance matrix** $\Sigma$ that captures correlations — but it makes one simplifying assumption: **all classes share the same $\Sigma$** (homoscedasticity). That single shared covariance causes the quadratic terms in the Bayes rule to **cancel**, leaving a **linear** decision boundary. So LDA = "Naive Bayes that *can* see correlations, at the cost of assuming every class has the same covariance shape." (If you also drop the *shared* assumption, you get QDA with quadratic boundaries — §8.)

---

## 2. Prerequisites

- **Naive Bayes / Bayes' theorem** (Topic 7) — LDA is the same generative idea with a richer covariance.
- **Multivariate Gaussian** $\mathcal{N}(\mu,\Sigma)$, its mean vector and covariance matrix.
- **Matrix algebra:** matrix–vector products, transpose, **matrix inverse**, **symmetric matrices**, **eigenvalues/eigenvectors**, **rank**.
- **Quadratic forms** $x^\top A x$.
- **Mahalanobis distance** $D^2=(x-\mu)^\top\Sigma^{-1}(x-\mu)$.
- **Variance/covariance** and the idea of within-group vs between-group spread.

---

## 3. Core idea — intuition first

Two overlapping clusters of points. You want **one line** to separate them, and a way to **score** any new point as "more like cluster 1 or cluster 2."

A naive idea: project everything onto the line joining the two class means and split at the midpoint. But that ignores **how spread out** each class is and **in which direction**. Fisher's insight: pick the projection direction that **pushes the class means far apart** while **keeping each class tight** — i.e. maximize *between-class separation* relative to *within-class scatter*:

$$\text{maximize}\quad \frac{(\text{distance between projected means})^2}{\text{within-class spread of projections}}$$

The generative twin says the same thing probabilistically: assume both classes are Gaussian bells of the **same shape** $\Sigma$, just centered at different means; then the optimal boundary is the set of points equally "Mahalanobis-close" to both means — and because the bells have the same shape, that boundary is a **straight line**.

> Headline intuition: **LDA separates by trading off "means far apart" against "classes not too spread," and (with shared covariance) the result is linear.**

---

## 4. Formal treatment

### 4.1 The generative model & assumptions

$$P(\mathbf{x}\mid y=k)=\mathcal{N}(\mathbf{x};\,\mu_k,\,\Sigma),\qquad P(y=k)=\pi_k$$

> **Assumptions (test the edges!):**
> 1. Each class-conditional is **Gaussian**.
> 2. **All classes share the same covariance** $\Sigma$ (homoscedastic) — *this is what makes LDA "linear."*
> 3. (Full $\Sigma$ allowed — correlations are modeled, unlike Naive Bayes.)

### 4.2 Linear discriminant function (multiclass)

Apply Bayes, take logs, and drop terms common to all classes (the $-\tfrac12\mathbf{x}^\top\Sigma^{-1}\mathbf{x}$ term cancels because $\Sigma$ is shared):

$$\boxed{\;\delta_k(\mathbf{x}) = \mathbf{x}^\top\Sigma^{-1}\mu_k \;-\; \tfrac12\,\mu_k^\top\Sigma^{-1}\mu_k \;+\; \log\pi_k\;}\qquad \hat{y}=\arg\max_k\,\delta_k(\mathbf{x})$$

$\delta_k$ is **linear in $\mathbf{x}$**, so every pairwise boundary $\{\delta_k=\delta_l\}$ is a **hyperplane**.

### 4.3 Two-class decision rule

Assign $\mathbf{x}$ to **class 1** iff $\delta_1(\mathbf{x})>\delta_2(\mathbf{x})$, i.e.

$$\boxed{\;\mathbf{x}^\top\Sigma^{-1}(\mu_1-\mu_2)\;>\;\tfrac12(\mu_1+\mu_2)^\top\Sigma^{-1}(\mu_1-\mu_2)\;-\;\log\frac{\pi_1}{\pi_2}\;}$$

- The **normal** to the boundary is $\Sigma^{-1}(\mu_1-\mu_2)$ (note: *not* $\mu_1-\mu_2$ unless $\Sigma\propto I$).
- **Priors only shift** the boundary (the $\log(\pi_1/\pi_2)$ term) — they do **not** rotate it.

### 4.4 Mahalanobis nearest-mean view (equal priors)

With $\pi_1=\pi_2$, the rule reduces to **assign $\mathbf{x}$ to the class with the smaller Mahalanobis distance to its mean**:

$$D_k^2(\mathbf{x})=(\mathbf{x}-\mu_k)^\top\Sigma^{-1}(\mathbf{x}-\mu_k),\qquad \hat{y}=\arg\min_k D_k^2$$

> Only when $\Sigma= \sigma^2 I$ does this become ordinary **Euclidean** nearest-mean. Otherwise the covariance "rescales and rotates" distances — this is a favourite GATE trap (§5 Problem 2).

### 4.5 Fisher's LDA — the projection / dimensionality-reduction view

Find the direction $\mathbf{w}$ maximizing the **Fisher criterion**:

$$J(\mathbf{w})=\frac{\mathbf{w}^\top S_B\,\mathbf{w}}{\mathbf{w}^\top S_W\,\mathbf{w}}$$

**Scatter matrices:**

$$S_W=\sum_{k}\sum_{i\in C_k}(\mathbf{x}_i-\mu_k)(\mathbf{x}_i-\mu_k)^\top \quad\text{(within-class)}$$
$$S_B=\sum_{k}N_k(\mu_k-\mu)(\mu_k-\mu)^\top \quad\text{(between-class; for 2 classes } S_B\propto(\mu_1-\mu_2)(\mu_1-\mu_2)^\top)$$

where $\mu$ is the overall mean. Maximizing $J$ gives the **generalized eigenvalue problem**:

$$S_B\mathbf{w}=\lambda S_W\mathbf{w}\;\Longleftrightarrow\; S_W^{-1}S_B\,\mathbf{w}=\lambda\mathbf{w}$$

**Two-class closed form (memorize):**

$$\boxed{\;\mathbf{w}\;\propto\;S_W^{-1}(\mu_1-\mu_2)\;}$$

> $S_W$ is the *pooled* (unnormalized) covariance: $\widehat{\Sigma}=S_W/(N-c)$, so the Fisher direction $S_W^{-1}(\mu_1-\mu_2)$ is the **same direction** as $\Sigma^{-1}(\mu_1-\mu_2)$ — the projection view and the generative boundary normal coincide.

**How many directions?** $S_B$ has **rank $\le c-1$** (it is built from $c$ mean-deviations summing to zero). Hence:

$$\boxed{\;\text{LDA yields at most } c-1 \text{ discriminant directions.}\;}$$

(2 classes → at most 1 direction; a 5-class problem → at most 4.)

### 4.6 LDA vs QDA (drop the shared-covariance assumption)

If each class has its **own** covariance $\Sigma_k$, the quadratic term no longer cancels:

$$\delta_k(\mathbf{x})=-\tfrac12\log|\Sigma_k|-\tfrac12(\mathbf{x}-\mu_k)^\top\Sigma_k^{-1}(\mathbf{x}-\mu_k)+\log\pi_k$$

This is **QDA**, with a **quadratic** boundary. So: **shared $\Sigma$ → LDA (linear); per-class $\Sigma_k$ → QDA (quadratic)**; **diagonal $\Sigma$ → Gaussian Naive Bayes.**

*(Derivations beyond writing down $\delta_k$ are not asked by GATE; the testable facts are the boundary linearity, the cancellation, and the closed-form direction.)*

---

## 5. Worked problems (GATE-style)

### Problem 1 — Fisher's optimal projection direction (NAT, linear algebra)

Two classes in 2-D:

- **Class 1:** $(3,4),\,(1,0)$
- **Class 2:** $(7,2),\,(5,6)$

Find the LDA (Fisher) projection direction $\mathbf{w}$.

**Step 1 — Class means.**
$$\mu_1=\Big(\tfrac{3+1}{2},\tfrac{4+0}{2}\Big)=(2,2),\qquad \mu_2=\Big(\tfrac{7+5}{2},\tfrac{2+6}{2}\Big)=(6,4)$$

**Step 2 — Within-class scatter $S_W$.**
Class 1 deviations: $(1,2),(-1,-2)$ →
$$S_1=\begin{bmatrix}1\\2\end{bmatrix}[1\,\,2]+\begin{bmatrix}-1\\-2\end{bmatrix}[-1\,\,-2]=\begin{bmatrix}2&4\\4&8\end{bmatrix}$$
Class 2 deviations: $(1,-2),(-1,2)$ →
$$S_2=\begin{bmatrix}2&-4\\-4&8\end{bmatrix}\;\Rightarrow\; S_W=S_1+S_2=\begin{bmatrix}4&0\\0&16\end{bmatrix}$$

**Step 3 — Invert $S_W$ and apply the formula.**
$$S_W^{-1}=\begin{bmatrix}\tfrac14&0\\0&\tfrac1{16}\end{bmatrix},\qquad \mu_1-\mu_2=(-4,-2)$$
$$\mathbf{w}\propto S_W^{-1}(\mu_1-\mu_2)=\Big(-\tfrac{4}{4},\,-\tfrac{2}{16}\Big)=(-1,\,-0.125)\;\propto\;\boxed{(8,\,1)}$$

**Interpretation (the point of the problem):** the *naive* mean-difference direction is $(\mu_1-\mu_2)\propto(2,1)$, but the true LDA direction is $(8,1)$ — far more aligned with the **x-axis**. Why? The within-class spread along $y$ is huge (variance 16) and along $x$ is small (variance 4), so LDA prefers to separate along the **low-variance** axis where the classes are crisp. **LDA $\neq$ "difference of means" whenever the within-class spread is not isotropic.**

---

### Problem 2 — LDA classification with non-identity covariance (MCQ — the Mahalanobis trap)

Shared covariance $\Sigma=\begin{bmatrix}9&0\\0&1\end{bmatrix}$, means $\mu_1=(0,0)$, $\mu_2=(6,2)$, **equal priors**. Classify the query $\mathbf{x}=(2,\,1.8)$.

**Naive (wrong) approach — Euclidean nearest mean:**
$$\|x-\mu_1\|^2=2^2+1.8^2=7.24,\qquad \|x-\mu_2\|^2=(-4)^2+(-0.2)^2=16.04$$
Euclidean would pick **class 1**. **But $\Sigma\neq I$, so this is invalid.**

**Correct — Mahalanobis distance** ($\Sigma^{-1}=\text{diag}(\tfrac19,1)$):
$$D_1^2=\frac{2^2}{9}+\frac{1.8^2}{1}=0.444+3.24=3.684$$
$$D_2^2=\frac{(-4)^2}{9}+\frac{(-0.2)^2}{1}=1.778+0.04=1.818$$
$D_2^2<D_1^2$ → **class 2**.

*(Check via the discriminant function, equal priors: $\delta_1=0$; $\delta_2=\mathbf{x}^\top\Sigma^{-1}\mu_2-\tfrac12\mu_2^\top\Sigma^{-1}\mu_2 = (2)(\tfrac{6}{9})+(1.8)(2)-\tfrac12[(6)(\tfrac69)+(2)(2)] = 4.933-4 = 0.933>0$ → class 2. ✓)*

**Answer: class 2.** Euclidean said class 1, LDA says class 2.

> **Why the flip:** along $y$ (variance 1, *trustworthy*) the query is almost on $\mu_2$ ($1.8$ vs $2$) and far from $\mu_1$. Along $x$ (variance 9, *noisy*) it sits nearer $\mu_1$. LDA **discounts the noisy x-axis** and trusts the crisp y-axis → class 2. Euclidean weighted both equally and got it wrong.

**(b) Effect of priors.** Class 1 wins instead iff $\delta_1>\delta_2$, i.e. $\log(\pi_1/\pi_2)>0.933$, i.e. prior odds $\pi_1/\pi_2>e^{0.933}\approx 2.54$. So if class 1 is more than ~2.5× as likely a priori, the decision flips back — **priors shift the boundary**.

---

### Problem 3 — Conceptual properties of LDA (MSQ)

Which of the following are **TRUE**?

- **(A)** LDA is an unsupervised method.
- **(B)** For a $c$-class problem, LDA produces at most $c-1$ discriminant directions.
- **(C)** LDA assumes all classes share the same covariance matrix.
- **(D)** If each class has its own covariance matrix, the Bayes-optimal boundary becomes quadratic (QDA).
- **(E)** LDA maximizes the total variance of the projected data, ignoring class labels.

**Solution.**
- **(A) False** — LDA *uses the labels* (it's supervised). Maximizing variance while ignoring labels is **PCA**.
- **(B) True** — $S_B$ has rank $\le c-1$, so at most $c-1$ useful directions.
- **(C) True** — the shared (homoscedastic) covariance is LDA's defining assumption and the reason its boundary is linear.
- **(D) True** — per-class covariances reinstate the quadratic term → QDA.
- **(E) False** — that's **PCA**, the unsupervised cousin.

**Answer: B, C, D.**

---

## 6. Common GATE question types on this topic

- **Compute the Fisher direction** $\mathbf{w}\propto S_W^{-1}(\mu_1-\mu_2)$ from a small dataset (Problem 1).
- **Classify a point** via the discriminant function / Mahalanobis distance, especially when $\Sigma\neq I$ (Problem 2).
- **"Maximum number of discriminant directions"** $=c-1$.
- **LDA vs PCA:** supervised (class separation) vs unsupervised (variance).
- **LDA vs QDA:** shared vs per-class covariance; linear vs quadratic boundary.
- **Effect of priors** (shift the boundary, don't rotate it); when does the decision flip.
- **Why the quadratic term cancels** (shared covariance) → linearity.
- **Scatter-matrix definitions / ranks**, generalized eigenproblem $S_W^{-1}S_B\mathbf{w}=\lambda\mathbf{w}$.

---

## 7. Traps, pitfalls & common mistakes

- **LDA ≠ PCA.** LDA is supervised and maximizes *class separability*; PCA is unsupervised and maximizes *variance*. Easy to swap under exam pressure.
- **Thinking LDA's boundary is quadratic.** It is **linear** (the quadratic term cancels under shared $\Sigma$). **QDA** is the quadratic one.
- **Using Euclidean nearest-mean when $\Sigma\neq I$.** You must use **Mahalanobis** distance; ignoring $\Sigma$ flips answers (Problem 2).
- **Forgetting at most $c-1$ directions.** You cannot project a 2-class problem onto more than **1** LDA direction.
- **Forgetting the prior term.** The $-\log(\pi_1/\pi_2)$ shifts the boundary; with imbalanced priors the midpoint rule is wrong.
- **$\mathbf{w}\propto(\mu_1-\mu_2)$ is wrong in general.** The correct direction is $S_W^{-1}(\mu_1-\mu_2)$; they coincide only when $S_W\propto I$.
- **Maximizing the ratio upside-down.** Maximize $\dfrac{w^\top S_B w}{w^\top S_W w}$ (between **over** within), not the reciprocal.
- **$S_W$ singular** when $N\le d$ or features are collinear — then $S_W^{-1}$ doesn't exist (small-sample problem; needs regularization/PCA first).
- **Sign/scale of $\mathbf{w}$ is arbitrary** — only the *direction* matters; $(8,1)$ and $(-8,-1)$ are the same projection.
- **Confusing $S_B$ and $S_W$**, or forgetting $S_B$ (two-class) is **rank 1**.

---

## 8. Connection forward

LDA finds a linear boundary by **modeling the class distributions** (means + a shared covariance) — it is *generative* and lets **every training point** (through the means and covariance) influence the boundary. This is powerful **when the Gaussian, equal-covariance assumption holds**, but fragile when it doesn't: non-Gaussian or unequal-covariance classes, outliers shifting the means/covariance, etc.

The next topic, **Support Vector Machine (SVM)**, attacks the same "find a linear separator" goal from the **opposite philosophy**: it makes **no distributional assumption** and instead chooses the **maximum-margin** hyperplane, caring only about the few **boundary points (support vectors)** and ignoring easy, far-away points entirely. So the move is **LDA → SVM = from "model the classes, then derive a boundary" to "directly optimize the widest-margin separating boundary, distribution-free"** — and SVM later adds **kernels** to bend that boundary non-linearly, something LDA's linearity cannot do.

*(Sideways link: LDA's linear discriminant for two equal-covariance Gaussians has the same form as **logistic regression**'s boundary and the **Gaussian Naive Bayes** boundary — three views of the linear-Gaussian story.)*

---

## 9. One-line cheat-sheet summary

**LDA:** Gaussian classes with a **shared full covariance** → **linear** discriminant $\delta_k(\mathbf{x})=\mathbf{x}^\top\Sigma^{-1}\mu_k-\tfrac12\mu_k^\top\Sigma^{-1}\mu_k+\log\pi_k$; equal priors ⇒ **nearest mean in Mahalanobis distance** (Euclidean only if $\Sigma\propto I$); Fisher projection $\mathbf{w}\propto S_W^{-1}(\mu_1-\mu_2)$ maximizing $\frac{w^\top S_B w}{w^\top S_W w}$, giving **at most $c-1$** directions; **supervised** (vs PCA's unsupervised), **linear** (vs QDA's quadratic per-class covariance); priors **shift** the boundary.

---

### ➡️ Next topic
**Topic 9 — Support Vector Machine (SVM)** (closing the Classification Methods cluster). Ask for it when ready.

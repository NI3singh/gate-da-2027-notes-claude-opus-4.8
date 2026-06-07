# Topic 18 — Dimensionality Reduction (Overview)

**Subject:** Machine Learning · **Section:** Unsupervised Learning

> **Scope:** This is a *framing / overview* topic. Its job is to explain **why** we shrink the number of features, set up the **two families** (feature selection vs feature extraction) and the **two axes** (linear vs nonlinear, supervised vs unsupervised), and hand off cleanly to **PCA (Topic 19)**, the headline method. The one genuinely computational thing GATE asks at the overview level — the **curse of dimensionality** numbers — gets full treatment; the heavy eigen-math is deferred to PCA.

---

## 0. Topic header & syllabus map

- **Topic:** Dimensionality Reduction (DR) — reducing the number of input features from $d$ to $k\ll d$ while preserving as much useful structure as possible.
- **Where it sits:** Comes **after** the clustering block (k-means, k-medoid, hierarchical), all of which depended on **distances** that misbehave in high dimensions. It also **echoes LDA (Topic 8)** — which, you'll see, was secretly a *supervised* dimensionality-reduction method met back in the supervised section. It **leads directly into PCA (Topic 19)**, the unsupervised linear DR workhorse, which closes the syllabus.
- **One line:** *Squeeze many correlated, noisy features into a few informative ones, so models compute faster, generalize better, and distances mean something again.*

---

## 1. Why this topic exists (motivation + connection)

Every method so far quietly assumed that the feature space was "nice": k-NN, k-means, and hierarchical clustering all rank points by **distance**, and regression/classification all fit parameters **per feature**. Two problems break this when $d$ (the number of features) is large:

1. **Distances stop being meaningful (the curse of dimensionality).** In high dimensions, *all* pairwise distances drift toward being equal — "nearest" and "farthest" neighbours become almost indistinguishable. That **directly undermines the previous three topics**: single-/complete-linkage, k-means' spheres, and k-NN all rely on a meaningful "closest," and high $d$ erodes it.
2. **Too many features hurt models and humans.** More features → more parameters → **more overfitting and more data needed**; redundant/correlated features cause **multicollinearity** (recall ridge regression, Topic 4); and you simply **cannot visualize** beyond 3-D.

**The bridge (clustering → DR).** Clustering tried to find structure *in* the feature space. Dimensionality reduction fixes the **feature space itself** first — projecting the data into a few well-chosen dimensions where distances are meaningful, noise is reduced, and computation is cheap. It is therefore a standard **preprocessing step before clustering or classification**, not a competitor to them.

**The bridge to what's next.** Once we agree we want "a few informative new axes that capture the variation and let us drop the rest," the obvious question is *which* axes. **PCA (Topic 19)** answers it: the orthogonal directions of **maximum variance**.

---

## 2. Prerequisites

- **Feature vector / feature space** — a sample is $\mathbf{x}\in\mathbb{R}^d$.
- **Distance & the curse callback** (Topic 15) — why distance-based methods need low, well-behaved $d$.
- **Variance and covariance** — DR is largely about where the data **varies** and which features are **redundant** (correlated).
- **Linear combinations of features** — feature extraction builds new features as weighted sums of old ones.
- **LDA (Topic 8)** — the supervised contrast to PCA.
- (Light, for the forward link) **eigenvalues/eigenvectors** — the engine of PCA.

---

## 3. Core idea — intuition first

**Analogy 1 — the shadow.** A 3-D object casts a 2-D shadow. A *bad* angle gives a useless blob; a *good* angle gives a shadow that still tells you what the object is. Dimensionality reduction is choosing the **best angle to flatten** high-dimensional data onto a few dimensions so its structure survives.

**Analogy 2 — redundant columns.** Suppose your dataset has both "height in cm" and "height in inches." That's **two columns but one real dimension** — perfectly correlated, so one is redundant. Real data is full of such near-duplications and correlations; DR **strips the redundancy**, keeping the few directions that genuinely carry information.

**The two ways to reduce $d$ (lock this in now):**

- **Feature selection** — *keep a subset of the original features, throw the rest away.* You still have real, interpretable features ("we kept age, income, tenure"). 
- **Feature extraction (transformation)** — *build $k$ brand-new features as combinations of the original $d$.* PCA's "principal components" are like this: each new axis is a weighted mix of the originals. More compact, but you lose the plain meaning of individual features.

**What "preserve structure" means.** Different methods preserve different things: keep the most **variance** (PCA), keep **class separability** (LDA), keep **pairwise distances / neighbourhoods** (t-SNE, MDS), or keep the ability to **reconstruct** the original with least error (PCA again, autoencoders).

---

## 4. Formal treatment

### 4.1 The problem statement

Given $n$ points $\mathbf{x}_i\in\mathbb{R}^d$, find a map $f:\mathbb{R}^d\to\mathbb{R}^k$ with $k\ll d$ producing $\mathbf{z}_i=f(\mathbf{x}_i)\in\mathbb{R}^k$ that retains the information needed for the downstream task (clustering, classification, visualization). For **linear** DR, $f$ is a projection: $\mathbf{z}=W^\top\mathbf{x}$ for some $d\times k$ matrix $W$.

### 4.2 The curse of dimensionality (the testable core)

Several distinct effects, all worsening as $d$ grows:

**(a) Volume / sparsity — neighbourhoods aren't "local."**
To capture a fraction $r$ of the volume of the unit hypercube $[0,1]^d$, a sub-cube must have edge length

$$
\boxed{\,e = r^{1/d}\,}\qquad(\text{since a sub-cube of edge }e\text{ has volume }e^d).
$$

As $d$ grows, $e\to 1$ even for tiny $r$: to grab just $10\%$ of the volume you must span almost the entire range of every axis (worked in Problem 1). So "local" neighbourhoods stop being local, and density-based intuition fails.

**(b) Distance concentration — "nearest" loses meaning.**
Under broad conditions, as $d\to\infty$,

$$
\frac{D_{\max}-D_{\min}}{D_{\min}}\;\longrightarrow\;0,
$$

i.e. the **nearest and farthest points become nearly equidistant**. This is the formal reason k-NN, k-means, and hierarchical linkage degrade in high $d$.

**(c) Empty space / corners.**
The volume of the unit hypersphere inscribed in the cube $[-1,1]^d$ shrinks to **zero** relative to the cube as $d\to\infty$ — almost all of a high-dimensional cube's volume sits in its **corners**, far from the centre. (Relatedly, most of a high-dimensional Gaussian's mass lies in a **thin shell**, not near the mean.)

**(d) Sample complexity / overfitting.**
The data needed to densely sample the space grows **exponentially** in $d$. With fixed $n$, more features mean more parameters, sparser coverage, and **higher variance / overfitting** (ties back to the bias–variance trade-off, Topic 11).

### 4.3 Two families of methods

**Feature selection** — keep a subset $S\subseteq\{1,\dots,d\}$ of original features. Three method types:

| Type | How it scores features | Cost | Example |
|---|---|---|---|
| **Filter** | a statistic independent of any model (correlation/MI with target, variance threshold, $\chi^2$) | cheap, model-agnostic | drop low-variance or low-correlation features |
| **Wrapper** | a model's validation performance on each candidate subset | expensive, model-specific | forward selection, backward elimination, RFE |
| **Embedded** | selection happens *during* model training | moderate | **LASSO/L1** zeroing coefficients; tree feature importance |

> Selection **keeps interpretability** (the surviving columns are real features) and is the right choice when you must explain *which* variables matter.

**Feature extraction (transformation)** — construct $k$ new features as functions of the original $d$:

- **Linear:** **PCA** (max variance, unsupervised), **LDA** (max class separability, supervised), factor analysis.
- **Nonlinear / manifold:** kernel PCA, **t-SNE**, UMAP, Isomap, LLE, **autoencoders** *(recognition level — GATE DA centres on PCA and LDA).*

> Extraction can pack more information into fewer dimensions, but the new axes are **mixtures**, so individual-feature interpretability is lost.

### 4.4 Two orthogonal axes that classify any DR method

- **Linear vs nonlinear** — does $f$ project onto a flat subspace (PCA, LDA) or bend to follow a curved manifold (t-SNE, autoencoders)?
- **Unsupervised vs supervised** — does it ignore labels and preserve generic structure (**PCA**: maximize variance using only $\mathbf{x}$), or use labels to preserve class structure (**LDA**: maximize between-class vs within-class scatter using $y$)?

$$
\boxed{\textbf{PCA: unsupervised, maximize variance.}\qquad \textbf{LDA: supervised, maximize class separation.}}
$$

This PCA-vs-LDA contrast is a perennial GATE one-liner — don't confuse them.

### 4.5 Benefits and costs (why/when to do it)

- **Benefits:** less computation & storage; less overfitting / better generalization; removes multicollinearity & noise; enables 2-D/3-D **visualization**; restores meaningful distances for clustering.
- **Costs / risks:** you **lose information** (some variance/structure is discarded); extracted features are **hard to interpret**; if done blindly it can **discard the very signal** a task needs (e.g. PCA keeps high-variance directions, which are not always the class-discriminative ones — that's exactly why LDA exists).

*Derivations skipped here on purpose:* the PCA optimization (eigen-decomposition of the covariance matrix, variance maximization, reconstruction-error minimization, choosing $k$ by explained variance) is the entire content of Topic 19, so it is only previewed here.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Curse of dimensionality: how "local" is a neighbourhood? (NAT)

In the unit hypercube $[0,1]^d$, what edge length $e$ must a sub-cube have to contain $10\%$ of the total volume, for $d=1,\,2,\,10,\,100$?

**Solution.** $e=r^{1/d}$ with $r=0.1$:
- $d=1:\ e=0.1$ (10% of the line).
- $d=2:\ e=0.1^{1/2}\approx 0.316$.
- $d=10:\ e=0.1^{1/10}=10^{-0.1}\approx \mathbf{0.794}$.
- $d=100:\ e=0.1^{1/100}=10^{-0.01}\approx \mathbf{0.977}$.

So in 100-D, capturing a mere $10\%$ of the volume requires spanning $\approx 97.7\%$ of **every** axis. "Nearby" points are not nearby at all — the geometric reason distance-based methods need low, reduced $d$.

---

### Problem 2 — Distance concentration (MCQ)

As the dimensionality $d\to\infty$ for data drawn from a fixed distribution, the contrast $\dfrac{D_{\max}-D_{\min}}{D_{\min}}$ between the farthest and nearest point distances tends to:

(A) $\infty$ &nbsp; (B) a positive constant &nbsp; (C) $0$ &nbsp; (D) it oscillates

**Solution.** **(C) $0$.** Distances **concentrate**: nearest and farthest neighbours become almost equidistant, so "nearest neighbour" loses discriminative power. This is precisely why k-NN (Topic 6) and the distance-based clustering methods (Topics 16–17) degrade in high dimensions and benefit from prior dimensionality reduction.

---

### Problem 3 — Classify the method (MSQ)

For each technique, state (i) selection vs extraction, (ii) supervised vs unsupervised, (iii) linear vs nonlinear.

| Method | Selection / Extraction | Supervised? | Linear? |
|---|---|---|---|
| **PCA** | Extraction | Unsupervised | Linear |
| **LDA** | Extraction | **Supervised** | Linear |
| **LASSO (L1)** | **Selection** (embedded) | Supervised | (linear model) |
| **Forward selection** | Selection (wrapper) | Supervised | — |
| **Correlation/variance threshold** | Selection (filter) | Unsupervised | — |
| **t-SNE** | Extraction | Unsupervised | **Nonlinear** |

*Typical question:* "Which of these are feature-**extraction** methods?" → **PCA, LDA, t-SNE.** "Which use labels?" → **LDA, LASSO, forward selection.**

---

### Problem 4 — True/false on DR (MSQ trap)

(A) Dimensionality reduction can reduce overfitting.
(B) Feature extraction preserves the interpretability of the original features.
(C) PCA requires class labels.
(D) Reducing dimensions always improves downstream accuracy.

**Solution.**
- (A) **True** — fewer features → fewer parameters → lower variance / less overfitting.
- (B) **False** — extracted features are *mixtures* of the originals; interpretability is lost (selection preserves it, extraction does not).
- (C) **False** — PCA is **unsupervised** (uses only $\mathbf{x}$). The label-using one is **LDA**.
- (D) **False** — DR discards information; if it drops task-relevant signal (e.g. PCA discarding a low-variance but class-discriminative direction), accuracy can **drop**.

**Answer: (A) only is true.** *(C) and PCA-vs-LDA is the classic trap.*

---

## 6. Common GATE question types on this topic

- **Curse-of-dimensionality computation:** edge length for a volume fraction ($e=r^{1/d}$), or qualitative distance-concentration reasoning.
- **Selection vs extraction** classification of a named method; **filter / wrapper / embedded** identification.
- **Supervised vs unsupervised / linear vs nonlinear** tagging (especially **PCA vs LDA**).
- **Why/when DR helps:** overfitting, multicollinearity, visualization, restoring meaningful distances.
- **True/false on properties:** information loss, interpretability, label requirement, no guaranteed accuracy gain.
- Conceptual lead-ins to PCA ("which directions does PCA keep?" → maximum variance).

---

## 7. Traps, pitfalls & common mistakes

- **PCA ↔ LDA confusion.** PCA = **unsupervised, max variance**; LDA = **supervised, max class separation**. The highest-variance direction is **not** always the most class-discriminative — this is the single most-tested distinction.
- **"DR always helps accuracy."** No — it **loses information**; only keep dimensions that matter for the task.
- **Confusing selection with extraction.** Selection keeps *real* features (interpretable); extraction builds *new mixed* features (not interpretable). LASSO selects; PCA extracts.
- **Forgetting to standardize before PCA/distance methods.** Features on larger scales dominate variance and distances; scale first (full discussion in Topic 19).
- **Thinking high variance = high information for a label.** Variance is task-agnostic; noise can have high variance. (Why LDA exists.)
- **Believing more features are always better.** Beyond a point, added features hurt (curse of dimensionality, overfitting) — the "peaking phenomenon."
- **Edge-length formula slip.** It's $e=r^{1/d}$ (the $d$-th root of the *fraction*), not $r/d$ or $r^d$.
- **Assuming DR removes only "useless" features.** Even good DR throws away *some* genuine structure; the goal is to lose the least that matters.

---

## 8. Connection forward

We now have the goal precisely framed: find **a few new axes** that (i) capture the most **variation**, (ii) are **uncorrelated** (so they don't repeat each other), and (iii) let us **reconstruct** the data with least error after dropping the rest. **Principal Component Analysis (Topic 19)** is the concrete answer:

- It computes the **covariance matrix** of the (standardized) data and takes its **eigenvectors** as the new axes — the **principal components**.
- The eigenvector with the **largest eigenvalue** is the direction of **maximum variance**; components are **orthogonal** (hence uncorrelated); and keeping the top $k$ **minimizes reconstruction error** for that $k$.
- You choose $k$ via the **explained-variance ratio** (e.g. "smallest $k$ retaining 95% of total variance").

So the syllabus closes: *measure similarity (15) → cluster flatly (16) → cluster hierarchically (17) → understand why high dimensions break all of this and how to fix the space (18) → do it concretely and optimally with PCA (19).*

---

## 9. One-line cheat-sheet summary

> **Dimensionality reduction** maps $\mathbb{R}^d\to\mathbb{R}^k$ ($k\ll d$) to beat the **curse of dimensionality** (volume sparsity $e=r^{1/d}\to1$, distance concentration $\tfrac{D_{\max}-D_{\min}}{D_{\min}}\to0$), cut overfitting/computation, and enable visualization. Two families: **feature selection** (keep real features — filter / wrapper / embedded, interpretable) vs **feature extraction** (new combined features — **PCA: unsupervised, max variance** vs **LDA: supervised, max class separation**; linear vs nonlinear like t-SNE). It **loses information**, so it never *guarantees* better accuracy.

---

### ➡️ Next topic (in order): **Topic 19 — Principal Component Analysis (PCA)** — covariance matrix, eigenvalues/eigenvectors, principal components as max-variance directions, explained-variance ratio for choosing $k$, the reconstruction-error view, and fully worked numerical PCA. Ask for it when ready.

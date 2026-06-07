# Topic 19 — Principal Component Analysis (PCA)

**Subject:** Machine Learning · **Section:** Unsupervised Learning *(this topic completes the section)*

> **Scope:** Full computational topic — the headline dimensionality-reduction method, and one of the most frequently and *numerically* tested items in GATE DA. You must be able to: build the **covariance matrix**, find **eigenvalues/eigenvectors** by hand (2×2), read off the **principal components** and **variance explained**, **project** points, choose **$k$** by explained variance, and compute **reconstruction error**. All of it is worked below.

---

## 0. Topic header & syllabus map

- **Topic:** Principal Component Analysis — an **unsupervised, linear feature-extraction** method that re-expresses data on new orthogonal axes (principal components) ordered by the variance they capture, so you can keep the top few and drop the rest.
- **Where it sits:** The concrete realization of the goals framed in Dimensionality Reduction (Topic 18). It is the **unsupervised counterpart of LDA** (Topic 8, supervised). It is the **final topic** of the Unsupervised Learning section and of this ML sequence.
- **One line:** *Rotate the axes to point along the directions of greatest spread (the eigenvectors of the covariance matrix), keep the top few, throw the rest away.*

---

## 1. Why this topic exists (motivation + connection)

The overview (Topic 18) ended with an exact wish-list for new axes: capture the most **variance**, be mutually **uncorrelated** (so they don't repeat information), and allow **reconstruction** of the data with the least error after dropping the rest. It did **not** say *how* to find such axes — that was deliberately left as the cliff-hanger.

**PCA is the answer, and it is provably optimal for that wish-list.** It computes the **covariance matrix** of the (centered, optionally standardized) data and takes its **eigenvectors** as the new axes. Three facts make it exactly fit the wish-list:
- The eigenvector with the **largest eigenvalue** is the direction of **maximum variance** (derived in §4.3).
- Eigenvectors of a symmetric matrix are **orthogonal**, so the new features are **uncorrelated**.
- Keeping the top $k$ eigenvectors **minimizes the mean squared reconstruction error** among all rank-$k$ linear projections.

**The bridge (DR overview → PCA):** "find max-variance, uncorrelated, least-error axes" $\Rightarrow$ "take the eigenvectors of the covariance matrix, ordered by eigenvalue." That single sentence is PCA.

**Connection back across the syllabus.** PCA contrasts with **LDA** (variance vs class separation), relieves the **multicollinearity** that motivated **ridge regression** (Topic 4), restores the meaningful distances that **clustering** (Topics 16–17) needs, and is the standard preprocessing step *before* those methods.

---

## 2. Prerequisites

- **Mean, variance, covariance** and the **covariance matrix** (symmetric $d\times d$).
- **Eigenvalues & eigenvectors** — solving $\det(S-\lambda I)=0$ and $(S-\lambda I)\mathbf{v}=\mathbf 0$ for $2\times2$ matrices.
- **Symmetric matrices / spectral theorem** — real eigenvalues, orthogonal eigenvectors.
- **Vector projection & unit vectors** — projecting $\mathbf{x}$ onto a unit direction $\mathbf{w}$ gives the scalar $\mathbf{w}^\top\mathbf{x}$.
- **Lagrange multipliers** (light) — for the variance-maximization derivation.
- **Dimensionality-reduction framing & PCA-vs-LDA** (Topic 18).

---

## 3. Core idea — intuition first

**Analogy — the best camera angle.** Imagine a flat, elongated swarm of points (like a tilted ellipse of data) floating in space. You want to photograph it from the angle that reveals the **most spread** — flatten it so the long axis of the ellipse runs left–right and the short axis is squashed. The **long axis is the first principal component** (most variance), the perpendicular **short axis is the second** (least variance). If the short axis carries almost no spread, you can drop it and lose almost nothing.

**Why variance = information (in PCA's worldview).** A direction along which the data barely varies tells you little — every point looks the same there. A direction with large variance separates the points. PCA therefore **keeps high-variance directions and discards low-variance ones**. (Caveat — variance is *task-agnostic*; the high-variance direction need not be the one that separates *classes*. That's why LDA exists. PCA never looks at labels.)

**Two equivalent ways to see PCA (both testable):**
1. **Maximize variance:** find the orthogonal directions along which the projected data spreads most.
2. **Minimize reconstruction error:** find the $k$-dimensional subspace onto which projecting, then re-expanding, distorts the data least.
These give the **same** answer — the top-$k$ eigenvectors of the covariance matrix.

**The new features are uncorrelated.** Because the principal axes are orthogonal and aligned with the spread, the data's covariance in the new coordinates is **diagonal** — the redundancy between correlated original features is removed.

---

## 4. Formal treatment

### 4.1 The PCA algorithm

```
Input: data matrix X (n rows = samples, d columns = features).
1. CENTER: subtract the column mean from each feature  →  X̃  (mean 0).
   (Optionally STANDARDIZE: also divide each column by its std-dev
    ⇒ this makes PCA use the CORRELATION matrix — see §4.6.)
2. COVARIANCE: S = (1/n) X̃ᵀ X̃      (a d×d symmetric, PSD matrix)
   (some texts use 1/(n−1); it only rescales eigenvalues, not the directions
    or the variance proportions.)
3. EIGEN-DECOMPOSE S: get eigenvalues λ₁ ≥ λ₂ ≥ … ≥ λ_d ≥ 0
   and orthonormal eigenvectors v₁, v₂, …, v_d.
4. The principal components are v₁ (1st PC, max variance), v₂, …
5. CHOOSE k via the explained-variance ratio (§4.5).
6. PROJECT: W = [v₁ … v_k] (d×k);  Z = X̃ W   (n×k scores = reduced data).
7. (Optional) RECONSTRUCT: X̂ = Z Wᵀ  (back in original d-space).
```

### 4.2 Key formulas (with conditions)

$$
\boxed{
\begin{aligned}
\textbf{Covariance matrix:}\quad & S=\frac1n \tilde X^\top \tilde X,\quad \tilde X=\text{centered data}\\[3pt]
\textbf{Eigen-equation (PCs):}\quad & S\mathbf{v}_i=\lambda_i\mathbf{v}_i,\quad \|\mathbf v_i\|=1,\ \mathbf v_i\perp\mathbf v_j\\[3pt]
\textbf{Variance along }\mathbf v_i:\quad & \mathbf v_i^\top S\,\mathbf v_i=\lambda_i\\[3pt]
\textbf{Total variance:}\quad & \operatorname{trace}(S)=\textstyle\sum_{i=1}^d \lambda_i=\textstyle\sum_{i=1}^d \mathrm{Var}(\text{feature }i)\\[3pt]
\textbf{Proportion explained by PC}_i:\quad & \frac{\lambda_i}{\sum_{j=1}^d \lambda_j}\\[3pt]
\textbf{Projection (score) of }\mathbf x:\quad & \mathbf z = W^\top(\mathbf x-\boldsymbol\mu),\quad W=[\mathbf v_1\cdots\mathbf v_k]\\[3pt]
\textbf{Mean sq. reconstruction error (keep top }k):\quad & \textstyle\sum_{j=k+1}^{d}\lambda_j
\end{aligned}}
$$

*Conditions:* requires **centering** before computing $S$; $S$ is **symmetric positive-semidefinite**, so all $\lambda_i\ge 0$ and eigenvectors are orthogonal (spectral theorem). The reconstruction-error identity holds because the discarded directions contribute exactly their variances $\lambda_{k+1},\dots,\lambda_d$.

### 4.3 Why eigenvectors? (variance-maximization derivation — GATE may ask)

Project the centered data onto a **unit** direction $\mathbf w$. The projected values are $\tilde X\mathbf w$, whose variance is
$$
\mathrm{Var}(\tilde X\mathbf w)=\mathbf w^\top S\,\mathbf w.
$$
Maximize this subject to $\mathbf w^\top\mathbf w=1$. Form the Lagrangian
$$
\mathcal L=\mathbf w^\top S\,\mathbf w-\lambda(\mathbf w^\top\mathbf w-1),\qquad
\frac{\partial\mathcal L}{\partial\mathbf w}=2S\mathbf w-2\lambda\mathbf w=0
\;\Longrightarrow\; \boxed{S\mathbf w=\lambda\mathbf w.}
$$
So the optimal $\mathbf w$ is an **eigenvector** of $S$, and the variance it captures is $\mathbf w^\top S\mathbf w=\mathbf w^\top(\lambda\mathbf w)=\lambda$. To **maximize** variance, choose the eigenvector with the **largest eigenvalue** ($\mathbf v_1$). The 2nd PC maximizes variance subject to being **orthogonal** to $\mathbf v_1$ → the next-largest eigenvalue, and so on. This is exactly why PCA = eigen-decomposition of the covariance matrix. ∎

### 4.4 Properties worth memorizing

- Eigenvalues are **non-negative** ($S$ is PSD); $\lambda_i=$ variance along $\mathbf v_i$.
- PCs are **orthonormal** and the projected features are **uncorrelated** (covariance of $Z$ is $\mathrm{diag}(\lambda_1,\dots,\lambda_k)$).
- **Total variance is preserved** across all $d$ PCs: $\sum\lambda_i=\operatorname{trace}(S)$.
- Number of **non-zero** eigenvalues $\le \min(n-1,\,d)$ (centering removes one degree of freedom).
- Eigenvectors are defined **up to sign** — $\mathbf v$ and $-\mathbf v$ are both valid PCs.
- PCA is a **rotation** (orthogonal transform) of the feature space, then truncation.

### 4.5 Choosing $k$ (how many components to keep)

- **Cumulative explained-variance ratio:** smallest $k$ with $\dfrac{\sum_{i=1}^k\lambda_i}{\sum_{j=1}^d\lambda_j}\ge \tau$ (commonly $\tau=0.90$ or $0.95$).
- **Scree plot / elbow:** plot $\lambda_i$ vs $i$; keep components before the curve flattens.
- **Kaiser criterion** (when using the **correlation** matrix): keep components with $\lambda_i>1$ *(recognition level)*.

### 4.6 Standardize or not? (covariance vs correlation matrix)

PCA is **not scale-invariant**: a feature measured in large units has large variance and will dominate the leading PCs purely because of its scale. If features are on **different scales/units**, **standardize** each to unit variance first — equivalently, run PCA on the **correlation matrix**. If features share comparable units and scale matters, the plain **covariance matrix** is appropriate. (This is a favourite trap.)

### 4.7 Relationship to SVD (recognition level)

PCA can be computed via the **Singular Value Decomposition** of the centered data: $\tilde X=U\Sigma V^\top$. The columns of $V$ are the **principal directions** (eigenvectors of $S=\tfrac1n \tilde X^\top\tilde X$), and the eigenvalues relate to singular values by $\lambda_i=\sigma_i^2/n$. SVD is the numerically preferred route, but for GATE-sized problems you eigen-decompose $S$ directly.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Full PCA from a given covariance matrix (multi-part NAT)

The covariance matrix of a centered 2-feature dataset is
$$
S=\begin{bmatrix}5 & 2\\ 2 & 5\end{bmatrix}.
$$
Find (a) the eigenvalues, (b) the principal components, (c) the proportion of variance explained by PC1, (d) the projection of the centered point $(4,2)$ onto each PC, (e) the mean squared reconstruction error if only PC1 is kept.

**(a) Eigenvalues.** $\det(S-\lambda I)=(5-\lambda)^2-2^2=0\Rightarrow(5-\lambda)^2=4\Rightarrow 5-\lambda=\pm2.$
$$
\boxed{\lambda_1=7,\quad \lambda_2=3.}
$$

**(b) Principal components (unit eigenvectors).**
- $\lambda_1=7$: $(5-7)v_1+2v_2=0\Rightarrow v_1=v_2\Rightarrow \mathbf v_1=\tfrac{1}{\sqrt2}(1,1)$.
- $\lambda_2=3$: $(5-3)v_1+2v_2=0\Rightarrow v_1=-v_2\Rightarrow \mathbf v_2=\tfrac{1}{\sqrt2}(1,-1)$.

**(c) Variance explained by PC1.** Total variance $=\operatorname{trace}(S)=10=\lambda_1+\lambda_2$.
$$
\frac{\lambda_1}{\lambda_1+\lambda_2}=\frac{7}{10}=\boxed{70\%}\quad(\text{PC2 explains }30\%).
$$

**(d) Projection of $(4,2)$.**
- Onto $\mathbf v_1$: $\tfrac{1}{\sqrt2}(1,1)\cdot(4,2)=\dfrac{4+2}{\sqrt2}=\dfrac{6}{\sqrt2}=3\sqrt2\approx \mathbf{4.24}$.
- Onto $\mathbf v_2$: $\tfrac{1}{\sqrt2}(1,-1)\cdot(4,2)=\dfrac{4-2}{\sqrt2}=\dfrac{2}{\sqrt2}=\sqrt2\approx \mathbf{1.41}$.

**(e) Reconstruction error keeping only PC1.** Equals the sum of discarded eigenvalues $=\lambda_2=\boxed{3}$ (i.e. $30\%$ of the total variance is lost).

---

### Problem 2 — PCA from raw data (multi-part NAT)

Four points: $P_1(2,1),\,P_2(1,2),\,P_3(-1,-2),\,P_4(-2,-1)$. Find (a) the mean, (b) the (population) covariance matrix, (c) the eigenvalues, (d) the first principal component, (e) the % variance captured by PC1.

**(a) Mean.** $\bar x=\tfrac{2+1-1-2}{4}=0$, $\bar y=\tfrac{1+2-2-1}{4}=0$ → mean $(0,0)$ (data already centered).

**(b) Covariance (divide by $n=4$).**
$$
\mathrm{Var}(x)=\tfrac{4+1+1+4}{4}=2.5,\quad \mathrm{Var}(y)=\tfrac{1+4+4+1}{4}=2.5,
$$
$$
\mathrm{Cov}(x,y)=\tfrac{(2)(1)+(1)(2)+(-1)(-2)+(-2)(-1)}{4}=\tfrac{8}{4}=2
\;\Rightarrow\; S=\begin{bmatrix}2.5 & 2\\ 2 & 2.5\end{bmatrix}.
$$

**(c) Eigenvalues.** $(2.5-\lambda)^2-4=0\Rightarrow 2.5-\lambda=\pm2\Rightarrow \boxed{\lambda_1=4.5,\ \lambda_2=0.5}$.

**(d) First PC.** $\lambda_1=4.5$: $(2.5-4.5)v_1+2v_2=0\Rightarrow v_1=v_2\Rightarrow \boxed{\mathbf v_1=\tfrac{1}{\sqrt2}(1,1)}$ (the $45^\circ$ line — exactly the direction the data spreads along). (PC2 $=\tfrac{1}{\sqrt2}(1,-1)$.)

**(e) Variance by PC1.** Total $=2.5+2.5=5=4.5+0.5$; ratio $=\dfrac{4.5}{5}=\boxed{90\%}$.

> Using sample covariance ($\div(n-1)=3$) instead gives $S=\begin{bmatrix}10/3 & 8/3\\ 8/3 & 10/3\end{bmatrix}$, eigenvalues $6$ and $\tfrac23$ — the **eigenvectors and the $90\%$ ratio are unchanged**; only the eigenvalue magnitudes scale. The variance proportions never depend on the $\div n$ vs $\div(n-1)$ choice.

---

### Problem 3 — Choosing $k$ by explained variance (NAT)

The covariance matrix of a 5-feature dataset has eigenvalues $\lambda=\{4.0,\,2.5,\,1.0,\,0.4,\,0.1\}$. What is the smallest number of principal components retaining at least $90\%$ of the total variance?

**Solution.** Total $=4.0+2.5+1.0+0.4+0.1=8.0$. Cumulative ratios:
- $k=1:\ 4.0/8=50\%$
- $k=2:\ 6.5/8=81.25\%$
- $k=3:\ 7.5/8=93.75\%\ \ge 90\%$ ✓

$k=2$ gives only $81.25\%<90\%$, so the answer is $\boxed{k=3}$ (retaining $93.75\%$).

---

### Problem 4 — Conceptual trap (MSQ): which statements about PCA are TRUE?

(A) The principal components are mutually orthogonal.
(B) The first principal component corresponds to the **smallest** eigenvalue of the covariance matrix.
(C) PCA is scale-invariant, so standardizing features makes no difference.
(D) The variance captured by the $i$-th principal component equals its eigenvalue $\lambda_i$.

**Solution.**
- (A) **True** — eigenvectors of a symmetric matrix are orthogonal.
- (B) **False** — the first PC is the **largest**-eigenvalue eigenvector (max variance).
- (C) **False** — PCA is **not** scale-invariant; standardization (correlation matrix) changes the result when features have different scales.
- (D) **True** — that is precisely what the eigenvalue measures.

**Answer: (A) and (D).** *(B and C are the classic distractors.)*

---

## 6. Common GATE question types on this topic

- **Given a covariance/correlation matrix:** find eigenvalues, the principal components, and the **% variance explained**.
- **From raw data:** center → covariance matrix → eigenvalues → first PC.
- **Project a point** onto the PC(s) (the score $\mathbf w^\top(\mathbf x-\boldsymbol\mu)$).
- **Choose $k$** for a given explained-variance threshold; or compute **reconstruction error** $=\sum_{j>k}\lambda_j$.
- **Total variance / trace** identities; "what fraction is lost if we keep $k$ PCs?"
- **Properties true/false:** orthogonality, eigenvalue = variance, PSD/non-negative eigenvalues, sign ambiguity, uncorrelated scores.
- **PCA vs LDA** (unsupervised max-variance vs supervised max-separation) and **standardization** effects.

---

## 7. Traps, pitfalls & common mistakes

- **Forgetting to center the data** before computing the covariance matrix. PCA assumes mean-zero features; skipping centering corrupts $S$ and the PCs.
- **Largest vs smallest eigenvalue.** PC1 = **largest** eigenvalue (maximum variance). Picking the smallest is the most common error.
- **Not standardizing when scales differ.** PCA is **not scale-invariant**; a large-unit feature dominates. Use the correlation matrix if units/scales differ.
- **PCA ↔ LDA confusion.** PCA ignores labels and maximizes variance; the highest-variance direction is **not** necessarily the most class-discriminative — that's LDA's job.
- **Un-normalized eigenvectors.** Principal components must be **unit** vectors; divide by the norm (e.g. $(1,1)\to(1,1)/\sqrt2$) before projecting.
- **$\div n$ vs $\div(n-1)$ panic.** It changes eigenvalue *magnitudes* but **not** the eigenvectors or the variance **proportions** — don't let it throw you.
- **Sign of an eigenvector.** $\mathbf v$ and $-\mathbf v$ are both correct; projections just flip sign — not an error.
- **Treating variance as task-relevance.** High variance $\ne$ useful for a label; PCA can discard a low-variance but discriminative direction.
- **Reconstruction-error formula.** Error from keeping the top $k$ is the sum of the **discarded** eigenvalues $\sum_{j>k}\lambda_j$, not the kept ones.
- **Characteristic-equation arithmetic.** For $\begin{bmatrix}a&b\\ b&c\end{bmatrix}$, eigenvalues are $\tfrac{(a+c)\pm\sqrt{(a-c)^2+4b^2}}{2}$ — a sign slip here ruins everything downstream.

---

## 8. Connection — closing the Unsupervised Learning section

PCA completes the unsupervised arc and ties back across the whole syllabus:

- **Back to clustering (16–17):** running PCA *first* shrinks $d$, defeats the curse of dimensionality, and restores meaningful distances — so k-means/hierarchical work better and faster on the reduced data. PCA + clustering is a standard pipeline.
- **Back to LDA (8):** same "find good axes" idea, opposite supervision — PCA maximizes total variance (unsupervised); LDA maximizes between-class vs within-class scatter (supervised). Knowing both, and when each is appropriate, is high-yield.
- **Back to ridge regression (4):** PCA removes the correlated/redundant directions that cause multicollinearity (principal-components regression is the explicit combination).
- **Beyond GATE (one line only):** nonlinear extensions (kernel PCA, autoencoders, t-SNE) handle curved manifolds PCA cannot — recognition level; GATE DA centres on linear PCA.

**The full unsupervised story, end to end:** *measure similarity (15) → partition flatly with k-means/k-medoid (16) → build the whole hierarchy with linkage (17) → recognize that high dimensions break distance-based methods (18) → fix it optimally by projecting onto the top variance directions with PCA (19).*

---

## 9. One-line cheat-sheet summary

> **PCA:** center (and standardize if scales differ → use the **correlation** matrix), form covariance $S=\tfrac1n\tilde X^\top\tilde X$, eigen-decompose; **principal components = eigenvectors**, ordered by eigenvalue, where $\lambda_i$ = variance along PC$_i$ and PC1 = **largest** $\lambda$. PCs are **orthogonal/uncorrelated**; total variance $=\sum\lambda_i=\operatorname{trace}(S)$; PC$_i$ explains $\lambda_i/\sum\lambda_j$; project via $\mathbf w^\top(\mathbf x-\boldsymbol\mu)$; keeping the top $k$ **minimizes reconstruction error** $=\sum_{j>k}\lambda_j$. Unsupervised, max-variance — **not** LDA, **not** scale-invariant.

---

### ✅ Section complete. This is the **final topic (19/19)** of the Machine Learning → Unsupervised Learning sequence; there is no next topic within it. For revision, the highest-yield computational drills across this section are: **k-means iterations (16), single-/complete-linkage merges (17), and the full PCA pipeline (19)** — practice those by hand under time. If you'd like, I can next produce a one-page **formula-and-trap revision sheet** spanning Topics 15–19, or begin notes for a different GATE DA subject.

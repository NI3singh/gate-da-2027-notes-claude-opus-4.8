# Topic 4 — Ridge Regression

**Subject:** Machine Learning
**Section:** Supervised Learning
**Position in syllabus:** The **regularised** repair of Multiple Linear Regression. It keeps the linear model and least-squares idea but adds an **L2 penalty** so the estimator survives multicollinearity and the $p\ge n$ case. This completes the linear-regression-for-continuous-targets family. *Comes after Multiple Linear Regression; leads into Logistic Regression.*

---

## 1. Why this topic exists (motivation + connection)

MLR's estimator $\hat{\boldsymbol\beta}=(X^\top X)^{-1}X^\top\mathbf y$ has a fatal weakness: when predictors are collinear or there are more predictors than observations, $X^\top X$ is **singular** (no inverse, no unique solution) or **near-singular** (the inverse explodes, so coefficients swing wildly with tiny data changes). OLS is then either undefined or useless for interpretation.

**Ridge Regression** removes this failure with one move: add a small multiple of the identity, $X^\top X+\lambda I$, before inverting. Equivalently, it minimises the squared error **plus a penalty on the size of the coefficients**. This always has a unique solution, shrinks the coefficients toward zero, and trades a little **bias** for a large drop in **variance** — often lowering total prediction error.

The bridge from the previous topic in one line: **Ridge exists because MLR's OLS breaks down when $X^\top X$ is singular/near-singular; it fixes this by regularising with $\lambda I$, guaranteeing invertibility and stabilising the estimates.** (Ridge is also called **Tikhonov regularisation** / **L2 regularisation**.)

---

## 2. Prerequisites

- **Topic 3 (MLR)** — OLS, design matrix, normal equations, and especially the **singularity / multicollinearity** problem ridge is built to solve.
- **Linear algebra** — symmetric matrices, **eigenvalues**, **positive (semi)definiteness**, matrix inverse.
- **Optimisation** — minimising a quadratic in $\boldsymbol\beta$; (for the constrained view) Lagrange multipliers.
- **Bias & variance of an estimator** — the language of the trade-off (full treatment is Topic 11).

---

## 3. Core idea — intuition first

OLS will do *anything* to drive the training error down — including handing huge, opposite-signed coefficients to two near-identical predictors so their effects nearly cancel. The fit looks great in-sample but the coefficients are nonsense and the model generalises badly.

Ridge says: *"Fit the data well, **but** keep the coefficients small."* It adds a cost $\lambda\sum_j\beta_j^2$ to the objective, so the optimiser is no longer allowed to inflate coefficients for free. The knob $\lambda$ controls the discipline: $\lambda=0$ is plain OLS (no discipline); as $\lambda$ grows, coefficients are squeezed harder toward zero; as $\lambda\to\infty$ they vanish. The sweet spot — found by cross-validation — sacrifices a touch of fit to gain a lot of stability.

A money analogy: OLS spends unlimited "coefficient budget" to chase the data; ridge puts you on a budget, so you spend it only on predictors that genuinely earn their keep.

---

## 4. Formal treatment

### The penalised objective and the estimator

> **Ridge objective (penalise slopes, not the intercept):**
> $$J(\boldsymbol\beta)=\underbrace{\|\mathbf y-X\boldsymbol\beta\|^2}_{\text{fit}}+\underbrace{\lambda\sum_{j=1}^{p}\beta_j^2}_{\text{L2 penalty}}=\|\mathbf y-X\boldsymbol\beta\|^2+\lambda\boldsymbol\beta^\top\boldsymbol\beta,\qquad \lambda\ge0$$
>
> **Closed-form solution:**
> $$\boxed{\ \hat{\boldsymbol\beta}_{\text{ridge}}=(X^\top X+\lambda I)^{-1}X^\top\mathbf y\ }$$

**Derivation** (GATE expects it): set the gradient to zero,
$$\nabla_{\boldsymbol\beta}J=-2X^\top\mathbf y+2X^\top X\boldsymbol\beta+2\lambda\boldsymbol\beta=\mathbf 0\;\Rightarrow\;(X^\top X+\lambda I)\hat{\boldsymbol\beta}=X^\top\mathbf y.$$

### Why $X^\top X+\lambda I$ is **always invertible** (the whole point)

$X^\top X$ is symmetric positive **semi**definite, so its eigenvalues satisfy $d_i\ge0$ (some may be $0$ ⇒ singular). Adding $\lambda I$ **shifts every eigenvalue up by $\lambda$**:

$$\text{eig}(X^\top X+\lambda I)=d_i+\lambda.$$

For $\lambda>0$ all eigenvalues are $>0$, so $X^\top X+\lambda I$ is positive **definite** and invertible — **even when $X^\top X$ is singular or $p>n$.** This is exactly why ridge works where OLS cannot.

### Limiting behaviour

- $\lambda=0$: $\hat{\boldsymbol\beta}_{\text{ridge}}=\hat{\boldsymbol\beta}_{\text{OLS}}$ (no penalty).
- $\lambda\to\infty$: the penalty dominates ⇒ $\hat{\boldsymbol\beta}_{\text{ridge}}\to\mathbf 0$ (slopes shrink to zero; with centred data the intercept stays at $\bar y$).

### The orthonormal case — clean shrinkage

If the columns are orthonormal, $X^\top X=I$, then $\hat{\boldsymbol\beta}_{\text{OLS}}=X^\top\mathbf y$ and

$$\hat{\boldsymbol\beta}_{\text{ridge}}=(I+\lambda I)^{-1}X^\top\mathbf y=\frac{1}{1+\lambda}\,\hat{\boldsymbol\beta}_{\text{OLS}}.$$

Every coefficient is scaled by the same factor $\tfrac{1}{1+\lambda}\in(0,1)$ — pure shrinkage, no selection. More generally, in a diagonalised (principal-axis) view a coordinate with eigenvalue $d$ is shrunk by $\dfrac{d}{d+\lambda}$, so **low-variance directions (small $d$) are damped most** — precisely the unstable directions multicollinearity creates.

### Bias–variance trade-off (the conceptual heart)

Ridge is a **biased** estimator for $\lambda>0$:

> $$\mathbb E[\hat{\boldsymbol\beta}_{\text{ridge}}]=(X^\top X+\lambda I)^{-1}X^\top X\,\boldsymbol\beta\ \neq\ \boldsymbol\beta$$
> $$\mathrm{Cov}(\hat{\boldsymbol\beta}_{\text{ridge}})=\sigma^2\,(X^\top X+\lambda I)^{-1}X^\top X\,(X^\top X+\lambda I)^{-1}$$

**Direction of the trade-off (what GATE tests):** as $\lambda\uparrow$, **bias increases** and **variance decreases**. Because $\text{MSE}=\text{bias}^2+\text{variance}$, there **provably exists some $\lambda>0$ for which ridge has strictly lower MSE than OLS** — you give up unbiasedness to win on total error. This is the same logic formalised in Topic 11.

### Constrained-optimisation (geometric) view

Ridge is equivalent to a budgeted least squares:

$$\min_{\boldsymbol\beta}\ \|\mathbf y-X\boldsymbol\beta\|^2\quad\text{subject to}\quad \sum_{j=1}^{p}\beta_j^2\le t,$$

with a one-to-one $\lambda\leftrightarrow t$ correspondence (Lagrangian). Picture elliptical RSS contours meeting a **spherical** constraint $\|\boldsymbol\beta\|^2\le t$: they touch on the sphere's smooth surface, so coefficients are **shrunk but rarely exactly zero**. *(Contrast — for awareness only: Lasso uses an L1 ball $\sum|\beta_j|\le t$, whose corners drive some coefficients to exactly $0$, giving variable selection. Ridge does not select; it shrinks.)*

### Practical conditions you must respect

- **Standardise the predictors first.** Unlike OLS, ridge is **not scale-invariant**: the penalty $\sum\beta_j^2$ treats all coefficients equally, but a coefficient's size depends on its predictor's scale. Centre and scale (mean $0$, unit variance) so the penalty is fair across predictors.
- **Do not penalise the intercept.** Centre $\mathbf y$ and the columns; the intercept is then $\bar y$ and is left out of the penalty.
- **Choose $\lambda$ by cross-validation**, not by training fit — $\lambda$ is a hyperparameter (Topic 12).

---

## 5. Worked problems (GATE-style)

### Problem 1 (NAT — compute, and see direction-dependent shrinkage)
Given $X^\top X=\begin{pmatrix}4&0\\0&1\end{pmatrix}$, $X^\top\mathbf y=\begin{pmatrix}8\\3\end{pmatrix}$, and $\lambda=1$, find the OLS and ridge estimates and compare the shrinkage.

**Solution.**
OLS: $\hat{\boldsymbol\beta}_{\text{OLS}}=\begin{pmatrix}1/4&0\\0&1\end{pmatrix}\begin{pmatrix}8\\3\end{pmatrix}=\begin{pmatrix}2\\3\end{pmatrix}.$

Ridge: $X^\top X+\lambda I=\begin{pmatrix}5&0\\0&2\end{pmatrix}\Rightarrow\hat{\boldsymbol\beta}_{\text{ridge}}=\begin{pmatrix}1/5&0\\0&1/2\end{pmatrix}\begin{pmatrix}8\\3\end{pmatrix}=\begin{pmatrix}1.6\\1.5\end{pmatrix}.$

Shrinkage factors $\tfrac{d}{d+\lambda}$: first coordinate $\tfrac{4}{5}=0.8$ ($2\to1.6$), second $\tfrac{1}{2}=0.5$ ($3\to1.5$). **The low-eigenvalue direction (the less-informative one) is shrunk far more** — ridge automatically distrusts the unstable directions.

### Problem 2 (NAT — orthonormal shrinkage, reverse)
For an orthonormal design ($X^\top X=I$), the OLS estimate of a coefficient is $6$. After ridge regression it becomes $4$. Find $\lambda$.

**Solution.** Orthonormal ⇒ $\hat\beta_{\text{ridge}}=\dfrac{1}{1+\lambda}\hat\beta_{\text{OLS}}$. So $4=\dfrac{6}{1+\lambda}\Rightarrow 1+\lambda=\dfrac{6}{4}=1.5\Rightarrow\boxed{\lambda=0.5}.$

### Problem 3 (MSQ — concept traps)
Which of the following are **true** of ridge regression?
(A) Ridge estimates are unbiased.
(B) $X^\top X+\lambda I$ is invertible for every $\lambda>0$, even when $X^\top X$ is singular.
(C) As $\lambda\to\infty$, the (penalised) coefficients tend to $0$.
(D) Ridge is scale-invariant, so standardising predictors is unnecessary.

**Solution.**
- (A) **False** — ridge is biased for $\lambda>0$; that is the price paid for lower variance.
- (B) **True** — eigenvalues become $d_i+\lambda>0$, making the matrix positive definite.
- (C) **True** — the penalty dominates and drives coefficients to $0$.
- (D) **False** — ridge is **not** scale-invariant; predictors must be standardised first.

**Answer: (B) and (C).**

---

## 6. Common GATE question types on this topic

- **Compute $\hat{\boldsymbol\beta}_{\text{ridge}}=(X^\top X+\lambda I)^{-1}X^\top\mathbf y$** for small/diagonal $X^\top X$.
- **Orthonormal shrinkage** — apply or invert the $\tfrac{1}{1+\lambda}$ factor; find $\lambda$ from a given shrink.
- **Invertibility / eigenvalues** — explain why adding $\lambda I$ makes the matrix invertible (eigenvalue shift to $d_i+\lambda$).
- **Limiting cases** — identify what happens at $\lambda=0$ and $\lambda\to\infty$.
- **Bias–variance reasoning** — direction of bias/variance with $\lambda$; why ridge can beat OLS in MSE.
- **Conceptual MSQ** — biasedness, scale-dependence (standardisation), shrink-not-select, L2 vs L1.
- **Multicollinearity remedy** — recognise ridge as the fix for ill-conditioned $X^\top X$.

---

## 7. Traps, pitfalls & common mistakes

- **"Ridge is unbiased."** No — it is **biased** for any $\lambda>0$. Unbiasedness is exactly what it trades away.
- **Forgetting to standardise.** OLS is scale-equivariant; **ridge is not.** Skipping standardisation lets the penalty unfairly punish small-scale predictors.
- **Penalising the intercept.** The intercept is usually left unpenalised (centre the data); penalising it distorts the baseline.
- **Expecting exact zeros.** Ridge **shrinks** coefficients but (almost) never sets them to exactly $0$ — that is **Lasso (L1)**, not ridge. Don't claim ridge does variable selection.
- **Thinking larger $\lambda$ always improves the model.** Too-large $\lambda$ over-shrinks ⇒ underfitting (high bias). $\lambda$ must be tuned (CV).
- **Sign of the penalty / matrix.** The objective adds $+\lambda\|\boldsymbol\beta\|^2$ and the estimator uses $X^\top X+\lambda I$ (plus, not minus).
- **Confusing $\lambda$ with a parameter to be estimated.** $\lambda$ is a **hyperparameter** chosen by cross-validation, not fit from the training residuals.
- **Believing ridge fixes coefficient *interpretability*.** It stabilises and shrinks; coefficients are biased toward $0$, so their magnitudes are not the unbiased "true" effects.

---

## 8. Connection forward

With SLR, MLR, and Ridge, we have the complete toolkit for predicting a **continuous** target with linear models — model form, the matrix solution, and regularisation for stability. The next step changes the **task**: the target is now a **class label**, not a number. **Logistic Regression** reuses the very same linear predictor $\eta=\mathbf x^\top\boldsymbol\beta$, but (i) passes it through the **sigmoid** $\sigma(\eta)=\frac{1}{1+e^{-\eta}}$ to output a **probability** in $(0,1)$, and (ii) is fit by **maximum likelihood / cross-entropy** instead of least squares — because squared error is the wrong loss for probabilities. The bridge: **Logistic Regression keeps the linear core of these models but adapts it from regression to binary classification by squashing the linear score into a probability and switching the loss; and the L2 penalty you just learned reappears directly as "regularised logistic regression."**

---

## 9. One-line cheat-sheet summary

**Ridge minimises $\|\mathbf y-X\boldsymbol\beta\|^2+\lambda\|\boldsymbol\beta\|^2$, giving $\hat{\boldsymbol\beta}=(X^\top X+\lambda I)^{-1}X^\top\mathbf y$ — always invertible (eigenvalues $d_i+\lambda>0$), biased but lower-variance, shrinking coefficients by $\tfrac{1}{1+\lambda}$ in the orthonormal case; standardise first, never penalise the intercept, and pick $\lambda$ by cross-validation.**

---

*Next topic (in order):* **Logistic Regression.**

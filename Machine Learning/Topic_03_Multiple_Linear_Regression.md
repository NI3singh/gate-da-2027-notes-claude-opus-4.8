# Topic 3 — Multiple Linear Regression (MLR)

**Subject:** Machine Learning
**Section:** Supervised Learning
**Position in syllabus:** The matrix generalisation of Simple Linear Regression to **many predictors at once**. Everything here is the engine room for the rest of regression: it introduces the design matrix, the normal equations in matrix form, the invertibility condition, and the goodness-of-fit tools. *Comes after Simple Linear Regression; leads into Ridge Regression.*

---

## 1. Why this topic exists (motivation + connection)

SLR explains $y$ using a **single** predictor. But almost nothing in the real world depends on just one thing — a house price depends on area *and* location *and* age *and* number of rooms simultaneously. **Multiple Linear Regression** lets the output depend linearly on **$p$ predictors at once**, and — crucially — it estimates each predictor's effect *while holding the others fixed*, something SLR cannot do (an SLR slope silently absorbs the influence of every variable you left out).

What SLR couldn't handle: multiple inputs, and "controlling for" confounders. MLR fixes both by stacking predictors into a **design matrix** $X$ and solving one matrix equation. Its own weakness then sets up the next topic: when the predictor columns are **correlated with each other** (multicollinearity) or there are **too many predictors relative to data** ($p$ large vs $n$), the matrix $X^\top X$ becomes singular or nearly so, and OLS either has no unique answer or produces wildly unstable, huge-variance coefficients. That failure is exactly what **Ridge Regression** is built to cure.

---

## 2. Prerequisites

- **Topic 2 (SLR)** — least squares, residuals, $R^2$, the slope-as-covariance idea (MLR is its matrix lift).
- **Matrix algebra** — multiplication, transpose, **inverse**, **rank**, **linear independence**, column space, orthogonal projection, positive (semi)definiteness.
- **Matrix/vector calculus** — gradient of a linear form and a quadratic form (used in the derivation, restated below).
- **Covariance matrix** — to read $\mathrm{Cov}(\hat{\boldsymbol\beta})$.

---

## 3. Core idea — intuition first

Picture each observation as a row: its $p$ feature values plus a leading $1$ (for the intercept). Stack all $n$ rows into a tall matrix $X$. We seek **one coefficient vector** $\boldsymbol\beta$ so that $X\boldsymbol\beta$ — a weighted blend of the columns — lands as close as possible to the observed target vector $\mathbf y$.

Geometrically: the achievable predictions $X\boldsymbol\beta$ sweep out a flat subspace (the **column space** of $X$) inside $\mathbb R^n$. The point in that subspace nearest to $\mathbf y$ is its **orthogonal (perpendicular) projection** $\hat{\mathbf y}$. "Least squares" *is* "drop a perpendicular from $\mathbf y$ onto the column space." The residual $\mathbf y-\hat{\mathbf y}$ is the perpendicular leg, so it is orthogonal to every predictor — which is precisely the normal equations.

---

## 4. Formal treatment

### The model and the design matrix

> **Model (with intercept, $p$ predictors):** $\;y_i=\beta_0+\beta_1x_{i1}+\cdots+\beta_px_{ip}+\varepsilon_i$
> **Matrix form:** $\;\mathbf y = X\boldsymbol\beta + \boldsymbol\varepsilon$

**Dimensions (memorise — directly tested):** with an intercept and $p$ predictors,

$$X:\ n\times(p+1),\qquad \boldsymbol\beta:\ (p+1)\times1,\qquad \mathbf y,\boldsymbol\varepsilon:\ n\times1,\qquad X^\top X:\ (p+1)\times(p+1)$$

The first column of $X$ is all $1$s (the intercept column).

### Assumptions (generalise SLR's; note the new one)

1. **Linearity in parameters** — $\mathbb E[\mathbf y\mid X]=X\boldsymbol\beta$.
2. **Zero-mean errors** — $\mathbb E[\boldsymbol\varepsilon]=\mathbf 0$.
3. **Spherical errors** — $\mathrm{Cov}(\boldsymbol\varepsilon)=\sigma^2 I_n$ (constant variance **and** uncorrelated).
4. **Full column rank** — $\mathrm{rank}(X)=p+1$ (no exact linear dependence among columns ⇒ **no perfect multicollinearity**). *This is the new, load-bearing assumption.* It forces $n\ge p+1$.
5. **(Inference only) Normality** — $\boldsymbol\varepsilon\sim\mathcal N(\mathbf 0,\sigma^2 I)$.

### OLS estimation — derivation

Minimise $\;\text{RSS}(\boldsymbol\beta)=\|\mathbf y-X\boldsymbol\beta\|^2=(\mathbf y-X\boldsymbol\beta)^\top(\mathbf y-X\boldsymbol\beta).$

Expand (using the scalar identity $\mathbf y^\top X\boldsymbol\beta=\boldsymbol\beta^\top X^\top\mathbf y$):

$$\text{RSS}=\mathbf y^\top\mathbf y-2\,\boldsymbol\beta^\top X^\top\mathbf y+\boldsymbol\beta^\top (X^\top X)\boldsymbol\beta.$$

Apply the **vector-calculus rules** $\;\nabla_{\boldsymbol\beta}(\mathbf a^\top\boldsymbol\beta)=\mathbf a\;$ and $\;\nabla_{\boldsymbol\beta}(\boldsymbol\beta^\top A\boldsymbol\beta)=2A\boldsymbol\beta\;$ (for symmetric $A$):

$$\nabla_{\boldsymbol\beta}\text{RSS}=-2X^\top\mathbf y+2(X^\top X)\boldsymbol\beta=\mathbf 0\;\Longrightarrow\;\boxed{(X^\top X)\,\hat{\boldsymbol\beta}=X^\top\mathbf y}\quad(\textbf{normal equations})$$

The Hessian is $2X^\top X$, which is positive **semi**definite always and positive **definite** exactly when $X$ has full column rank — so the solution is a **unique global minimum iff** $\mathrm{rank}(X)=p+1$.

> **Closed-form OLS estimator**
> $$\hat{\boldsymbol\beta}=(X^\top X)^{-1}X^\top\mathbf y$$
> **Condition:** $X^\top X$ must be invertible $\iff$ $X$ has full column rank $\iff$ columns linearly independent $\iff$ no perfect multicollinearity. If it fails, **no unique OLS solution exists.**

### Geometry: the hat (projection) matrix

$$\hat{\mathbf y}=X\hat{\boldsymbol\beta}=\underbrace{X(X^\top X)^{-1}X^\top}_{H,\ \text{hat matrix}}\,\mathbf y,\qquad \mathbf e=\mathbf y-\hat{\mathbf y}=(I-H)\mathbf y$$

> **Properties of $H$ (favourite MCQ material):**
> - **Symmetric:** $H^\top=H$. **Idempotent:** $H^2=H$ (so is $I-H$).
> - $HX=X$, and $X^\top\mathbf e=\mathbf 0$ (residuals orthogonal to all predictors).
> - $\mathrm{trace}(H)=p+1=\mathrm{rank}(X)$ (number of fitted parameters); $\mathrm{trace}(I-H)=n-p-1$.
> - $\hat{\mathbf y}$ is the orthogonal projection of $\mathbf y$ onto $\mathrm{Col}(X)$; $\mathbf e$ lives in the orthogonal complement.

### Sampling properties of $\hat{\boldsymbol\beta}$

Since $\hat{\boldsymbol\beta}=\boldsymbol\beta+(X^\top X)^{-1}X^\top\boldsymbol\varepsilon$:

> $$\mathbb E[\hat{\boldsymbol\beta}]=\boldsymbol\beta\ \ (\textbf{unbiased}),\qquad \mathrm{Cov}(\hat{\boldsymbol\beta})=\sigma^2 (X^\top X)^{-1}$$

This generalises SLR's $\mathrm{Var}(\hat\beta_1)=\sigma^2/S_{xx}$. **Key reading for the next topic:** when $X^\top X$ is near-singular, the entries of $(X^\top X)^{-1}$ explode, so the coefficient variances blow up — the symptom Ridge tames. Under assumptions 1–4, OLS is **BLUE** (**Gauss–Markov**).

### Noise variance & degrees of freedom

> $$\hat\sigma^2=\frac{\text{RSS}}{\,n-p-1\,}\quad(\text{unbiased; d.o.f.}=n-\underbrace{(p+1)}_{\#\text{params}})$$

### Goodness of fit

With an intercept, the decomposition $\text{TSS}=\text{ESS}+\text{RSS}$ still holds, and

$$R^2=\frac{\text{ESS}}{\text{TSS}}=1-\frac{\text{RSS}}{\text{TSS}}=R_{\text{multiple}}^2,\qquad R=\mathrm{corr}(\mathbf y,\hat{\mathbf y})$$

**Critical caveat:** $R^2$ **never decreases** when you add a predictor (RSS can only shrink), so it cannot tell whether a new predictor is *worth it*. Use the penalised version:

> **Adjusted $R^2$**
> $$\bar R^2=1-\frac{\text{RSS}/(n-p-1)}{\text{TSS}/(n-1)}=1-(1-R^2)\,\frac{n-1}{n-p-1}$$
> $\bar R^2$ **rises only if** the added predictor reduces RSS more than its 1-d.o.f. cost; otherwise it falls. $\bar R^2\le R^2$ always, and $\bar R^2$ can be negative.

*(Overall-significance test, if asked: $F=\dfrac{\text{ESS}/p}{\text{RSS}/(n-p-1)}\sim F_{p,\,n-p-1}$ under $H_0:\beta_1=\cdots=\beta_p=0$.)*

### Multicollinearity (the seed of Ridge) and VIF

- **Perfect multicollinearity** — a predictor is an exact linear combination of others ⇒ $X^\top X$ **singular** ⇒ OLS undefined.
- **Near multicollinearity** — predictors highly correlated ⇒ $X^\top X$ **ill-conditioned** ⇒ huge coefficient variances, unstable estimates, nonsensical signs, extreme sensitivity to tiny data changes. *Predictions can still be fine; individual coefficients become unreliable.*
- **Detection — Variance Inflation Factor:** $\;\mathrm{VIF}_j=\dfrac{1}{1-R_j^2}$, where $R_j^2$ is from regressing $x_j$ on all other predictors. $\mathrm{VIF}=1$ ⇒ no collinearity; $\mathrm{VIF}\gtrsim5\text{–}10$ ⇒ a problem.

### Coefficient interpretation

$\beta_j$ = expected change in $y$ for a **one-unit increase in $x_j$ with all other predictors held constant** (a *partial* effect). This "holding others fixed" clause is what separates an MLR coefficient from an SLR slope.

---

## 5. Worked problems (GATE-style)

### Problem 1 (NAT + concept — scaffolding & invertibility)
An MLR with an intercept is fit to $n=10$ observations and $p=3$ predictors.
(a) Give the dimensions of $X$, $\boldsymbol\beta$, $X^\top X$, and $\hat{\mathbf y}$.
(b) State the residual degrees of freedom.
(c) Suppose the third predictor satisfies $x_3=2x_1+5$ for every observation. Does a unique OLS estimate exist? Why?

**Solution.**
(a) $X:10\times4$, $\boldsymbol\beta:4\times1$, $X^\top X:4\times4$, $\hat{\mathbf y}=X\boldsymbol\beta:10\times1$.
(b) d.o.f. $=n-(p+1)=10-4=\boxed{6}$.
(c) The column for $x_3$ equals $2\cdot(\text{column }x_1)+5\cdot(\text{intercept column})$ — an **exact linear dependence** among the 4 columns. So $\mathrm{rank}(X)\le3<4$, making $X^\top X$ **singular** (non-invertible). **No unique OLS solution exists** — this is perfect multicollinearity, the very situation Ridge resolves.

### Problem 2 (NAT — solve the normal equations)
For a two-coefficient regression the normal equations reduce to
$$X^\top X=\begin{pmatrix}2&1\\1&2\end{pmatrix},\qquad X^\top\mathbf y=\begin{pmatrix}5\\4\end{pmatrix}.$$
Find $\hat{\boldsymbol\beta}$.

**Solution.** $\det(X^\top X)=2\cdot2-1\cdot1=3$, so $(X^\top X)^{-1}=\dfrac13\begin{pmatrix}2&-1\\-1&2\end{pmatrix}$.
$$\hat{\boldsymbol\beta}=\frac13\begin{pmatrix}2&-1\\-1&2\end{pmatrix}\begin{pmatrix}5\\4\end{pmatrix}=\frac13\begin{pmatrix}10-4\\-5+8\end{pmatrix}=\frac13\begin{pmatrix}6\\3\end{pmatrix}=\boxed{\begin{pmatrix}2\\1\end{pmatrix}}.$$

### Problem 3 (NAT — $R^2$ vs adjusted $R^2$, the model-comparison trap)
With $n=20$: Model A uses $p=2$ predictors and gives $R^2=0.80$. Adding a third predictor (Model B, $p=3$) raises $R^2$ to $0.81$. Compute $\bar R^2$ for both and decide whether the new predictor helped.

**Solution.** $\bar R^2=1-(1-R^2)\frac{n-1}{n-p-1}$.
- Model A: $1-(0.20)\cdot\frac{19}{17}=1-0.20(1.1176)=1-0.2235=\boxed{0.776}$.
- Model B: $1-(0.19)\cdot\frac{19}{16}=1-0.19(1.1875)=1-0.2256=\boxed{0.774}$.

$R^2$ rose ($0.80\to0.81$) but $\bar R^2$ **fell** ($0.776\to0.774$). The third predictor reduced RSS by less than its d.o.f. cost, so it did **not** improve the model. *Lesson: never compare models of different sizes by $R^2$.*

---

## 6. Common GATE question types on this topic

- **Dimensions & d.o.f.** — sizes of $X,\boldsymbol\beta,X^\top X,\hat{\mathbf y}$; residual d.o.f. $=n-p-1$.
- **Solve $\hat{\boldsymbol\beta}=(X^\top X)^{-1}X^\top\mathbf y$** from a given (usually $2\times2$) system or from $X^\top X,X^\top\mathbf y$.
- **Invertibility / uniqueness** — "does a unique OLS estimate exist?" via rank / linear-dependence reasoning.
- **Hat-matrix properties** — identify symmetric, idempotent, $\mathrm{trace}(H)=p+1$, projection statements.
- **$R^2$ vs adjusted $R^2$** — compute, and reason about adding/removing predictors.
- **$\mathrm{Cov}(\hat{\boldsymbol\beta})=\sigma^2(X^\top X)^{-1}$** and $\hat\sigma^2=\text{RSS}/(n-p-1)$.
- **Multicollinearity / VIF** — detect, explain consequences, compute $\mathrm{VIF}=1/(1-R_j^2)$.
- **Coefficient interpretation** — the "holding others constant" meaning.

---

## 7. Traps, pitfalls & common mistakes

- **Wrong parameter count.** With an intercept there are $p+1$ parameters; d.o.f. is $n-p-1$ (not $n-p$, not $n-1$). Forgetting the intercept column is the classic dimension slip.
- **Treating $R^2$ as a model-selection score.** It only ever rises with more predictors — use **adjusted $R^2$** (or AIC/BIC, $F$-test) to compare sizes.
- **$R^2=r^2$ is SLR-only.** In MLR, $R^2$ is the squared *multiple* correlation $\mathrm{corr}(\mathbf y,\hat{\mathbf y})^2$, not a single pairwise correlation.
- **Assuming $(X^\top X)^{-1}$ always exists.** It fails under perfect multicollinearity or when $p+1>n$. Then OLS has no unique solution — the doorway to Ridge.
- **Dummy-variable trap.** Encoding a $k$-category variable with $k$ dummies *and* keeping the intercept creates an exact dependence (the dummies sum to the intercept column) ⇒ singular $X^\top X$. Use $k-1$ dummies.
- **Confusing "predictions are fine" with "coefficients are fine."** Near-multicollinearity can leave $\hat{\mathbf y}$ accurate while individual $\hat\beta_j$ are garbage with huge variance.
- **Misreading a coefficient.** $\beta_j$ is a *partial* effect (others held fixed); it generally differs from the SLR slope of $y$ on $x_j$ alone.
- **Matrix order errors.** It is $(X^\top X)^{-1}X^\top\mathbf y$ — keep the transpose placement and multiplication order exact; $X^\top X$ is $(p+1)\times(p+1)$, not $n\times n$.
- **Correlation ≠ causation**, and a high $R^2$ never proves the model is "correct."

---

## 8. Connection forward

MLR's Achilles heel is the inverse $(X^\top X)^{-1}$: when predictors are collinear or $p$ is large relative to $n$, $X^\top X$ is singular or ill-conditioned, so OLS is undefined or has exploding variance. **Ridge Regression** removes this failure by adding a small $\lambda I$ before inverting:

$$\hat{\boldsymbol\beta}_{\text{ridge}}=(X^\top X+\lambda I)^{-1}X^\top\mathbf y,\qquad \lambda>0,$$

which is **always invertible** (it makes the matrix positive definite), **shrinks** the coefficients toward $0$, and trades a little bias for a large drop in variance. The bridge in one line: **Ridge exists because MLR's OLS breaks down when $X^\top X$ is singular/near-singular; it fixes this by regularising $X^\top X$ with $\lambda I$, guaranteeing invertibility and stabilising the estimates.**

---

## 9. One-line cheat-sheet summary

**MLR fits $\mathbf y=X\boldsymbol\beta+\boldsymbol\varepsilon$ by OLS $\hat{\boldsymbol\beta}=(X^\top X)^{-1}X^\top\mathbf y$ (unique iff $X$ is full-rank); $\hat{\mathbf y}=H\mathbf y$ projects $\mathbf y$ onto $\mathrm{Col}(X)$, $\mathrm{Cov}(\hat{\boldsymbol\beta})=\sigma^2(X^\top X)^{-1}$, d.o.f. $=n-p-1$, and you compare models with adjusted $R^2$, not $R^2$.**

---

*Next topic (in order):* **Ridge Regression.**

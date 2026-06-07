# Topic 5 — Logistic Regression

**Subject:** Machine Learning
**Section:** Supervised Learning
**Position in syllabus:** The hinge between regression and classification. It keeps the **linear predictor** of the regression family but targets a **binary class label**, squashing the linear score into a probability. It is the first **classification** method and the standard **discriminative** linear classifier. *Comes after Ridge Regression; the next topic (k-Nearest Neighbour) begins a new chat.*

---

## 1. Why this topic exists (motivation + connection)

Everything so far predicted a **continuous** number. But a huge class of problems asks a **yes/no** question: is this email spam, will this loan default, is this tumour malignant? Here $y\in\{0,1\}$, and the natural quantity to predict is $P(Y=1\mid x)$ — a **probability**.

Why not just run linear regression on the 0/1 labels? Three things break: (i) $X\boldsymbol\beta$ is unbounded, so it can predict probabilities below $0$ or above $1$ — nonsense; (ii) the relationship between predictors and a probability **saturates** near $0$ and $1$, not a straight line; (iii) the errors are **Bernoulli**, not Gaussian/homoscedastic, so least-squares assumptions fail. **Logistic Regression** fixes all three by passing the linear predictor through the **sigmoid**, which maps $(-\infty,\infty)\to(0,1)$, and by fitting with **maximum likelihood (cross-entropy)** instead of least squares.

Bridge from the previous topic: **Logistic Regression reuses the linear core $x^\top\boldsymbol\beta$ of SLR/MLR/Ridge but adapts it from regression to binary classification — squashing the score into a probability via the sigmoid and switching the loss to cross-entropy; and the L2 penalty from Ridge carries over directly as "regularised logistic regression."**

---

## 2. Prerequisites

- **Topics 2–4** — the linear predictor $x^\top\boldsymbol\beta$, and L2 regularisation (it reappears here).
- **Topic 1** — the classification task, loss functions, and the discriminative-vs-generative split.
- **Probability** — Bernoulli distribution, conditional probability, **odds**, likelihood and log-likelihood.
- **Calculus** — chain rule, the sigmoid's derivative, gradients, **convexity**; iterative optimisation (gradient ascent / Newton's method).
- **Exponentials & logarithms.**

---

## 3. Core idea — intuition first

Start with the familiar linear score $\eta=\beta_0+\beta_1x_1+\cdots+\beta_px_p$. This score lives on the whole real line — perfect for "how strongly do the features vote for class 1," but it isn't a probability. So feed it through an **S-shaped squashing function**, the sigmoid: very negative scores map near $0$, very positive scores near $1$, and a score of $0$ maps to exactly $0.5$.

The right way to read the model is through **odds**. The sigmoid is built so that the **log-odds of class 1 is exactly the linear score**: $\log\frac{p}{1-p}=x^\top\boldsymbol\beta$. So logistic regression is "linear regression on the log-odds." That makes each coefficient interpretable: bumping a feature by one unit adds $\beta_j$ to the log-odds, i.e. **multiplies the odds by $e^{\beta_j}$**. To *decide*, threshold the probability at $0.5$ — which, beautifully, corresponds to a **straight-line (hyperplane) boundary** in feature space.

---

## 4. Formal treatment

### The model

> **Linear predictor (logit):** $\;\eta(x)=\beta_0+\beta_1x_1+\cdots+\beta_px_p=x^\top\boldsymbol\beta$
> **Sigmoid (logistic) function:** $\;\sigma(z)=\dfrac{1}{1+e^{-z}}=\dfrac{e^{z}}{1+e^{z}}$
> **Model:** $\;p(x)=P(Y=1\mid x)=\sigma(x^\top\boldsymbol\beta)$

**Sigmoid properties (used constantly):** range $(0,1)$; $\sigma(0)=0.5$; monotonically increasing; symmetric $\sigma(-z)=1-\sigma(z)$; and the derivative

$$\boxed{\ \sigma'(z)=\sigma(z)\,[1-\sigma(z)]\ }$$

### Odds, log-odds, and coefficient meaning

$$\frac{p(x)}{1-p(x)}=e^{x^\top\boldsymbol\beta}\quad\Longleftrightarrow\quad \boxed{\ \log\frac{p(x)}{1-p(x)}=x^\top\boldsymbol\beta\ }\ (\textbf{logit link})$$

> **Interpretation of $\beta_j$ (with other predictors held fixed):** a one-unit increase in $x_j$ adds $\beta_j$ to the **log-odds** and multiplies the **odds** by $e^{\beta_j}$ (the **odds ratio**). $\beta_j>0\Rightarrow$ odds rise; $\beta_j=0\Rightarrow$ odds unchanged (OR $=1$); $\beta_j<0\Rightarrow$ odds fall.

### Decision rule — and why the boundary is linear

Predict $\hat y=1$ if $p(x)\ge0.5$. Since $\sigma$ is increasing and $\sigma(0)=0.5$,

$$p(x)\ge0.5\iff x^\top\boldsymbol\beta\ge0,$$

so the **decision boundary $x^\top\boldsymbol\beta=0$ is a hyperplane** — logistic regression is a **linear classifier**, even though $p(x)$ is nonlinear in $x$. (A general threshold $t$ gives the boundary $x^\top\boldsymbol\beta=\log\frac{t}{1-t}$ — still linear; lowering $t$ catches more positives at the cost of more false positives.)

### Estimation — Maximum Likelihood (no closed form!)

The labels are Bernoulli with $p_i=\sigma(x_i^\top\boldsymbol\beta)$:

$$L(\boldsymbol\beta)=\prod_{i=1}^{n}p_i^{\,y_i}(1-p_i)^{1-y_i},\qquad \ell(\boldsymbol\beta)=\sum_{i=1}^{n}\big[y_i\log p_i+(1-y_i)\log(1-p_i)\big].$$

> **Maximising the log-likelihood $\ell$ is identical to minimising the negative log-likelihood, a.k.a. the cross-entropy / log loss.**

**Why not squared error?** Squared error composed with the sigmoid is **non-convex** in $\boldsymbol\beta$ (many local minima), whereas the cross-entropy is **convex** in $\boldsymbol\beta$ — a single global optimum (unique if $X$ is full-rank and the classes are not perfectly separable).

**Gradient (derive — GATE-relevant).** Using $\sigma'=\sigma(1-\sigma)$ and the chain rule, the messy terms collapse:

$$\frac{\partial\ell}{\partial\beta_j}=\sum_{i=1}^{n}\big(y_i-p_i\big)x_{ij}\qquad\Longleftrightarrow\qquad \nabla\ell=X^\top(\mathbf y-\mathbf p).$$

This mirrors linear regression's residual gradient, but with $p_i=\sigma(x_i^\top\boldsymbol\beta)$ in place of $\hat y_i$. Setting $\nabla\ell=\mathbf 0$ gives **transcendental** equations with **no closed-form solution** — unlike OLS. Solve **iteratively**:

- **Gradient ascent:** $\boldsymbol\beta\leftarrow\boldsymbol\beta+\alpha\,X^\top(\mathbf y-\mathbf p)$.
- **Newton–Raphson / IRLS:** $\boldsymbol\beta\leftarrow\boldsymbol\beta+(X^\top W X)^{-1}X^\top(\mathbf y-\mathbf p)$, with $W=\mathrm{diag}\big(p_i(1-p_i)\big)$. The Hessian $-X^\top WX$ is negative semidefinite, confirming concavity of $\ell$.

### Regularisation, multiclass, and the generative contrast

- **Regularised logistic regression:** minimise $-\ell(\boldsymbol\beta)+\lambda\|\boldsymbol\beta\|^2$ (L2, from Ridge) — also cures the separation blow-up below.
- **Multiclass ($K$ classes):** **softmax / multinomial** logistic regression, $P(Y=k\mid x)=\dfrac{e^{x^\top\boldsymbol\beta_k}}{\sum_{j}e^{x^\top\boldsymbol\beta_j}}$; or one-vs-rest.
- **Discriminative model:** logistic regression models $P(Y\mid X)$ **directly** and assumes only that the **log-odds is linear** — it makes *no* assumption about the distribution of $X$. (Upcoming generative methods — Naive Bayes, LDA — instead model $P(X\mid Y)$; LDA also yields a linear boundary but only under Gaussian, equal-covariance assumptions.)

### Assumptions / conditions

1. **Binary (or categorical) outcome**; observations **independent**.
2. **Log-odds linear** in the predictors (the model can be wrong if the true relationship is curved).
3. **No perfect multicollinearity** (else $\boldsymbol\beta$ not identifiable).
4. **No requirement** of normality or equal variance of predictors (a key contrast with LDA).
5. **No perfect (complete) separation** — if a hyperplane separates the classes exactly, the MLE **diverges** ($\|\boldsymbol\beta\|\to\infty$, sigmoid $\to$ step). Regularisation prevents this.

*(Threshold choice trades off error types; precision, recall, and accuracy are read off the confusion matrix and depend on the chosen threshold.)*

---

## 5. Worked problems (GATE-style)

### Problem 1 (NAT — probability, classification, odds, boundary)
A fitted model has $\log\dfrac{p}{1-p}=1+2x_1-x_2$. For the point $x_1=1,\,x_2=1$:
(a) predicted probability $p$ and predicted class; (b) the odds, and the effect on the odds of increasing $x_1$ by $1$; (c) the decision boundary at threshold $0.5$.

**Solution.**
(a) Logit $=1+2(1)-1=2$, so $p=\sigma(2)=\dfrac{1}{1+e^{-2}}\approx\dfrac{1}{1.1353}\approx\boxed{0.881}$. Since $p\ge0.5$, predict **class 1**.
(b) Odds $=e^{2}\approx\boxed{7.39}$ (check: $0.881/0.119\approx7.39$). Increasing $x_1$ by $1$ multiplies the odds by $e^{\beta_1}=e^{2}\approx7.39$.
(c) Boundary: $1+2x_1-x_2=0\Rightarrow\boxed{x_2=1+2x_1}$ — a straight line (linear classifier).

### Problem 2 (NAT — gradient component)
Three observations have true labels $y=(1,0,1)$ and current predicted probabilities $p=(0.6,0.3,0.8)$. The feature $x_1$ takes values $(2,1,3)$. Find the gradient of the log-likelihood w.r.t. $\beta_1$.

**Solution.** $\dfrac{\partial\ell}{\partial\beta_1}=\sum_i(y_i-p_i)x_{i1}=(1-0.6)(2)+(0-0.3)(1)+(1-0.8)(3)=0.8-0.3+0.6=\boxed{1.1}.$
(The positive gradient says: to increase the likelihood, push $\beta_1$ **up** in a gradient-ascent step.)

### Problem 3 (MSQ — concept traps)
Which statements about logistic regression are **true**?
(A) It is a regression method whose output is a continuous, unbounded value.
(B) At threshold $0.5$, its decision boundary is **linear** in the features.
(C) Its parameters have a closed-form solution, like ordinary least squares.
(D) The **log-odds** is a linear function of the predictors.

**Solution.**
- (A) **False** — it is a **classification** method; the output is a probability in $(0,1)$, thresholded to a class.
- (B) **True** — $p\ge0.5\iff x^\top\boldsymbol\beta\ge0$, a hyperplane.
- (C) **False** — **no closed form**; fit iteratively by MLE (gradient ascent / Newton–IRLS).
- (D) **True** — that is the logit-link definition $\log\frac{p}{1-p}=x^\top\boldsymbol\beta$.

**Answer: (B) and (D).**

---

## 6. Common GATE question types on this topic

- **Evaluate $p(x)=\sigma(x^\top\boldsymbol\beta)$** and classify at a threshold.
- **Odds / log-odds / odds ratio** — compute, and interpret $e^{\beta_j}$.
- **Decision boundary** — write $x^\top\boldsymbol\beta=0$ and identify it as linear.
- **Cross-entropy / log loss** — compute for given $(y_i,p_i)$.
- **Gradient / one update step** — apply $\nabla\ell=X^\top(\mathbf y-\mathbf p)$.
- **Sigmoid algebra** — $\sigma(0)=0.5$, $\sigma(-z)=1-\sigma(z)$, $\sigma'=\sigma(1-\sigma)$.
- **Conceptual** — classification not regression, linear boundary, convex loss, no closed form, discriminative vs generative, separation issue.

---

## 7. Traps, pitfalls & common mistakes

- **"Logistic regression is regression."** It is **classification**. The name refers to the logistic *function*, not a continuous target. (Classic GATE bait, foreshadowed in Topic 1.)
- **Expecting a closed form.** There is **none**; the MLE is found iteratively. Don't write a "$(X^\top X)^{-1}\dots$" formula for it.
- **Boundary vs probability.** $p(x)$ is **nonlinear** in $x$, but the **decision boundary is linear**. Both facts are true simultaneously.
- **Misreading $\beta_j$.** It changes the **log-odds** additively / the **odds** multiplicatively ($\times e^{\beta_j}$) — **not** the probability linearly.
- **Using squared-error loss.** Wrong (non-convex with the sigmoid); use **cross-entropy** (convex).
- **Forgetting the unit.** $\sigma$ and logs are typically computed with **natural** base $e$ in these formulas; keep $e$ consistent.
- **Perfect separation.** If the classes are linearly separable, the MLE coefficients diverge to $\pm\infty$ — apply regularisation; don't report "infinite" coefficients as a valid fit.
- **Multicollinearity.** Same identifiability problem as MLR — collinear predictors make coefficients unstable.
- **Threshold is a choice.** $0.5$ is only the default; changing it shifts the confusion matrix (precision/recall trade-off), so accuracy alone can mislead on imbalanced data.

---

## 8. Connection forward

Logistic regression is **parametric** and **discriminative**: it commits to a specific shape — log-odds *linear* in the features — learns a fixed coefficient vector $\boldsymbol\beta$, and produces a single global linear boundary. The next method drops these commitments entirely. **k-Nearest Neighbour (k-NN)** is **non-parametric** and **instance-based**: it fits *no* parameters and assumes *no* boundary shape; it simply stores the training data and classifies a new point by the majority vote of its nearest neighbours, so it can carve out highly nonlinear boundaries. The bridge: **logistic regression assumes a parametric (linear-in-log-odds) form and learns weights once; k-NN makes no such assumption and lets the local data decide at prediction time — trading model assumptions and fast prediction for heavy memory and computation per query.**

*(Note: k-Nearest Neighbour begins a **new chat** — carry this bridge across so the story stays connected.)*

---

## 9. One-line cheat-sheet summary

**Logistic regression models $P(Y{=}1\mid x)=\sigma(x^\top\boldsymbol\beta)$ with linear log-odds $\log\frac{p}{1-p}=x^\top\boldsymbol\beta$; it is a discriminative **classifier** with a **linear** boundary $x^\top\boldsymbol\beta=0$, fit by maximising the (convex) Bernoulli log-likelihood via the iterative gradient $\nabla\ell=X^\top(\mathbf y-\mathbf p)$ — no closed form — where $e^{\beta_j}$ is the odds ratio per unit of $x_j$.**

---

*This completes the Regression chat. Next topic (in a new chat):* **k-Nearest Neighbour.**

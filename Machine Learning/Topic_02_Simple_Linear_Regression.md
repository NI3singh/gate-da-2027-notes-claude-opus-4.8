# Topic 2 — Simple Linear Regression (SLR)

**Subject:** Machine Learning
**Section:** Supervised Learning
**Position in syllabus:** The first **concrete model** of the Supervised Learning block. It takes the abstract "learn $f$, minimise a loss" idea from Topic 1 and makes it fully explicit on the simplest possible case: **one** predictor, a **straight-line** fit, trained by **least squares**. *Comes after Regression & Classification Problems; leads into Multiple Linear Regression.*

---

## 1. Why this topic exists (motivation + connection)

Topic 1 told us *what* a regression problem is (predict a continuous $y$ from features $x$, minimise squared error) but gave no machine to actually produce a prediction. **Simple Linear Regression is that first machine.** It assumes the relationship between one input $x$ and the output $y$ is a straight line and finds the *best* line by minimising the squared prediction error.

What the previous topic couldn't do: it stopped at "the optimal predictor is $\mathbb{E}[Y\mid X]$" — a population quantity we can't compute from a finite sample. SLR makes a workable assumption — that $\mathbb{E}[Y\mid X]$ is **linear** in $x$ — so we can *estimate* it from data in closed form. Its limitation in turn forces the **next** topic: real problems have many predictors, not one, which generalises SLR into **Multiple Linear Regression**.

---

## 2. Prerequisites

- **Topic 1** — supervised setup ($D=\{(x_i,y_i)\}$), squared-error loss, empirical-risk minimisation, the regression task.
- **Mean, variance, covariance, correlation** — the slope is built entirely from these.
- **Partial derivatives & unconstrained minimisation** — setting $\partial(\text{RSS})/\partial\beta = 0$.
- **Basic summation algebra** — manipulating $\sum x_i,\ \sum x_i y_i,$ etc.

---

## 3. Core idea — intuition first

You have a scatter of points and you believe $y$ rises (or falls) roughly **linearly** with $x$. You want the single straight line $\hat y = \hat\beta_0 + \hat\beta_1 x$ that sits "closest" to the cloud. "Closest" is defined as **smallest total squared vertical gap** between each point and the line — square the residuals so positives and negatives don't cancel, and so big misses are penalised hard.

The slope $\hat\beta_1$ answers: *"if $x$ goes up by 1 unit, by how much does the predicted $y$ change?"* The intercept $\hat\beta_0$ is the predicted $y$ when $x=0$. Two facts make the whole topic intuitive:

- the best line **always passes through the centre of mass** $(\bar x, \bar y)$, and
- the slope is just **covariance of $x,y$ divided by variance of $x$** — it points the line along the direction the data trends, scaled by how much $x$ spreads.

---

## 4. Formal treatment

### The model

> **Population (true) model:** $\;y_i = \beta_0 + \beta_1 x_i + \varepsilon_i$
> **Fitted model:** $\;\hat y_i = \hat\beta_0 + \hat\beta_1 x_i$
> **Residual:** $\;e_i = y_i - \hat y_i$

Here $\beta_0,\beta_1$ are unknown true parameters; $\hat\beta_0,\hat\beta_1$ are our estimates; $\varepsilon_i$ is random noise.

### Assumptions (state these — GATE tests when they break)

1. **Linearity in parameters** — $\mathbb{E}[y\mid x]=\beta_0+\beta_1 x$.
2. **Zero-mean errors** — $\mathbb{E}[\varepsilon_i]=0$.
3. **Homoscedasticity** — $\mathrm{Var}(\varepsilon_i)=\sigma^2$, *constant* for all $i$.
4. **Independence / no autocorrelation** — $\mathrm{Cov}(\varepsilon_i,\varepsilon_j)=0$ for $i\neq j$.
5. **(For inference only) Normality** — $\varepsilon_i\sim\mathcal N(0,\sigma^2)$. *Not needed* for the point estimates or for Gauss–Markov.
6. **Predictor varies** — $S_{xx}=\sum(x_i-\bar x)^2>0$ (the $x_i$ are not all identical), else the slope is undefined.

### Least-squares estimation (OLS)

Minimise the **residual sum of squares**:

$$\text{RSS}(\beta_0,\beta_1)=\sum_{i=1}^{n}\big(y_i-\beta_0-\beta_1 x_i\big)^2$$

**Derivation of the normal equations** (GATE does expect this). Set both partials to zero:

$$\frac{\partial\text{RSS}}{\partial\beta_0}=-2\sum\big(y_i-\beta_0-\beta_1 x_i\big)=0\ \Rightarrow\ \sum y_i=n\beta_0+\beta_1\sum x_i \quad\text{(NE1)}$$

$$\frac{\partial\text{RSS}}{\partial\beta_1}=-2\sum x_i\big(y_i-\beta_0-\beta_1 x_i\big)=0\ \Rightarrow\ \sum x_iy_i=\beta_0\sum x_i+\beta_1\sum x_i^2 \quad\text{(NE2)}$$

From (NE1): $\hat\beta_0=\bar y-\hat\beta_1\bar x$. Substitute into (NE2) and simplify (using $\sum x_iy_i-n\bar x\bar y=S_{xy}$ and $\sum x_i^2-n\bar x^2=S_{xx}$) to get $\hat\beta_1=S_{xy}/S_{xx}$. The objective is a convex paraboloid, so this stationary point is the **global minimum** (valid because $S_{xx}>0$). ∎

> **Closed-form OLS estimators**
> $$\boxed{\ \hat\beta_1=\frac{S_{xy}}{S_{xx}}=\frac{\sum(x_i-\bar x)(y_i-\bar y)}{\sum(x_i-\bar x)^2}=\frac{n\sum x_iy_i-\sum x_i\sum y_i}{\,n\sum x_i^2-(\sum x_i)^2\,}\ }$$
> $$\boxed{\ \hat\beta_0=\bar y-\hat\beta_1\bar x\ }$$
> **Condition:** requires $S_{xx}>0$. The two right-hand forms are algebraically identical; use the **summation form** when given $\sum x,\sum y,\sum xy,\sum x^2$, and the **deviation form** when given raw points.

**Link to correlation (memorise):**

$$\hat\beta_1=r\,\frac{s_y}{s_x},\qquad r=\frac{S_{xy}}{\sqrt{S_{xx}\,S_{yy}}}$$

So the slope's **sign equals the sign of the correlation** $r$.

### Algebraic properties of the OLS fit (with intercept)

These hold *automatically* whenever an intercept is included — frequent MCQ fodder:

- $\sum e_i = 0$ — residuals sum to zero (from NE1).
- $\sum x_i e_i = 0$ — residuals are orthogonal to the predictor (from NE2).
- $\sum \hat y_i e_i = 0$ — residuals orthogonal to fitted values.
- The line passes through $(\bar x,\bar y)$, and $\overline{\hat y}=\bar y$.

### Goodness of fit

$$\underbrace{\sum(y_i-\bar y)^2}_{\text{TSS}=S_{yy}}=\underbrace{\sum(\hat y_i-\bar y)^2}_{\text{ESS (regression)}}+\underbrace{\sum(y_i-\hat y_i)^2}_{\text{RSS (error)}}$$

**Condition:** this decomposition $\text{TSS}=\text{ESS}+\text{RSS}$ holds **only when an intercept is in the model** (the cross-term vanishes because $\sum e_i=0$ and $\sum \hat y_i e_i=0$).

> **Coefficient of determination**
> $$R^2=\frac{\text{ESS}}{\text{TSS}}=1-\frac{\text{RSS}}{\text{TSS}},\qquad 0\le R^2\le 1$$
> **For SLR only:** $\;R^2=r^2\;$ (the square of the correlation coefficient).
> Handy identity: $\;\text{ESS}=\hat\beta_1^2\,S_{xx}=\hat\beta_1\,S_{xy}=S_{xy}^2/S_{xx}.$

*Why $R^2=r^2$:* since $\hat y_i-\bar y=\hat\beta_1(x_i-\bar x)$, ESS $=\hat\beta_1^2 S_{xx}=S_{xy}^2/S_{xx}$, so $R^2=\dfrac{S_{xy}^2/S_{xx}}{S_{yy}}=\dfrac{S_{xy}^2}{S_{xx}S_{yy}}=r^2$.

### Estimating the noise variance & estimator variances

> $$\hat\sigma^2=\frac{\text{RSS}}{n-2}\quad(\text{unbiased; }n-2\text{ d.o.f. because 2 params are estimated})$$
> $$\mathrm{Var}(\hat\beta_1)=\frac{\sigma^2}{S_{xx}},\qquad \mathrm{Var}(\hat\beta_0)=\sigma^2\!\left(\frac1n+\frac{\bar x^2}{S_{xx}}\right)$$
> **Conditions:** assumptions 1–4 hold and $x_i$ treated as fixed. Note $\mathrm{Var}(\hat\beta_1)\downarrow$ as the spread of $x$ ($S_{xx}$) grows — *more spread-out predictors give more precise slopes.*

Under assumptions 1–4, OLS is **BLUE** (Best Linear Unbiased Estimator) — the **Gauss–Markov theorem**: among all linear unbiased estimators, OLS has the smallest variance. (Normality is *not* required for this; it is only needed to run $t$-tests / build confidence intervals, where the slope's test statistic is $t=\hat\beta_1/\widehat{\text{se}}(\hat\beta_1)$ with $n-2$ d.o.f.)

### Effect of linear transformations of the variables (classic trap)

If $x'=ax+b$ and $y'=cy+d$ (with $a,c\neq0$), the regression of $y'$ on $x'$ has:

$$\hat\beta_1'=\frac{c}{a}\,\hat\beta_1,\qquad \hat\beta_0'=c\,\hat\beta_0+d-\frac{c}{a}\hat\beta_1\,b$$

Correlation $|r|$ and $R^2$ are **unchanged**; $r$ flips sign iff exactly one of $a,c$ is negative. *Note the slope divides by the $x$-scale $a$ but multiplies by the $y$-scale $c$.*

---

## 5. Worked problems (GATE-style)

### Problem 1 (NAT — full computation)
Data: $x=(1,2,3,4,5)$, $y=(2,4,5,4,5)$. (a) Find the least-squares line. (b) Predict $y$ at $x=6$. (c) Find $R^2$.

**Solution.** $\bar x=3,\ \bar y=4$.

| $x_i-\bar x$ | $y_i-\bar y$ | product | $(x_i-\bar x)^2$ |
|---|---|---|---|
| $-2$ | $-2$ | $4$ | $4$ |
| $-1$ | $0$ | $0$ | $1$ |
| $0$ | $1$ | $0$ | $0$ |
| $1$ | $0$ | $0$ | $1$ |
| $2$ | $1$ | $2$ | $4$ |

$S_{xy}=6,\ S_{xx}=10,\ S_{yy}=4+0+1+0+1=6$.

(a) $\hat\beta_1=\dfrac{6}{10}=0.6$, $\hat\beta_0=4-0.6(3)=2.2$ ⟹ $\boxed{\hat y=2.2+0.6x}$.

(b) $\hat y(6)=2.2+0.6(6)=\boxed{5.8}$.

(c) $\text{ESS}=\hat\beta_1 S_{xy}=0.6(6)=3.6$, so $R^2=3.6/6=\boxed{0.6}$ (and $r=\sqrt{0.6}\approx0.775$).

*Sanity checks GATE rewards:* fitted values $(2.8,3.4,4.0,4.6,5.2)$ give residuals $(-0.8,0.6,1.0,-0.6,-0.2)$ which **sum to 0** ✔, $\text{RSS}=2.4$, and $\hat\sigma^2=2.4/(5-2)=0.8$.

### Problem 2 (NAT — correlation / reverse regression)
For a dataset, $\bar x=10,\ \bar y=20,\ s_x=2,\ s_y=5,\ r=0.8$.
(a) Regression line of $y$ on $x$. (b) Slope of regression of $x$ on $y$. (c) Verify the product of the two slopes equals $r^2$.

**Solution.**
(a) $\hat\beta_1=r\,s_y/s_x=0.8\cdot\frac{5}{2}=2$; $\hat\beta_0=20-2(10)=0$ ⟹ $\boxed{\hat y=2x}$.
(b) slope of $x$ on $y$: $b_{xy}=r\,s_x/s_y=0.8\cdot\frac{2}{5}=\boxed{0.32}$.
(c) $b_{yx}\cdot b_{xy}=2\times0.32=0.64=0.8^2=r^2$ ✔.

*Takeaway:* the two regression lines differ (slopes $2$ vs reciprocal-side $0.32$); their product is always $r^2$, and they coincide only when $r=\pm1$.

### Problem 3 (MCQ — transformation trap)
The regression of $y$ on $x$ is $\hat y=3+2x$. Units are changed to $x'=2x$ and $y'=3y$. The regression of $y'$ on $x'$ is:
(A) $\hat{y}'=9+3x'$  (B) $\hat{y}'=3+4x'$  (C) $\hat{y}'=9+12x'$  (D) $\hat{y}'=3+2x'$

**Solution.** Use $\hat\beta_1'=\frac{c}{a}\hat\beta_1=\frac{3}{2}(2)=3$ and (with $b=d=0$) $\hat\beta_0'=c\hat\beta_0=3(3)=9$. Or substitute $x=x'/2,\ y=y'/3$ into $y=3+2x$: $\;y'/3=3+x'\Rightarrow y'=9+3x'$.
**Answer: (A).** *Trap:* doubling $x$ does **not** double the slope — it *halves* the slope's $x$-part; tripling $y$ triples it.

---

## 6. Common GATE question types on this topic

- **Compute $\hat\beta_0,\hat\beta_1$** from raw points **or** from summary stats $(\sum x,\sum y,\sum xy,\sum x^2,n)$.
- **Predict** $\hat y$ at a given $x$; or recover an unknown $x$/$y$ from a fitted line.
- **Goodness of fit** — compute $R^2$, TSS/ESS/RSS, or exploit $R^2=r^2$.
- **Correlation ⇄ slope** — get $r$ from slopes, or slope from $r,s_x,s_y$; product-of-slopes $=r^2$.
- **Residual / line properties** — "which is true?" ($\sum e_i=0$, passes through $(\bar x,\bar y)$, orthogonality).
- **Transformation effects** — how rescaling/shifting $x$ or $y$ changes slope, intercept, $r$, $R^2$.
- **Variance / d.o.f.** — $\hat\sigma^2=\text{RSS}/(n-2)$, $\mathrm{Var}(\hat\beta_1)=\sigma^2/S_{xx}$, BLUE/Gauss–Markov statements.

---

## 7. Traps, pitfalls & common mistakes

- **Intercept formula sign.** It is $\hat\beta_0=\bar y-\hat\beta_1\bar x$ (minus), *not* $\bar y+\hat\beta_1\bar x$.
- **Mixing the two slope forms.** The summation form has $n\sum x_iy_i-\sum x_i\sum y_i$ on top and $n\sum x_i^2-(\sum x_i)^2$ on the bottom — don't drop the $n$.
- **$R^2=r^2$ is SLR-only.** It does **not** carry over to multiple regression (there $R^2$ is the squared *multiple* correlation, not a single $r$).
- **TSS = ESS + RSS needs an intercept.** Through-the-origin models break the decomposition and can even give $R^2<0$.
- **Regression isn't symmetric.** $y$-on-$x$ ≠ $x$-on-$y$ unless $r=\pm1$; never invert a fitted line by simple algebra to predict the other direction.
- **Correlation $r$ ≠ slope.** Same sign, but different magnitude and units; $r\in[-1,1]$ while the slope is unbounded.
- **Degrees of freedom.** Divide RSS by $n-2$ (not $n$ or $n-1$) for an unbiased $\hat\sigma^2$.
- **Extrapolation.** The line is only trustworthy within the observed $x$-range; predicting far outside it is unjustified (and a favourite conceptual question).
- **Outliers / high-leverage points.** Squared loss makes a single far point drag the line — recognise this if a question flags an unusual observation.
- **Causation.** A good fit / high $R^2$ shows association, **not** that $x$ causes $y$.

---

## 8. Connection forward

SLR captures only **one** predictor and a **single straight line**. Real data has several inputs at once (price depends on size *and* location *and* age). **Multiple Linear Regression** is the direct generalisation: replace the scalar $x$ by a feature vector and the two scalars $(\beta_0,\beta_1)$ by a coefficient vector, turning the normal equations into the compact matrix form $\hat{\boldsymbol\beta}=(X^\top X)^{-1}X^\top y$. Everything here — least squares, residual orthogonality, $R^2$, the variance formula — reappears in matrix dress. The bridge: **Multiple LR exists because SLR fails when more than one predictor influences $y$; it generalises SLR by stacking predictors into a matrix and inverting $X^\top X$ instead of dividing by $S_{xx}$.**

---

## 9. One-line cheat-sheet summary

**SLR fits $\hat y=\hat\beta_0+\hat\beta_1x$ by least squares: $\hat\beta_1=S_{xy}/S_{xx}=r\,s_y/s_x$, $\hat\beta_0=\bar y-\hat\beta_1\bar x$; the line passes through $(\bar x,\bar y)$, residuals sum to zero, $R^2=r^2$, and $\hat\sigma^2=\text{RSS}/(n-2)$.**

---

*Next topic (in order):* **Multiple Linear Regression.**

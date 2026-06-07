# Topic 14 — Central Limit Theorem (CLT)

*Subject: Probability and Statistics · Section: Sampling Theory & Statistical Inference*

---

## 0. Topic header & syllabus map

- **Topic:** Central Limit Theorem (CLT)
- **Subject:** Probability and Statistics (GATE DA)
- **Where it sits:** This is the **opening topic of the Sampling Theory & Statistical Inference block**. It comes immediately **after** *Sampling distributions (t & chi-squared)* and leads directly **into** *Confidence Intervals* and then *Hypothesis Testing (z/t/chi-squared tests)*. The CLT is the single result that makes the entire inference block work, so everything after this leans on it.

---

## 1. Why this topic exists (motivation + connection)

In real life we never see the whole population — we see a **sample**, compute a number like the sample mean $\bar X$, and want to say something about the unknown population mean $\mu$. But $\bar X$ is a *random* quantity: a different sample gives a different $\bar X$. To quantify "how far is $\bar X$ likely to be from $\mu$," we need the **distribution of $\bar X$** (its *sampling distribution*).

**What the previous topic couldn't do.** The $t$ and $\chi^2$ sampling distributions you just learned are *exact* — but only when the **population itself is normal**. Real populations are frequently *not* normal: incomes are right-skewed, waiting times are exponential, counts are discrete (binomial/Poisson). For those, the exact sampling distribution of $\bar X$ is messy or unknown. We needed a result that works **regardless of the population's shape**.

**What CLT delivers.** It says: add up (or average) many independent observations, and the result becomes **approximately normal** as the sample size grows — *no matter what distribution the individual observations came from*, as long as the variance is finite. This is why the normal distribution shows up everywhere and why one set of normal-based formulas can serve almost any population.

**Forward link.** Once we know $\bar X \approx \mathcal N(\mu,\sigma^2/n)$, we can invert that statement to build **confidence intervals** for $\mu$ (next topic) and to run **z-tests** (topic 16). The CLT *is* the engine behind those.

---

## 2. Prerequisites

- **Expectation & variance** of random variables; **linearity of expectation**.
- **Variance of a sum of independent variables:** $\operatorname{Var}\!\left(\sum X_i\right)=\sum\operatorname{Var}(X_i)$ for independent (or merely uncorrelated) variables.
- **Normal & standard normal distribution**: z-scores, and reading the standard normal CDF $\Phi(z)$ from a table.
- **Binomial and Poisson distributions** (their means and variances) — needed for the normal approximations.
- The idea of **i.i.d.** (independent and identically distributed) observations.
- (From topic 13) The notion of a **sampling distribution** — the distribution of a statistic over repeated samples.

---

## 3. Core idea — intuition first

**Averaging smooths and centers.** Roll one fair die: the outcome is *uniform* over $\{1,\dots,6\}$ — perfectly flat, no peak. Now average **two** dice: the result is *triangular*, peaking at $3.5$ (because a middling average like $3.5$ can happen many ways — $(1,6),(2,5),(3,4),\dots$ — while an extreme average like $1$ needs *both* dice to be $1$, which is rare). Average **many** dice and the histogram becomes a smooth **bell curve** centered at $3.5$.

So even though one die is flat, the *average* of many dice is bell-shaped. The mechanism is general:

> Extreme averages require **all** observations to be extreme **in the same direction** (rare). Middling averages can arise in **many** combinations (common). Counting these combinations pushes the distribution of the average toward the symmetric, peaked, normal shape.

Two things also happen as $n$ grows: the distribution of $\bar X$ **stays centered at $\mu$** (averaging doesn't bias), and it gets **narrower** — its spread is $\sigma/\sqrt n$, which shrinks with $n$. That shrinking spread is the *standard error*, and it's why larger samples give more reliable estimates.

The CLT is the precise version of this picture: **whatever** the starting shape, the standardized average converges to the *same* universal bell curve — the standard normal.

---

## 4. Formal treatment

### 4.1 Properties of the sample mean (exact — hold for *any* $n$ and *any* distribution with finite mean/variance)

Let $X_1,\dots,X_n$ be i.i.d. with $E[X_i]=\mu$ and $\operatorname{Var}(X_i)=\sigma^2$. Define $\displaystyle \bar X=\frac1n\sum_{i=1}^n X_i$.

$$E[\bar X]=\frac1n\sum_{i=1}^n E[X_i]=\frac{n\mu}{n}=\mu$$

$$\operatorname{Var}(\bar X)=\frac{1}{n^2}\sum_{i=1}^n \operatorname{Var}(X_i)=\frac{n\sigma^2}{n^2}=\frac{\sigma^2}{n}
\qquad\Longrightarrow\qquad
\boxed{\operatorname{SE}(\bar X)=\dfrac{\sigma}{\sqrt n}}$$

| Quantity | Value | Condition |
|---|---|---|
| $E[\bar X]$ | $\mu$ | finite mean exists |
| $\operatorname{Var}(\bar X)$ | $\sigma^2/n$ | **independence** (or uncorrelated) + finite variance |
| **Standard error** $\operatorname{SE}(\bar X)$ | $\sigma/\sqrt n$ | same as above |

> ⚠️ These are **not** CLT. They are exact algebra and require **no** assumption about shape. The independence assumption is what kills the cross-terms in the variance — drop it and $\operatorname{Var}(\bar X)\ne\sigma^2/n$. CLT adds a *separate* claim about the **shape** of $\bar X$.

### 4.2 The Central Limit Theorem (classical / Lindeberg–Lévy form)

Let $X_1,X_2,\dots$ be **i.i.d.** with mean $\mu$ and **finite** variance $\sigma^2\in(0,\infty)$. Then as $n\to\infty$:

$$\frac{\bar X-\mu}{\sigma/\sqrt n}\ \xrightarrow{\ d\ }\ \mathcal N(0,1)
\qquad\Longleftrightarrow\qquad
\frac{S_n-n\mu}{\sigma\sqrt n}\ \xrightarrow{\ d\ }\ \mathcal N(0,1),\quad S_n=\sum_{i=1}^n X_i$$

Here "$\xrightarrow{d}$" means **convergence in distribution**: the CDF of the left side approaches the standard normal CDF $\Phi$. The **mean form** and **sum form** are equivalent (just multiply/divide by $n$).

**Working ("finite-$n$") approximations** used to solve problems:

$$\bar X \ \overset{\text{approx}}{\sim}\ \mathcal N\!\left(\mu,\ \frac{\sigma^2}{n}\right),
\qquad
S_n \ \overset{\text{approx}}{\sim}\ \mathcal N\!\left(n\mu,\ n\sigma^2\right)$$

Standardize and use the z-table:
$$z=\frac{\bar X-\mu}{\sigma/\sqrt n}\quad(\text{for a mean}),\qquad z=\frac{S_n-n\mu}{\sigma\sqrt n}\quad(\text{for a sum}).$$

**Conditions / assumptions (memorize — GATE tests the edges):**

1. **Independence** of the $X_i$.
2. **Identically distributed** (classical form). *(Generalizations relax this — see 4.6.)*
3. **Finite variance:** $\sigma^2<\infty$. **This is essential.** Heavy-tailed distributions with infinite variance (e.g. **Cauchy**) do **not** obey the CLT.
4. **Large enough $n$.** The approximation *improves* with $n$. Common rule of thumb: $n\ge 30$. But this is only a guide — the more skewed/heavy-tailed the population, the larger the $n$ needed.

### 4.3 Exact special case: a normal population

If the population is **already normal**, then $\bar X$ is **exactly** normal for **every** $n$ (even $n=1$):
$$X_i\sim\mathcal N(\mu,\sigma^2)\ \Longrightarrow\ \bar X\sim\mathcal N\!\left(\mu,\tfrac{\sigma^2}{n}\right)\ \text{exactly.}$$
No "large $n$" is needed and no approximation is involved — a sum of independent normals is normal.

### 4.4 Normal approximation to the Binomial (De Moivre–Laplace) — a CLT special case

A $\text{Binomial}(n,p)$ variable is a **sum of $n$ i.i.d. Bernoulli$(p)$** indicators, so by CLT, for large $n$:
$$X\sim\text{Binomial}(n,p)\ \overset{\text{approx}}{\sim}\ \mathcal N\big(np,\ np(1-p)\big).$$

**Validity rule:** the approximation is reasonable when $np\ge 5$ **and** $n(1-p)\ge 5$ (some texts use $10$). It works best near $p=0.5$ and needs larger $n$ when $p$ is near $0$ or $1$.

**Continuity correction.** We are approximating a **discrete** distribution by a **continuous** one, so each integer $k$ is spread over the interval $[k-0.5,\,k+0.5]$. Let $Y\sim\mathcal N\big(np,np(1-p)\big)$:

| Discrete probability | Continuity-corrected normal version |
|---|---|
| $P(X=k)$ | $P(k-0.5\le Y\le k+0.5)$ |
| $P(X\le k)$ | $P(Y\le k+0.5)$ |
| $P(X< k)=P(X\le k-1)$ | $P(Y\le k-0.5)$ |
| $P(X\ge k)$ | $P(Y\ge k-0.5)$ |
| $P(X> k)=P(X\ge k+1)$ | $P(Y\ge k+0.5)$ |
| $P(a\le X\le b)$ | $P(a-0.5\le Y\le b+0.5)$ |

> The rule of thumb: **expand the integer interval outward by $0.5$ on each end** before standardizing.

### 4.5 Normal approximation to the Poisson

A $\text{Poisson}(\lambda)$ can be written as a sum of $n$ i.i.d. $\text{Poisson}(\lambda/n)$ pieces, so for large $\lambda$:
$$X\sim\text{Poisson}(\lambda)\ \overset{\text{approx}}{\sim}\ \mathcal N(\lambda,\ \lambda)\qquad(\text{rule of thumb }\lambda\gtrsim 10).$$
*Quick example:* if $X\sim\text{Poisson}(100)$, then $P(X\le 110)\approx P\!\big(Z\le \tfrac{110.5-100}{\sqrt{100}}\big)=P(Z\le 1.05)\approx 0.853$ (with continuity correction, since $\sqrt{\lambda}=10$).

### 4.6 Generalizations (beyond GATE — one line)

The **Lyapunov** and **Lindeberg** CLTs relax the "identically distributed" requirement (independent but differently distributed variables, under mild moment/uniform-smallness conditions, still sum to a normal). **GATE DA only tests the classical i.i.d. version**, so we don't go further.

### 4.7 On the proof (skipped, with reason)

The rigorous proof uses **characteristic functions / moment generating functions**: one shows the MGF of the standardized average converges to $e^{t^2/2}$, the MGF of $\mathcal N(0,1)$. **GATE never asks you to prove the CLT**, so we skip the full proof; the one-line takeaway is just that *the standardized average's MGF → the standard-normal MGF*, which forces the limiting distribution to be normal.

### 4.8 Standard-normal values you'll reuse

| $z$ | $\Phi(z)$ | | $z$ | $\Phi(z)$ |
|---|---|---|---|---|
| $1.00$ | $0.8413$ | | $1.645$ | $0.9500$ |
| $1.10$ | $0.8643$ | | $1.96$ | $0.9750$ |
| $1.20$ | $0.8849$ | | $2.00$ | $0.9772$ |
| $1.28$ | $0.8997$ | | $2.33$ | $0.9901$ |
| $1.50$ | $0.9332$ | | $2.575$ | $0.9950$ |

*(Symmetry: $\Phi(-z)=1-\Phi(z)$.)*

---

## 5. Worked problems (GATE-style)

### Problem 1 — Probability for a sample mean (NAT)

A population has mean $\mu=100$ and standard deviation $\sigma=24$. A random sample of $n=36$ observations is drawn. Find $P(\bar X>104)$.

**Solution.**
Standard error: $\operatorname{SE}=\dfrac{\sigma}{\sqrt n}=\dfrac{24}{\sqrt{36}}=\dfrac{24}{6}=4.$
By the CLT, $\bar X\approx\mathcal N(100,\,4^2)$. Standardize:
$$z=\frac{104-100}{4}=1.$$
$$P(\bar X>104)=P(Z>1)=1-\Phi(1)=1-0.8413=\boxed{0.1587}.$$

*Note:* nothing was assumed about the shape of the population — only that $\mu,\sigma$ are finite and $n=36$ is "large." (If the population were stated to be normal, the same answer would be **exact**.)

---

### Problem 2 — Sum of i.i.d. variables / total load (NAT)

A container is loaded with $n=100$ boxes whose weights are i.i.d. with mean $50$ kg and standard deviation $5$ kg. Find the probability that the **total** weight exceeds $5100$ kg.

**Solution.**
For the sum $S=\sum_{i=1}^{100}X_i$:
$$E[S]=n\mu=100\times50=5000,\qquad \operatorname{Var}(S)=n\sigma^2=100\times25=2500\ \Rightarrow\ \operatorname{SD}(S)=\sqrt{2500}=50.$$
By CLT, $S\approx\mathcal N(5000,\,50^2)$:
$$z=\frac{5100-5000}{50}=2,\qquad P(S>5100)=1-\Phi(2)=1-0.9772=\boxed{0.0228}.$$

> **Scaling trap check:** the SD of the **sum** is $\sigma\sqrt n=5\sqrt{100}=50$ — *not* $\sigma/\sqrt n$. Use $\sigma\sqrt n$ for sums and $\sigma/\sqrt n$ for means.

---

### Problem 3 — Normal approximation to the binomial, with continuity correction (MCQ — trap-style)

A fair die is rolled $180$ times. Using the normal approximation, find $P(\text{at most }35\text{ sixes})$.

(A) $0.8413$  (B) $0.8643$  (C) $0.7286$  (D) $0.5398$

**Solution.**
Number of sixes $X\sim\text{Binomial}(180,\,\tfrac16)$.
$$np=180\cdot\tfrac16=30,\qquad np(1-p)=180\cdot\tfrac16\cdot\tfrac56=25\ \Rightarrow\ \text{SD}=5.$$
Validity: $np=30\ge5$ and $n(1-p)=150\ge5$ ✓.

We need $P(X\le 35)$. **With continuity correction**, $P(X\le35)\approx P(Y\le 35.5)$ where $Y\sim\mathcal N(30,25)$:
$$z=\frac{35.5-30}{5}=1.1,\qquad P(Y\le35.5)=\Phi(1.1)=\boxed{0.8643}\ \Rightarrow\ \textbf{(B)}.$$

**Why the trap option (A) is there:** without the continuity correction you'd compute $z=\frac{35-30}{5}=1$, giving $\Phi(1)=0.8413$ — option (A). GATE deliberately lists both. **Always apply the $\pm0.5$ correction** when approximating a discrete variable.

---

### Problem 4 — Minimum sample size for a target precision (NAT — bridge to confidence intervals)

A population has standard deviation $\sigma=15$. We want the sample mean to lie **within $3$ units** of the true mean $\mu$ with probability at least $0.95$. Find the **minimum** sample size $n$.

**Solution.**
We require $P\big(|\bar X-\mu|\le 3\big)\ge 0.95$. Standardize using $\operatorname{SE}=\sigma/\sqrt n$:
$$P\!\left(|Z|\le \frac{3}{15/\sqrt n}\right)\ge0.95.$$
A central probability of $0.95$ corresponds to $|Z|\le 1.96$. So we need:
$$\frac{3\sqrt n}{15}\ge 1.96\ \Rightarrow\ \sqrt n\ge \frac{1.96\times15}{3}=1.96\times5=9.8\ \Rightarrow\ n\ge 96.04.$$
Since $n$ must be an integer, $\boxed{n=97}$.

> This is **exactly** a confidence-interval margin-of-error calculation: the margin is $E=z_{\alpha/2}\,\sigma/\sqrt n$, and solving for $n$ gives $n=\big(z_{\alpha/2}\,\sigma/E\big)^2$. Here $\big(1.96\cdot15/3\big)^2=96.04\to97$. Topic 15 will name this formula explicitly.

---

## 6. Common GATE question types on this topic

- **Probability for a sample mean** in a range $\Rightarrow$ compute $\operatorname{SE}=\sigma/\sqrt n$, standardize, read $\Phi$.
- **Probability for a sum/total** of i.i.d. variables $\Rightarrow$ use $E[S]=n\mu$, $\operatorname{SD}(S)=\sigma\sqrt n$.
- **Normal approximation to the binomial** (and Poisson), almost always testing whether you applied the **continuity correction** correctly.
- **Minimum sample size** for a stated margin and confidence (direct bridge to confidence intervals).
- **Conceptual / MSQ:** which statement of the CLT is correct; what does it claim and *not* claim; the **finite-variance** requirement; whether it applies to the *mean/sum* (yes) or to *individual observations* (no).
- **Standard-error computation** given $\sigma$ and $n$, or back-solving $n$ from a given SE.

---

## 7. Traps, pitfalls & common mistakes

1. **Standard error vs population SD.** The spread of $\bar X$ is $\sigma/\sqrt n$, **not** $\sigma$. Forgetting the $\sqrt n$ is the single most common slip.
2. **Variance vs standard deviation.** If the problem gives a **variance**, take the square root before forming $z$. ($z$ uses the SD/SE in the denominator.)
3. **Sum vs mean scaling.** SD of a **sum** $=\sigma\sqrt n$; SD of a **mean** $=\sigma/\sqrt n$. These move in opposite directions in $n$ — don't swap them.
4. **Continuity correction.** (a) Forgetting it entirely (off by $0.5$). (b) Applying it in the **wrong direction** — remember "expand the integer interval outward by $0.5$." For $P(X\le k)$ use $k+0.5$; for $P(X\ge k)$ use $k-0.5$.
5. **CLT is about the average/sum, not individual data.** Individual observations keep their original (possibly skewed) distribution; only $\bar X$ (or $S_n$) becomes normal.
6. **Finite variance is mandatory.** Distributions like the **Cauchy** (infinite/undefined variance) do **not** satisfy the CLT — the sample mean does *not* settle into a normal.
7. **Normal population $\Rightarrow$ exact, any $n$.** If told the population is normal, $\bar X$ is *exactly* normal even for tiny $n$ — don't wait for "$n\ge30$" and don't call it an approximation.
8. **"$n\ge30$" is a guideline, not a theorem.** Heavily skewed populations may need much larger $n$; near-symmetric ones converge faster.
9. **Independence is required** for $\operatorname{Var}(\bar X)=\sigma^2/n$ and for the classical CLT. Correlated (e.g., non-random or clustered) samples break the formula.
10. **Known vs unknown $\sigma$.** The clean $z=(\bar X-\mu)/(\sigma/\sqrt n)$ uses the **true** $\sigma$. If $\sigma$ is unknown and estimated by the sample SD $s$ with **small $n$ from a normal population**, you must switch to the **$t$-distribution** (topic 13 / topic 16). For **large $n$**, $t\approx z$, so the CLT-based normal is fine.

---

## 8. Connection forward

The CLT says $\bar X\approx\mathcal N(\mu,\sigma^2/n)$, i.e. $P\!\big(|\bar X-\mu|\le z_{\alpha/2}\,\sigma/\sqrt n\big)\approx 1-\alpha$. **Rearranging that inequality to isolate $\mu$** produces the interval
$$\bar X \pm z_{\alpha/2}\,\frac{\sigma}{\sqrt n},$$
which is precisely a **confidence interval** for $\mu$ — the next topic. The sample-size calculation in Problem 4 is the same idea solved for $n$. And when we instead *fix* a hypothesized $\mu_0$ and ask whether $\bar X$ is "too far" from it, we get the **z-test** of topic 16. In short: **CLT is the foundation; confidence intervals and z-tests are it, rearranged.**

---

## 9. One-line cheat-sheet summary

For i.i.d. data with finite mean $\mu$ and variance $\sigma^2$, the sample mean is approximately $\mathcal N(\mu,\sigma^2/n)$ for large $n$ **regardless of the population's shape** — standardize with $z=\dfrac{\bar X-\mu}{\sigma/\sqrt n}$ (use $\sigma\sqrt n$ for a **sum**), and apply a $\pm0.5$ **continuity correction** when approximating a discrete distribution (e.g. $\text{Binomial}(n,p)\approx\mathcal N(np,\,np(1-p))$).

---

**Next topic (logically next in order): Topic 15 — Confidence Intervals**, which directly inverts this CLT statement to build interval estimates for $\mu$ (and $p$). Ask for it when ready.

# Topic 13 — Sampling Distributions (t-Distribution & Chi-Squared Distribution)

> **File this under:** Probability & Statistics → *Probability Distributions* (closing topic — the bridge into inference).

---

## 0. Topic header & syllabus map

- **Topic:** The two **sampling distributions** GATE DA tests — the **Chi-Squared ($\chi^2$)** distribution and the **$t$-distribution** — together with the idea of a *sampling distribution* itself.
- **Subject:** Probability and Statistics (GATE Data Science & AI).
- **Where it sits:** The **last topic of the Probability Distributions section**. It comes right after **Topic 12 (Continuous Distributions)** and is *built entirely out of the standard normal* you learned there. It is the direct foundation for the inference section: **CLT (Topic 14), Confidence Intervals (Topic 15), and the $t$-test / $\chi^2$-test (Topic 16).** Both distributions you meet here reappear as the *engines* of those tests.

---

## 1. Why this topic exists (motivation + connection)

Topics 11–12 gave you the distribution of a **single random variable** $X$. But statistics is almost never about one observation — it is about **summaries of a whole sample**: the sample mean $\bar X$, the sample variance $S^2$. Because a sample is random, **any statistic computed from it is itself a random variable**, with its own distribution across all possible samples. That distribution is called a **sampling distribution**, and we *must* know it to do any inference (estimate a parameter, build a confidence interval, run a test).

- **What the previous topic couldn't do:** The normal (Topic 12) lets you compute probabilities for $\bar X$ **only when the population $\sigma$ is known** (via $Z=\frac{\bar X-\mu}{\sigma/\sqrt n}$). In real data $\sigma$ is almost *never* known — we estimate it with the sample SD $S$. Plugging a *random, estimated* $S$ in place of the *fixed* $\sigma$ injects extra uncertainty that the normal cannot capture → we need the **$t$-distribution**. And the normal says nothing about the distribution of the **variance estimate $S^2$** itself → we need the **$\chi^2$-distribution**.
- **What the next topics build on it:** The **CLT** (Topic 14) explains *why* $\bar X$ is (approximately) normal even for non-normal populations — which is what justifies using these tools. **Confidence intervals** (Topic 15) use $t$ for the mean (σ unknown) and $\chi^2$ for the variance. **Hypothesis tests** (Topic 16) are literally named after them: the **$t$-test** and the **$\chi^2$-test**.

**The internal bridge (read once — it makes both distributions one idea):**

> Both are *manufactured from the standard normal* $Z\sim N(0,1)$ of Topic 12.
> - **Chi-Squared:** add up squared standard normals — $\chi^2_k=Z_1^2+\cdots+Z_k^2$. It is the natural home of **sums of squares**, hence of **sample variance**.
> - **$t$:** divide a standard normal by the square root of an *independent* chi-squared over its df — $T=\dfrac{Z}{\sqrt{\chi^2_k/k}}$. So **a $t$-variable literally contains a $\chi^2$ in its denominator** — you cannot understand $t$ without $\chi^2$. The $\chi^2$ in the denominator is exactly the randomness of the estimated $S$ that the plain normal ignored.

---

## 2. Prerequisites

- **Standard normal $N(0,1)$ and standardization** (Topic 12): $Z=\frac{X-\mu}{\sigma}$, the table $\Phi$, symmetry. Everything here is built from $Z$.
- **Sampling fact for the mean:** for an i.i.d. sample from $N(\mu,\sigma^2)$, $\ \bar X\sim N\!\big(\mu,\tfrac{\sigma^2}{n}\big)$, so the **standard error** is $\sigma/\sqrt n$ and $\ \dfrac{\bar X-\mu}{\sigma/\sqrt n}\sim N(0,1)$.
- **Expectation & variance** (Topic 9): including, for the standard normal, $E[Z^2]=1$ and $E[Z^4]=3$ (used to get the $\chi^2$ mean and variance).
- **Sample mean & sample variance definitions** (Topic 10): $\bar X=\frac1n\sum X_i$ and $S^2=\dfrac{1}{n-1}\sum (X_i-\bar X)^2$ (note the $n-1$).
- **Exponential** (Topic 12): used for a neat special-case bridge ($\chi^2_2$ is an exponential).

---

## 3. Core idea — intuition first

**What a "sampling distribution" is.** Draw a sample, compute a number (say $\bar X$). Draw another sample, compute it again — you get a *different* number. Repeat conceptually over all possible samples: the spread of those numbers is the **sampling distribution of the statistic**. A statistic is a random variable; a sampling distribution is just *its* distribution. Inference is impossible without it, because to say "how surprising is this $\bar X$?" you must know how $\bar X$ would vary by chance.

**Why a new distribution for the variance ($\chi^2$).** $S^2$ is built from a **sum of squared deviations**. Squares are non-negative, so $S^2\ge0$ and its distribution is **right-skewed**, living on $[0,\infty)$ — nothing like the symmetric normal. If the data are normal, the (scaled) sum of squares follows the chi-squared. Intuitively, **adding more squared normal pieces** (more df) makes the total bigger and, by the CLT, eventually bell-shaped — which is why $\chi^2_k$ drifts toward symmetry as $k$ grows.

**Why a new distribution for the mean when σ is unknown ($t$).** With σ known, $\frac{\bar X-\mu}{\sigma/\sqrt n}=Z$ is exactly standard normal. With σ unknown we substitute $S$: $\frac{\bar X-\mu}{S/\sqrt n}$. Now the denominator is **random** — by bad luck $S$ can come out small, making the ratio shoot to extreme values more often than $Z$ would. Result: a symmetric bell **with fatter tails** than the normal — the $t$-distribution. The fewer the observations (small df), the less we trust $S$, the fatter the tails. As $n\to\infty$, $S\to\sigma$ becomes reliable and **$t\to N(0,1)$.**

**Analogy.** $Z$ is "measuring with a perfect ruler" (σ known). $t$ is "measuring with a ruler whose own length you estimated from the same data" (σ from $S$) — extra wobble, so you must allow more room for surprise. The more tick-marks you used to calibrate the ruler (higher df), the closer it gets to perfect.

---

## 4. Formal treatment

### 4A. The sampling-distribution setup (background you'll reuse)

For an i.i.d. sample $X_1,\dots,X_n$ from $N(\mu,\sigma^2)$:

| Statistic | Sampling distribution | Condition |
|---|---|---|
| Sample mean $\bar X$ | $N\!\big(\mu,\ \sigma^2/n\big)$ | normal population (exact); any population (approx, by CLT) |
| Standardized mean, σ **known** | $\dfrac{\bar X-\mu}{\sigma/\sqrt n}\sim N(0,1)$ | σ known |
| Standardized mean, σ **unknown** | $\dfrac{\bar X-\mu}{S/\sqrt n}\sim t_{n-1}$ | normal population |
| Scaled sample variance | $\dfrac{(n-1)S^2}{\sigma^2}\sim \chi^2_{n-1}$ | normal population |

**Key theorem (needed to build $t$):** For an i.i.d. **normal** sample, $\bar X$ and $S^2$ are **independent**. (This independence is a special property of the normal; GATE does not ask the proof, but you use the fact.)

---

### 4B. Chi-Squared distribution $\;\chi^2_k$ ($k$ degrees of freedom)

**Definition (this is the thing to know cold):**
$$\boxed{\,\text{If } Z_1,\dots,Z_k \stackrel{\text{iid}}{\sim} N(0,1)\ \text{independent, then } \ \chi^2_k := Z_1^2+Z_2^2+\cdots+Z_k^2\,}$$

So **chi-squared with $k$ df is a sum of $k$ squared independent standard normals.**

| Quantity | Value | Condition / note |
|---|---|---|
| Support | $[0,\infty)$ | sum of squares ⇒ non-negative |
| Shape | right-skewed; → symmetric/normal as $k\uparrow$ | skewness $=\sqrt{8/k}\to0$ |
| Mean | $E[\chi^2_k]=k$ | |
| Variance | $\mathrm{Var}(\chi^2_k)=2k$ | |
| Mode | $\max(k-2,\ 0)$ | |
| **Additivity** | $\chi^2_{k_1}+\chi^2_{k_2}=\chi^2_{k_1+k_2}$ | **independent** summands |
| **Single square** | $Z^2\sim\chi^2_1$ | special case $k=1$ |

**Derivation of mean & variance (clean and GATE-relevant):** Using $E[Z^2]=\mathrm{Var}(Z)+(E[Z])^2=1+0=1$,
$$E[\chi^2_k]=\sum_{i=1}^k E[Z_i^2]=k.$$
Using the fourth moment of the standard normal $E[Z^4]=3$, so $\mathrm{Var}(Z^2)=E[Z^4]-(E[Z^2])^2=3-1=2$, and independence:
$$\mathrm{Var}(\chi^2_k)=\sum_{i=1}^k \mathrm{Var}(Z_i^2)=2k.$$

**The application that makes it matter — sample variance.** For an i.i.d. $N(\mu,\sigma^2)$ sample,
$$\frac{(n-1)S^2}{\sigma^2}\sim \chi^2_{n-1},\qquad\text{while}\qquad \sum_{i=1}^n\frac{(X_i-\mu)^2}{\sigma^2}\sim \chi^2_{n}.$$
**Why one df is lost ($n-1$, not $n$):** when you use the *true* mean $\mu$ you have $n$ independent squared normals → $\chi^2_n$. When you use the *sample* mean $\bar X$, the deviations satisfy the constraint $\sum_i (X_i-\bar X)=0$; once $n-1$ deviations are known the last is forced, so only $n-1$ are "free" → $\chi^2_{n-1}$. **Estimating one parameter from the data costs one degree of freedom.**

**Two distinct GATE uses of $\chi^2$ (don't confuse them):**
1. **Distribution of (scaled) sample variance** — as above; basis of CIs/tests for $\sigma^2$.
2. **The $\chi^2$ test statistic** $\displaystyle\sum\frac{(O_i-E_i)^2}{E_i}$ for goodness-of-fit and independence — *approximately* $\chi^2$ (details in Topic 16). Note both forms are "sums of squared standardized quantities," which is *why* both lead to $\chi^2$.

**Neat bridges (worth a line):**
- $\chi^2_2$ is exactly an **exponential with mean 2** (its density is $\tfrac12 e^{-x/2}$ — recognise the Exp$(\lambda=\tfrac12)$ from Topic 12). Check: mean $=2=k$, var $=4=2k$. ✓
- $\chi^2$ is a special case of the **gamma** distribution (shape $k/2$, scale $2$) — *gamma itself is outside the DA syllabus*, mentioned only for context.

*PDF (for reference only — GATE does not ask you to use it; numeric work uses tables):* $f(x)=\dfrac{1}{2^{k/2}\Gamma(k/2)}\,x^{k/2-1}e^{-x/2},\ x>0.$

---

### 4C. $t$-distribution $\;t_k$ ($k$ degrees of freedom)

**Definition (know this cold):**
$$\boxed{\,\text{If } Z\sim N(0,1) \text{ and } V\sim \chi^2_k \text{ are independent, then } \ T:=\dfrac{Z}{\sqrt{V/k}}\sim t_k\,}$$

A standard normal divided by the square root of an **independent** chi-squared over its df. The $\chi^2$ in the denominator is the *estimated-variance randomness*.

| Quantity | Value | Condition / note |
|---|---|---|
| Support | $(-\infty,\infty)$ | |
| Shape | symmetric about $0$, bell-shaped, **heavier tails** than $N(0,1)$ | |
| Mean | $0$ | for $k>1$ (undefined for $k=1$) |
| Variance | $\dfrac{k}{k-2}$ | for $k>2$ ($>1$, so wider than normal); infinite for $1<k\le2$; undefined for $k=1$ |
| Limit | $t_k\to N(0,1)$ as $k\to\infty$ | tails thin out as df grows |
| Special case | $t_1$ = **Cauchy** (no mean, no variance) | |

**The application — the $t$-statistic (σ unknown).** For an i.i.d. $N(\mu,\sigma^2)$ sample,
$$T=\frac{\bar X-\mu}{S/\sqrt n}\sim t_{n-1}.$$
**Why this is a $t$ — the construction made explicit** (this derivation *is* the bridge between 4B and 4C):
$$T=\frac{\bar X-\mu}{S/\sqrt n}=\frac{\dfrac{\bar X-\mu}{\sigma/\sqrt n}}{\dfrac{S}{\sigma}}=\frac{Z}{\sqrt{S^2/\sigma^2}}=\frac{Z}{\sqrt{\dfrac{(n-1)S^2/\sigma^2}{\,n-1\,}}}=\frac{Z}{\sqrt{V/(n-1)}},$$
where $Z=\frac{\bar X-\mu}{\sigma/\sqrt n}\sim N(0,1)$, $V=\frac{(n-1)S^2}{\sigma^2}\sim\chi^2_{n-1}$, and $Z\perp V$ (the independence theorem of 4A). Hence $T\sim t_{n-1}$. ∎

> **$z$ vs $t$ — the decision rule (a GATE staple):**
> - **σ known** → use $Z=\frac{\bar X-\mu}{\sigma/\sqrt n}\sim N(0,1)$.
> - **σ unknown** (estimate with $S$) → use $T=\frac{\bar X-\mu}{S/\sqrt n}\sim t_{n-1}$.
> For large $n$ the two coincide (since $t_{n-1}\to N(0,1)$), but the *exact* small-sample answer for unknown σ is always $t$.

*PDF (reference only — not examinable to use):* $f(t)=\dfrac{\Gamma\!\big(\frac{k+1}{2}\big)}{\sqrt{k\pi}\,\Gamma\!\big(\frac{k}{2}\big)}\Big(1+\dfrac{t^2}{k}\Big)^{-(k+1)/2}.$

---

### 4D. Degrees of freedom — the one concept tying it together

**Degrees of freedom (df)** = the number of independent quantities free to vary after imposing the constraints used to compute a statistic. **Each parameter you estimate from the data removes one df.**

| Situation | df | Reason |
|---|---|---|
| $\sum (X_i-\mu)^2/\sigma^2$ (true mean) | $n$ | $n$ independent squares |
| $\sum (X_i-\bar X)^2/\sigma^2$ (sample mean) | $n-1$ | constraint $\sum(X_i-\bar X)=0$ |
| One-sample $t$ for the mean | $n-1$ | $S$ estimates σ using $\bar X$ |
| $\chi^2$ goodness-of-fit (forward, Topic 16) | $c-1-m$ | $c$ categories, $-1$ for total, $-m$ for $m$ estimated parameters |
| $\chi^2$ test of independence ($r\times c$ table) | $(r-1)(c-1)$ | row/column totals fixed |

---

### 4E. One-glance comparison (revision gold)

| | Chi-Squared $\chi^2_k$ | $t$-distribution $t_k$ |
|---|---|---|
| **Built from $Z$** | $Z_1^2+\cdots+Z_k^2$ | $\dfrac{Z}{\sqrt{\chi^2_k/k}}$ |
| **Support** | $[0,\infty)$ | $(-\infty,\infty)$ |
| **Shape** | right-skewed (→ normal as $k\uparrow$) | symmetric, heavy-tailed (→ $N(0,1)$ as $k\uparrow$) |
| **Mean** | $k$ | $0$ ($k>1$) |
| **Variance** | $2k$ | $\dfrac{k}{k-2}$ ($k>2$) |
| **Models** | distribution of sample **variance**; GoF/independence test | distribution of standardized **mean** when σ unknown |
| **Used in (Topic 16)** | $\chi^2$-test | $t$-test |

---

## 5. Worked problems (GATE-style)

### Problem 1 — Chi-squared mean, variance & additivity (NAT)
Let $V\sim\chi^2_{15}$. (i) Find $E[V]$ and $\mathrm{Var}(V)$. (ii) If $V=V_1+V_2$ with $V_1\sim\chi^2_{6}$ and $V_2$ an independent chi-squared, find the df of $V_2$.

**Solution.** (i) $E[V]=k=15$, $\mathrm{Var}(V)=2k=30$.
(ii) By additivity $\chi^2_{6}+\chi^2_{d}=\chi^2_{6+d}=\chi^2_{15}\Rightarrow d=9$.
**Answers: $15$, $30$, and $\text{df}(V_2)=9$.**

---

### Problem 2 — Sample variance & the df loss (NAT trap)
A random sample of size $n=16$ is drawn from $N(\mu,\sigma^2)$. State the distribution (with mean and variance) of $W=\dfrac{(n-1)S^2}{\sigma^2}$. Also state the distribution of $U=\displaystyle\sum_{i=1}^{16}\dfrac{(X_i-\mu)^2}{\sigma^2}$.

**Solution.** Using the sample mean: $W\sim\chi^2_{15}$, with mean $15$ and variance $30$.
Using the *true* mean $\mu$: $U\sim\chi^2_{16}$.
**Answers: $W\sim\chi^2_{15}$ (mean 15, var 30); $U\sim\chi^2_{16}$.** *Trap:* writing $\chi^2_{16}$ for $W$ — you lose one df because $\bar X$ (not $\mu$) is used.

---

### Problem 3 — Identify the statistic's distribution (MCQ)
A sample of size $25$ is drawn from a normal population with **unknown** variance. The statistic $\dfrac{\bar X-\mu}{S/\sqrt n}$ follows:
(A) $N(0,1)$ (B) $t_{24}$ (C) $t_{25}$ (D) $\chi^2_{24}$

**Solution.** σ unknown, estimated by $S$ ⇒ $t$ with $n-1=24$ df.
**Answer: (B) $t_{24}$.** *Trap:* (A) is for σ *known*; (C) forgets the $-1$.

---

### Problem 4 — Variance of a $t$ (NAT)
Find the variance of the $t$-distribution with $5$ degrees of freedom.

**Solution.** $\mathrm{Var}(t_k)=\dfrac{k}{k-2}=\dfrac{5}{5-2}=\dfrac{5}{3}\approx1.667.$
**Answer: $5/3\approx1.667$.** *(It exceeds $1$, reflecting the heavier-than-normal tails; as $k\to\infty$ it would shrink to $1$.)*

---

### Problem 5 — Construction identification (MCQ)
$Z\sim N(0,1)$ and $V\sim\chi^2_{12}$ are independent. The distribution of $T=\dfrac{Z}{\sqrt{V/12}}$ is:
(A) $\chi^2_{12}$ (B) $t_{12}$ (C) $N(0,1)$ (D) $t_{13}$

**Solution.** This is exactly the definition of the $t$ with $k=12$ df.
**Answer: (B) $t_{12}$.**

---

### Problem 6 — Degrees of freedom for $\chi^2$ tests (NAT, forward-looking but in scope)
(i) A six-sided die is tested for fairness by a goodness-of-fit test (6 outcome categories, no parameters estimated from the data). State the df.
(ii) A $3\times4$ contingency table is tested for independence of the two attributes. State the df.

**Solution.** (i) $\text{df}=c-1=6-1=5$. (ii) $\text{df}=(r-1)(c-1)=(3-1)(4-1)=6$.
**Answers: $5$ and $6$.**

---

### Problem 7 — Properties (MSQ)
Which of the following are **correct**?
(A) A chi-squared random variable is always non-negative.
(B) The $t$-distribution is symmetric about $0$.
(C) As the degrees of freedom increase, $t_k$ approaches $N(0,1)$.
(D) The mean of $\chi^2_k$ equals $2k$.

**Solution.**
- (A) **True** — it is a sum of squares, so $\ge0$ with support $[0,\infty)$.
- (B) **True** — built from the symmetric $Z$ in the numerator; symmetric about $0$.
- (C) **True** — heavier tails thin out; $t_\infty=N(0,1)$.
- (D) **False** — the **mean is $k$**; $2k$ is the **variance**.

**Answer: (A), (B), (C).** *(D) swaps mean and variance — a classic trap.*

---

## 6. Common GATE question types on this topic

1. **State mean/variance** of $\chi^2_k$ ($k$, $2k$) or $t_k$ ($0$, $\tfrac{k}{k-2}$), with conditions.
2. **Use additivity** of independent chi-squareds; recognise $Z^2\sim\chi^2_1$.
3. **Identify the sampling distribution** of a given statistic — $\bar X$, $\frac{(n-1)S^2}{\sigma^2}$, the $z$- vs $t$-statistic.
4. **Get the degrees of freedom right** — $n-1$ for one-sample mean/variance; $c-1-m$ and $(r-1)(c-1)$ for $\chi^2$ tests.
5. **Choose $z$ or $t$** based on whether σ is known (a recurring conceptual MCQ).
6. **Recognise the constructions** $\chi^2_k=\sum Z_i^2$ and $t_k=\frac{Z}{\sqrt{\chi^2_k/k}}$ from a worded description.
7. **Limiting behaviour** — $t_k\to N(0,1)$; $\chi^2_k$ becomes symmetric as $k$ grows.
8. **True/false on properties** — symmetry, non-negativity, heavy tails, mean/variance mix-ups.

---

## 7. Traps, pitfalls & common mistakes

- **Degrees of freedom off by one.** One-sample mean/variance uses $n-1$, not $n$ — you "spend" one df estimating $\mu$ with $\bar X$. (Use $n$ *only* when the true $\mu$ is given.)
- **Swapping the $\chi^2$ mean and variance.** Mean $=k$, **variance $=2k$**. Don't write mean $=2k$.
- **Using $z$ when σ is unknown.** If you estimate σ by $S$, the exact distribution is $t_{n-1}$, not $N(0,1)$ (they only merge for large $n$).
- **Thinking $t$ is just "a slightly different normal."** It has **fatter tails** and **variance $\frac{k}{k-2}>1$**; for small df the difference is large.
- **$t$ moments at small df.** Mean is undefined at $k=1$ (Cauchy); variance is infinite for $1<k\le2$ and undefined at $k=1$. State the condition $k>2$ before quoting $\frac{k}{k-2}$.
- **Forgetting the independence requirement.** $\chi^2$ additivity needs **independent** summands; the $t$ construction needs $Z\perp\chi^2$ (true for normal samples because $\bar X\perp S^2$).
- **Expecting $\chi^2$ to be symmetric.** It is **right-skewed**; only approaches symmetry as $k\to\infty$. Its support is $[0,\infty)$, so it has no left tail.
- **Confusing the two roles of $\chi^2$.** "Distribution of $S^2$" (df $=n-1$) is *different* from "the $\chi^2$ test statistic $\sum\frac{(O-E)^2}{E}$" (df from categories/table) — same distribution family, different df rules.
- **Reading the wrong tail of the table.** $\chi^2$ tables and $t$ tables are usually indexed by *upper-tail* area and by df — pick the right df *and* the right tail; for $t$, use symmetry for the lower tail.
- **Treating $S$ as a constant.** The whole reason $t$ exists is that $S$ is **random**; never standardize with $S$ and then call the result normal.

---

## 8. Connection forward

This topic hands the inference section its tools:

- **Topic 14 (Central Limit Theorem):** justifies the approximate normality of $\bar X$ for *non-normal* populations, which is what lets us deploy $z$/$t$ machinery beyond strictly normal data. (It also explains why $\chi^2_k$ and $t_k$ themselves approach the normal for large $k$.)
- **Topic 15 (Confidence Intervals):** the **$t$-distribution** gives the interval for $\mu$ when σ is unknown — $\bar X\pm t_{n-1,\,\alpha/2}\,\frac{S}{\sqrt n}$; the **$\chi^2$-distribution** gives the interval for $\sigma^2$ — $\Big(\frac{(n-1)S^2}{\chi^2_{\text{upper}}},\frac{(n-1)S^2}{\chi^2_{\text{lower}}}\Big)$.
- **Topic 16 (Hypothesis Testing):** the **$t$-test** uses $T=\frac{\bar X-\mu_0}{S/\sqrt n}\sim t_{n-1}$; the **$\chi^2$-test** uses $\sum\frac{(O-E)^2}{E}$ against $\chi^2$ critical values, with the df rules from §4D.

> *(Per your roadmap, the next topic starts a new chat with the **Central Limit Theorem**, which opens the Sampling Theory & Statistical Inference section.)*

---

## 9. One-line cheat-sheet summary

**$\chi^2_k=\sum_{i=1}^k Z_i^2$:** support $[0,\infty)$, right-skewed, **mean $k$, variance $2k$**, additive, $Z^2\sim\chi^2_1$; sample variance gives $\frac{(n-1)S^2}{\sigma^2}\sim\chi^2_{n-1}$ (lose 1 df for $\bar X$). **$t_k=\dfrac{Z}{\sqrt{\chi^2_k/k}}$:** symmetric, heavy-tailed, **mean $0$ ($k>1$), variance $\frac{k}{k-2}$ ($k>2$)**, $\to N(0,1)$ as $k\to\infty$; use $T=\frac{\bar X-\mu}{S/\sqrt n}\sim t_{n-1}$ when **σ is unknown** (else use $z$).

---

### ➡️ Next topic (new chat / new section)
**Topic 14 — Central Limit Theorem**, opening *Sampling Theory & Statistical Inference*. Start the new chat when ready; it will explain why $\bar X$ is approximately normal and tie together the $z$/$t$/$\chi^2$ machinery you've now assembled.

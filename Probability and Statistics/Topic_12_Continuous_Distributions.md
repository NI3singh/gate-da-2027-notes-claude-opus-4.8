# Topic 12 — Continuous Distributions (Uniform, Exponential, Normal & Standard Normal)

> **File this under:** Probability & Statistics → *Probability Distributions* (continuous half).

---

## 0. Topic header & syllabus map

- **Topic:** The continuous "named" distributions GATE DA tests: **Continuous Uniform, Exponential, Normal, Standard Normal.**
- **Subject:** Probability and Statistics (GATE Data Science & AI).
- **Where it sits:** Second half of the Probability Distributions section. It comes right after **Topic 11 (Discrete Distributions)** — these are the continuous mirror images of those — and it is the **foundation for Topic 13 (Sampling Distributions: $t$ & $\chi^2$)**, which are *built directly out of the normal*. Practically, the **normal** is the most heavily tested object in the entire GATE statistics syllabus.

---

## 1. Why this topic exists (motivation + connection)

Topic 11 modeled things you can **count**: heads, calls, defectives. But a huge fraction of real quantities are not countable lists — they are **measurements on a continuum**: a waiting time, a length, a voltage, a measurement error, a test score. For these, the machinery of Topic 7 (PDF & CDF) applies: probability is an **area under a density curve**, and $P(X=\text{exact value})=0$. This topic gives the four standard densities everyone reaches for.

- **What the previous topic couldn't do:** A PMF puts positive probability on individual points; that's wrong for continuous quantities. The **discrete uniform** could model "pick a random integer $1$–$6$" but not "pick a random real in $[0,1]$." The **Poisson** could count how many events happen in an hour but had no way to express *how long you wait* until the next one — that gap is exactly what the **exponential** fills.
- **What the next topic builds on it:** Sampling distributions (Topic 13) are assembled from the normal — **$\chi^2$ is a sum of squared standard normals**, and **$t$ is a standard normal divided by a (scaled) $\chi^2$**. The whole inference pipeline (CLT → confidence intervals → $z$/$t$/$\chi^2$ tests, Topics 14–16) runs on the normal. Master the normal here and the rest of the syllabus is bookkeeping.

**The four as one connected story (and a bridge back to Topic 11):**

> **Continuous Uniform** = "a random real number in $[a,b]$, all equally likely" — the flat-density analogue of the discrete uniform. →
> **Exponential** = the *waiting time* between events of a **Poisson process**. Same rate $\lambda$ as the Poisson from Topic 11: **Poisson counts events per interval; Exponential measures time between events.** It is *memoryless*. →
> **Normal** = the bell curve that sums and averages of almost anything converge to (CLT) — the universal model for "natural variation + measurement error." The **binomial → normal** as $n$ grows (De Moivre–Laplace). →
> **Standard Normal** $N(0,1)$ = the normal rescaled to $\mu=0,\sigma=1$; the single reference scale we **standardize** every normal problem onto so one $\Phi$-table answers them all.

---

## 2. Prerequisites

- **Continuous RV, PDF & CDF** (Topic 7): $\int_{-\infty}^{\infty} f(x)\,dx=1$; $P(a\le X\le b)=\int_a^b f$; $F(x)=\int_{-\infty}^x f$; $f=F'$.
- **Expectation & variance for continuous RVs** (Topic 9): $E[X]=\int x f(x)\,dx$, $\mathrm{Var}(X)=E[X^2]-(E[X])^2$; **linearity** and, for independent variables, **variances add**.
- **Poisson distribution** (Topic 11): you'll connect it to the exponential.
- **Basic calculus:** integration by parts; $\int_0^\infty e^{-\lambda x}dx=\tfrac1\lambda$; $\ln 2\approx0.693$.
- **Key contrast to remember:** for a *continuous* RV, $P(X=c)=0$, so $P(X\le x)=P(X<x)$ — **endpoints don't change probabilities** (the opposite of the discrete case).

---

## 3. Core idea — intuition first

Three questions identify the right continuous model:

1. **A random real number in an interval, every value equally likely?** (a bus arrives "uniformly" in the next 10 min; a rounding error in $[-0.5,0.5]$) → **Continuous Uniform**. The density is a flat rectangle of height $\tfrac{1}{b-a}$; probability of a sub-interval is just *its length ÷ total length*.

2. **Time you wait for the next event, or a lifetime, where the process has no memory?** (time to next phone call, until a bulb fails, until a radioactive decay) → **Exponential**. The density starts highest at $0$ and decays; long waits are possible but increasingly unlikely. Its defining feature: **memorylessness** — a used component that hasn't failed is "as good as new." This is the continuous twin of the Poisson: if events arrive at rate $\lambda$ (Poisson), the *gaps* between them are Exponential($\lambda$).

3. **A quantity that is the sum/average of many small independent effects, clustering symmetrically around a centre?** (heights, exam scores, measurement noise, sample means) → **Normal**. Bell-shaped, symmetric about $\mu$, spread set by $\sigma$. By the CLT (Topic 14), averages of almost *anything* look normal — which is why it's everywhere.

**Why we "standardize":** There is **no elementary antiderivative** of $e^{-z^2/2}$, so we cannot write the normal CDF in closed form. The workaround: tabulate the CDF $\Phi(z)$ once for the **standard** normal $N(0,1)$, and convert every other normal to it with
$$Z=\frac{X-\mu}{\sigma}.$$
One table, every normal question. Standardizing is *the* master skill of this topic.

**Quick analogy for memorylessness:** A discrete "coin until first head" forgets past failures (geometric). The exponential is its continuous version: however long you've already waited, the *remaining* wait has the same distribution as a fresh start.

---

## 4. Formal treatment

Throughout, "PDF" is the density; probabilities are areas (integrals) under it.

### 4A. Continuous Uniform $\;X\sim U(a,b)$

$$\boxed{\,f(x)=\dfrac{1}{b-a},\quad a\le x\le b\quad(\text{and }0\text{ otherwise})\,}$$

**Conditions:** $a<b$; the density is *constant* on $[a,b]$ (all values equally likely).

| Quantity | Formula | Note |
|---|---|---|
| CDF | $F(x)=\dfrac{x-a}{b-a}$ for $a\le x\le b$ (0 below $a$, 1 above $b$) | a straight ramp |
| Mean | $\mu=\dfrac{a+b}{2}$ | the midpoint |
| Variance | $\sigma^2=\dfrac{(b-a)^2}{12}$ | depends only on the **width**, not location |
| Interval probability | $P(c\le X\le d)=\dfrac{d-c}{b-a}$ for $[c,d]\subseteq[a,b]$ | length ÷ total length |

**Short derivation of the variance:**
$$E[X]=\int_a^b \frac{x}{b-a}\,dx=\frac{a+b}{2},\quad E[X^2]=\int_a^b \frac{x^2}{b-a}\,dx=\frac{a^2+ab+b^2}{3},$$
$$\mathrm{Var}=\frac{a^2+ab+b^2}{3}-\Big(\frac{a+b}{2}\Big)^2=\frac{(b-a)^2}{12}.$$

> **Mirror to Topic 11:** discrete uniform on $\{1,\dots,n\}$ had variance $\frac{n^2-1}{12}$; continuous uniform on $[a,b]$ has $\frac{(b-a)^2}{12}$. Same "$/12$" family.

---

### 4B. Exponential $\;X\sim \mathrm{Exp}(\lambda)$

$$\boxed{\,f(x)=\lambda e^{-\lambda x},\quad x\ge 0\quad(\lambda>0)\,}$$

**Conditions:** $x\ge 0$; constant rate $\lambda$. Models waiting times / lifetimes of a memoryless process.

| Quantity | Formula | Note |
|---|---|---|
| CDF | $F(x)=1-e^{-\lambda x},\ x\ge0$ | |
| Survival ("more than $x$") | $P(X>x)=e^{-\lambda x}$ | use this constantly |
| Mean | $E[X]=\dfrac{1}{\lambda}$ | |
| Variance | $\mathrm{Var}(X)=\dfrac{1}{\lambda^2}$ | so **SD $=\frac1\lambda=$ mean** |
| Median | $\dfrac{\ln 2}{\lambda}\approx\dfrac{0.693}{\lambda}$ | $<$ mean (right-skewed) |
| MGF (optional) | $\dfrac{\lambda}{\lambda-t}$ for $t<\lambda$ | |

> ⚠️ **Parametrization trap.** Two conventions exist:
> - **Rate** form (GATE default): $f(x)=\lambda e^{-\lambda x}$, mean $=1/\lambda$.
> - **Scale/mean** form: $f(x)=\frac{1}{\theta}e^{-x/\theta}$, mean $=\theta$.
>
> If a problem says "mean lifetime $=1000$ hours," then $\theta=1000$ and $\lambda=1/1000$. Decide which symbol the question is using **before** plugging in.

**Mean by integration by parts** (sometimes asked):
$$E[X]=\int_0^\infty x\,\lambda e^{-\lambda x}\,dx=\Big[-x e^{-\lambda x}\Big]_0^\infty+\int_0^\infty e^{-\lambda x}\,dx=0+\frac1\lambda=\frac1\lambda.$$

**Memoryless property (the signature — and the only continuous distribution with it):**
$$\boxed{\,P(X>s+t\mid X>s)=P(X>t)\,}$$
**Proof:** $\displaystyle P(X>s+t\mid X>s)=\frac{P(X>s+t)}{P(X>s)}=\frac{e^{-\lambda(s+t)}}{e^{-\lambda s}}=e^{-\lambda t}=P(X>t).$ ∎

> **Bridge to Topic 11 (test favourite):** Events of a **Poisson process** with rate $\lambda$ ⇔ inter-event **waiting times are Exponential($\lambda$)** with the *same* $\lambda$.
> - "How many events in time $t$?" → Poisson with mean $\lambda t$.
> - "How long until the next event?" → Exponential with rate $\lambda$, so $P(\text{wait}>t)=e^{-\lambda t}$.
> Notice $P(\text{0 events in time }t)=e^{-\lambda t}\cdot\frac{(\lambda t)^0}{0!}=e^{-\lambda t}=P(\text{wait}>t)$ — the same number, two viewpoints.

---

### 4C. Normal $\;X\sim N(\mu,\sigma^2)$

$$\boxed{\,f(x)=\dfrac{1}{\sigma\sqrt{2\pi}}\,\exp\!\Big(-\dfrac{(x-\mu)^2}{2\sigma^2}\Big),\quad -\infty<x<\infty\,}$$

**Conditions:** defined on all reals; $\sigma>0$. Parameters are literally the **mean $\mu$** and **variance $\sigma^2$**.

| Property | Statement |
|---|---|
| Shape | symmetric bell about $\mu$; **mean = median = mode $=\mu$** |
| Spread | inflection points at $\mu\pm\sigma$ |
| Mean / Variance | $E[X]=\mu,\ \mathrm{Var}(X)=\sigma^2$ |
| **Linear transform** | $aX+b\sim N(a\mu+b,\ a^2\sigma^2)$ for constants $a\ne0,b$ — *still normal* |
| **Sum (independent)** | $N(\mu_1,\sigma_1^2)+N(\mu_2,\sigma_2^2)=N(\mu_1+\mu_2,\ \sigma_1^2+\sigma_2^2)$ |
| **Difference (independent)** | $X-Y\sim N(\mu_1-\mu_2,\ \sigma_1^2+\sigma_2^2)$ — variances **add**, never subtract |
| MGF (optional) | $\exp\!\big(\mu t+\tfrac12\sigma^2 t^2\big)$ |

*Derivation of $E[X]=\mu$, $\mathrm{Var}=\sigma^2$ from the integral is **skipped** — it needs the Gaussian integral $\int e^{-z^2/2}dz=\sqrt{2\pi}$, which GATE never asks you to perform. Just use the results.*

**Empirical (68–95–99.7) rule** — worth memorizing for quick estimates:
$$P(\mu\!-\!\sigma<X<\mu\!+\!\sigma)\approx0.68,\quad P(\mu\!-\!2\sigma<\cdots<\mu\!+\!2\sigma)\approx0.95,\quad P(\mu\!-\!3\sigma<\cdots)\approx0.997.$$

---

### 4D. Standard Normal $\;Z\sim N(0,1)$ and standardization

The normal with $\mu=0,\ \sigma=1$. Its PDF is $\phi(z)=\frac{1}{\sqrt{2\pi}}e^{-z^2/2}$ and its CDF is written $\Phi(z)$ (no closed form — read from a table).

**Standardization** (the workhorse):
$$Z=\frac{X-\mu}{\sigma}\ \sim N(0,1),\qquad P(a<X<b)=\Phi\!\Big(\frac{b-\mu}{\sigma}\Big)-\Phi\!\Big(\frac{a-\mu}{\sigma}\Big).$$
*Why it works:* by linearity $E[Z]=\frac{E[X]-\mu}{\sigma}=0$ and $\mathrm{Var}(Z)=\frac{1}{\sigma^2}\mathrm{Var}(X)=1$; and a linear function of a normal is normal.

**Symmetry identity (used in almost every problem):**
$$\boxed{\,\Phi(-z)=1-\Phi(z)\,}\qquad\Rightarrow\qquad P(Z>z)=1-\Phi(z),\quad P(|Z|<z)=2\Phi(z)-1.$$

**Mini $\Phi$-table (memorize the starred rows — they reappear in CIs & tests):**

| $z$ | $\Phi(z)$ | | $z$ | $\Phi(z)$ |
|---|---|---|---|---|
| 0.00 | 0.5000 | | 1.645 ★ | 0.9500 |
| 1.00 | 0.8413 | | 1.96 ★ | 0.9750 |
| 1.28 | 0.8997 | | 2.00 | 0.9772 |
| 1.50 | 0.9332 | | 2.33 | 0.9901 |
| 1.90 | 0.9713 | | 2.576 ★ | 0.9950 |

The starred values give the famous two-sided coverages: $P(|Z|<1.645)=0.90$, $P(|Z|<1.96)=0.95$, $P(|Z|<2.576)=0.99$ — the exact numbers you'll quote for **90%/95%/99% confidence intervals** (Topic 15).

---

### 4E. Normal approximations (connect back to Topic 11, forward to Topic 14)

**Normal approximation to the Binomial (De Moivre–Laplace).** If $X\sim\mathrm{Bin}(n,p)$ with $n$ large,
$$X\ \approx\ N(np,\ npq).$$
**Validity rule of thumb:** $np\ge5$ **and** $nq\ge5$ (use $\ge10$ to be safe).
**Continuity correction** (because we approximate a *discrete* count by a *continuous* curve — add/subtract $0.5$):
$$P(X\le k)\approx\Phi\!\Big(\frac{k+0.5-np}{\sqrt{npq}}\Big),\qquad P(X\ge k)\approx 1-\Phi\!\Big(\frac{k-0.5-np}{\sqrt{npq}}\Big).$$

**Normal approximation to the Poisson.** For large $\lambda$ (rule of thumb $\lambda\ge10$): $X\sim\mathrm{Poi}(\lambda)\approx N(\lambda,\lambda)$, again with continuity correction.

(The full justification is the CLT in Topic 14; this is its first concrete appearance.)

---

### 4F. One-glance comparison table (revision gold)

| | Continuous Uniform $U(a,b)$ | Exponential$(\lambda)$ | Normal $N(\mu,\sigma^2)$ | Standard Normal $N(0,1)$ |
|---|---|---|---|---|
| **Models** | random real, equally likely | waiting time / lifetime | natural variation, sums/averages | reference scale |
| **Support** | $[a,b]$ | $[0,\infty)$ | $(-\infty,\infty)$ | $(-\infty,\infty)$ |
| **PDF** | $\frac{1}{b-a}$ | $\lambda e^{-\lambda x}$ | $\frac{1}{\sigma\sqrt{2\pi}}e^{-(x-\mu)^2/2\sigma^2}$ | $\frac{1}{\sqrt{2\pi}}e^{-z^2/2}$ |
| **CDF** | $\frac{x-a}{b-a}$ | $1-e^{-\lambda x}$ | no closed form → $\Phi(\frac{x-\mu}{\sigma})$ | $\Phi(z)$ (table) |
| **Mean** | $\frac{a+b}{2}$ | $\frac{1}{\lambda}$ | $\mu$ | $0$ |
| **Variance** | $\frac{(b-a)^2}{12}$ | $\frac{1}{\lambda^2}$ | $\sigma^2$ | $1$ |
| **Signature** | flat density | memoryless; SD = mean | symmetric bell | $\Phi(-z)=1-\Phi(z)$ |

---

## 5. Worked problems (GATE-style)

### Problem 1 — Continuous Uniform (NAT)
$X\sim U(2,8)$. Find (i) $P(3<X<5)$ and (ii) $\mathrm{Var}(X)$.

**Solution.** Width $b-a=6$.
(i) $P(3<X<5)=\dfrac{5-3}{6}=\dfrac{1}{3}\approx0.333.$
(ii) $\mathrm{Var}=\dfrac{(8-2)^2}{12}=\dfrac{36}{12}=3.$
**Answers: $1/3$ and $3$.**

---

### Problem 2 — Uniform, conditional probability (NAT trap)
$X\sim U(0,10)$. Find $P(X>7\mid X>4)$.

**Solution.** $P(X>7\mid X>4)=\dfrac{P(X>7)}{P(X>4)}=\dfrac{3/10}{6/10}=\dfrac{1}{2}=0.5.$
**Answer: $0.5$.** *(For the uniform — unlike the exponential — the conditional probability does change with the conditioning event; the uniform is **not** memoryless.)*

---

### Problem 3 — Exponential, Poisson bridge (NAT)
Phone calls arrive as a Poisson process at **$4$ per hour**. Find the probability that the waiting time until the next call **exceeds $30$ minutes**.

**Solution.** Inter-arrival time $T\sim\mathrm{Exp}(\lambda)$ with $\lambda=4$ per hour. $30$ min $=0.5$ h.
$$P(T>0.5)=e^{-\lambda(0.5)}=e^{-2}\approx0.135.$$
**Answer: $\approx0.135$.** (Identical to the Poisson statement "0 calls in 0.5 h" $=e^{-2}$.)

---

### Problem 4 — Exponential, memoryless (NAT trap)
The lifetime of a bulb is exponential with **mean $1000$ hours**. Given that it has already lasted $500$ hours, find the probability it lasts **at least another $1000$ hours**.

**Solution.** Mean $=1000\Rightarrow\lambda=1/1000$. By memorylessness the $500$ hours already survived are irrelevant:
$$P(X>1500\mid X>500)=P(X>1000)=e^{-1000/1000}=e^{-1}\approx0.368.$$
**Answer: $\approx0.368$.** *Trap:* computing $P(X>1500)/P(X>500)$ the long way is fine and gives the same $e^{-1}$ — but the slick route is to remember "exponential forgets."

---

### Problem 5 — Normal, standardization both tails (NAT)
$X\sim N(50,100)$ (so $\sigma=10$). Find $P(35<X<65)$.

**Solution.** Standardize: $z_1=\frac{35-50}{10}=-1.5$, $z_2=\frac{65-50}{10}=1.5$.
$$P(35<X<65)=\Phi(1.5)-\Phi(-1.5)=2\Phi(1.5)-1=2(0.9332)-1=0.8664.$$
**Answer: $\approx0.866$.**

---

### Problem 6 — Normal, reverse (find the cutoff) (NAT)
Scores are $N(100,15^2)$. Find the score $x$ exceeded by only the **top $5\%$**.

**Solution.** $P(X>x)=0.05\Rightarrow P(Z>z)=0.05\Rightarrow z=1.645$ (since $\Phi(1.645)=0.95$).
$$x=\mu+z\sigma=100+1.645(15)=100+24.675\approx124.7.$$
**Answer: $\approx124.7$.** *(Read the table "backwards": probability → $z$ → $x=\mu+z\sigma$.)*

---

### Problem 7 — Sum of independent normals (NAT)
$X\sim N(2,9)$ and $Y\sim N(3,16)$ are independent. Find $P(X+Y>10)$.

**Solution.** $X+Y\sim N(2+3,\ 9+16)=N(5,25)$, so $\sigma=5$.
$$P(X+Y>10)=P\!\Big(Z>\frac{10-5}{5}\Big)=P(Z>1)=1-0.8413=0.1587.$$
**Answer: $\approx0.159$.** *Trap:* variances add ($9+16=25$), **not** the standard deviations ($3+4=7$ would be wrong).

---

### Problem 8 — Normal approximation to Binomial (NAT, bridge + continuity correction)
A fair coin is tossed $100$ times. Approximate $P(\text{at least }60\text{ heads})$.

**Solution.** $X\sim\mathrm{Bin}(100,0.5)$: $np=50$, $npq=25$, $\sigma=5$. Check validity: $np=nq=50\ge5$ ✓.
With continuity correction, $P(X\ge60)=P(X>59.5)$:
$$\approx 1-\Phi\!\Big(\frac{59.5-50}{5}\Big)=1-\Phi(1.9)=1-0.9713=0.0287.$$
**Answer: $\approx0.029$.** *(Without the correction you'd get $1-\Phi(2)=0.0228$; the exact binomial is $\approx0.0284$, so the continuity correction is clearly the better estimate.)*

---

### Problem 9 — Conceptual relationships (MSQ)
Which of the following are **correct**?
(A) For $X\sim\mathrm{Exp}(\lambda)$, the mean equals the standard deviation.
(B) Standardizing **any** random variable via $Z=(X-\mu)/\sigma$ makes it follow $N(0,1)$.
(C) For any continuous random variable, $P(X=c)=0$ for every constant $c$.
(D) The exponential distribution is memoryless: $P(X>s+t\mid X>s)=P(X>t)$.

**Solution.**
- (A) **True** — both equal $1/\lambda$.
- (B) **False** — standardizing only forces mean $0$, SD $1$; it does **not** change the *shape*. The result is $N(0,1)$ **only if $X$ was already normal.**
- (C) **True** — point probabilities are zero for continuous RVs (area of a line).
- (D) **True** — proved in §4B; this is the exponential's defining property.

**Answer: (A), (C), (D).** *(B) is the classic misconception.*

---

## 6. Common GATE question types on this topic

1. **Uniform interval / conditional probability** — length ratios; note the uniform is *not* memoryless (Problems 1–2).
2. **Exponential survival & memoryless** — $P(X>x)=e^{-\lambda x}$; "given it survived $s$, lasts $t$ more" $=e^{-\lambda t}$ (Problems 3–4).
3. **Poisson ↔ Exponential conversion** — "rate per interval" ↔ "waiting time" (Problem 3).
4. **Normal standardization** — single tail, two tails, or $P(|X-\mu|<k\sigma)$ (Problem 5).
5. **Reverse normal** — given a probability/percentile, find the cutoff $x=\mu+z\sigma$ (Problem 6).
6. **Linear combinations of independent normals** — new mean/variance, then standardize (Problem 7).
7. **Normal approximation to Binomial/Poisson** — with continuity correction (Problem 8).
8. **Identify-the-distribution / true-false on properties** — mean=SD (exp), symmetry (normal), $P(X=c)=0$, memorylessness (Problem 9).
9. **Find the rate/parameter** from a given mean, median, or probability.

---

## 7. Traps, pitfalls & common mistakes

- **Standardizing makes it normal? No.** $Z=(X-\mu)/\sigma$ fixes mean and SD but **not shape**. Only a *normal* becomes $N(0,1)$.
- **SDs don't add.** For independent sums/differences, **variances** add: $\sigma^2=\sigma_1^2+\sigma_2^2$, then $\sigma=\sqrt{\sigma_1^2+\sigma_2^2}$ — never $\sigma_1+\sigma_2$. And for a *difference* you still **add** the variances.
- **Exponential parametrization.** "Mean $=m$" ⇒ $\lambda=1/m$. Don't plug the mean in where the rate belongs (or vice-versa).
- **Forgetting continuity correction** in normal-to-binomial problems — off by a noticeable amount; use $k\pm0.5$.
- **Variance of $N(\mu,\sigma^2)$ vs SD.** The second parameter is the **variance** $\sigma^2$. If told "$N(50,100)$," then $\sigma=10$, not $100$.
- **Endpoint fuss for continuous RVs.** $P(X\le x)=P(X<x)$; $\le$ vs $<$ never matters for a continuous distribution (it *does* for discrete — careful when switching between Topics 11 and 12).
- **Uniform variance uses the width.** $\frac{(b-a)^2}{12}$ — not $\frac{b^2-a^2}{12}$ and not location-dependent.
- **Reverse-normal sign errors.** "Top 5%" needs $z=+1.645$; "bottom 5%" needs $z=-1.645$. Draw the curve and shade.
- **Reading $\Phi$ vs upper tail.** Tables usually give $\Phi(z)=P(Z\le z)$; for $P(Z>z)$ take $1-\Phi(z)$, and use $\Phi(-z)=1-\Phi(z)$ for negatives.
- **Median ≠ mean for the exponential.** Median $=\frac{\ln2}{\lambda}\approx0.693\,\text{mean}$ — it's right-skewed, so don't use the mean as the 50th percentile.

---

## 8. Connection forward

This topic is the launchpad for all statistical inference:

- **Topic 13 (Sampling Distributions):** the **chi-squared** distribution is defined as a sum of squared independent standard normals $\big(\chi^2_k=\sum_{i=1}^k Z_i^2\big)$; the **$t$-distribution** is $T=\dfrac{Z}{\sqrt{\chi^2_k/k}}$ — both are *manufactured from the normal you just learned*.
- **Topic 14 (Central Limit Theorem):** formalizes why the normal is universal — sample means/sums of i.i.d. variables become approximately normal, which is exactly the binomial- and Poisson-to-normal approximations of §4E.
- **Topics 15–16 (Confidence intervals & hypothesis tests):** the starred $z$-values ($1.645,1.96,2.576$) become CI multipliers, and standardization becomes the **$z$-test** statistic; the exponential/Poisson rate estimation and the normal model feed the tests directly.

---

## 9. One-line cheat-sheet summary

**Uniform $U(a,b)$:** $f=\frac1{b-a}$, mean $\frac{a+b}2$, var $\frac{(b-a)^2}{12}$, prob = length ratio. **Exponential$(\lambda)$:** $f=\lambda e^{-\lambda x}$, $P(X>x)=e^{-\lambda x}$, mean = SD $=\frac1\lambda$, **memoryless**, = Poisson waiting time. **Normal $N(\mu,\sigma^2)$:** symmetric bell, $aX+b$ & independent sums stay normal (variances add). **Standard $N(0,1)$:** standardize $Z=\frac{X-\mu}\sigma$, use $\Phi$ with $\Phi(-z)=1-\Phi(z)$; remember $1.645/1.96/2.576$ for $90/95/99\%$.

---

### ➡️ Next topic (in order)
**Topic 13 — Sampling Distributions ($t$ and $\chi^2$).** Ask for it when ready; we'll build both directly out of the standard normal from this topic and set up everything needed for confidence intervals and hypothesis tests.

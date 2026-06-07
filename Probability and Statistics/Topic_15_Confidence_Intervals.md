# Topic 15 — Confidence Intervals (CI)

*Subject: Probability and Statistics · Section: Sampling Theory & Statistical Inference*

---

## 0. Topic header & syllabus map

- **Topic:** Confidence Intervals (interval estimation)
- **Subject:** Probability and Statistics (GATE DA)
- **Where it sits:** The **middle topic of the Statistical Inference block**. It comes immediately **after** the *Central Limit Theorem* (which supplies the sampling distribution of $\bar X$) and leads directly **into** *Hypothesis Testing (z/t/chi-squared tests)*. A two-sided confidence interval and a two-sided test are two views of the *same* calculation, so this topic is the natural bridge between estimation and testing.

---

## 1. Why this topic exists (motivation + connection)

A **point estimate** like "the sample mean is $70$" is a single number with **no statement of reliability**. Two different samples give two different means, so by itself $\bar X=70$ tells us nothing about how close $70$ is to the true $\mu$. We need to report an estimate **together with its uncertainty** — an *interval*, plus a number saying how trustworthy that interval is.

**What the previous topic set up.** The CLT gave us exactly the missing ingredient: the *distribution* of the estimator, $\bar X\approx\mathcal N(\mu,\sigma^2/n)$. That tells us $\bar X$ usually lands within a few standard errors of $\mu$. A **confidence interval is the CLT statement turned inside out**: instead of "$\bar X$ is probably near $\mu$," we rearrange algebra to say "$\mu$ is probably within $\bar X\pm\text{margin}$." Without the CLT (topic 14) and the $t$/$\chi^2$ sampling distributions (topic 13), there would be no critical value and no margin to write down.

**Forward link.** Hypothesis testing asks the *reverse* question: given a *claimed* value $\mu_0$, is our sample consistent with it? It turns out $\mu_0$ lies inside the $(1-\alpha)$ confidence interval **iff** a two-sided test does **not** reject $\mu_0$ at level $\alpha$. So mastering CIs is half of mastering the next topic.

---

## 2. Prerequisites

- **Central Limit Theorem** and the **standard error** $\sigma/\sqrt n$ (topic 14).
- **Standard normal** distribution and reading **critical values** $z_{\alpha/2}$ from the table.
- **t-distribution** and its **degrees of freedom**, and the **chi-squared distribution** (topic 13) — needed for the mean-with-unknown-$\sigma$ interval and the variance interval.
- **Sample mean $\bar X$, sample variance $s^2$, sample proportion $\hat p$** as estimators.
- Basic algebra of rearranging an inequality (the whole derivation is one rearrangement).

---

## 3. Core idea — intuition first

Imagine drawing many samples, each giving a $\bar X$, and around each $\bar X$ you draw a fixed-width "net" of $\pm$ a couple of standard errors. Because $\bar X$ lands near $\mu$ most of the time (CLT), **most of those nets will cover the true $\mu$**. If you size the net so that it covers $\mu$ in $95\%$ of repeated samples, you've built a **95% confidence interval**.

The subtle part is *what the 95% describes*. The true $\mu$ is a **fixed (unknown) constant** — it does not move. What's random is the **interval** (it shifts with each sample). So:

> **"95% confidence" is a property of the procedure**: in the long run, $95\%$ of the intervals it produces will trap $\mu$. It is **not** a probability statement about one already-computed interval — once you write down $(66.08,\,73.92)$, $\mu$ is either in it or not; there's no "95% chance" left.

Trade-off intuition (very testable): to be *more* confident you need a *wider* net (you'd say "I'm 99% sure $\mu$ is in this big range"); to get a *narrower*, more precise net at the same confidence you must collect *more data* (the net's half-width is $\propto 1/\sqrt n$).

---

## 4. Formal treatment

### 4.1 The universal template

Every confidence interval in this syllabus has the same shape:

$$\boxed{\ \text{estimate}\ \pm\ \underbrace{(\text{critical value})\times(\text{standard error})}_{\text{margin of error }E}\ }$$

$$\hat\theta\ \pm\ c\cdot\operatorname{SE}(\hat\theta).$$

The **confidence level** is $1-\alpha$ (e.g. $0.95$); $\alpha$ is the total tail probability left outside the interval, split as $\alpha/2$ in each tail for a two-sided CI.

### 4.2 Derivation by inverting the CLT (mean, $\sigma$ known)

From the CLT, $\dfrac{\bar X-\mu}{\sigma/\sqrt n}\approx Z\sim\mathcal N(0,1)$. By definition of the critical value $z_{\alpha/2}$ (the point with area $\alpha/2$ to its right):

$$P\!\left(-z_{\alpha/2}\le \frac{\bar X-\mu}{\sigma/\sqrt n}\le z_{\alpha/2}\right)=1-\alpha.$$

Multiply through by $\sigma/\sqrt n$ and rearrange to isolate $\mu$:

$$P\!\left(\bar X-z_{\alpha/2}\frac{\sigma}{\sqrt n}\ \le\ \mu\ \le\ \bar X+z_{\alpha/2}\frac{\sigma}{\sqrt n}\right)=1-\alpha.$$

The interval between those endpoints is the $(1-\alpha)$ CI for $\mu$. **GATE does expect you to be able to reproduce this inversion** — it's short and conceptual.

### 4.3 CI for a population mean $\mu$

**Case A — $\sigma$ known (or $n$ large): the z-interval.**
$$\bar X\ \pm\ z_{\alpha/2}\,\frac{\sigma}{\sqrt n}
\qquad\text{conditions: }\sigma\text{ known; population normal OR }n\text{ large (CLT).}$$

**Case B — $\sigma$ unknown, $n$ small, population $\approx$ normal: the t-interval.**
$$\bar X\ \pm\ t_{\alpha/2,\,n-1}\,\frac{s}{\sqrt n}
\qquad\text{conditions: }\sigma\text{ unknown (use sample SD }s\text{); population approximately normal; df}=n-1.$$

- $s$ is the **sample standard deviation**; the degrees of freedom are $n-1$.
- Because the $t$-distribution has heavier tails, $t_{\alpha/2,n-1}>z_{\alpha/2}$, so the **t-interval is always wider** than the z-interval for the same data — the price of not knowing $\sigma$.
- As $n\to\infty$, $t_{\alpha/2,n-1}\to z_{\alpha/2}$, so for **large $n$** you may use $z$ even with $s$ (the two agree). This is exactly the CLT showing up again.

**Decision rule (memorize):**

| Situation | Use |
|---|---|
| $\sigma$ known | **z** |
| $\sigma$ unknown, $n$ small, population normal | **t** with $n-1$ df |
| $\sigma$ unknown, $n$ large | **z** (with $s$) — t and z coincide |

### 4.4 CI for a population proportion $p$ (Wald interval)

For a sample of $n$ independent Bernoulli trials with $X$ successes, $\hat p=X/n$. By the CLT, $\hat p\approx\mathcal N\!\big(p,\,p(1-p)/n\big)$, so:
$$\hat p\ \pm\ z_{\alpha/2}\sqrt{\frac{\hat p(1-\hat p)}{n}}
\qquad\text{conditions: large }n,\ \text{specifically } n\hat p\ge5\ \text{and } n(1-\hat p)\ge5.$$

*(The true $p$ in the SE is replaced by $\hat p$; that's why this needs a large sample.)*

### 4.5 CI for a population variance $\sigma^2$ (chi-squared)

This is where the $\chi^2$ sampling distribution from topic 13 pays off. For a **normal** population, $\dfrac{(n-1)s^2}{\sigma^2}\sim\chi^2_{n-1}$, which gives:

$$\left(\ \frac{(n-1)s^2}{\chi^2_{\alpha/2,\,n-1}}\ ,\ \frac{(n-1)s^2}{\chi^2_{1-\alpha/2,\,n-1}}\ \right)
\qquad\text{condition: population is }\textbf{normal}\text{ (this CI is *not* robust to non-normality).}$$

- $\chi^2_{\alpha/2,n-1}$ is the **upper** critical value (area $\alpha/2$ to the right) and goes in the **denominator of the lower bound**; the smaller value $\chi^2_{1-\alpha/2,n-1}$ gives the upper bound.
- Because $\chi^2$ is **skewed (asymmetric)**, this interval is **not symmetric** around $s^2$ — don't write it as $s^2\pm\text{something}$. (For $\sigma$, take square roots of both endpoints.)

### 4.6 Margin of error & sample size

The half-width is the **margin of error** $E=c\cdot\operatorname{SE}$. Solving for $n$:

$$\textbf{Mean: }\quad E=z_{\alpha/2}\frac{\sigma}{\sqrt n}\ \Longrightarrow\ n=\left(\frac{z_{\alpha/2}\,\sigma}{E}\right)^{2}.$$

$$\textbf{Proportion: }\quad n=\frac{z_{\alpha/2}^{2}\,p(1-p)}{E^{2}}\ \xrightarrow[\ p\text{ unknown}\ ]{\ \text{worst case}\ } n=\frac{z_{\alpha/2}^{2}\,(0.25)}{E^{2}}.$$

**Always round $n$ UP** to the next integer (to *guarantee* margin $\le E$). For an unknown proportion, $p(1-p)$ is maximized at $p=0.5$ (value $0.25$), giving the safest (largest) sample size.

### 4.7 One-sided confidence bounds

Sometimes only an upper *or* lower bound matters (e.g., "with 95% confidence, mean defect rate is **at most** ___"). Then the entire $\alpha$ sits in **one** tail, so the critical value is $z_{\alpha}$ (not $z_{\alpha/2}$):

$$\text{Upper bound: }\ \mu\le \bar X+z_{\alpha}\frac{\sigma}{\sqrt n},
\qquad
\text{Lower bound: }\ \mu\ge \bar X-z_{\alpha}\frac{\sigma}{\sqrt n}.$$

E.g. a 95% **one-sided** bound uses $z_{0.05}=1.645$, whereas a 95% **two-sided** CI uses $z_{0.025}=1.96$. Mixing these up is a classic error.

### 4.8 What changes the width

CI width $=2E=2c\cdot\operatorname{SE}$. Hence:

| Change | Effect on width | Why |
|---|---|---|
| Confidence level $\uparrow$ (90%→99%) | **wider** | larger critical value $c$ |
| Sample size $n\uparrow$ | **narrower** | $\operatorname{SE}\propto 1/\sqrt n$ (to *halve* width, *quadruple* $n$) |
| Population SD $\sigma\uparrow$ | **wider** | larger $\operatorname{SE}$ |

### 4.9 Critical-value quick reference

**Two-sided $z_{\alpha/2}$:**

| Confidence | $\alpha$ | $z_{\alpha/2}$ |
|---|---|---|
| 90% | 0.10 | 1.645 |
| 95% | 0.05 | 1.960 |
| 99% | 0.01 | 2.576 |

**One-sided $z_{\alpha}$:** 90% → 1.282, 95% → 1.645, 99% → 2.326.

**Two-sided $t_{\alpha/2,\,n-1}$ (95%), selected df:** df $9\to2.262$, $15\to2.131$, $24\to2.064$, $\infty\to1.96$.

---

## 5. Worked problems (GATE-style)

### Problem 1 — z-interval for a mean, $\sigma$ known (NAT)

A sample of $n=64$ items has sample mean $\bar X=70$. The population SD is known to be $\sigma=16$. Construct a **95%** confidence interval for $\mu$.

**Solution.**
$\operatorname{SE}=\dfrac{\sigma}{\sqrt n}=\dfrac{16}{\sqrt{64}}=\dfrac{16}{8}=2.$ For 95%, $z_{0.025}=1.96$, so
$$E=1.96\times2=3.92,\qquad \text{CI}=70\pm3.92=\boxed{(66.08,\ 73.92)}.$$

---

### Problem 2 — t-interval for a mean, $\sigma$ unknown (MCQ — trap-style)

From a normal population, a sample of $n=16$ gives $\bar X=50$ and sample SD $s=8$. Find the **95%** CI for $\mu$.

(A) $(46.08,\ 53.92)$  (B) $(45.74,\ 54.26)$  (C) $(48.00,\ 52.00)$  (D) $(44.00,\ 56.00)$

**Solution.**
Since $\sigma$ is **unknown** and $n$ is small, use the **t-interval** with df $=n-1=15$, so $t_{0.025,15}=2.131$.
$$\operatorname{SE}=\frac{s}{\sqrt n}=\frac{8}{\sqrt{16}}=\frac{8}{4}=2,\qquad E=2.131\times2=4.262,$$
$$\text{CI}=50\pm4.262=\boxed{(45.74,\ 54.26)}\ \Rightarrow\ \textbf{(B)}.$$

**Trap (A):** wrongly using $z=1.96$ gives $E=3.92$ and the *too-narrow* interval $(46.08,53.92)$. Because $t_{0.025,15}=2.131>1.96$, the correct t-interval is **wider**. When $\sigma$ is unknown and $n$ is small, **use $t$, not $z$**.

---

### Problem 3 — CI for a proportion (NAT)

In a survey of $n=400$ voters, $100$ support a proposal. Build a **95%** CI for the true proportion $p$.

**Solution.**
$\hat p=\dfrac{100}{400}=0.25.$ Check validity: $n\hat p=100\ge5$, $n(1-\hat p)=300\ge5$ ✓.
$$\operatorname{SE}=\sqrt{\frac{\hat p(1-\hat p)}{n}}=\sqrt{\frac{0.25\times0.75}{400}}=\sqrt{0.00046875}=0.02165,$$
$$E=1.96\times0.02165=0.0424,\qquad \text{CI}=0.25\pm0.0424=\boxed{(0.208,\ 0.292)}.$$

---

### Problem 4 — Sample size for a proportion, $p$ unknown (NAT)

A pollster wants a **95%** CI for a proportion with margin of error **at most $0.03$**, but has no prior estimate of $p$. What is the minimum required sample size?

**Solution.**
With $p$ unknown, use the worst case $p(1-p)=0.25$ (at $p=0.5$):
$$n=\frac{z_{\alpha/2}^2\,p(1-p)}{E^2}=\frac{(1.96)^2(0.25)}{(0.03)^2}=\frac{3.8416\times0.25}{0.0009}=\frac{0.9604}{0.0009}=1067.1.$$
Round **up**: $\boxed{n=1068}$.

*(This is why national polls so often quote roughly $\pm3\%$ at about $1000$–$1100$ respondents.)*

---

### Problem 5 — Interpretation (MSQ — pure concept)

A 95% confidence interval for $\mu$ is computed as $(66.08,\,73.92)$. Which statement(s) are **correct**?

(i) There is a 95% probability that $\mu$ lies between $66.08$ and $73.92$.
(ii) If the sampling procedure were repeated many times, about 95% of the resulting intervals would contain $\mu$.
(iii) 95% of the sample data values lie between $66.08$ and $73.92$.
(iv) Increasing the confidence level to 99% (same data) would make the interval wider.

**Solution.**
- **(i) Incorrect.** $\mu$ is a fixed constant and the interval is already computed; the 95% describes the *long-run behaviour of the method*, not the probability for this one realized interval.
- **(ii) Correct.** This is the proper frequentist interpretation.
- **(iii) Incorrect.** A CI is a statement about the **parameter $\mu$**, not about where individual data points fall (that would be a different "prediction/tolerance" interval).
- **(iv) Correct.** Higher confidence ⇒ larger critical value ⇒ wider interval.

**Answer: (ii) and (iv).**

---

## 6. Common GATE question types on this topic

- **Compute a z-interval** for $\mu$ given $\sigma$ and $n$ (plug-in).
- **Compute a t-interval** for $\mu$ when $\sigma$ is unknown — testing whether you pick $t$ (df $=n-1$) over $z$.
- **Compute a CI for a proportion** $p$, including checking the large-sample validity condition.
- **CI for variance/SD** via $\chi^2$ — testing that you use the asymmetric formula and the right tail values.
- **Minimum sample size** for a target margin (mean or proportion; worst-case $p=0.5$).
- **Effect of changing** confidence level / $n$ / $\sigma$ on the width (conceptual MCQ).
- **Interpretation MSQs** — the frequentist vs. "probability of this interval" distinction.
- **One-sided bound** problems — using $z_\alpha$ instead of $z_{\alpha/2}$.

---

## 7. Traps, pitfalls & common mistakes

1. **z vs t.** $\sigma$ **unknown** + small $n$ ⇒ use **t** with $n-1$ df. Using $z$ gives an interval that is **too narrow** (over-confident). For large $n$ they coincide.
2. **Degrees of freedom $=n-1$**, not $n$, for the t- and chi-squared-based intervals.
3. **Two-sided vs one-sided critical value.** Two-sided 95% uses $z_{0.025}=1.96$; one-sided 95% uses $z_{0.05}=1.645$. Don't swap.
4. **Standard error vs SD.** The margin uses $\sigma/\sqrt n$ (or $s/\sqrt n$), **not** $\sigma$. (Same $\sqrt n$ trap as in the CLT topic.)
5. **Variance vs SD given.** If the problem states a *variance*, take its square root before forming the SE.
6. **Interpretation error.** "95% probability that $\mu$ is in *this* interval" is **wrong**. The randomness is in the interval/procedure; $\mu$ is fixed. The 95% is long-run coverage.
7. **CI for variance is not symmetric.** Because $\chi^2$ is skewed, never write $s^2\pm$ something; use the two-tail $\chi^2$ formula, with the **larger** $\chi^2$ value under the **lower** bound.
8. **Proportion validity.** The Wald interval needs $n\hat p\ge5$ and $n(1-\hat p)\ge5$; for tiny counts it can even produce limits below 0 or above 1, which is nonsensical.
9. **Rounding sample size.** Always round **up** — rounding down would let the margin exceed the target.
10. **Robustness asymmetry.** The mean CI is robust (CLT protects it for large $n$ even if the population isn't normal); the **variance CI is *not* robust** — it genuinely requires normality.
11. **Wider ≠ better.** A wider interval is *more confident* but *less precise* (less informative). High confidence and high precision pull in opposite directions for fixed $n$.

---

## 8. Connection forward

The bridge to **Hypothesis Testing** is **duality**. A two-sided $(1-\alpha)$ CI is the set of all parameter values that a two-sided test at level $\alpha$ would **not** reject:

$$\mu_0\in\Big(\bar X\pm z_{\alpha/2}\tfrac{\sigma}{\sqrt n}\Big)\ \Longleftrightarrow\ \text{the test of } H_0:\mu=\mu_0 \text{ is *not* rejected at level }\alpha.$$

So a CI is "all the null hypotheses you'd accept," and a test is "is *this particular* $\mu_0$ inside the interval?" The next topic formalizes this from the testing side: null/alternative hypotheses, significance level $\alpha$, test statistics, p-values, and the **z-test, t-test, and chi-squared test** — each of which reuses exactly the standard errors, critical values, and distributions you just used to build intervals.

---

## 9. One-line cheat-sheet summary

A confidence interval is $\text{estimate}\pm(\text{critical value})\times\text{SE}$: use $\bar X\pm z_{\alpha/2}\,\sigma/\sqrt n$ when $\sigma$ is known (or $n$ large), $\bar X\pm t_{\alpha/2,n-1}\,s/\sqrt n$ when $\sigma$ is unknown and $n$ small, $\hat p\pm z_{\alpha/2}\sqrt{\hat p(1-\hat p)/n}$ for a proportion, and the asymmetric $\chi^2$ formula for a variance — and remember the "95%" describes the **procedure's long-run coverage**, not one computed interval.

---

**Next topic (logically next in order): Topic 16 — Hypothesis Testing (z-test, t-test, chi-squared test)**, the dual of confidence intervals and the final topic of this section. Ask for it when ready.

# Topic 16 — Hypothesis Testing — **Part 1: The Framework & the z-Test**

*Subject: Probability and Statistics · Section: Sampling Theory & Statistical Inference (final topic)*

> This topic is split into two parts to keep full depth.
> **Part 1 (this file):** the universal testing framework (hypotheses, errors, power, p-values, decision rules) **+ the z-test family**.
> **Part 2 (ask next):** the **t-test** (one-sample, paired, two-sample) and the **chi-squared test** (goodness-of-fit, independence, variance).

---

## 0. Topic header & syllabus map

- **Topic:** Hypothesis Testing (z-test, t-test, chi-squared test) — *Part 1*
- **Subject:** Probability and Statistics (GATE DA)
- **Where it sits:** The **final topic of the Statistical Inference block**, immediately after *Confidence Intervals*. A two-sided test is the **dual** of a confidence interval (established at the end of topic 15), and the test statistics reuse the **CLT** standard errors (topic 14) and the **t/χ² sampling distributions** (topic 13). Part 1 builds the shared machinery and the z-test; Part 2 swaps in the t- and χ²-distributions.

---

## 1. Why this topic exists (motivation + connection)

A confidence interval (previous topic) answers *"which parameter values are plausible?"* — it hands you a range. But many real questions demand a **decision**, not a range: *Is the new drug better than placebo? Did the filling machine drift off its 500 ml target? Is this coin fair? Are two groups genuinely different?* For those we need a formal **yes/no rule** that comes with a **controlled error rate**.

**What the previous topic couldn't do.** A CI tells you the plausible values but doesn't, by itself, package a *decision procedure* whose false-positive rate you can pin down in advance. Hypothesis testing does exactly that: it pits a default claim (the **null hypothesis**) against a rival (the **alternative**), and rejects the default only when the data would be surprising under it — while guaranteeing that the chance of a *wrong rejection* stays at most $\alpha$ (e.g. 5%).

**Connections.** The test statistic $z=\dfrac{\bar X-\mu_0}{\sigma/\sqrt n}$ is just the CLT-standardized mean, evaluated at the *claimed* $\mu_0$. And the **CI↔test duality** (topic 15) means everything you already computed for intervals transfers directly. Where the CI *contained* $\mu_0$, the test *fails to reject* it.

**Forward (within this topic).** Part 2 keeps this entire framework unchanged and only replaces the **reference distribution**: the **t** when $\sigma$ is unknown and $n$ is small, and the **χ²** for variances, goodness-of-fit, and independence.

---

## 2. Prerequisites

- **CLT** and the **standard error** $\sigma/\sqrt n$ (topic 14).
- **Confidence intervals**, **critical values** $z_{\alpha/2}$, $z_\alpha$, and the **CI↔test duality** (topic 15).
- **Standard normal** distribution: z-scores, reading $\Phi$ and tail probabilities.
- **Sample mean $\bar X$, sample proportion $\hat p$** as estimators.
- Basic probability of events (to reason about Type I / Type II error rates).

---

## 3. Core idea — intuition first

Hypothesis testing is **proof by contradiction with a probability budget**, or equivalently the **courtroom** model:

- The defendant is **presumed innocent** — this is the **null hypothesis $H_0$** ("no effect," "status quo," "machine on target").
- The prosecution must show evidence **beyond reasonable doubt** to convict — i.e. reject $H_0$.
- We **assume $H_0$ is true**, then ask: *how surprising is the data we actually observed, if $H_0$ really held?* If the observed result sits far out in the tail (very unlikely under $H_0$), that's our "beyond reasonable doubt," and we **reject $H_0$**. If the data is unremarkable under $H_0$, we **fail to reject** it.

Two things to internalize immediately:

1. **"Fail to reject" is NOT "prove innocent."** Not convicting a defendant doesn't prove innocence — it means insufficient evidence. Likewise we never "accept $H_0$"; we only "fail to reject" it. *Absence of evidence is not evidence of absence.*
2. **$\alpha$ is the budget for convicting an innocent.** The **significance level $\alpha$** is the probability we're willing to tolerate of rejecting a *true* $H_0$ (a wrongful conviction). We **fix $\alpha$ before seeing the data** (commonly $0.05$).

The **test statistic** measures *how many standard errors the data sits from what $H_0$ predicts*. Far out ⇒ reject.

---

## 4. Formal treatment

### 4.1 The six ingredients of every test

1. **Null hypothesis $H_0$** — the default claim, always stated with **equality** at the boundary ($\mu=\mu_0$, $p=p_0$, $\sigma^2=\sigma_0^2$). The null distribution is computed *at* this value.
2. **Alternative hypothesis $H_1$ (or $H_a$)** — what we suspect. Its form fixes the tails:
   - **Two-sided:** $H_1:\mu\ne\mu_0$
   - **Right-tailed:** $H_1:\mu>\mu_0$
   - **Left-tailed:** $H_1:\mu<\mu_0$
   *(For one-sided cases $H_0$ may be written $\mu\le\mu_0$ or $\mu\ge\mu_0$, but the test is always evaluated at the boundary $\mu_0$.)*
3. **Test statistic** — a quantity whose distribution **under $H_0$** is known (here, standard normal).
4. **Significance level $\alpha$** — the tolerated Type I error rate, **fixed in advance**.
5. **Rejection region / decision rule** — either compare the statistic to a **critical value**, or compare the **p-value** to $\alpha$.
6. **Decision + conclusion in context** — "reject $H_0$" or "fail to reject $H_0$," then state what that means for the real question.

### 4.2 Two equivalent decision rules

**(I) Critical-value method.** Reject $H_0$ if the test statistic lands in the rejection region:

| $H_1$ | Reject $H_0$ when |
|---|---|
| $\mu\ne\mu_0$ (two-tailed) | $\lvert z\rvert>z_{\alpha/2}$ |
| $\mu>\mu_0$ (right-tailed) | $z>z_{\alpha}$ |
| $\mu<\mu_0$ (left-tailed) | $z<-z_{\alpha}$ |

**(II) p-value method.** The **p-value** is the probability, *computed under $H_0$*, of getting a test statistic **at least as extreme** as the one observed:

| $H_1$ | p-value |
|---|---|
| two-tailed | $2\,P(Z>\lvert z_{\text{obs}}\rvert)$ |
| right-tailed | $P(Z>z_{\text{obs}})$ |
| left-tailed | $P(Z<z_{\text{obs}})$ |

**Reject $H_0$ iff p-value $\le\alpha$.** The two methods *always* give the same decision.

> **What the p-value is NOT:** it is **not** $P(H_0\text{ true})$ and not the probability you made a mistake. It is $P(\text{data this extreme}\mid H_0\text{ true})$.

### 4.3 Type I & Type II errors and power (heavily tested)

|  | **$H_0$ is true** | **$H_0$ is false** |
|---|---|---|
| **Reject $H_0$** | **Type I error** (prob $=\alpha$) | Correct decision (prob $=1-\beta=$ **power**) |
| **Fail to reject $H_0$** | Correct decision (prob $=1-\alpha$) | **Type II error** (prob $=\beta$) |

- **Type I error (false positive):** rejecting a *true* $H_0$. Its probability is exactly the significance level, $\boxed{P(\text{Type I})=\alpha}$ — and **we choose it**.
- **Type II error (false negative):** failing to reject a *false* $H_0$. Its probability $\boxed{P(\text{Type II})=\beta}$ depends on the *true* parameter value, on $n$, on $\sigma$, and on $\alpha$.
- **Power $=1-\beta$:** the probability of correctly detecting a real effect.

**Key relationships (memorize):**

| Change | Effect |
|---|---|
| Lower $\alpha$ (stricter), fixed $n$ | $\beta$ **rises**, power **falls** (the two errors trade off) |
| Larger sample size $n$ | $\beta$ **falls**, power **rises** (can reduce *both* errors) |
| Larger true effect $\lvert\mu_1-\mu_0\rvert$ | power **rises** |
| Larger $\sigma$ | power **falls** |

### 4.4 The z-test family — conditions and statistics

A **z-test** is used when the test statistic is (approximately) **standard normal under $H_0$** — i.e. $\sigma$ is **known** (or $n$ large enough for the CLT), or we're testing proportions with large $n$.

**(a) One-sample mean, $\sigma$ known**
$$z=\frac{\bar X-\mu_0}{\sigma/\sqrt n}\qquad\text{conditions: }\sigma\text{ known; population normal OR }n\text{ large (CLT).}$$

**(b) One-sample proportion**
$$z=\frac{\hat p-p_0}{\sqrt{\dfrac{p_0(1-p_0)}{n}}}\qquad\text{conditions: }np_0\ge5,\ n(1-p_0)\ge5.$$
> ⚠️ The SE here uses the **null value $p_0$**, *not* $\hat p$ — because the null distribution is computed *assuming $H_0$ is true*. (Contrast with the proportion **CI**, which uses $\hat p$ since there is no null there.)

**(c) Two-sample means, $\sigma_1,\sigma_2$ known**
$$z=\frac{(\bar X_1-\bar X_2)-(\mu_1-\mu_2)_0}{\sqrt{\dfrac{\sigma_1^2}{n_1}+\dfrac{\sigma_2^2}{n_2}}}\qquad\text{usually }(\mu_1-\mu_2)_0=0;\ \text{conditions: }\sigma\text{'s known, large/normal.}$$

**(d) Two-sample proportions** *(less common in GATE DA; uses a pooled estimate under $H_0$)*
$$z=\frac{\hat p_1-\hat p_2}{\sqrt{\hat p(1-\hat p)\!\left(\tfrac1{n_1}+\tfrac1{n_2}\right)}},\qquad \hat p=\frac{x_1+x_2}{n_1+n_2}.$$

### 4.5 Critical-value quick reference (z)

| $\alpha$ | Two-tailed $z_{\alpha/2}$ | One-tailed $z_{\alpha}$ |
|---|---|---|
| 0.10 | $\pm1.645$ | $1.282$ |
| 0.05 | $\pm1.960$ | $1.645$ |
| 0.01 | $\pm2.576$ | $2.326$ |

### 4.6 CI ↔ test duality (recap, now from the testing side)

For a two-sided test at level $\alpha$:
$$\text{reject } H_0:\mu=\mu_0 \ \text{at level }\alpha
\quad\Longleftrightarrow\quad
\mu_0\ \notin\ \Big(\bar X\pm z_{\alpha/2}\tfrac{\sigma}{\sqrt n}\Big).$$
So you can answer a two-sided test simply by checking whether $\mu_0$ lies outside the $(1-\alpha)$ CI — no separate computation needed.

---

## 5. Worked problems (GATE-style) — Part 1

### Problem 1 — One-sample z-test for a mean, two-tailed (both methods) (NAT/MCQ)

A filling machine should dispense a mean of $\mu_0=500$ ml with known SD $\sigma=12$ ml. A sample of $n=36$ bottles has mean $\bar X=504$ ml. At $\alpha=0.05$, test whether the mean has **changed**.

**Solution.**
$H_0:\mu=500$ vs $H_1:\mu\ne500$ (two-tailed). Standard error $=\dfrac{12}{\sqrt{36}}=2$.
$$z=\frac{504-500}{2}=2.0.$$
- **Critical-value method:** $z_{0.025}=1.96$. Since $\lvert z\rvert=2.0>1.96$, **reject $H_0$**.
- **p-value method:** $p=2\,P(Z>2.0)=2(1-0.9772)=2(0.0228)=0.0456<0.05$, **reject $H_0$**.

**Conclusion:** there is significant evidence (at 5%) that the mean fill has changed.

*Duality check:* the 95% CI is $504\pm1.96(2)=(500.08,\,507.92)$, which **excludes** 500 — consistent with rejecting $H_0$.

---

### Problem 2 — One-sample z-test for a proportion, right-tailed (NAT)

A supplier claims its defect rate is **at most 10%**. In a sample of $n=200$ items, $30$ are defective. At $\alpha=0.05$, is there evidence the defect rate **exceeds** 10%?

**Solution.**
$H_0:p=0.10$ vs $H_1:p>0.10$ (right-tailed). $\hat p=\dfrac{30}{200}=0.15$.
SE **under $H_0$** (uses $p_0=0.10$): $\sqrt{\dfrac{0.10\times0.90}{200}}=\sqrt{0.00045}=0.02121$.
$$z=\frac{0.15-0.10}{0.02121}=2.357.$$
Critical value (right tail, $\alpha=0.05$) is $1.645$. Since $2.357>1.645$, **reject $H_0$**. (p-value $=P(Z>2.357)\approx0.009<0.05$.)

**Conclusion:** evidence that the true defect rate exceeds 10%.
*(Note the SE used $p_0=0.10$, not $\hat p=0.15$ — this is the proportion-test trap.)*

---

### Problem 3 — Two-sample z-test for difference of means, two-tailed (MCQ)

Two production lines have known SDs. Line A: $n_1=50$, $\bar X_1=120$, $\sigma_1=10$. Line B: $n_2=50$, $\bar X_2=115$, $\sigma_2=8$. At $\alpha=0.05$, test whether the line means differ.

**Solution.**
$H_0:\mu_1=\mu_2$ vs $H_1:\mu_1\ne\mu_2$.
$$\text{SE}=\sqrt{\frac{10^2}{50}+\frac{8^2}{50}}=\sqrt{\frac{100}{50}+\frac{64}{50}}=\sqrt{2+1.28}=\sqrt{3.28}=1.811.$$
$$z=\frac{(120-115)-0}{1.811}=\frac{5}{1.811}=2.761.$$
Since $\lvert z\rvert=2.761>1.96$, **reject $H_0$**. (p-value $=2P(Z>2.76)\approx0.0058<0.05$.)

**Conclusion:** the two line means differ significantly at 5%.

---

### Problem 4 — Type II error and power (NAT — the conceptual one)

A right-tailed z-test for a mean uses $H_0:\mu=50$ vs $H_1:\mu>50$, with $\sigma=10$, $n=25$, $\alpha=0.05$.
(a) State the rejection rule in terms of $\bar X$.
(b) If the **true** mean is actually $\mu=55$, find the Type II error probability $\beta$ and the **power**.

**Solution.**
SE $=\dfrac{10}{\sqrt{25}}=2$.

**(a)** Right-tailed, $\alpha=0.05\Rightarrow$ reject when $z>1.645$, i.e.
$$\frac{\bar X-50}{2}>1.645\ \Longrightarrow\ \bar X>50+1.645(2)=53.29.$$
So: **reject $H_0$ if $\bar X>53.29$.**

**(b)** A Type II error = *failing to reject* $H_0$ (i.e. $\bar X\le53.29$) when the truth is $\mu=55$:
$$\beta=P(\bar X\le 53.29\mid \mu=55)=P\!\left(Z\le\frac{53.29-55}{2}\right)=P(Z\le-0.855)\approx0.196.$$
$$\textbf{Power}=1-\beta\approx0.804.$$

**Interpretation:** even though the true mean really is above 50, this test detects it only about **80%** of the time at $n=25$. Increasing $n$, increasing the true gap, or raising $\alpha$ would push the power higher; a larger $\sigma$ would lower it.

---

## 6. Common GATE question types on this topic (Part 1)

- **Set up $H_0$/$H_1$** correctly, especially deciding **one- vs two-tailed** from the wording ("changed/differs" ⇒ two-tailed; "exceeds/greater/less" ⇒ one-tailed).
- **Compute the z-statistic** and decide via critical value **or** p-value.
- **z-test for a proportion** — remembering the SE uses $p_0$.
- **Two-sample z-test** for a difference of means.
- **Type I / Type II errors and power:** identify which is which, compute $\beta$ and power, and reason about the $\alpha$–$\beta$ and $n$–power trade-offs.
- **p-value interpretation** and comparison to $\alpha$.
- **CI↔test duality:** decide a two-sided test directly from a confidence interval.

---

## 7. Traps, pitfalls & common mistakes (Part 1)

1. **"Fail to reject" ≠ "accept $H_0$."** We never *prove* $H_0$; insufficient evidence is not proof of the null.
2. **The p-value is not $P(H_0\text{ true})$.** It is $P(\text{data this extreme}\mid H_0)$.
3. **One- vs two-tailed critical value.** Two-tailed uses $z_{\alpha/2}$; one-tailed uses $z_{\alpha}$. Correspondingly, a two-tailed p-value **doubles** the one-tail probability.
4. **Proportion SE.** The **test** uses $p_0$ in the SE; the **CI** uses $\hat p$. Swapping them is a classic error.
5. **Type I vs Type II.** $\alpha=P(\text{reject true }H_0)$ is set by you; $\beta=P(\text{keep false }H_0)$ depends on the true parameter, $n$, and $\sigma$. They are **not** "$\alpha+\beta=1$."
6. **The error trade-off.** Lowering $\alpha$ (for fixed $n$) **raises** $\beta$. Only increasing $n$ can shrink both.
7. **Rejection direction.** Right-tailed rejects for **large** $z$; left-tailed rejects for **small (very negative)** $z$. Don't reject on the wrong side.
8. **Compare like with like.** Compare the **statistic to a critical value**, *or* the **p-value to $\alpha$** — never a statistic to $\alpha$ or a p-value to a critical value.
9. **$\sigma$ known ⇒ z; $\sigma$ unknown (small $n$) ⇒ t** (Part 2). Don't use z when only the sample SD $s$ is available with small $n$.
10. **Statistical ≠ practical significance.** With large $n$, even a trivially small effect can be "significant"; significance doesn't mean the effect is important.

---

## 8. Connection forward

**Within this topic (to Part 2):** the *entire* framework above — $H_0/H_1$, $\alpha$, Type I/II errors, power, p-values, the decision rules, and the duality with CIs — stays **exactly the same**. Only two things change in Part 2: the **test statistic** and its **reference distribution**:
- **t-test:** when $\sigma$ is unknown and $n$ is small, replace $\sigma$ by $s$ and compare to a **$t_{n-1}$** distribution (one-sample, paired, and two-sample versions).
- **chi-squared test:** for a single **variance**, and for **goodness-of-fit** and **independence** of categorical data, the statistic is compared to a **$\chi^2$** distribution.

**Beyond the syllabus (context only):** this machinery is the backbone of A/B testing, ML model comparison, and feature-significance checks — useful to know, but not examinable detail.

---

## 9. One-line cheat-sheet summary (Part 1)

A z-test assumes $H_0$, computes $z=\dfrac{\text{estimate}-\text{null}}{\text{SE}}$ (SE uses **$\sigma$** for a mean, **$p_0$** for a proportion), and **rejects $H_0$ when $\lvert z\rvert$ exceeds the critical value** ($z_{\alpha/2}$ two-tailed, $z_\alpha$ one-tailed) or equivalently when **p $\le\alpha$**; Type I error $=\alpha$ (chosen), Type II error $=\beta$, power $=1-\beta$, and lowering $\alpha$ raises $\beta$ for fixed $n$.

---

**This is Part 1 of Topic 16.** Ask for **Part 2 — the t-Test (one-sample, paired, two-sample) and the Chi-Squared Test (goodness-of-fit, independence, variance)** to complete this topic and the Statistical Inference section.

# Topic 16 — Hypothesis Testing — **Part 2: The t-Test & the Chi-Squared Test**

*Subject: Probability and Statistics · Section: Sampling Theory & Statistical Inference (final topic)*

> **Continues Part 1.** Part 1 built the *shared* framework — $H_0/H_1$, one- vs two-tailed logic, the critical-value and p-value decision rules, Type I/II errors, power, and the CI↔test duality. **None of that is repeated here.** Part 2 keeps all of it and only changes the **test statistic** and its **reference distribution**.

---

## 0. Topic header & syllabus map

- **Topic:** Hypothesis Testing — *Part 2:* the **t-test** (one-sample, paired, two-sample) and the **chi-squared test** (goodness-of-fit, independence, variance).
- **Subject:** Probability and Statistics (GATE DA)
- **Where it sits:** Completes the **final topic of the Statistical Inference block**. The t-test directly uses the **t-distribution** and the χ²-test the **chi-squared distribution** from topic 13; the framework is from Part 1. After this, the Probability & Statistics syllabus is complete.

---

## 1. Why this part exists (motivation + connection)

The z-test of Part 1 needed the population SD $\sigma$ to be **known** — but in practice we almost **never** know $\sigma$; we only have the sample SD $s$. And the z-test only handled **numeric means/proportions** — it said nothing about **categorical data** (does this die fit a fair distribution? are two attributes associated?) or about **the spread itself** (has variability changed?). Part 2 fills both gaps:

- **t-test:** when $\sigma$ is unknown and $n$ is small, estimating $\sigma$ by $s$ injects *extra* uncertainty. The standardized statistic is then **no longer standard normal** — it follows the heavier-tailed **t-distribution** with $n-1$ degrees of freedom (topic 13). This is the precise fix for "z-test but $\sigma$ unknown."
- **chi-squared test:** comparing **observed vs. expected counts** produces a statistic that follows the **χ² distribution** — used for goodness-of-fit, independence in contingency tables, and (via $\frac{(n-1)s^2}{\sigma^2}\sim\chi^2_{n-1}$) testing a **variance**.

**Connection.** As $n\to\infty$, $t_{n-1}\to z$ — so the t-test *becomes* the z-test for large samples (the CLT again). And the χ² variance test is exactly the **testing dual** of the variance confidence interval from topic 15.

---

## 2. Prerequisites

- **Part 1**: the full testing framework (hypotheses, $\alpha$, errors, power, p-value/critical-value rules, duality).
- **t-distribution** (heavier tails, parameter = degrees of freedom) and **chi-squared distribution** (right-skewed, non-negative) — topic 13.
- **Sample variance $s^2$** and **sample SD $s$**; the result $\dfrac{(n-1)s^2}{\sigma^2}\sim\chi^2_{n-1}$ for a normal population.
- Reading **t** and **χ²** critical-value tables.

---

## 3. Core idea — intuition first

**Why t, not z?** When $\sigma$ is known, $\dfrac{\bar X-\mu_0}{\sigma/\sqrt n}$ is standard normal. But replacing $\sigma$ with the *estimated* $s$ means the denominator is itself random and "wobbly," especially for small $n$. That extra wobble makes large statistic values more likely than the normal predicts — so the correct reference curve has **fatter tails**: the **t-distribution**. Fewer degrees of freedom ⇒ fatter tails ⇒ larger critical values ⇒ a **more conservative** (harder-to-reject) test. As $n$ grows, $s\to\sigma$, the wobble vanishes, and t collapses back to z.

**Why χ² for counts?** A χ² test asks: *how far are the observed category counts from what the model expects?* For each category compute the standardized squared gap $\dfrac{(O-E)^2}{E}$ and add them up. Small total ⇒ data fits the model; large total ⇒ the model is wrong. Because **every** mismatch — too many *or* too few — makes a *positive* contribution, the χ² goodness-of-fit and independence tests are **always right-tailed**: we only reject for a *large* χ². (Squaring removes the sign, so there's no "left side" to worry about.)

---

## 4. Formal treatment

### 4.1 The t-test family (σ unknown)

**(a) One-sample t-test for a mean**
$$t=\frac{\bar X-\mu_0}{s/\sqrt n},\qquad \text{df}=n-1.$$
*Conditions:* $\sigma$ unknown (use sample SD $s$); population **approximately normal** (matters for small $n$; for large $n$ the CLT makes it robust and $t\approx z$).

**(b) Paired (matched) t-test — two *dependent* samples**
Use when the two samples are **linked** (same subjects measured before/after, or matched pairs). Form the differences $d_i=x_i-y_i$ and run a one-sample t on them:
$$t=\frac{\bar d-\mu_{d,0}}{s_d/\sqrt n},\qquad \text{df}=n-1,$$
where $\bar d$ and $s_d$ are the mean and SD of the **differences**, $n$ = number of **pairs**, and usually $\mu_{d,0}=0$.
*Conditions:* differences approximately normal; pairs independent of each other.

**(c) Two-sample t-test — two *independent* groups**

*Pooled (assume equal population variances)* — the standard GATE form:
$$t=\frac{(\bar X_1-\bar X_2)-(\mu_1-\mu_2)_0}{s_p\sqrt{\tfrac1{n_1}+\tfrac1{n_2}}},\qquad
s_p^2=\frac{(n_1-1)s_1^2+(n_2-1)s_2^2}{n_1+n_2-2},\qquad \text{df}=n_1+n_2-2.$$
$s_p^2$ is the **pooled variance** (a weighted average of the two sample variances). Usually $(\mu_1-\mu_2)_0=0$.
*Conditions:* independent groups; each population approx. normal; **equal variances**.

*Welch's t (unequal variances)* — use $\text{SE}=\sqrt{s_1^2/n_1+s_2^2/n_2}$ with the Welch–Satterthwaite df (a non-integer formula rarely hand-computed in GATE). Mentioned for completeness; GATE typically expects the **pooled** version.

**Decision:** identical to Part 1 — compare $t$ to $t_{\alpha/2,\text{df}}$ (two-tailed) or $t_{\alpha,\text{df}}$ (one-tailed), or compare the p-value to $\alpha$.

### 4.2 The chi-squared test family

All χ² tests on counts use the same statistic, compared to the **upper-tail** critical value $\chi^2_{\alpha,\text{df}}$ (right-tailed):
$$\boxed{\ \chi^2=\sum_i\frac{(O_i-E_i)^2}{E_i}\ },\qquad O=\text{observed},\ E=\text{expected under }H_0.$$
*General condition:* every **expected** count $E_i\ge 5$ (rule of thumb); if some are smaller, **combine categories**.

**(a) Goodness-of-fit (GOF)** — does an observed frequency distribution match a hypothesized one?
- $E_i=n\,p_i$ where $p_i$ are the hypothesized category probabilities.
- $H_0$: data follow the specified distribution; $H_1$: they do not.
- $\text{df}=k-1-m$, where $k=$ number of categories and $m=$ number of parameters **estimated from the data**. (Fully specified distribution ⇒ $m=0$ ⇒ $\text{df}=k-1$; e.g. fitting a Poisson with $\lambda$ estimated ⇒ $m=1$ ⇒ $\text{df}=k-2$.)

**(b) Test of independence** — are two categorical variables associated? (Data in an $r\times c$ contingency table.)
- Expected cell count: $\displaystyle E_{ij}=\frac{(\text{row}_i\text{ total})(\text{col}_j\text{ total})}{\text{grand total}}.$
- $H_0$: the two variables are **independent**; $H_1$: they are **associated**.
- $\text{df}=(r-1)(c-1).$
- *(The **test of homogeneity** — comparing the distribution of one variable across several populations — uses the identical statistic and df; only the sampling story differs.)*

**(c) Test for a single variance** (normal population) — the testing dual of the variance CI:
$$\chi^2=\frac{(n-1)s^2}{\sigma_0^2},\qquad \text{df}=n-1.$$
- $H_0:\sigma^2=\sigma_0^2$. Can be **one- or two-tailed**.
- Two-tailed rejection region: $\chi^2<\chi^2_{1-\alpha/2,\,n-1}$ **or** $\chi^2>\chi^2_{\alpha/2,\,n-1}$ (asymmetric, like the variance CI).
- *Condition:* population **normal** (this test is **not** robust to non-normality).

### 4.3 Degrees-of-freedom cheat-table (memorize)

| Test | df |
|---|---|
| One-sample t | $n-1$ |
| Paired t | $n-1$ ($n$ = #pairs) |
| Two-sample pooled t | $n_1+n_2-2$ |
| χ² goodness-of-fit | $k-1-m$ |
| χ² independence ($r\times c$) | $(r-1)(c-1)$ |
| χ² for a variance | $n-1$ |

### 4.4 A useful identity

For one degree of freedom, $\chi^2_{1}=Z^2$. Hence $\chi^2_{0.05,1}=3.841=(1.96)^2$ — the χ²-with-1-df test and the two-sided z-test are the same test in disguise (e.g. a $2\times2$ independence test equals a two-proportion z-test).

---

## 5. Worked problems (GATE-style) — Part 2

*(t-table values used: $t_{0.025,15}=2.131$, $t_{0.05,9}=1.833$, $t_{0.025,20}=2.086$.
χ²-table values used: $\chi^2_{0.05,5}=11.070$, $\chi^2_{0.05,1}=3.841$, $\chi^2_{0.025,24}=39.364$, $\chi^2_{0.975,24}=12.401$.)*

### Problem 1 — One-sample t-test, two-tailed (NAT) — and a z-vs-t contrast

A process target is $\mu_0=100$. A sample of $n=16$ gives $\bar X=96$ and $s=8$. At $\alpha=0.05$, test whether the mean differs from 100. (Population approx. normal, $\sigma$ unknown.)

**Solution.**
$H_0:\mu=100$ vs $H_1:\mu\ne100$. Use **t** (σ unknown, small $n$), df $=15$.
$$\text{SE}=\frac{s}{\sqrt n}=\frac{8}{4}=2,\qquad t=\frac{96-100}{2}=-2.0.$$
Critical value $t_{0.025,15}=2.131$. Since $\lvert t\rvert=2.0<2.131$, **fail to reject $H_0$**.

**Conclusion:** insufficient evidence that the mean differs from 100.
**Teaching point:** with a *z*-test this same $\lvert z\rvert=2.0$ would *exceed* $1.96$ and reject — but the t-test's larger critical value ($2.131$) makes it **more conservative**, so we don't reject. That gap is the cost of not knowing $\sigma$.

---

### Problem 2 — Paired t-test, one-tailed (NAT)

Ten subjects' weights are recorded before and after a program. The differences $d_i=(\text{before}-\text{after})$ have mean $\bar d=3$ kg and SD $s_d=4$ kg ($n=10$ pairs). At $\alpha=0.05$, is there evidence of weight **loss**?

**Solution.**
This is **paired** data (same subjects twice). $H_0:\mu_d=0$ vs $H_1:\mu_d>0$ (right-tailed). df $=9$.
$$\text{SE}=\frac{s_d}{\sqrt n}=\frac{4}{\sqrt{10}}=1.265,\qquad t=\frac{3-0}{1.265}=2.372.$$
Critical value $t_{0.05,9}=1.833$ (one-tailed). Since $2.372>1.833$, **reject $H_0$**.

**Conclusion:** significant evidence of weight loss.
*(Trap avoided: because the two readings are on the **same** subjects, this is paired — using an independent two-sample test here would be wrong.)*

---

### Problem 3 — Two-sample pooled t-test, two-tailed (MCQ)

Independent groups. Group 1: $n_1=10$, $\bar X_1=52$, $s_1=3$. Group 2: $n_2=12$, $\bar X_2=48$, $s_2=4$. Assume equal variances. At $\alpha=0.05$, do the means differ?

**Solution.**
$H_0:\mu_1=\mu_2$ vs $H_1:\mu_1\ne\mu_2$. Pooled variance, df $=n_1+n_2-2=20$:
$$s_p^2=\frac{(10-1)3^2+(12-1)4^2}{20}=\frac{81+176}{20}=\frac{257}{20}=12.85,\quad s_p=3.585.$$
$$\text{SE}=s_p\sqrt{\tfrac1{10}+\tfrac1{12}}=3.585\sqrt{0.18333}=3.585(0.4282)=1.535,$$
$$t=\frac{52-48}{1.535}=2.606.$$
Critical value $t_{0.025,20}=2.086$. Since $\lvert t\rvert=2.606>2.086$, **reject $H_0$**.

**Conclusion:** the two group means differ significantly at 5%.

---

### Problem 4 — Chi-squared goodness-of-fit (NAT)

A die is rolled $120$ times with results: face 1→18, 2→23, 3→16, 4→21, 5→18, 6→24. At $\alpha=0.05$, test whether the die is **fair**.

**Solution.**
$H_0$: fair die ($p_i=\tfrac16$ each) vs $H_1$: not fair. Expected each $=120/6=20$. df $=k-1=5$.
$$\chi^2=\sum\frac{(O-E)^2}{E}=\frac{(-2)^2+3^2+(-4)^2+1^2+(-2)^2+4^2}{20}=\frac{4+9+16+1+4+16}{20}=\frac{50}{20}=2.5.$$
Critical value $\chi^2_{0.05,5}=11.070$. Since $2.5<11.070$, **fail to reject $H_0$**.

**Conclusion:** no evidence the die is unfair. (All $E_i=20\ge5$, so the test is valid; the test is **right-tailed**.)

---

### Problem 5 — Chi-squared test of independence, $2\times2$ (NAT)

Is treatment associated with outcome? Observed counts:

| | Success | Failure | Row total |
|---|---|---|---|
| **Treatment A** | 50 | 50 | 100 |
| **Treatment B** | 30 | 70 | 100 |
| **Col total** | 80 | 120 | **200** |

Test at $\alpha=0.05$.

**Solution.**
$H_0$: treatment and outcome **independent** vs $H_1$: **associated**. df $=(2-1)(2-1)=1$.
Expected counts $E_{ij}=\dfrac{(\text{row})(\text{col})}{200}$: $E(\text{A,Succ})=\frac{100\cdot80}{200}=40$, $E(\text{A,Fail})=60$, $E(\text{B,Succ})=40$, $E(\text{B,Fail})=60$.
$$\chi^2=\frac{(50-40)^2}{40}+\frac{(50-60)^2}{60}+\frac{(30-40)^2}{40}+\frac{(70-60)^2}{60}=2.5+1.667+2.5+1.667=8.333.$$
Critical value $\chi^2_{0.05,1}=3.841$. Since $8.333>3.841$, **reject $H_0$**.

**Conclusion:** there is a significant association between treatment and outcome.
*(Note: for $2\times2$ tables some texts apply **Yates' continuity correction**, $\sum\frac{(\lvert O-E\rvert-0.5)^2}{E}$; GATE generally uses the **uncorrected** statistic shown here.)*

---

### Problem 6 — Chi-squared test for a variance, two-tailed (NAT)

A normal population is claimed to have variance $\sigma_0^2=4$. A sample of $n=25$ gives $s^2=8$. At $\alpha=0.05$, test whether the variance **differs** from 4.

**Solution.**
$H_0:\sigma^2=4$ vs $H_1:\sigma^2\ne4$. df $=24$.
$$\chi^2=\frac{(n-1)s^2}{\sigma_0^2}=\frac{24\times8}{4}=48.$$
Two-tailed rejection region: $\chi^2<\chi^2_{0.975,24}=12.401$ or $\chi^2>\chi^2_{0.025,24}=39.364$. Since $48>39.364$, **reject $H_0$**.

**Conclusion:** evidence that the variance differs from 4 (in fact, it is larger).
*(This mirrors the asymmetric variance CI of topic 15: $48$ falls outside the acceptance band $[12.401,\,39.364]$.)*

---

## 6. Common GATE question types on this topic (Part 2)

- **Pick the right test:** $\sigma$ unknown + small $n$ ⇒ **t**; categorical counts ⇒ **χ²**; spread/variance ⇒ **χ² variance test**.
- **Paired vs independent:** decide whether two samples are *linked* (paired t) or *separate groups* (two-sample t) — a recurring trap.
- **Compute pooled variance and the two-sample t**, with df $=n_1+n_2-2$.
- **χ² goodness-of-fit:** compute expected counts, the statistic, and especially the **df** ($k-1-m$).
- **χ² independence:** compute $E_{ij}$ from row/column totals; df $=(r-1)(c-1)$.
- **χ² variance test** and its asymmetric rejection region.
- **Identify the correct degrees of freedom** for each test (the single most common χ²/t slip).

---

## 7. Traps, pitfalls & common mistakes (Part 2)

1. **z vs t.** Known $\sigma$ ⇒ z; unknown $\sigma$ with small $n$ ⇒ **t** (df $=n-1$). Using z understates the critical value and over-rejects. For large $n$, t $\approx$ z.
2. **Wrong degrees of freedom.** Use the df cheat-table (4.3). Confusing $n-1$ (one-sample/variance) with $n_1+n_2-2$ (two-sample pooled) or $(r-1)(c-1)$ (independence) is the top error.
3. **Paired vs independent.** Same subjects measured twice (before/after, matched) ⇒ **paired** t on the differences. Two separate groups ⇒ **two-sample** t. Choosing wrong changes the statistic and df entirely.
4. **Pooled variance is weighted.** $s_p^2$ weights by $n_i-1$; it is **not** the simple average $\tfrac{s_1^2+s_2^2}{2}$ unless $n_1=n_2$.
5. **χ² uses counts, never percentages/proportions.** Convert any rates to actual frequencies first.
6. **Expected, not observed, in the denominator.** The statistic is $\sum\frac{(O-E)^2}{E}$ — divide by $E$.
7. **GOF/independence are right-tailed only.** Squaring removes signs, so every deviation inflates χ²; reject only for **large** χ². (Only the *variance* test can be two-tailed.)
8. **$E_i\ge5$ validity.** If expected counts are too small, the χ² approximation fails — combine categories.
9. **GOF df subtracts estimated parameters.** Fitting a distribution whose parameters were estimated from the data lowers df by $m$ (e.g. Poisson with estimated $\lambda$ ⇒ df $=k-2$).
10. **Variance test needs normality.** Unlike the mean tests (CLT-protected), the χ² variance test is genuinely sensitive to non-normal populations.
11. **Conclusions are about the parameter/association, not individuals**, and "fail to reject" still never means "$H_0$ proven."

---

## 8. Connection forward (and section wrap-up)

This completes **Hypothesis Testing** and the entire **Sampling Theory & Statistical Inference** section. The whole block now hangs together as one story:

- the **CLT** (14) gave the sampling distribution of $\bar X$;
- **confidence intervals** (15) inverted it into interval estimates;
- **hypothesis testing** (16) turned the same machinery into decisions — the **z-test** when $\sigma$ is known (Part 1), the **t-test** when it isn't, and the **χ²-test** for categorical data and variances (Part 2).

The choice among tests is just: *what parameter, is $\sigma$ known, how big is $n$, and is the data numeric or categorical?* Beyond GATE, this exact toolkit powers **A/B testing, model comparison, and feature-significance** decisions in data science — context worth knowing, but not examinable detail.

---

## 9. One-line cheat-sheet summary (Part 2)

Use $t=\dfrac{\bar X-\mu_0}{s/\sqrt n}$ (df $n-1$) when $\sigma$ is unknown — paired t runs this on the differences (df $n-1$), pooled two-sample t uses $s_p$ with df $n_1+n_2-2$; for counts use $\chi^2=\sum\frac{(O-E)^2}{E}$, **right-tailed**, with df $=k-1-m$ (goodness-of-fit), $(r-1)(c-1)$ (independence), and use $\chi^2=\dfrac{(n-1)s^2}{\sigma_0^2}$ (df $n-1$) for a variance.

---

**Topic 16 is now complete (Parts 1 + 2), finishing the Probability & Statistics — Statistical Inference section.** If you'd like, I can assemble a one-page **formula-and-df master sheet** spanning topics 14–16 for last-day revision.

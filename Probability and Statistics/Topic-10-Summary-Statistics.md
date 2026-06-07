# Topic 10 — Summary Statistics (Mean, Median, Mode, Standard Deviation, Covariance & Correlation)

*Subject: Probability and Statistics (GATE DA) · Cluster: Random Variables & Expectation (final topic)*

> **What's new vs Topic 9.** Topic 9 defined $E[X]$ and $\mathrm{Var}(X)$ as *operators on random variables* (the distributional/population view). Topic 10 turns these into a **descriptive toolkit**: alternative centres (**median, mode**), the **sample** versions you compute from raw data (with the $n-1$ subtlety), and the **full** covariance → **correlation** story that Topic 9 only previewed for variance‑of‑a‑sum.

---

## 0. Topic header & syllabus map

- **Topic:** Summary Statistics — central tendency (mean, median, mode), spread (standard deviation & friends), and bivariate association (covariance, correlation)
- **Subject:** Probability and Statistics
- **Where it sits:** **Final** topic of the *Random Variables & Expectation* block. It comes right after **Expectation & Variance (Topic 9)** and closes the cluster by giving the *vocabulary of summaries* we will report for every distribution from here on. The next topic begins a **new chat — Topic 11: Discrete Distributions** — where these summaries get computed for named PMFs.

---

## 1. Why this topic exists (motivation + connection)

- **The real problem.** We usually face one of two situations Topic 9 didn't address: (i) we hold raw **data** (a sample), not a known distribution, and must summarise it; (ii) even with a distribution, the *mean* is only one notion of "centre," and for **two** variables we want a clean, **unit‑free** measure of how they move together.
- **What the previous topic couldn't do.** Topic 9 treated the mean as *the* centre, with no account of **robustness** (outliers), **skew**, or **categorical** data, and no descriptive **sample** formulas. Covariance appeared only as a tool for $\mathrm{Var}(X+Y)$; its magnitude depends on **units**, so it can't be compared across variable pairs.
- **The fix.** A descriptive kit: **central tendency** (mean / median / mode), **spread** (SD, range, IQR), and **bivariate association** (covariance, and its normalised version **correlation** $\rho\in[-1,1]$). Plus the **sample** counterparts for actual data.

---

## 2. Prerequisites

- **Topic 9** — $E[X]$, $\mathrm{Var}(X)$, $\mathrm{Cov}$ (preview), LOTUS, linearity, variance algebra.
- **Topic 7** — CDF and **percentiles/median** via $F(m)=\tfrac12$; continuous mode via $f'(x)=0$.
- **Topic 6** — PMF and **mode** as the most probable value (argmax).
- **Topic 8** — joint distributions and **independence** (for the uncorrelated‑vs‑independent distinction).

---

## 3. Core idea — intuition first

A summary statistic compresses a distribution (or a dataset) into a few interpretable numbers along two axes:

- **Where is it centred?** Three honest answers, each "typical" in a different sense:
  - **Mean** — the *balance point* (Topic 9's $E[X]$). Uses every value, so a single huge outlier drags it.
  - **Median** — the *middle* value (50th percentile). Half the mass lies on each side; an outlier barely moves it (**robust**).
  - **Mode** — the *most common* value / the peak of the PMF/PDF. The only centre that works for purely categorical data.
- **How spread out is it?** **Standard deviation** (typical distance from the mean), with cruder cousins **range** (max − min) and **IQR** ($Q_3-Q_1$, robust).

For **two** variables, **covariance** asks "when $X$ is above its mean, does $Y$ tend to be above (positive) or below (negative) its mean?" But its size is in mixed units (e.g. ₹·kg), so we rescale by the two SDs to get **correlation** $\rho$ — a pure number in $[-1,1]$ where $\pm1$ means a perfect *straight‑line* relationship and $0$ means no *linear* trend.

**Skew, intuitively.** If centre measures disagree, the distribution is lopsided: a long **right** tail pulls the *mean* above the *median* (right/positive skew); a long **left** tail pulls it below.

---

## 4. Formal treatment

### 4.1 Central tendency

| Measure | Distribution (population) | Data (sample, size $n$) |
|---|---|---|
| **Mean** | $\mu=E[X]=\sum x\,p(x)$ or $\int x f(x)\,dx$ | $\bar{x}=\dfrac{1}{n}\sum_{i=1}^n x_i$ |
| **Median** $m$ | smallest $m$ with $F(m)\ge\tfrac12$ (continuous: solve $F(m)=\tfrac12$) | middle of sorted data ($n$ odd) or mean of the two middles ($n$ even) |
| **Mode** | value(s) maximising $p(x)$ (discrete) or $f(x)$ (continuous: $f'=0$ or boundary) | most frequent value(s) |

*Continuous‑mode mechanics:* for $f(x)=6x(1-x)$ on $[0,1]$, $f'(x)=6(1-2x)=0\Rightarrow$ **mode $=\tfrac12$**. (Always check endpoints too; e.g. a monotone density has its mode at a boundary.)

**Quartiles & percentiles.** The $p$‑th percentile $x_p$ solves $F(x_p)=p$. $Q_1=x_{0.25}$, $Q_2=$ median, $Q_3=x_{0.75}$.

**Mean–median–mode & skew (testable rule).** The **skewness** $\gamma_1=\dfrac{E[(X-\mu)^3]}{\sigma^3}$ (3rd standardised moment) signs the lopsidedness:

- **Symmetric** ($\gamma_1=0$): $\text{mean}=\text{median}=\text{mode}$.
- **Right / positive skew** ($\gamma_1>0$, long right tail): typically $\text{mode}<\text{median}<\text{mean}$.
- **Left / negative skew** ($\gamma_1<0$, long left tail): typically $\text{mean}<\text{median}<\text{mode}$.

*(Kurtosis $=\dfrac{E[(X-\mu)^4]}{\sigma^4}$ measures tail‑heaviness — know the name; rarely computed in GATE DA.)*

**Choosing a centre:** mean for symmetric, outlier‑free data; **median** for skewed data or when outliers are present; **mode** for categorical/most‑likely questions.

### 4.2 Spread

| Measure | Distribution (population) | Data (sample) |
|---|---|---|
| **Variance** | $\sigma^2=\mathrm{Var}(X)=E[X^2]-\mu^2$ | $s^2=\dfrac{1}{n-1}\displaystyle\sum_{i=1}^n (x_i-\bar{x})^2$ |
| **Std. deviation** | $\sigma=\sqrt{\sigma^2}$ | $s=\sqrt{s^2}$ |
| **Range** | $\max-\min$ (crude, outlier‑sensitive) | same |
| **IQR** | $Q_3-Q_1$ (robust) | same |

**Why $n-1$ (Bessel's correction).** For a *sample*, dividing by $n-1$ (not $n$) makes $s^2$ an **unbiased** estimator of the population $\sigma^2$: $E[s^2]=\sigma^2$. Dividing the *whole population's* squared deviations uses $n$. **Use $n-1$ for sample variance/SD, $n$ for population.** *(The unbiasedness reasoning links forward to estimation/inference.)*

**Standardisation.** $Z=\dfrac{X-\mu}{\sigma}$ has mean $0$, SD $1$ — dimensionless, comparable across variables (the backbone of the $z$‑score and Topic 12's standard normal).

### 4.3 Covariance — full treatment

$$\boxed{\,\mathrm{Cov}(X,Y)=E\big[(X-\mu_X)(Y-\mu_Y)\big]=E[XY]-E[X]E[Y]\,}\qquad \text{sample: } s_{XY}=\frac{1}{n-1}\sum_i (x_i-\bar x)(y_i-\bar y).$$

| Property | Statement |
|---|---|
| Self | $\mathrm{Cov}(X,X)=\mathrm{Var}(X)$ |
| Symmetry | $\mathrm{Cov}(X,Y)=\mathrm{Cov}(Y,X)$ |
| **Bilinearity** | $\mathrm{Cov}(aX+b,\,cY+d)=ac\,\mathrm{Cov}(X,Y)$ — **constants $b,d$ drop out** |
| Additivity | $\mathrm{Cov}(X,\,Y+Z)=\mathrm{Cov}(X,Y)+\mathrm{Cov}(X,Z)$ |
| Variance link | $\mathrm{Var}(aX+bY)=a^2\mathrm{Var}(X)+b^2\mathrm{Var}(Y)+2ab\,\mathrm{Cov}(X,Y)$ |
| Independence | $X\perp Y\Rightarrow \mathrm{Cov}(X,Y)=0$ — **converse false** |
| Cauchy–Schwarz | $\lvert\mathrm{Cov}(X,Y)\rvert\le \sigma_X\sigma_Y$ |

**Sign:** $\mathrm{Cov}>0$ ⇒ move together; $<0$ ⇒ move oppositely; $=0$ ⇒ no *linear* co‑movement. **Scale:** magnitude depends on units, so it is **not** comparable across variable pairs — hence correlation.

### 4.4 Correlation (Pearson)

$$\boxed{\,\rho_{XY}=\frac{\mathrm{Cov}(X,Y)}{\sigma_X\,\sigma_Y}\,}\qquad \text{sample: } r=\frac{\sum_i(x_i-\bar x)(y_i-\bar y)}{\sqrt{\sum_i(x_i-\bar x)^2}\,\sqrt{\sum_i(y_i-\bar y)^2}}.$$
*Conditions:* $\sigma_X,\sigma_Y>0$.

| Property | Statement |
|---|---|
| Range | $-1\le \rho\le 1$ (from Cauchy–Schwarz) |
| Unit‑free | $\rho$ is dimensionless (the $n-1$'s cancel in $r$) |
| Perfect linear | $\rho=\pm1 \iff Y=aX+b$ **exactly**, with $\operatorname{sign}(\rho)=\operatorname{sign}(a)$ |
| Transform | $\rho(aX+b,\,cY+d)=\operatorname{sign}(ac)\,\rho(X,Y)$ — shifts don't matter; a negative scale **flips the sign** |
| Uncorrelated | $\rho=0 \iff \mathrm{Cov}=0$; **$X\perp Y\Rightarrow\rho=0$ but not conversely** |
| **Linear only** | $\rho$ captures **linear** association; a strong *curved* relationship can have $\rho=0$ |

> **One line forward (ML):** for a vector of variables, all pairwise covariances form the **covariance matrix** (symmetric, positive semidefinite) — the object PCA diagonalises. Not developed here.

> **Derivations GATE does not ask:** proofs of Cauchy–Schwarz, of $E[s^2]=\sigma^2$, and of the $\rho=\pm1$ characterisation. Know and *use* them.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Mean, median, mode, and population vs sample SD (NAT) ★trap

Data: $\{3,\,5,\,5,\,7,\,10\}$. Find the mean, median, mode, the **sample** SD, and the **population** SD. Comment on skew.

**Solution.**
Sum $=30$, $n=5$ ⇒ **mean $\bar x=6$**. Sorted $3,5,5,7,10$ ⇒ **median $=5$** (3rd value). **Mode $=5$** (appears twice).
Deviations from $6$: $-3,-1,-1,1,4$; squares $9,1,1,1,16$; $\sum=28$.
$$s^2=\frac{28}{n-1}=\frac{28}{4}=7\Rightarrow \boxed{s=\sqrt7\approx 2.646}\ (\text{sample}),\qquad \sigma^2=\frac{28}{n}=\frac{28}{5}=5.6\Rightarrow \boxed{\sigma\approx 2.366}\ (\text{population}).$$
**Skew:** mean ($6$) $>$ median ($5$), so the data is **right‑skewed** — the value $10$ pulls the mean above the median.

> **Trap:** using $n=5$ for a *sample* variance gives $5.6$ instead of $7$. Match the divisor to the question: $n-1$ for a sample, $n$ for a full population.

### Problem 2 — Covariance and correlation from a joint PMF (NAT)

Joint PMF of $(X,Y)$: $p(0,0)=0.4,\ p(0,1)=0.2,\ p(1,0)=0.1,\ p(1,1)=0.3$. Find $\mathrm{Cov}(X,Y)$ and $\rho_{XY}$.

**Solution.**
Marginals: $p_X(1)=0.1+0.3=0.4$ (so $p_X(0)=0.6$); $p_Y(1)=0.2+0.3=0.5$.
$E[X]=0.4,\ E[Y]=0.5$. Since $X,Y\in\{0,1\}$: $E[X^2]=0.4$, $E[Y^2]=0.5$.
$E[XY]=\sum xy\,p(x,y)=1\cdot1\cdot 0.3=0.3$ (only the $(1,1)$ cell contributes).
$$\mathrm{Cov}(X,Y)=0.3-(0.4)(0.5)=\boxed{0.1}>0.$$
$\mathrm{Var}(X)=0.4-0.16=0.24$, $\mathrm{Var}(Y)=0.5-0.25=0.25$, so $\sigma_X=\sqrt{0.24}\approx0.490$, $\sigma_Y=0.5$.
$$\rho_{XY}=\frac{0.1}{0.490\times 0.5}=\frac{0.1}{0.245}\approx \boxed{0.408}.$$
A moderate **positive** linear association. *(As a check, $X,Y$ are dependent: $p(1,1)=0.3\ne p_X(1)p_Y(1)=0.20$ — consistent with $\rho\ne 0$.)*

### Problem 3 — Uncorrelated but dependent (the $\rho=0$ trap) (MSQ) ★trap

$X$ is uniform on $\{-1,0,1\}$ (each prob $\tfrac13$); let $Y=X^2$. Show $\rho_{XY}=0$ yet $X,Y$ are dependent.

**Solution.**
By symmetry $E[X]=0$ and $E[X^3]=\tfrac{(-1)^3+0+1^3}{3}=0$. Then
$$\mathrm{Cov}(X,Y)=E[XY]-E[X]E[Y]=E[X\cdot X^2]-0=E[X^3]=0\ \Rightarrow\ \boxed{\rho_{XY}=0}.$$
But $Y$ is a **deterministic function** of $X$ ($Y=X^2$), so they are as dependent as possible. Formally, $P(X=0,Y=0)=P(X=0)=\tfrac13$, whereas $P(X=0)P(Y=0)=\tfrac13\cdot\tfrac13=\tfrac19\ne\tfrac13$ — **dependent.**

> **Lesson:** $\rho$ detects only *linear* association. Zero correlation never proves independence; here a perfect *quadratic* dependence is invisible to $\rho$.

### Problem 4 — Covariance bilinearity & correlation under linear maps (NAT)

Given $\mathrm{Var}(X)=4$, $\mathrm{Var}(Y)=9$, $\mathrm{Cov}(X,Y)=-3$. Find **(a)** $\rho_{XY}$, **(b)** $\mathrm{Var}(2X-3Y)$, **(c)** $\rho(2X+1,\,-3Y+5)$.

**Solution.**
(a) $\rho_{XY}=\dfrac{-3}{\sqrt4\,\sqrt9}=\dfrac{-3}{2\cdot3}=\boxed{-0.5}$.
(b) $\mathrm{Var}(2X-3Y)=2^2(4)+(-3)^2(9)+2(2)(-3)\mathrm{Cov}(X,Y)=16+81+(-12)(-3)=16+81+36=\boxed{133}$.
(c) Shifts ($+1,+5$) don't affect correlation; scales are $a=2>0$, $c=-3<0$, so
$$\rho(2X+1,-3Y+5)=\operatorname{sign}(2\cdot(-3))\,\rho_{XY}=(-1)(-0.5)=\boxed{+0.5}.$$
The single negative scale **flips the sign** of the correlation.

---

## 6. Common GATE question types on this topic

- Compute **mean / median / mode** from a dataset or a distribution; find **percentiles/quartiles/IQR**.
- Compare **mean vs median** to infer **skew** direction; apply the mean–median–mode ordering.
- Compute **population vs sample** variance/SD (the $n$ vs $n-1$ choice).
- Compute **$\mathrm{Cov}$ and $\rho$** from a joint PMF/PDF or from paired data.
- Use **covariance bilinearity** to get $\mathrm{Var}$ of a linear combination.
- Effect of **linear transforms** on mean, variance, covariance, correlation (sign flips, shifts dropping out).
- Identify **uncorrelated‑but‑dependent** cases; recall $\rho$ measures *linear* association only.
- State the **range of $\rho$** and the meaning of $\rho=\pm1$, $\rho=0$.

---

## 7. Traps, pitfalls & common mistakes

- **Uncorrelated ≠ independent.** $\mathrm{Cov}=0$ (i.e. $\rho=0$) does **not** imply independence; only **independent ⇒ uncorrelated**. (#1 conceptual trap — see Problem 3.)
- **$\rho$ is linear‑only.** A strong nonlinear relationship can give $\rho\approx0$. "No correlation" ≠ "no relationship."
- **Sample uses $n-1$, population uses $n$.** Using the wrong divisor for variance/SD/covariance is a frequent slip; know that $n-1$ yields the *unbiased* estimate.
- **Mean ≠ median for skewed data**, and the mean is pulled toward the long tail. Don't assume symmetry.
- **Mode may be non‑unique or absent**; for a continuous density it's the **peak** (interior $f'=0$ *or* a boundary). Check endpoints.
- **Covariance scale/units.** Its magnitude isn't comparable across variable pairs — that's the whole reason to normalise to $\rho$.
- **Shifts vs scales.** Additive constants drop out of $\mathrm{Cov}$ and leave $\rho$ unchanged; **a negative scale flips $\rho$'s sign**; $\mathrm{Var}$ scales by the square.
- **$|\rho|>1$ is impossible** — a computed value outside $[-1,1]$ means an arithmetic error.
- **$\mathrm{Var}(X+Y)$ needs the $2\,\mathrm{Cov}$ term** unless the variables are uncorrelated (recap from Topic 9).
- **Outlier sensitivity.** Mean, SD, and range are easily distorted by outliers; median and IQR are robust — pick the right summary for the data.
- **Correlation ≠ causation** (interpretation questions).

---

## 8. Connection forward

This closes the **Random Variables & Expectation** block: you can now describe *any* distribution by its centre, spread, and (for pairs) association. The new chat opens with **Topic 11 — Discrete Distributions** (uniform, Bernoulli, binomial, Poisson), where each named PMF gets exactly these summaries — its mean, variance, mode and shape — computed with Topic 9's operators and reported in Topic 10's language. **Topic 12** does the same for continuous families (uniform, exponential, normal), whose mean‑median‑mode coincidence/skew you'll now recognise instantly. Further out, **Topics 13–16** treat the **sample mean and sample variance as random variables** and study their *sampling distributions* — promoting these descriptive statistics into the inferential tools (CLT, confidence intervals, $z$/$t$/$\chi^2$ tests) that finish the subject. In ML, the **covariance matrix** hinted at here is the core object of PCA.

---

## 9. One-line cheat-sheet summary

Centre: **mean** (balance point, outlier‑sensitive), **median** (50th percentile, robust), **mode** (peak); spread: **SD $=\sqrt{\mathrm{Var}}$** (population $\div n$, sample $\div(n-1)$); association: **$\mathrm{Cov}(X,Y)=E[XY]-E[X]E[Y]$** (scale‑dependent) and **$\rho=\mathrm{Cov}/(\sigma_X\sigma_Y)\in[-1,1]$** (unit‑free, *linear only*) — independent ⇒ $\rho=0$ but **not** conversely, and $\rho=\pm1$ iff an exact straight line.

---

**Next topic (begins a new chat): Topic 11 — Discrete Distributions (uniform, Bernoulli, binomial, Poisson).** Each named PMF will be summarised with the mean/variance/mode/correlation tools built across Topics 9–10.

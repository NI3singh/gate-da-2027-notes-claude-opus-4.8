# Topic 9 — Expectation, Variance and Conditional Expectation/Variance

*Subject: Probability and Statistics (GATE DA) · Cluster: Random Variables & Expectation*

> **Boundary note.** This topic owns the **expectation operator** $E[\cdot]$, **variance**, and their **conditional** versions plus the two great laws (total expectation, total variance). The *descriptive* re-readings — "mean vs median vs mode," sample SD, and the full theory of **covariance & correlation** — are **Topic 10**. Covariance is introduced here only as much as the variance‑of‑a‑sum formula demands.

---

## 0. Topic header & syllabus map

- **Topic:** Expectation, Variance, Conditional Expectation & Conditional Variance (with the laws of total expectation and total variance)
- **Subject:** Probability and Statistics
- **Where it sits:** Fourth topic of the *Random Variables & Expectation* block. It comes right after **Conditional PDF (Topic 8)** — which built whole conditional *distributions* — and extracts from any distribution its key **scalars**: where it's centred ($E[X]$) and how spread out it is ($\mathrm{Var}(X)$). Conditioning (Topic 8) + averaging (here) fuse into $E[X\mid Y]$, $\mathrm{Var}(X\mid Y)$ and the powerful **tower** and **Eve's law**. Leads into **Topic 10 — Summary Statistics**.

---

## 1. Why this topic exists (motivation + connection)

- **The real problem.** A full distribution (PMF/PDF) is a lot of information. To compare, decide, or report, we want **single numbers**: "on average, how much?" and "how variable is it?"
- **What the previous topic couldn't do.** Topics 6–8 describe distributions but produce no scalar summary — there is no built‑in "average value" or "spread" in a density; you cannot compare two distributions' centres or risks just by staring at $f$.
- **The fix.** **Expectation** is the probability‑weighted average — the distribution's *centre of mass*. **Variance** is the average squared deviation from that centre — its *spread*. And conditioning (Topic 8) combined with averaging gives **conditional** expectation/variance, whose **total‑law** versions let us compute otherwise‑hard $E$ and $\mathrm{Var}$ by *splitting a problem into stages*.

---

## 2. Prerequisites

- **Topics 6–7** — PMF and PDF/CDF (expectation is defined as a sum/integral against them).
- **Topic 8** — joint, marginal and **conditional** densities/PMFs; **independence** (needed for product/sum rules and for the laws).
- **Topic 3** — independence of events.
- **Calculus & series** — summation, definite/improper integrals, convergence.

---

## 3. Core idea — intuition first

**Expectation = balance point.** Place a mass $p_X(x)$ at each value $x$ (discrete) or smear mass with density $f_X(x)$ (continuous). The value at which the rod *balances* is $E[X]=\sum x\,p(x)$ or $\int x f(x)\,dx$. It need not be a value $X$ ever takes (the mean of a die is 3.5).

**Variance = average squared swing.** Measure each value's distance from the balance point, **square** it (so left and right deviations don't cancel and big deviations are penalised), and average: $\mathrm{Var}(X)=E[(X-\mu)^2]$. Its square root, the **standard deviation** $\sigma$, lives in the same units as $X$.

**Conditioning + averaging.** Given partial information $Y=y$, Topic 8 gave the *conditional distribution*; averaging it gives the **conditional mean** $E[X\mid Y=y]$ — your best single guess for $X$ once you know $Y=y$. Letting $y$ vary makes $E[X\mid Y]$ a **random variable**. Two laws then say: the overall average is the *average of the conditional averages* (**tower**), and the overall spread is *average within‑group spread* + *spread of the group means* (**Eve's law**).

**Tiny example.** $E[X]$ for $f(x)=2x$ on $[0,1]$ leans toward 1 (more mass on the right): $E[X]=\int_0^1 x\cdot2x\,dx=\tfrac23$, exactly as the rising density suggests.

---

## 4. Formal treatment

### 4.1 Expectation (mean) — definition & existence

$$\text{Discrete: } E[X]=\sum_x x\,p_X(x)\qquad\qquad \text{Continuous: } E[X]=\int_{-\infty}^{\infty} x\,f_X(x)\,dx.$$
*Existence condition:* the sum/integral must converge **absolutely** — $\sum|x|p(x)<\infty$ or $\int|x|f(x)\,dx<\infty$. If it diverges, **$E[X]$ does not exist** (some heavy‑tailed distributions have no mean). $E[c]=c$ for a constant $c$.

### 4.2 LOTUS — "Law of the Unconscious Statistician"

To average a *function* of $X$ you do **not** need the distribution of $g(X)$ first:
$$\boxed{\,E[g(X)]=\sum_x g(x)\,p_X(x)\quad\text{or}\quad \int_{-\infty}^{\infty} g(x)\,f_X(x)\,dx\,}$$
Two‑variable form: $E[g(X,Y)]=\sum_x\sum_y g(x,y)\,p_{X,Y}(x,y)$ or $\iint g(x,y)\,f_{X,Y}(x,y)\,dx\,dy$. *Condition:* absolute convergence.

### 4.3 Properties of expectation

| Property | Statement | Condition |
|---|---|---|
| **Linearity** | $E[aX+bY+c]=aE[X]+bE[Y]+c$ | **always** — even if $X,Y$ are *dependent* |
| Constant | $E[c]=c$ | — |
| Monotonicity | $X\le Y$ (always) $\Rightarrow E[X]\le E[Y]$ | — |
| Product (indep.) | $E[XY]=E[X]E[Y]$; more generally $E[g(X)h(Y)]=E[g(X)]E[h(Y)]$ | **$X\perp Y$ only** |
| Jensen | $\phi$ convex $\Rightarrow E[\phi(X)]\ge \phi(E[X])$ (reverse if concave) | $\phi$ convex/concave |

- **Linearity is the single most useful fact in the topic** and needs *no* independence. (Independence is needed for *products*, not sums.)
- *Special case of Jensen:* $E[X^2]\ge (E[X])^2$ (since $x^2$ is convex) — this is exactly why $\mathrm{Var}\ge 0$.
- **Best constant predictor (MMSE):** the constant $c$ minimising $E[(X-c)^2]$ is $c=E[X]$, with minimum value $\mathrm{Var}(X)$.

### 4.4 Variance & standard deviation

$$\mathrm{Var}(X)=E\big[(X-\mu)^2\big]\stackrel{\text{(expand)}}{=}\boxed{\,E[X^2]-\big(E[X]\big)^2\,},\qquad \sigma_X=\sqrt{\mathrm{Var}(X)}.$$
*(One‑line derivation: $E[(X-\mu)^2]=E[X^2-2\mu X+\mu^2]=E[X^2]-2\mu\,E[X]+\mu^2=E[X^2]-\mu^2$.)*
- $\mathrm{Var}(X)\ge 0$; $=0$ iff $X$ is constant (almost surely). Variance has **squared** units; SD has the **same units** as $X$.

**Covariance (introduced only as needed — full theory in Topic 10):**
$$\mathrm{Cov}(X,Y)=E\big[(X-\mu_X)(Y-\mu_Y)\big]=E[XY]-E[X]E[Y].$$

**Properties of variance:**

| Property | Statement | Condition |
|---|---|---|
| Shift | $\mathrm{Var}(X+b)=\mathrm{Var}(X)$ | constant $b$ — shifts don't change spread |
| Scale | $\mathrm{Var}(aX)=a^2\,\mathrm{Var}(X)$ | note $a^2$, and $\sigma_{aX+b}=\lvert a\rvert\sigma_X$ |
| Sum | $\mathrm{Var}(X\pm Y)=\mathrm{Var}(X)+\mathrm{Var}(Y)\pm 2\,\mathrm{Cov}(X,Y)$ | always |
| Sum (indep./uncorr.) | $\mathrm{Var}(X\pm Y)=\mathrm{Var}(X)+\mathrm{Var}(Y)$ | $\mathrm{Cov}=0$ — **variances add, even for $X-Y$** |
| General | $\mathrm{Var}\!\big(\sum_i a_iX_i\big)=\sum_i a_i^2\mathrm{Var}(X_i)+2\!\!\sum_{i<j}a_ia_j\mathrm{Cov}(X_i,X_j)$ | always |

### 4.5 Conditional expectation

- **As a number** (for fixed $y$): $\displaystyle E[X\mid Y=y]=\sum_x x\,p_{X\mid Y}(x\mid y)$ or $\int_{-\infty}^{\infty} x\,f_{X\mid Y}(x\mid y)\,dx$ — built on Topic 8's conditional distribution.
- **As a random variable:** $E[X\mid Y]$ is the function $g(Y)$ with $g(y)=E[X\mid Y=y]$. Being a function of the random $Y$, it is itself random.
- Conditional LOTUS & linearity hold: $E[aX_1+bX_2\mid Y]=aE[X_1\mid Y]+bE[X_2\mid Y]$.
- **Taking out what is known:** $E\big[h(Y)\,X\mid Y\big]=h(Y)\,E[X\mid Y]$, and $E[h(Y)\mid Y]=h(Y)$. *(Given $Y$, any function of $Y$ is a constant.)*

### 4.6 Law of total expectation (tower property)

$$\boxed{\,E[X]=E\big[E[X\mid Y]\big]\,}\quad\Longleftrightarrow\quad E[X]=\sum_y E[X\mid Y=y]\,p_Y(y)\ \text{ or }\ \int E[X\mid Y=y]\,f_Y(y)\,dy.$$
*Condition:* expectations exist. Reads as **"average the conditional averages."** The outer expectation is over $Y$. Generalises to $E[g(X)]=E\big[E[g(X)\mid Y]\big]$.

### 4.7 Conditional variance & law of total variance (Eve's law)

$$\mathrm{Var}(X\mid Y=y)=E[X^2\mid Y=y]-\big(E[X\mid Y=y]\big)^2,\qquad \mathrm{Var}(X\mid Y)\ \text{is a random variable.}$$
$$\boxed{\,\mathrm{Var}(X)=\underbrace{E\big[\mathrm{Var}(X\mid Y)\big]}_{\text{avg. within-group spread}}+\underbrace{\mathrm{Var}\big(E[X\mid Y]\big)}_{\text{spread of group means}}\,}$$
*Condition:* variances exist. **Two terms — forgetting the second ("between") term is the classic error.** Interpretation = unexplained + explained variance (the probabilistic ANOVA / bias–variance backbone).

*Short derivation (optional but worth knowing).* By tower, $E[X^2]=E\big[\mathrm{Var}(X\mid Y)+(E[X\mid Y])^2\big]=E[\mathrm{Var}(X\mid Y)]+E\big[(E[X\mid Y])^2\big]$. Subtract $(E[X])^2=(E[E[X\mid Y]])^2$; the last two terms combine into $\mathrm{Var}(E[X\mid Y])$. ∎

**Best predictor (MMSE), the conditional version.** Among **all** functions $g(Y)$, the one minimising $E[(X-g(Y))^2]$ is $g(Y)=E[X\mid Y]$ — the *regression function*. Its minimum error is $E[\mathrm{Var}(X\mid Y)]$. (This is why $E[X\mid Y]$ is the target of regression in ML.)

### 4.8 Named consequence — random sums (Wald's identities)

For $S=\sum_{i=1}^{N} X_i$ with $X_i$ i.i.d. (mean $\mu$, variance $\sigma^2$), $N$ a non‑negative integer RV **independent of the $X_i$**:
$$\boxed{\,E[S]=E[N]\,\mu\,}\qquad\boxed{\,\mathrm{Var}(S)=E[N]\,\sigma^2+\mathrm{Var}(N)\,\mu^2\,}$$
*Why:* $E[S\mid N=n]=n\mu\Rightarrow E[S]=\mu E[N]$ (tower); the variance follows from Eve's law with $E[S\mid N]=N\mu$, $\mathrm{Var}(S\mid N)=N\sigma^2$.

### 4.9 Supplementary tail bounds & moments *(appear occasionally)*

- **Markov:** for $X\ge 0$, $a>0$: $\;P(X\ge a)\le \dfrac{E[X]}{a}.$
- **Chebyshev:** $\;P\big(|X-\mu|\ge k\sigma\big)\le \dfrac{1}{k^2}$, equivalently $P\big(|X-\mu|\ge a\big)\le \dfrac{\sigma^2}{a^2}$. *(An upper bound — usually loose, not the exact probability. It is the bridge to the CLT/concentration story in Topic 14.)*
- **Moments:** $n$‑th moment $E[X^n]$; $n$‑th central moment $E[(X-\mu)^n]$ (variance is the 2nd). The **MGF** $M_X(t)=E[e^{tX}]$ generates moments via $E[X^n]=M_X^{(n)}(0)$ — *rarely required in GATE DA; know it exists.*

> **Derivations GATE does not ask:** proofs of Jensen, Markov, Chebyshev, and the MMSE optimality of $E[X\mid Y]$. Know the statements and use them; GATE never asks for the proofs.

---

## 5. Worked problems (GATE-style)

### Problem 1 — $E[X]$, $E[X^2]$, $\mathrm{Var}$, and LOTUS (NAT)

$X$ has density $f(x)=2x$ on $[0,1]$, else 0. Find $E[X]$, $E[X^2]$, $\mathrm{Var}(X)$, $\sigma_X$, and $E[1/X]$.

**Solution.**
$$E[X]=\int_0^1 x\cdot 2x\,dx=\int_0^1 2x^2\,dx=\tfrac23,\qquad E[X^2]=\int_0^1 x^2\cdot 2x\,dx=\int_0^1 2x^3\,dx=\tfrac12.$$
$$\mathrm{Var}(X)=E[X^2]-(E[X])^2=\tfrac12-\tfrac49=\tfrac{9-8}{18}=\boxed{\tfrac{1}{18}}\approx 0.0556,\qquad \sigma_X=\tfrac{1}{3\sqrt2}\approx 0.236.$$
LOTUS: $E[1/X]=\int_0^1 \tfrac1x\cdot 2x\,dx=\int_0^1 2\,dx=\boxed{2}.$

> Note $E[1/X]=2$ but $1/E[X]=\tfrac32=1.5$. They differ because $E[g(X)]\ne g(E[X])$ in general; since $1/x$ is convex, **Jensen** guarantees $E[1/X]\ge 1/E[X]$ ($2\ge1.5$ ✓).

### Problem 2 — Linearity & variance algebra (the sign / constant traps) (NAT) ★trap

$X,Y$ are **independent** with $E[X]=2,\ \mathrm{Var}(X)=3$ and $E[Y]=-1,\ \mathrm{Var}(Y)=4$. Find
**(a)** $E[2X-3Y+1]$, **(b)** $\mathrm{Var}(2X-3Y+1)$, **(c)** $E[X^2]$, **(d)** $\mathrm{Var}(X-Y)$.

**Solution.**
(a) $2(2)-3(-1)+1=4+3+1=\mathbf{8}$. *(linearity — no independence needed.)*
(b) $\mathrm{Var}(2X-3Y+1)=2^2\mathrm{Var}(X)+(-3)^2\mathrm{Var}(Y)=4(3)+9(4)=12+36=\mathbf{48}$. *(the "$+1$" drops; both terms are positive even though it is $-3Y$.)*
(c) $E[X^2]=\mathrm{Var}(X)+(E[X])^2=3+4=\mathbf{7}$.
(d) $\mathrm{Var}(X-Y)=\mathrm{Var}(X)+\mathrm{Var}(Y)=3+4=\mathbf{7}$ (independent). **Not** $3-4=-1$ — variance can never be negative; for a difference of independent RVs the variances still **add**.

### Problem 3 — Conditional expectation from a joint density + tower check (NAT)

$(X,Y)$ has joint density $f_{X,Y}(x,y)=x+y$ on the unit square $[0,1]^2$ (from Topic 8: $f_X(x)=x+\tfrac12$, $f_{X\mid Y}(x\mid y)=\tfrac{x+y}{y+1/2}$). **(a)** Find $E[X\mid Y=y]$. **(b)** Verify $E\big[E[X\mid Y]\big]=E[X]$.

**Solution.**
(a) $\displaystyle E[X\mid Y=y]=\int_0^1 x\cdot\frac{x+y}{y+\tfrac12}\,dx=\frac{1}{y+\tfrac12}\Big(\tfrac13+\tfrac{y}{2}\Big)=\boxed{\dfrac{2+3y}{3(1+2y)}}.$ *(Check $y=0$: $\tfrac23$; $y=1$: $\tfrac59$ — both match direct integration.)*
(b) Using $f_Y(y)=y+\tfrac12=\tfrac{1+2y}{2}$, the $(1+2y)$ cancels neatly:
$$E\big[E[X\mid Y]\big]=\int_0^1 \frac{2+3y}{3(1+2y)}\cdot\frac{1+2y}{2}\,dy=\int_0^1 \frac{2+3y}{6}\,dy=\frac{1}{6}\Big(2+\tfrac32\Big)=\frac{7}{12}.$$
Direct: $E[X]=\int_0^1 x\big(x+\tfrac12\big)\,dx=\tfrac13+\tfrac14=\tfrac{7}{12}$ ✓. The tower holds.

### Problem 4 — Both laws at once: a two‑stage (hierarchical) model (NAT) ★showcase

Draw $p\sim\text{Uniform}(0,1)$; then flip a coin with success probability $p$ and set $X=1$ for heads, $0$ for tails. Find $E[X]$ and $\mathrm{Var}(X)$ — and confirm $\mathrm{Var}(X)$ via the law of total variance.

**Solution.**
Given $p$: $E[X\mid p]=p$ and $\mathrm{Var}(X\mid p)=p(1-p)$ (a Bernoulli($p$)).
**Tower:** $E[X]=E[\,E[X\mid p]\,]=E[p]=\boxed{\tfrac12}.$
Since $X\in\{0,1\}$ with $P(X=1)=\tfrac12$, directly $\mathrm{Var}(X)=\tfrac12\cdot\tfrac12=\boxed{\tfrac14}.$
**Eve's law check.** With $E[p]=\tfrac12$, $E[p^2]=\int_0^1 p^2\,dp=\tfrac13$, $\mathrm{Var}(p)=\tfrac13-\tfrac14=\tfrac{1}{12}$:
$$E[\mathrm{Var}(X\mid p)]=E[p-p^2]=\tfrac12-\tfrac13=\tfrac16,\qquad \mathrm{Var}(E[X\mid p])=\mathrm{Var}(p)=\tfrac{1}{12}.$$
Sum $=\tfrac16+\tfrac{1}{12}=\tfrac{2+1}{12}=\tfrac{3}{12}=\tfrac14$ ✓ — matches. *(This is exactly how Wald‑type and mixture variances are computed.)*

---

## 6. Common GATE question types on this topic

- Compute $E[X]$, $E[X^2]$, $\mathrm{Var}(X)$, $\sigma_X$ from a **PMF or PDF**.
- Use **LOTUS** for $E[g(X)]$ without finding $g(X)$'s distribution.
- Apply **linearity** $E[aX+bY+c]$ and **variance algebra** $\mathrm{Var}(aX+bY)$ (independent, or with covariance).
- $E[XY]$ for **independent vs dependent**; spot the *uncorrelated ≠ independent* subtlety (Topic 10).
- **Conditional expectation** $E[X\mid Y=y]$ from a joint distribution.
- **Law of total expectation** (two‑stage / mixture / **random‑sum / Wald**).
- **Law of total variance / variance decomposition.**
- **Markov / Chebyshev** tail bounds (occasionally).
- The **constant** minimising MSE ($=E[X]$); the **function of $Y$** minimising MSE ($=E[X\mid Y]$).

---

## 7. Traps, pitfalls & common mistakes

- **$\mathrm{Var}(X-Y)$ still adds.** For independent/uncorrelated RVs $\mathrm{Var}(X-Y)=\mathrm{Var}(X)+\mathrm{Var}(Y)$, **not** the difference. Variance is never negative. (#1 trap.)
- **Variance scaling is $a^2$, not $a$.** $\mathrm{Var}(aX+b)=a^2\mathrm{Var}(X)$; the additive constant $b$ contributes **0**.
- **$E[g(X)]\ne g(E[X])$ (Jensen).** In particular $E[X^2]\ne(E[X])^2$ and $E[1/X]\ne 1/E[X]$. Don't "pull $g$ outside."
- **$E[XY]=E[X]E[Y]$ requires independence** (or at least zero covariance) — not true in general.
- **Linearity does *not* require independence** (reverse trap): $E[X+Y]=E[X]+E[Y]$ always holds.
- **Existence.** $E[X]$ needs **absolute** convergence; heavy‑tailed RVs may have no mean or no variance.
- **$E[X\mid Y]$ is a random variable**; $E[X\mid Y=y]$ is a number. The tower's outer expectation is over $Y$: $E[E[X\mid Y]]=E[X]$ (a number), not $E[X\mid Y]$.
- **Taking out what's known.** $E[h(Y)\,X\mid Y]=h(Y)E[X\mid Y]$ — functions of the conditioning variable come **out**; forgetting this corrupts conditional computations.
- **Eve's law has two terms.** Using only $E[\mathrm{Var}(X\mid Y)]$ and dropping $\mathrm{Var}(E[X\mid Y])$ **undercounts** the variance.
- **Units / standardising.** SD shares units with $X$; variance is squared. The standardised variable $Z=\frac{X-\mu}{\sigma}$ has mean $0$, variance $1$ (used constantly from Topic 12 onward).
- **Negative variance = error.** If a computation yields $\mathrm{Var}<0$, you dropped a sign or a covariance term.
- **Chebyshev is a bound.** It gives an upper limit on a tail probability, usually loose — never the exact value.

---

## 8. Connection forward

Topic 10 (**Summary Statistics**) re‑reads $E[X]$ as the **mean** and compares it with the **median** and **mode** (which coincide only for symmetric/unimodal shapes), formalises **SD** as the standard measure of spread, and completes the **covariance/correlation** story we only previewed here (turning $\mathrm{Cov}$ into the unit‑free correlation $\rho=\mathrm{Cov}(X,Y)/(\sigma_X\sigma_Y)$ and clarifying *uncorrelated ≠ independent*). From there, **Topics 11–12** simply *apply* these formulas to compute $E$ and $\mathrm{Var}$ of the named distributions, and **Topic 14 (CLT)** studies how the **mean and variance of an average** behave — with **Chebyshev** the first crude version of that concentration story. In ML, **Eve's law** is the seed of the bias–variance decomposition.

---

## 9. One-line cheat-sheet summary

$E[X]$ is the probability‑weighted average ($\sum xp$ or $\int xf$) and $\mathrm{Var}(X)=E[X^2]-\mu^2$; **linearity** $E[aX+bY+c]=aE[X]+bE[Y]+c$ holds *always*, while $\mathrm{Var}(aX+b)=a^2\mathrm{Var}(X)$ and $\mathrm{Var}(X\pm Y)=\mathrm{Var}X+\mathrm{Var}Y\pm2\mathrm{Cov}$; condition‑then‑average with the **tower** $E[X]=E[E[X\mid Y]]$ and **Eve's law** $\mathrm{Var}(X)=E[\mathrm{Var}(X\mid Y)]+\mathrm{Var}(E[X\mid Y])$.

---

**Next topic (in order): Topic 10 — Summary Statistics (mean, median, mode, standard deviation, covariance & correlation).** It re-reads $E[X]$ and $\mathrm{Var}(X)$ as descriptive measures and completes the covariance → correlation story previewed here.

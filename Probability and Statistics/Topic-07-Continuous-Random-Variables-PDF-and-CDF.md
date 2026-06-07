# Topic 7 — Continuous Random Variables, PDF and CDF

*Subject: Probability and Statistics (GATE DA) · Cluster: Random Variables & Expectation*

> **Terminology note (read first).** The official syllabus phrase *"probability distribution function"* refers, in this topic, to the **probability density function (PDF)** — the function you *integrate*. The **cumulative distribution function (CDF)** is its running integral. Throughout: **PDF = density $f(x)$**, **CDF = $F(x)=P(X\le x)$**.

---

## 0. Topic header & syllabus map

- **Topic:** Continuous Random Variables, Probability Density Function (PDF) & Cumulative Distribution Function (CDF)
- **Subject:** Probability and Statistics
- **Where it sits:** Second topic of the *Random Variables & Expectation* block. It comes immediately after **Discrete RVs & PMF (Topic 6)** and is its mirror image: same questions, but for variables whose values fill a continuum rather than a list. The **CDF you met in Topic 6** is the object that carries over unchanged — here it becomes smooth, and its slope is the density. Leads into **Topic 8 — Conditional PDF**, which restricts and renormalizes the density you define here.

---

## 1. Why this topic exists (motivation + connection)

- **The real problem.** Many quantities are *continuous*: a waiting time, a measured length, a temperature, a voltage. Their possible values form an **interval**, not a list — between any two values there is always another.
- **What the previous topic couldn't do.** A PMF assigns *positive* probability to individual points and **sums** them. For a continuous quantity there are *uncountably many* possible values; if each had positive probability the total would blow past 1. The only consistent choice is $P(X=x)=0$ for **every** single $x$ — so the PMF is identically zero and carries **no information**, and summation has nothing to sum.
- **The fix.** Stop asking *"how much probability sits exactly at this point"* (always 0) and instead ask *"how much probability per unit length near this point."* That rate is the **density** $f(x)$. Probability becomes **area under $f$**. Crucially, the **CDF $F(x)=P(X\le x)$** from Topic 6 still works perfectly — it is the bridge — and its **derivative is the density**.
- **Forward.** Topic 8 conditions this density on an event or another variable (conditional PDF); Topic 12 fills in the *named* continuous distributions (uniform, exponential, normal) as special cases of exactly this framework.

---

## 2. Prerequisites

- **Topic 6** — discrete RVs, and especially the **CDF and all its properties** (non-decreasing, right-continuous, limits 0 and 1). The CDF is the shared backbone.
- **Integral calculus** — definite & indefinite integrals, the **Fundamental Theorem of Calculus** ($\frac{d}{dx}\int_a^x f = f(x)$), integrating polynomials and $e^{kx}$.
- **Improper integrals** — $\int_{-\infty}^{\infty}$, and convergence of such integrals.
- **Differentiation** — needed for $f = F'$ and for the Jacobian in transformations.
- **Topic 4 ideas** (conditional/joint probability) — reused next, in Topic 8.

---

## 3. Core idea — intuition first

In Topic 6 probability was **piled** onto isolated points. Now imagine **smearing** the same total amount of probability (which must still equal 1) **continuously** along the number line, like spreading a fixed kilogram of butter over a slice of bread. The density $f(x)$ is *how thickly the butter lies at position $x$* — grams **per centimetre**, not grams. To get the actual probability (the grams) in a region you must **integrate** the thickness over that region.

Two consequences fall out immediately and define the whole topic:

1. **The probability "at a single point" is the area over a single point — which is zero.** A vertical line has no width, hence no area: $P(X=x)=0$. This is the headline difference from discrete RVs.
2. **The density is *not* a probability.** Thickness can be large; $f(x)$ may exceed 1 (a thin tall slice). What is bounded by 1 is the *area*, never the height.

**Tiny example.** Let $f(x)=2x$ for $0\le x\le 1$ (probability spread lightly near 0, heavily near 1). The chance $X$ lands in the right half is the area $\int_{0.5}^{1} 2x\,dx = [x^2]_{0.5}^{1}=1-0.25=0.75$ — far more likely to land near 1, exactly as the rising density suggests.

**Reading the CDF.** $F(x)$ is the **accumulated** butter from the far left up to $x$. It rises smoothly from 0 to 1, and its **slope at $x$ is precisely the thickness $f(x)$** there.

---

## 4. Formal treatment

### Definitions

- **Continuous random variable.** $X$ is continuous if its CDF $F_X$ is continuous everywhere; equivalently, there is a non-negative function $f_X$ (the density) with
$$F_X(x) = \int_{-\infty}^{x} f_X(t)\,dt \quad\text{for all } x.$$
- **Support.** $S_X = \{\, x : f_X(x) > 0 \,\}$ (up to sets of measure zero). Integrate **only** over the support.

### Properties of a valid PDF

| | Property | Condition / meaning |
|---|---|---|
| **P1** | $f_X(x) \ge 0$ for all $x$ | density is non‑negative |
| **P2** | $\displaystyle\int_{-\infty}^{\infty} f_X(x)\,dx = 1$ | total area (total probability) is 1 |
| **P3** | $\displaystyle P(X \in A) = \int_A f_X(x)\,dx$ | probability of a value‑set $A$ is its area |

*Conditions:* $f_X\ge 0$ and the improper integral in P2 **converges** to exactly 1. Note P1 allows $f_X(x)>1$ — heights are unbounded; only areas are capped.

### PDF ↔ CDF (the two-way link)

$$\boxed{\,F_X(x)=\int_{-\infty}^{x} f_X(t)\,dt\,}\qquad\qquad \boxed{\,f_X(x)=\dfrac{d}{dx}F_X(x)\,}$$
*Condition on the right:* $f_X(x)=F_X'(x)$ holds **wherever $F_X$ is differentiable**. At isolated kinks $F_X$ may fail to be differentiable; you may define $f_X$ arbitrarily there (a measure‑zero set never affects any probability).

### Properties of the CDF (identical list to Topic 6 — this is the bridge)

- **C1.** Non-decreasing.
- **C2.** Continuous everywhere (no jumps), for a purely continuous RV. *(In general the CDF is only right‑continuous; a jump would signal a point mass — see the note on mixed distributions below.)*
- **C3.** $\lim_{x\to-\infty}F_X(x)=0$, $\lim_{x\to+\infty}F_X(x)=1$.
- **C4.** $P(a < X \le b) = F_X(b) - F_X(a)$.

### Probability of an interval — and the "all four are equal" fact

$$P(a \le X \le b) = \int_a^b f_X(x)\,dx = F_X(b) - F_X(a).$$
Because $P(X=a)=P(X=b)=0$ for a continuous RV,
$$\boxed{P(a\le X\le b)=P(a< X< b)=P(a< X\le b)=P(a\le X< b).}$$
The strict-vs-non-strict distinction that mattered so much for discrete RVs **disappears** here. (Watch the reverse trap below.)

### Density interpretation (infinitesimal)

$$P(x < X \le x+dx) \approx f_X(x)\,dx.$$
"Density × width ≈ probability." This is why $f_X$ is a *rate*, not a probability.

### Median and quantiles

- **Median** $m$: the value with half the probability on each side, i.e. solve $F_X(m)=\tfrac12$.
- **$p$-th quantile** $x_p$: solve $F_X(x_p)=p$. *(For a continuous, strictly increasing $F$ on the support, $x_p$ is unique.)*

### Functions of a continuous RV: $Y=g(X)$

**Method A — CDF method (universal, always safe).**
1. Write $F_Y(y)=P\big(g(X)\le y\big)$.
2. Solve the inequality $g(X)\le y$ for $X$ to get an $X$-region.
3. Express that probability through $F_X$ (or as an integral of $f_X$).
4. Differentiate: $f_Y(y)=F_Y'(y)$. **Determine the support of $Y$ separately** (the image of $S_X$).

**Method B — change-of-variables / Jacobian formula.**

- *If $g$ is monotonic and differentiable* on the support, with inverse $x=g^{-1}(y)$:
$$f_Y(y) = f_X\!\big(g^{-1}(y)\big)\,\left|\dfrac{d}{dy}\,g^{-1}(y)\right| = \dfrac{f_X(x)}{|g'(x)|}\bigg|_{\,x=g^{-1}(y)}.$$
- *If $g$ is not monotonic* (several $x$ give the same $y$), **sum over all real roots** $x_1,\dots,x_k$ of $g(x)=y$:
$$\boxed{\,f_Y(y) = \sum_{i=1}^{k} \dfrac{f_X(x_i)}{|g'(x_i)|}\,}.$$
This is the continuous analogue of Topic 6's *"sum the PMF over every preimage."*

**Key special case — linear transform $Y=aX+b$ ($a\ne 0$).** Here $g^{-1}(y)=\frac{y-b}{a}$ and $|g'|=|a|$, so
$$f_Y(y)=\frac{1}{|a|}\,f_X\!\left(\frac{y-b}{a}\right).$$
(Scaling by $a$ stretches the axis by $|a|$, so the density must shrink by $1/|a|$ to keep total area 1.)

### Expectation — *preview only* (fully in Topic 9)

So that density problems are self-contained:
$$E[X]=\int_{-\infty}^{\infty} x\,f_X(x)\,dx,\qquad E[g(X)]=\int_{-\infty}^{\infty} g(x)\,f_X(x)\,dx\ \text{(LOTUS)},$$
each valid **provided the integral converges absolutely**. Variance and conditional expectation are Topic 9 — do not develop them here.

### One line on *mixed* distributions (be aware, rarely tested)

If a CDF is mostly smooth but has a **jump** at some point $a$, that jump (size $F(a)-F(a^-)$) is an actual **point mass** $P(X=a)>0$ — such an $X$ is *mixed*, neither purely continuous nor purely discrete. For a *mixed* RV the "$P(X=x)=0$" rule fails **at the jump points**. GATE DA mostly tests purely continuous RVs, but recognise a jump for what it is.

> **Derivations GATE does not ask (skipped on purpose):** the measure-theoretic definition of absolute continuity / existence of densities, and the formal proof of the change-of-variables formula. GATE never requires these.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Constant, CDF, interval probability, median (NAT)

$X$ has density $f(x)=c\,x^2$ for $0\le x\le 2$, and 0 otherwise.
**(a)** Find $c$. **(b)** Find $F(x)$. **(c)** $P(X\le 1)$ and $P(0.5<X<1.5)$. **(d)** The median.

**Solution.**
(a) $\displaystyle\int_0^2 c\,x^2\,dx = c\cdot\frac{8}{3}=1 \Rightarrow \boxed{c=\tfrac38}$.
(b) For $0\le x\le 2$: $F(x)=\int_0^x \tfrac38 t^2\,dt = \tfrac38\cdot\tfrac{x^3}{3}=\dfrac{x^3}{8}$. So
$$F(x)=\begin{cases}0 & x<0\\ x^3/8 & 0\le x\le 2\\ 1 & x>2.\end{cases}\quad(\text{check }F(2)=1\ \checkmark)$$
(c) $P(X\le 1)=F(1)=\mathbf{1/8}=0.125$. $\;P(0.5<X<1.5)=F(1.5)-F(0.5)=\dfrac{1.5^3-0.5^3}{8}=\dfrac{3.375-0.125}{8}=\dfrac{3.25}{8}=\mathbf{0.40625}$.
(d) $F(m)=\tfrac12 \Rightarrow \dfrac{m^3}{8}=\tfrac12 \Rightarrow m^3=4 \Rightarrow m=\sqrt[3]{4}\approx \mathbf{1.587}$.

### Problem 2 — From CDF to PDF; $P(X=\text{point})=0$ (MCQ/MSQ)

A continuous RV has CDF $F(x)=0$ for $x<0$, $F(x)=x^2$ for $0\le x<1$, $F(x)=1$ for $x\ge 1$.
**(a)** Verify it is a valid CDF and find $f(x)$. **(b)** $P(X\le 0.5)$, $P(X=0.5)$, $P(X>0.5)$. **(c)** $P(0.25<X<0.75)$.

**Solution.**
(a) Non-decreasing on $[0,1)$ since $F'=2x\ge 0$; continuous at $0$ ($F=0$) and at $1$ ($F=1$); limits $0$ and $1$. ✓ Valid. Then
$$f(x)=F'(x)=\begin{cases}2x & 0<x<1\\ 0 & \text{otherwise.}\end{cases}\quad\Big(\int_0^1 2x\,dx=1\ \checkmark\Big)$$
(b) $P(X\le 0.5)=F(0.5)=\mathbf{0.25}$; $\;P(X=0.5)=\mathbf{0}$ (continuous RV); $\;P(X>0.5)=1-F(0.5)=\mathbf{0.75}$.
(c) $P(0.25<X<0.75)=F(0.75)-F(0.25)=0.5625-0.0625=\mathbf{0.5}$.

> **Takeaway:** every point has probability 0, so $P(X\le0.5)=P(X<0.5)$ here — the opposite of the discrete world.

### Problem 3 — Transformation, monotonic (Jacobian) (NAT)

$X$ has density $f_X(x)=2x$ on $[0,1]$. Let $Y=X^2$. Find $f_Y(y)$.

**Solution (CDF method).** For $0\le y\le 1$: $F_Y(y)=P(X^2\le y)=P(0\le X\le\sqrt y)=F_X(\sqrt y)$. Since $F_X(x)=\int_0^x 2t\,dt=x^2$, we get $F_Y(y)=(\sqrt y)^2=y$. Differentiate:
$$f_Y(y)=\begin{cases}1 & 0<y<1\\ 0 & \text{otherwise.}\end{cases}$$
So $Y\sim\text{Uniform}(0,1)$.
**Check (Jacobian).** $g(x)=x^2$ is increasing on $(0,1)$, $x=\sqrt y$, $\frac{dx}{dy}=\frac{1}{2\sqrt y}$:
$f_Y(y)=f_X(\sqrt y)\big|\tfrac{1}{2\sqrt y}\big|=2\sqrt y\cdot\frac{1}{2\sqrt y}=1$ ✓.

### Problem 4 — Transformation, non-monotonic (the branch trap) (NAT)

$X\sim\text{Uniform}(-1,1)$, i.e. $f_X(x)=\tfrac12$ on $[-1,1]$. Let $Y=X^2$. Find $f_Y(y)$.

**Solution.** Now $g(x)=x^2$ is **not** monotonic on $(-1,1)$. For $0<y<1$ the equation $x^2=y$ has **two** roots $x=\pm\sqrt y$, both inside $(-1,1)$, with $|g'(\pm\sqrt y)|=2\sqrt y$. Summing over both branches:
$$f_Y(y)=\frac{f_X(\sqrt y)}{2\sqrt y}+\frac{f_X(-\sqrt y)}{2\sqrt y}=\frac{1/2}{2\sqrt y}+\frac{1/2}{2\sqrt y}=\boxed{\frac{1}{2\sqrt y}},\quad 0<y<1.$$
**Check (CDF method):** $F_Y(y)=P(-\sqrt y\le X\le \sqrt y)=\int_{-\sqrt y}^{\sqrt y}\tfrac12\,dx=\sqrt y$, so $f_Y=\frac{d}{dy}\sqrt y=\frac{1}{2\sqrt y}$ ✓, and $\int_0^1\frac{1}{2\sqrt y}\,dy=1$ ✓.

> **Trap:** dropping the negative branch gives $\frac{1}{4\sqrt y}$ — exactly **half** the right answer. The same $g(x)=x^2$ gave a *uniform* density in Problem 3 but a *different* density here, purely because the domain made it non-monotonic. **Always count branches.**

### Problem 5 — Which functions are valid PDFs? (MSQ)

Which of the following are valid probability density functions? (Select all.)
- **(a)** $f(x)=2$ for $0\le x\le 0.5$, else 0.
- **(b)** $f(x)=3x^2$ for $-1\le x\le 1$, else 0.
- **(c)** $f(x)=\tfrac38 x^2$ for $0\le x\le 2$, else 0.
- **(d)** $f(x)=1-x$ for $0\le x\le 2$, else 0.

**Solution.**
- (a) $f\ge 0$ ✓; $\int_0^{0.5} 2\,dx = 1$ ✓. **VALID** — note $f=2>1$ is perfectly fine (height is not a probability).
- (b) $f\ge 0$ ✓ but $\int_{-1}^{1} 3x^2\,dx=[x^3]_{-1}^{1}=2\ne 1$. **INVALID** (area is 2).
- (c) $f\ge 0$ ✓; $\int_0^2 \tfrac38 x^2\,dx=1$ ✓ (Problem 1). **VALID.**
- (d) At $x=2$, $f=-1<0$. **INVALID** (negative; also $\int_0^2(1-x)\,dx=0$).

**Answer: (a) and (c).** This single question hits all three failure modes: density *may* exceed 1 (a), but must be **non-negative** (d fails) and must **integrate to exactly 1** (b fails).

---

## 6. Common GATE question types on this topic

- Find the **normalizing constant $c$** in a density (finite or infinite support; ensure the integral converges).
- Compute $P(a<X<b)$, $P(X>a)$ by **integrating $f$** or via $F(b)-F(a)$.
- Go **$f\to F$** (integrate, piecewise) or **$F\to f$** (differentiate); evaluate $F$ at a point.
- Find the **median / quantile / percentile** from $F$.
- **Transformation $Y=g(X)$**: find $f_Y$ or $F_Y$ — linear ($aX+b$), square ($X^2$), exponential ($e^X$), etc.; **monotonic and non-monotonic**.
- **"Which is a valid PDF / CDF"** identification (MSQ) — check non-negativity, area 1, and (for CDFs) monotone + limits.
- Combined with Topic 9: compute $E[X]$, $\mathrm{Var}(X)$, $E[g(X)]$ from a density.

---

## 7. Traps, pitfalls & common mistakes

- **Reverse strict/non-strict trap.** For a *continuous* RV all four interval forms are equal because $P(X=x)=0$ — so do **not** waste effort separating them. **But** if the CDF has a **jump** (mixed RV), $P(X=\text{jump point})>0$ and the equality fails *there*; don't blindly assume zero.
- **Density is not a probability.** $f(x)$ may be $>1$; never report "$P(X=x)=f(x)$." Only **areas** are probabilities.
- **Support discipline when integrating.** Integrate **only** where $f>0$. Plugging limits outside the support over- or under-counts. In transformations, **the support of $Y$ is the image of the support of $X$** — getting it wrong is a classic error.
- **Branch counting in transformations.** For non-monotonic $g$, sum $f_X(x_i)/|g'(x_i)|$ over **all** real roots; missing a branch (the $X^2$ trap) typically halves the density.
- **Absolute value in the Jacobian.** Use $|g'(x)|$ (or $\left|\tfrac{dx}{dy}\right|$). Omitting it can produce a **negative** density — instantly wrong.
- **$f=F'$ only where $F$ is differentiable.** At kinks, define $f$ however you like; it doesn't change any probability (measure‑zero set).
- **Validity checks.** After solving for $c$: confirm $f\ge 0$ on the *whole* support **and** that the (possibly improper) integral converges to **exactly 1**, not just to a finite number.
- **"Probability distribution function" ≠ CDF.** In this syllabus that phrase means the **density**; the cumulative one is the **CDF**. Don't differentiate/integrate the wrong object.
- **Median.** Solve $F(m)=\tfrac12$, not $f(m)=\tfrac12$. For a symmetric density, the median equals the centre of symmetry.
- **Upper/lower limit mix-up.** $P(a<X\le b)=F(b)-F(a)$ with $b$ the **upper** limit; swapping gives a negative probability.

---

## 8. Connection forward

Topic 8 (**Conditional PDF**) takes the density defined here and **conditions** it. Just as Topic 4 turned $P(A)$ into $P(A\mid B)=P(A\cap B)/P(B)$, we will turn $f_X(x)$ into a *conditional* density by **restricting to a sub-region and renormalizing so the area is again 1** — e.g., the density of $X$ given $X>a$, or, in a joint setting, the density of $X$ given $Y=y$. The integral machinery (area = probability, $f=F'$) carries over verbatim; only the normalizing constant changes. Further out, **Topic 9** computes $E[X]=\int x f$, and **Topic 12** names the standard densities (uniform, exponential, normal) you've been working with in disguise.

---

## 9. One-line cheat-sheet summary

A continuous RV has $P(X=x)=0$; its **density** $f\ge 0$ integrates to 1, **probability is area** $\int_a^b f=F(b)-F(a)$, the **CDF** $F(x)=\int_{-\infty}^x f$ satisfies $f=F'$, and for $Y=g(X)$ transform by $f_Y(y)=\sum_i f_X(x_i)/|g'(x_i)|$ over **all** roots $x_i$ of $g(x)=y$ (or use the CDF method) — density can exceed 1, only the area can't.

---

**Next topic (in order): Topic 8 — Conditional PDF.** It restricts and renormalizes the density defined here, bringing Topic 4's conditioning idea into the continuous world.

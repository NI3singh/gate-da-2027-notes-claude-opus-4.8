# Topic 8 — Conditional PDF

*Subject: Probability and Statistics (GATE DA) · Cluster: Random Variables & Expectation*

> **Scope note.** "Conditional PDF" cannot be defined without **joint** and **marginal** densities — the conditional density of $X$ given $Y$ is literally *built* from them. So this topic necessarily packages: (i) conditioning a continuous RV on an **event**, (ii) **joint & marginal** densities (the continuous version of Topic 6's joint PMF and Topic 4's joint/marginal probability), (iii) the **conditional density** $f_{X\mid Y}$, (iv) **independence** of continuous RVs, and (v) **continuous Bayes**. Conditional *expectation* is previewed only — it is fully Topic 9.

---

## 0. Topic header & syllabus map

- **Topic:** Conditional PDF (with joint & marginal densities, independence, and continuous Bayes)
- **Subject:** Probability and Statistics
- **Where it sits:** Third topic of the *Random Variables & Expectation* block. It comes right after **Continuous RVs, PDF & CDF (Topic 7)** — which described a single continuous variable *in isolation* — and adds the ability to **update that description given information**. It is the continuous twin of **Topic 4** (conditional probability) and **Topic 5** (Bayes). It leads into **Topic 9 — Expectation, Variance & Conditional Expectation/Variance**, which *averages* against the conditional densities defined here.

---

## 1. Why this topic exists (motivation + connection)

- **The real problem.** We rarely sit in pure ignorance. We learn partial facts — "the wait has already exceeded 5 minutes," "the first sensor read 3.2," "$Y$ came out equal to $y$" — and we want to **revise** the distribution of $X$ in light of that.
- **What the previous topic couldn't do.** A plain density $f_X$ describes $X$ *alone*; it has no slot for "given that…" and no way to encode a *relationship* between two variables. There is no operation inside a single‑variable PDF that incorporates an observation.
- **The fix.** A **conditional PDF** restricts the density to the region consistent with what we know and **renormalizes to area 1**. When the condition is "$Y=y$" — a *probability‑zero* event for a continuous $Y$ — we cannot divide probabilities (that would be $0/0$), so we **define** the conditional density through the joint and marginal densities. This is exactly Topic 4's $P(A\mid B)=P(A\cap B)/P(B)$ and Topic 5's Bayes, rewritten for densities.

---

## 2. Prerequisites

- **Topic 7** — single‑variable continuous density, CDF, *area = probability*, normalization to 1, and the fact $P(Y=y)=0$.
- **Topic 4** — conditional probability $P(A\mid B)=P(A\cap B)/P(B)$, the multiplication rule, and the **law of total probability**.
- **Topic 5** — **Bayes' theorem** (we build its continuous form here).
- **Topic 6** — **joint PMF** and **marginalization** (the discrete parallel of everything below).
- **Double integrals** — integrating a 2‑variable function over a region, and *integrating out* one variable (limits may be **functions** of the kept variable).

---

## 3. Core idea — intuition first

Conditioning means **zoom in, then rescale to area 1**.

**Conditioning on an event** (say $X>a$): throw away the part of the density outside the event, keep the rest, and **scale it up** by dividing by $P(X>a)$ so the surviving piece again has total area 1 (probability must still sum to one *within the new, restricted world*).

**Conditioning on $Y=y$**: picture the joint density as a *surface* $f_{X,Y}(x,y)$ sitting over the $xy$‑plane. Slice it with the vertical plane $Y=y$. The cross‑section you expose — read as a curve in $x$ — has the right **shape** for "$X$ given $Y=y$," but the **wrong total area** (it isn't 1). The area of that slice, in the marginal sense, is exactly $f_Y(y)$. So divide the slice by $f_Y(y)$:
$$f_{X\mid Y}(x\mid y) = \frac{f_{X,Y}(x,y)}{f_Y(y)}.$$
That single rescaling turns a slice into an honest probability density.

**Tiny example.** If $X$ has density $f(x)=3x^2$ on $[0,1]$ and we learn $X>\tfrac12$, the surviving piece is $3x^2$ on $(\tfrac12,1]$ with area $P(X>\tfrac12)=\tfrac78$. Scaling up: $f_{X\mid X>1/2}(x)=\dfrac{3x^2}{7/8}=\dfrac{24x^2}{7}$ on $(\tfrac12,1]$ — same shape, now integrating to 1.

---

## 4. Formal treatment

### 4.1 Conditioning on an event $B$ with $P(B)>0$

For an event $B=\{X\in S\}$ (e.g. $X>a$, or $a<X<b$):
$$\boxed{\,f_{X\mid B}(x) = \dfrac{f_X(x)}{P(B)}\ \text{for } x\in S,\quad 0\ \text{otherwise}\,}\qquad P(B)=\int_S f_X(x)\,dx.$$
*Conditions:* $P(B)>0$. The result is a valid density on $S$ (it integrates to 1). The **conditional CDF** is $F_{X\mid B}(x)=P(X\le x\mid B)$, and conditional probabilities are $P(X\in A\mid B)=\dfrac{P(X\in A\cap S)}{P(B)}=\displaystyle\int_{A\cap S} f_{X\mid B}(x)\,dx.$

### 4.2 Joint PDF (two continuous variables)

A pair $(X,Y)$ has joint density $f_{X,Y}(x,y)$ if

| | Property | Meaning |
|---|---|---|
| **J1** | $f_{X,Y}(x,y)\ge 0$ | non‑negative |
| **J2** | $\displaystyle\iint_{\mathbb{R}^2} f_{X,Y}(x,y)\,dx\,dy = 1$ | total volume is 1 |
| **J3** | $\displaystyle P\big((X,Y)\in R\big)=\iint_R f_{X,Y}(x,y)\,dx\,dy$ | probability of a region = its volume |

*(Probability is now **volume** under a surface, just as in 1‑D it was area under a curve.)*

### 4.3 Marginal PDF — integrate the other variable out

$$\boxed{\,f_X(x)=\int_{-\infty}^{\infty} f_{X,Y}(x,y)\,dy\,}\qquad \boxed{\,f_Y(y)=\int_{-\infty}^{\infty} f_{X,Y}(x,y)\,dx\,}$$
*Condition (critical):* integrate over the **full range of the other variable allowed by the support**. If the support couples the variables (e.g. $0<x<y<1$), the limits are **functions** of the kept variable, *not* fixed constants. This is exactly Topic 4's marginalization / Topic 6's "sum out the other variable," now an integral.

### 4.4 Conditional PDF of $X$ given $Y=y$ (the headline)

$$\boxed{\,f_{X\mid Y}(x\mid y)=\dfrac{f_{X,Y}(x,y)}{f_Y(y)}\,}\qquad\text{(requires }f_Y(y)>0).$$
- This is a **valid density in $x$ for each fixed $y$**: $\displaystyle\int_{-\infty}^{\infty} f_{X\mid Y}(x\mid y)\,dx = \frac{1}{f_Y(y)}\int_{-\infty}^{\infty} f_{X,Y}(x,y)\,dx = \frac{f_Y(y)}{f_Y(y)}=1.$
- **It integrates to 1 over $x$, not over $y$.** The conditioning value $y$ is *frozen*.
- Conditional CDF / probability: $F_{X\mid Y}(x\mid y)=\displaystyle\int_{-\infty}^{x} f_{X\mid Y}(t\mid y)\,dt$, and $P(X\in A\mid Y=y)=\displaystyle\int_A f_{X\mid Y}(x\mid y)\,dx.$
- Symmetrically, $f_{Y\mid X}(y\mid x)=f_{X,Y}(x,y)/f_X(x)$.

> **Why we can't use $P(A\cap B)/P(B)$ here.** $B=\{Y=y\}$ has $P(B)=0$, so that ratio is $0/0$. The density definition above is the rigorous replacement; it agrees with the limit of $P(X\le x\mid y<Y\le y+\varepsilon)$ as $\varepsilon\to 0$. (The deeper subtlety — that conditioning on different probability‑zero events can give different answers, the *Borel paradox* — is beyond GATE.)

### 4.5 Multiplication rule, total probability, continuous Bayes

- **Multiplication (chain) rule:** $\;f_{X,Y}(x,y)=f_{X\mid Y}(x\mid y)\,f_Y(y)=f_{Y\mid X}(y\mid x)\,f_X(x).$
- **Law of total probability (continuous):** $\;f_Y(y)=\displaystyle\int_{-\infty}^{\infty} f_{Y\mid X}(y\mid x)\,f_X(x)\,dx.$
- **Bayes' theorem (continuous):**
$$\boxed{\,f_{X\mid Y}(x\mid y)=\dfrac{f_{Y\mid X}(y\mid x)\,f_X(x)}{\displaystyle\int_{-\infty}^{\infty} f_{Y\mid X}(y\mid x')\,f_X(x')\,dx'}\,}$$
This *flips* the conditioning — "posterior $\propto$ likelihood $\times$ prior," the engine behind Bayesian ML. *(Mixed case: if one variable is discrete and the other continuous, replace the corresponding integral by a sum and the matching density by a PMF — the logic is identical.)*

### 4.6 Independence of continuous RVs

$$X \perp Y \iff f_{X,Y}(x,y)=f_X(x)\,f_Y(y)\ \text{for all }(x,y)\iff f_{X\mid Y}(x\mid y)=f_X(x)\ \text{(whenever defined)}.$$
**Practical test (two parts, both required):**
1. The **support is a product/rectangle** — the allowed range of $x$ does **not** depend on $y$ (and vice versa); **and**
2. the density **factors** as $g(x)\,h(y)$ on that support.
If the support is *not* rectangular (a triangle, a disc, $x<y$, …), $X$ and $Y$ are **dependent** — no matter how nicely the formula factors.

### 4.7 The discrete parallel (bridge to Topic 6)

Everything mirrors the discrete case with **sums** in place of integrals:
$$p_{X\mid Y}(x\mid y)=\frac{p_{X,Y}(x,y)}{p_Y(y)},\qquad p_Y(y)=\sum_x p_{X,Y}(x,y),\qquad X\perp Y \iff p_{X,Y}=p_X p_Y.$$
Same idea, same Bayes flip — only the integration symbol changes.

### 4.8 Conditional expectation — *preview only* (fully in Topic 9)

For completeness: $E[X\mid Y=y]=\displaystyle\int_{-\infty}^{\infty} x\,f_{X\mid Y}(x\mid y)\,dx$, and the **law of total expectation** $E[X]=\displaystyle\int E[X\mid Y=y]\,f_Y(y)\,dy=E\big[E[X\mid Y]\big]$. These — plus conditional variance and the law of total variance — are **Topic 9**; don't develop them here.

> **Derivations GATE does not ask (skipped on purpose):** the rigorous limiting construction of $f_{X\mid Y}$ (and the Borel paradox), and measure‑theoretic conditional expectation. GATE never requires these.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Conditioning on an event (restrict + renormalize) (NAT)

$X$ has density $f_X(x)=3x^2$ on $[0,1]$, else 0. **(a)** Find $f_{X\mid X>1/2}(x)$. **(b)** Compute $P\big(X>\tfrac34 \,\big|\, X>\tfrac12\big)$.

**Solution.**
$P(X>\tfrac12)=\int_{1/2}^{1} 3x^2\,dx=[x^3]_{1/2}^{1}=1-\tfrac18=\tfrac78.$
(a) $f_{X\mid X>1/2}(x)=\dfrac{3x^2}{7/8}=\dfrac{24x^2}{7}$ for $\tfrac12<x\le 1$, else 0. *(Check: $\int_{1/2}^1 \tfrac{24}{7}x^2\,dx=1$ ✓.)*
(b) $P\big(X>\tfrac34\mid X>\tfrac12\big)=\dfrac{P(X>3/4)}{P(X>1/2)}=\dfrac{1-(3/4)^3}{7/8}=\dfrac{1-27/64}{7/8}=\dfrac{37/64}{56/64}=\boxed{\dfrac{37}{56}\approx 0.661}.$

### Problem 2 — Joint → marginal → conditional (MCQ/NAT)

$(X,Y)$ has joint density $f_{X,Y}(x,y)=c\,(x+y)$ on the unit square $0\le x\le1,\ 0\le y\le1$, else 0.
**(a)** Find $c$. **(b)** Find $f_X(x)$ and $f_{X\mid Y}(x\mid y)$. **(c)** Are $X,Y$ independent? **(d)** $P\big(X<\tfrac12 \,\big|\, Y=\tfrac12\big)$.

**Solution.**
(a) $\displaystyle\int_0^1\!\!\int_0^1 c(x+y)\,dx\,dy=c\int_0^1\!\Big(\tfrac12+y\Big)dy=c\big[\tfrac{y}{2}+\tfrac{y^2}{2}\big]_0^1=c\cdot1=1\Rightarrow \boxed{c=1}.$
(b) $f_X(x)=\int_0^1 (x+y)\,dy=\big[xy+\tfrac{y^2}{2}\big]_0^1=x+\tfrac12$ on $[0,1]$; by symmetry $f_Y(y)=y+\tfrac12$. Then
$$f_{X\mid Y}(x\mid y)=\frac{x+y}{\,y+\tfrac12\,},\quad 0\le x\le 1\ \ (\text{fixed }y).\qquad(\text{Check: }\textstyle\int_0^1\frac{x+y}{y+1/2}dx=\frac{1/2+y}{y+1/2}=1\ \checkmark)$$
(c) $f_X f_Y=(x+\tfrac12)(y+\tfrac12)\ne x+y$ (e.g. at $x=y=0$: $\tfrac14\ne0$). **Dependent.**
(d) $f_{X\mid Y}(x\mid\tfrac12)=\dfrac{x+1/2}{1/2+1/2}=x+\tfrac12$, so $P\big(X<\tfrac12\mid Y=\tfrac12\big)=\int_0^{1/2}(x+\tfrac12)\,dx=\big[\tfrac{x^2}{2}+\tfrac{x}{2}\big]_0^{1/2}=\tfrac18+\tfrac14=\boxed{\tfrac38=0.375}.$

### Problem 3 — Independence + the support trap (MSQ) ★trap

$(X,Y)$ has joint density $f_{X,Y}(x,y)=8xy$ on the triangle $0<x<y<1$, else 0.
**(a)** Verify it is a density. **(b)** Find $f_X,f_Y$. **(c)** Independent? **(d)** Find $f_{X\mid Y}(x\mid y)$.

**Solution.**
(a) $\displaystyle\int_0^1\!\!\int_0^{y} 8xy\,dx\,dy=\int_0^1 8y\cdot\tfrac{y^2}{2}\,dy=\int_0^1 4y^3\,dy=1$ ✓. *(Note the inner limit is $0\to y$ — the support couples the variables.)*
(b) $f_Y(y)=\int_0^{y} 8xy\,dx=4y^3$ for $0<y<1$. $\;f_X(x)=\int_{x}^{1} 8xy\,dy=4x(1-x^2)=4x-4x^3$ for $0<x<1$.
(c) **Dependent.** The support is a **triangle**, so the allowed $x$‑range ($0<x<y$) depends on $y$ — a product support fails immediately, regardless of any factoring. *(Indeed $f_Xf_Y=(4x-4x^3)(4y^3)\ne 8xy$.)*
(d) $f_{X\mid Y}(x\mid y)=\dfrac{8xy}{4y^3}=\dfrac{2x}{y^2}$ for $0<x<y$, else 0. *(Check: $\int_0^{y}\tfrac{2x}{y^2}\,dx=1$ ✓.)* Note the conditional support $0<x<y$ **moves with $y$**.

> **The trap:** candidates "test independence" by checking only whether the formula factors and miss that a triangular/curved support **by itself** forces dependence. Always check the support shape *first*.

### Problem 4 — Continuous Bayes / total probability (NAT) ★harder

Let $X\sim\text{Uniform}(0,1)$, and given $X=x$ let $Y\mid X=x \sim \text{Uniform}(0,x)$, i.e. $f_{Y\mid X}(y\mid x)=\dfrac1x$ for $0<y<x<1$.
**(a)** Find the joint density and the marginal $f_Y(y)$. **(b)** Find the posterior $f_{X\mid Y}(x\mid y)$.

**Solution.**
(a) Multiplication rule: $f_{X,Y}(x,y)=f_{Y\mid X}(y\mid x)\,f_X(x)=\dfrac1x\cdot 1=\dfrac1x$ on $\{0<y<x<1\}$. Marginal (total probability):
$$f_Y(y)=\int_{x=y}^{1}\frac1x\,dx=[\ln x]_{y}^{1}=-\ln y,\quad 0<y<1.\qquad(\text{valid: }-\ln y>0)$$
(b) Bayes:
$$f_{X\mid Y}(x\mid y)=\frac{f_{X,Y}(x,y)}{f_Y(y)}=\frac{1/x}{-\ln y}=\boxed{\dfrac{1}{-\,x\ln y}},\quad y<x<1.$$
*(Check normalization: $\int_y^1 \dfrac{1}{-x\ln y}\,dx=\dfrac{1}{-\ln y}\int_y^1\dfrac{dx}{x}=\dfrac{-\ln y}{-\ln y}=1$ ✓.)*
Here the prior on $X$ was flat, but observing $Y=y$ rules out all $x\le y$ and **tilts** the posterior toward larger $x$ — Bayes updating in action.

---

## 6. Common GATE question types on this topic

- **Condition on an event** ($X>a$, $a<X<b$): find the renormalized conditional density and a conditional probability.
- **Given a joint density:** find the constant $c$, the **marginals**, the **conditional density**, and $P(X\in A\mid Y=y)$.
- **Independence test** from a joint density — check **rectangular support** *and* **factorization**.
- **Continuous Bayes / posterior density**; the **law of total probability** for densities.
- **Mixed** discrete–continuous conditioning (sum on one side, integral on the other).
- Combined with Topic 9: **conditional expectation** $E[X\mid Y=y]$ from the conditional density.

---

## 7. Traps, pitfalls & common mistakes

- **Conditioning on $Y=y$ is conditioning on a probability‑zero event.** You **cannot** write $P(A\cap B)/P(B)$ (it is $0/0$); you **must** use $f_{X\mid Y}=f_{X,Y}/f_Y$. This is the conceptual headline of the topic.
- **Independence needs a product support *and* factorization.** A density that factors algebraically but lives on a **non‑rectangular** support (triangle, disc, $x<y$) describes **dependent** variables. Check the support shape first — it's the #1 GATE error here.
- **Renormalize after restricting.** Conditioning on an event means **dividing by $P(\text{event})$**; skip it and your "density" won't integrate to 1.
- **The conditioning value is frozen; integrate over the *other* variable.** $f_{X\mid Y}(x\mid y)$ must integrate to 1 **over $x$** (with $y$ fixed), not over $y$. Integrating the wrong variable is a classic slip.
- **The conditional support can depend on $y$.** As in Problem 3, $f_{X\mid Y}(x\mid y)$ lived on $0<x<y$. Using fixed limits $0$ to $1$ gives the wrong probability.
- **Marginalizing with the wrong limits.** When the support couples the variables, the integration limits are **functions** of the kept variable. Using constant $0$–$1$ limits on a triangle double‑counts.
- **Bayes direction.** $f_{X\mid Y}\ne f_{Y\mid X}$ in general — don't swap them; the denominator is the marginal of the **conditioning** variable.
- **$f_Y(y)=0\Rightarrow$ undefined.** The conditional density needs $f_Y(y)>0$.
- **Density, not probability (carried from Topic 7).** $f_{X\mid Y}(x\mid y)$ is a density and may exceed 1; never read it as "the probability that $X=x$ given $Y=y$."

---

## 8. Connection forward

Topic 9 (**Expectation, Variance & Conditional Expectation/Variance**) does the one thing we've been holding back: it **averages** against the densities built here. The conditional density $f_{X\mid Y}(\cdot\mid y)$ is precisely the weighting used to define $E[X\mid Y=y]=\int x\,f_{X\mid Y}(x\mid y)\,dx$ and $\mathrm{Var}(X\mid Y=y)$. Those feed the **law of total expectation** $E[X]=E\big[E[X\mid Y]\big]$ and the **law of total variance** $\mathrm{Var}(X)=E[\mathrm{Var}(X\mid Y)]+\mathrm{Var}(E[X\mid Y])$ — both heavily used in GATE and in ML (e.g., the bias–variance story). In short: this topic builds the *conditional distributions*; Topic 9 extracts their *numbers*.

---

## 9. One-line cheat-sheet summary

To condition on an **event**, restrict the density and divide by the event's probability; to condition on **$Y=y$** (a zero‑probability event) use $f_{X\mid Y}(x\mid y)=f_{X,Y}(x,y)/f_Y(y)$ with $f_Y(y)=\int f_{X,Y}\,dx$; **independence** $\iff f_{X,Y}=f_Xf_Y$ on a **rectangular** support; and **continuous Bayes** flips the conditioning via posterior $\propto$ likelihood $\times$ prior.

---

**Next topic (in order): Topic 9 — Expectation, Variance and Conditional Expectation/Variance.** It averages against the (conditional) densities defined here to produce $E[X]$, $\mathrm{Var}(X)$, $E[X\mid Y]$, and the laws of total expectation and variance.

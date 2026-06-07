# Topic 4 — Taylor Series

> **Subject:** Calculus and Optimization
> **Section:** Calculus & Single-Variable Optimization (Topic 4 of 5)
> **GATE DA syllabus line covered:** *"…differentiability, Taylor series, maxima and minima…"* (the *Taylor series* part)

---

## 0. Topic header & syllabus map

- **Topic:** Taylor Series
- **Belongs to:** Calculus and Optimization → the *Calculus & single-variable optimization* cluster.
- **Where it sits:** **Topic 4 of 5.** It comes **after Differentiability** (Topic 3) — it is built entirely from higher-order derivatives at a point — and leads into the final topic, **Maxima, Minima & Single-Variable Optimization** (Topic 5), whose second-derivative test is just the quadratic Taylor expansion in disguise.

In one line: *the tangent line was the best straight-line copy of $f$ near a point; the Taylor series is the best polynomial copy, matching $f$'s value, slope, curvature, and every higher derivative.*

---

## 1. Why this topic exists (motivation + connection)

**The real problem it solves.** We can only *evaluate* polynomials by hand — they need just $+,-,\times$. But the functions we actually care about ($e^x$, $\sin x$, $\ln x$) have no finite arithmetic recipe. Taylor series lets us replace a complicated function, **near a chosen point**, by a polynomial that agrees with it to any order we want — with a remainder we can bound. That single idea powers calculators, numerical methods, error analysis, and (crucially for us) limit evaluation and optimization.

**What the previous topic (Differentiability) couldn't do.** The derivative handed us the **tangent line** — the best *first-order* (linear) approximation near $a$. But a line ignores curvature. The tangent to $\sin x$ at $0$ is just $y=x$, which is poor once $x$ leaves a tiny neighbourhood. Differentiability, using **one** derivative, can only do linear. Taylor uses $f''(a),f'''(a),\dots$ to add curvature and ever-finer corrections, producing a polynomial that hugs the curve far more tightly.

**The bridge, stated plainly:** *Taylor series exists because the tangent line (first order, one derivative) is too crude away from the point; feeding in all the higher derivatives at the point builds a polynomial matching $f$ to any order.*

**What the next topic builds on it.** At an interior critical point $a$ (where $f'(a)=0$), the **second-order** Taylor expansion collapses to
$$f(x)-f(a)\approx \tfrac12 f''(a)(x-a)^2,$$
so the **sign of $f''(a)$** decides local min vs local max. That is precisely the **second-derivative test** of Topic 5 — Taylor is its justification.

---

## 2. Prerequisites

- **Topic 3 — Differentiability:** higher-order derivatives $f',f'',\dots,f^{(n)}$; the tangent line; LMVT.
- **Topic 2 — Limits:** indeterminate forms and L'Hôpital (Taylor is the faster alternative for many limits).
- **Topic 1 — Functions:** even/odd symmetry, standard functions.
- **Algebra:** factorials, $\Sigma$-notation, polynomial arithmetic, the geometric series.

---

## 3. Core idea — intuition first

We want a polynomial $P(x)$ that **agrees with $f$ at $x=a$ to high order**: same value, same slope, same curvature, same third derivative, and so on. The more derivatives we match, the better $P$ tracks $f$ near $a$.

**Analogy (a tailor fitting a curve).** First pin the point (0th order: a constant). Then match the slope (1st order: the tangent line). Then bend it to match the curvature (2nd order: a parabola). Each new term is a *correction* that shrinks the previous approximation's error.

**Why the $\tfrac{1}{k!}$ shows up.** The coefficient of $(x-a)^k$ is $\dfrac{f^{(k)}(a)}{k!}$. Differentiating $(x-a)^k$ exactly $k$ times produces a factor $k!$; dividing by $k!$ is what makes the $k$-th derivative of $P$ equal $f^{(k)}(a)$ at $a$. So each coefficient is engineered to match one more derivative.

**Tiny example.** Approximating $e^x$ near $0$: $\ 1\ \to\ 1+x\ \to\ 1+x+\tfrac{x^2}{2}\ \to\ \dots$ — each step matches one more derivative and fits a wider range.

---

## 4. Formal treatment

### 4.1 Taylor polynomial and Taylor series

The **$n$-th Taylor polynomial** of $f$ about $a$ (needs $f$ to be $n$-times differentiable at $a$):
$$P_n(x)=\sum_{k=0}^{n}\frac{f^{(k)}(a)}{k!}(x-a)^k=f(a)+f'(a)(x-a)+\frac{f''(a)}{2!}(x-a)^2+\cdots+\frac{f^{(n)}(a)}{n!}(x-a)^n.$$

The **Taylor series** ($n\to\infty$; needs $f$ infinitely differentiable near $a$):
$$f(x)=\sum_{k=0}^{\infty}\frac{f^{(k)}(a)}{k!}(x-a)^k,$$
valid **where it converges AND the remainder $\to 0$** (see §4.3).

### 4.2 Maclaurin series — the special case $a=0$

$$f(x)=\sum_{k=0}^{\infty}\frac{f^{(k)}(0)}{k!}x^k.$$
*Maclaurin is just "Taylor at $0$" — not a separate object.*

### 4.3 Taylor's theorem with remainder (the rigorous version)

Write $f(x)=P_n(x)+R_n(x)$, where $R_n$ is the **remainder (truncation error)**.

> **Lagrange form of the remainder.** If $f$ is $(n+1)$-times differentiable on the interval between $a$ and $x$, then there exists $c$ between $a$ and $x$ with
> $$R_n(x)=\frac{f^{(n+1)}(c)}{(n+1)!}(x-a)^{n+1}.$$

- **Beautiful link to Topic 3:** the case $n=0$ reads $f(x)=f(a)+f'(c)(x-a)$ — that is exactly **Lagrange's MVT**. So Taylor's theorem *generalises* the Mean Value Theorem to higher order.
- The series converges to $f$ at $x$ $\iff R_n(x)\to0$ as $n\to\infty$.
- **Error bound:** $\ \big|R_n(x)\big|\le \dfrac{M}{(n+1)!}\,|x-a|^{\,n+1}$, where $M=\max\big|f^{(n+1)}\big|$ on the interval between $a$ and $x$. Use this for "bound the error" and "how many terms for given accuracy."

*(Cauchy and integral forms of the remainder also exist; Lagrange is the GATE-relevant one — one line is enough on the others.)*

### 4.4 Standard Maclaurin expansions — MEMORISE (with intervals of validity)

$$e^{x}=\sum_{k=0}^{\infty}\frac{x^k}{k!}=1+x+\frac{x^2}{2!}+\frac{x^3}{3!}+\cdots\qquad(\text{all }x)$$

$$\sin x=\sum_{k=0}^{\infty}\frac{(-1)^k x^{2k+1}}{(2k+1)!}=x-\frac{x^3}{3!}+\frac{x^5}{5!}-\cdots\qquad(\text{all }x)$$

$$\cos x=\sum_{k=0}^{\infty}\frac{(-1)^k x^{2k}}{(2k)!}=1-\frac{x^2}{2!}+\frac{x^4}{4!}-\cdots\qquad(\text{all }x)$$

$$\ln(1+x)=\sum_{k=1}^{\infty}\frac{(-1)^{k-1}x^k}{k}=x-\frac{x^2}{2}+\frac{x^3}{3}-\cdots\qquad(-1<x\le1)$$

$$\frac{1}{1-x}=\sum_{k=0}^{\infty}x^k=1+x+x^2+\cdots,\qquad \frac{1}{1+x}=1-x+x^2-\cdots\qquad(|x|<1)$$

$$(1+x)^{\alpha}=\sum_{k=0}^{\infty}\binom{\alpha}{k}x^k=1+\alpha x+\frac{\alpha(\alpha-1)}{2!}x^2+\cdots\qquad(|x|<1,\ \alpha\notin\mathbb{Z}_{\ge0})$$

$$\tan x=x+\frac{x^3}{3}+\frac{2x^5}{15}+\cdots\ \Big(|x|<\tfrac{\pi}{2}\Big),\qquad \arctan x=x-\frac{x^3}{3}+\frac{x^5}{5}-\cdots\ \big(|x|\le1\big)$$

> **Convergence at a glance:** $e^x,\sin,\cos$ converge for **all** $x$; $\ \ln(1+x),\ \tfrac{1}{1\pm x},\ (1+x)^\alpha,\ \arctan$ are valid only on **restricted intervals** ($|x|<1$ etc.). The generalised binomial coefficient is $\binom{\alpha}{k}=\dfrac{\alpha(\alpha-1)\cdots(\alpha-k+1)}{k!}$.

### 4.5 Building new series from known ones (the fast way)

Computing many derivatives by hand is slow. Instead, manufacture series from the standard list:

- **Substitution:** $e^{-x^2}=\sum\dfrac{(-x^2)^k}{k!}=1-x^2+\dfrac{x^4}{2!}-\cdots$
- **Multiply / divide** two known series (collect by powers).
- **Differentiate / integrate term-by-term** (valid inside the radius of convergence): integrating $\dfrac{1}{1+x}$ gives $\ln(1+x)$; differentiating $\dfrac{1}{1-x}$ gives $\dfrac{1}{(1-x)^2}=\sum k x^{k-1}$.
- **Add / scale** series.

These tricks answer "find the coefficient of $x^n$ in …" almost instantly.

### 4.6 Convergence of a power series (proportionate — appears occasionally)

A power series $\sum c_k(x-a)^k$ has a **radius of convergence $R$**: it converges for $|x-a|<R$, diverges for $|x-a|>R$, and the **endpoints** must be checked separately.

> **Ratio test:** $\ R=\displaystyle\lim_{k\to\infty}\left|\frac{c_k}{c_{k+1}}\right|$ (when the limit exists), equivalently $\dfrac1R=\lim\left|\dfrac{c_{k+1}}{c_k}\right|$.

Examples: $e^x\Rightarrow R=\infty$; geometric / $\ln(1+x)$ / $\arctan\Rightarrow R=1$. *(GATE DA leans more on expansion + limits + error bounds than on convergence radius, but it can surface — know the ratio test.)*

### 4.7 Linear and quadratic approximation (the bridge)

- **Linear (1st order):** $\ f(x)\approx f(a)+f'(a)(x-a)$ — exactly the tangent line / order-1 Taylor polynomial. Used for quick estimates and error propagation.
- **Quadratic (2nd order):** $\ f(x)\approx f(a)+f'(a)(x-a)+\tfrac12 f''(a)(x-a)^2$ — adds curvature.
- **Little-$o$ statement:** $\ f(x)=P_n(x)+o\big((x-a)^n\big)$ as $x\to a$ (the error is negligible compared to $(x-a)^n$).

The quadratic form is the seed of the second-derivative test (Topic 5): at a critical point the linear term dies, leaving the curvature term to classify the extremum.

---

## 5. Worked problems (GATE-style)

### Problem 1 — A limit that Taylor crushes (and L'Hôpital labours over) *(NAT)*

Evaluate $\displaystyle\lim_{x\to0}\frac{\tan x-\sin x}{x^{3}}.$

**Solution.** Expand both to order $x^3$:
$$\tan x=x+\frac{x^3}{3}+\cdots,\qquad \sin x=x-\frac{x^3}{6}+\cdots.$$
$$\tan x-\sin x=\Big(\frac{x^3}{3}+\frac{x^3}{6}\Big)+\cdots=x^3\Big(\frac{2+1}{6}\Big)+\cdots=\frac{x^3}{2}+\cdots.$$
$$\Rightarrow\ \lim_{x\to0}\frac{\tan x-\sin x}{x^3}=\boxed{\tfrac12}.$$

> **Why Taylor here:** L'Hôpital on this $\tfrac00$ form needs **three** differentiations with messy $\sec^2,\sec^2\tan$ terms. Taylor reads the answer off the leading coefficient. *Trap:* you must keep terms **up to $x^3$** — truncating at $x^1$ wrongly gives $0/x^3$.

---

### Problem 2 — A high derivative via series substitution *(MCQ / NAT)*

Find $f^{(4)}(0)$ for $f(x)=e^{-x^{2}}$.

**Solution.** Differentiating $e^{-x^2}$ four times directly is painful. Instead substitute into the $e^u$ series with $u=-x^2$:
$$e^{-x^2}=\sum_{k=0}^{\infty}\frac{(-x^2)^k}{k!}=1-x^2+\frac{x^4}{2!}-\frac{x^6}{3!}+\cdots$$
The coefficient of $x^4$ is $\dfrac{1}{2!}=\dfrac12$. Since the coefficient of $x^4$ equals $\dfrac{f^{(4)}(0)}{4!}$,
$$\frac{f^{(4)}(0)}{4!}=\frac12\ \Rightarrow\ f^{(4)}(0)=4!\cdot\tfrac12=24\cdot\tfrac12=\boxed{12}.$$

> **Lesson:** **coefficient of $x^k$** $\times\,k!$ **$=f^{(k)}(0)$**. (Also note: $e^{-x^2}$ is even, so all *odd* derivatives at $0$ vanish — a free sanity check.)

---

### Problem 3 — Bounding the truncation error (Lagrange remainder) *(NAT, trap)*

Approximate $e^{0.1}$ by its second-degree Maclaurin polynomial and bound the error.

**Solution.** $P_2(x)=1+x+\dfrac{x^2}{2}$, so
$$P_2(0.1)=1+0.1+\frac{(0.1)^2}{2}=1+0.1+0.005=1.105.$$
The Lagrange remainder for $f(x)=e^x$, $a=0$, $n=2$ is
$$R_2(0.1)=\frac{f'''(c)}{3!}(0.1)^3=\frac{e^{c}}{6}(0.001),\qquad c\in(0,0.1).$$
We don't know $c$, so we **bound** $f'''=e^c$ by its maximum on $[0,0.1]$: $e^c\le e^{0.1}<1.2$. Hence
$$\big|R_2(0.1)\big|\le \frac{1.2}{6}\,(0.001)=0.2\times0.001=2\times10^{-4}.$$
$$\boxed{\,e^{0.1}\approx1.105,\quad \text{error}<2\times10^{-4}.}$$
*(Check: true $e^{0.1}=1.10517\ldots$, actual error $\approx1.7\times10^{-4}<2\times10^{-4}$.)* ✔

> **Trap:** $c$ is unknown — you must use the **maximum** of $f^{(n+1)}$ on the interval, not plug in a particular $c$ or use $f^{(n+1)}(0)$. Picking any valid, easy upper bound (here $1.2$) is the move.

---

## 6. Common GATE question types on this topic

- **Evaluate a limit** with Taylor expansions (especially when L'Hôpital is ugly) — very common.
- **Find the Maclaurin/Taylor series** of a function, usually by **substitution** into a known series.
- **Extract a coefficient** or compute $f^{(n)}(0)$ / $f^{(n)}(a)$ via "coefficient $\times\,k!=f^{(k)}$."
- **Approximate a value** to stated accuracy; **decide how many terms** are needed.
- **Bound the truncation error** using the Lagrange remainder.
- **Linear / quadratic approximation** of a function near a point.
- **Radius / interval of convergence** of a power series (ratio test).
- **Recognise a numerical series** as a known value ($e$, $\ln 2$, $\tfrac{\pi}{4}$, …).

---

## 7. Traps, pitfalls & common mistakes

- **Keep enough terms.** For $\frac{x-\sin x}{x^3}$ you must expand $\sin x$ through $x^3$; stopping too early yields a spurious $0$ or wrong constant. **Match the expansion order to the denominator's power** (and go one further if leading terms cancel).
- **Factorials in coefficients.** Coefficient of $(x-a)^k$ is $\dfrac{f^{(k)}(a)}{k!}$ — *not* $f^{(k)}(a)$. Conversely $f^{(k)}(a)=k!\times(\text{coefficient})$.
- **Respect intervals of convergence.** $e^x,\sin,\cos$ work for all $x$; but the $\ln(1+x)$, $\tfrac{1}{1-x}$, $(1+x)^\alpha$, $\arctan$ series are valid only for $|x|<1$ (or the stated range). Using $\ln(1+x)$'s series at $x=2$ is meaningless.
- **Lagrange remainder uses the MAX of $f^{(n+1)}$** over the interval with an **unknown $c$** — bound it, never evaluate at a chosen point.
- **Maclaurin = Taylor at $0$.** If asked to expand about $a\neq0$, you must use $(x-a)$ powers and derivatives **at $a$**, not at $0$.
- **Sign / alternation slips** in $\sin,\cos,\ln(1+x),\arctan$ — re-derive the $(-1)^k$ pattern if unsure.
- **Substitution discipline:** plugging $g(x)$ into a series means replacing $x$ *everywhere and raising to the power* (e.g. $e^{-x^2}$ uses $(-x^2)^k$, giving alternating signs and only even powers).
- **Smooth $\neq$ analytic.** Having all derivatives doesn't guarantee the Taylor series converges *back to $f$*; that needs $R_n\to0$. (Counterexample: $f(x)=e^{-1/x^2}$, $f(0)=0$, has every derivative $0$ at $0$, so its Maclaurin series is $\equiv0\neq f$. Beyond GATE — one line.)
- **Polynomial vs series:** $P_n$ is a *finite approximation*; the Taylor *series* is the infinite sum. Don't conflate them.
- **Use parity:** even functions have only even powers (odd functions only odd powers) — saves work and catches sign errors.
- When using Taylor in a limit, expand numerator and denominator to **consistent order**, simplify, *then* divide.

---

## 8. Connection forward

Topic 5, **Maxima, Minima & Single-Variable Optimization**, is the direct payoff of §4.7. At an interior critical point $a$ (where $f'(a)=0$), the linear Taylor term vanishes and
$$f(x)-f(a)\approx\frac{f''(a)}{2!}(x-a)^2.$$
Since $(x-a)^2\ge0$, the **sign of $f''(a)$** alone decides the verdict: $f''(a)>0\Rightarrow$ local **minimum** (upward parabola), $f''(a)<0\Rightarrow$ local **maximum**. If $f''(a)=0$, the next nonzero Taylor derivative settles it (e.g. a nonzero $f'''$ ⇒ inflection, not an extremum). That *is* the second-derivative test — Taylor is its proof. The series we built here becomes the engine for classifying optima.

---

## 9. One-line cheat-sheet summary

> $f(x)=\sum_{k\ge0}\dfrac{f^{(k)}(a)}{k!}(x-a)^k$ (Maclaurin if $a=0$), with **Lagrange remainder** $R_n=\dfrac{f^{(n+1)}(c)}{(n+1)!}(x-a)^{n+1}$ (the $n{=}0$ case is LMVT). Memorise the standard expansions ($e^x,\sin,\cos$ for all $x$; $\ln(1+x),\tfrac{1}{1-x},(1+x)^\alpha,\arctan$ for $|x|<1$). Use Taylor to crush limits (**keep enough terms!**), extract coefficients (**coeff $\times\,k!=f^{(k)}(a)$**), and bound errors (**max of $f^{(n+1)}$, unknown $c$**).

---

### ▶ Next topic
**Topic 5 — Maxima, Minima & Single-Variable Optimization** (critical points, first- and second-derivative tests, global extrema on intervals) — the final topic of this section. Ask for it when ready.

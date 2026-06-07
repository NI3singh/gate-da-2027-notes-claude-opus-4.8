# Topic 3 — Differentiability

> **Subject:** Calculus and Optimization
> **Section:** Calculus & Single-Variable Optimization (Topic 3 of 5)
> **GATE DA syllabus line covered:** *"…continuity and differentiability, Taylor series…"* (the *differentiability* part)

---

## 0. Topic header & syllabus map

- **Topic:** Differentiability
- **Belongs to:** Calculus and Optimization → the *Calculus & single-variable optimization* cluster.
- **Where it sits:** **Topic 3 of 5.** It comes **after Limits and Continuity** (Topic 2) — whose limit machinery *defines* the derivative — and leads into **Taylor Series** (Topic 4), which uses derivatives of *all* orders. *(Note: the immediate predecessor is Limits and Continuity, not Functions; the derivative is a limit, so that's the load-bearing connection.)*

In one line: *continuity says the graph connects; differentiability says the graph is locally smooth enough to have a single well-defined tangent slope at the point.*

---

## 1. Why this topic exists (motivation + connection)

**The real problem it solves.** We constantly need the **instantaneous rate of change**: the velocity at one instant (not the average over a trip), the slope of a curve at a single point, the marginal cost of one more unit, the sensitivity of a loss to a parameter. Average rates over an interval aren't enough — we want the rate *at a point*. The **derivative** delivers exactly that, as the limit of average rates as the interval shrinks to zero.

**What the previous topic (Limits & Continuity) couldn't do.** Continuity told us a function has no break — but "no break" says nothing about *how steeply* it changes, and it doesn't hand us a tangent line. A continuous graph can be jagged with corners everywhere (even continuous-but-nowhere-differentiable functions exist). To get an actual *number* for the slope, we take the limit of the secant slope $\frac{f(x)-f(a)}{x-a}$ — a $\tfrac00$ limit, precisely the kind Topic 2 taught us to evaluate. So differentiability is a **stronger** condition than continuity: it demands not just connectedness but local linearity.

**The bridge, stated plainly:** *Differentiability exists because continuity tells us the graph connects but not how fast it rises; we take the limit of the average slope to manufacture the instantaneous slope.*

**What the next topic builds on it.** **Taylor Series** approximates $f$ near $a$ using $f(a),f'(a),f''(a),\dots$ — it is "local linearity (one derivative) upgraded to a local polynomial fit (many derivatives)." The tangent line is literally the first-order Taylor polynomial. And **optimization** (Topic 5) finds maxima/minima via $f'(x)=0$ and classifies them via $f''$. The derivative built here powers both remaining topics.

---

## 2. Prerequisites

- **Topic 2 — Limits & Continuity:** one-sided limits, $\tfrac00$ evaluation, L'Hôpital, the three-condition continuity test, IVT/EVT.
- **Topic 1 — Functions:** standard-function library, composition, inverse functions, piecewise functions, monotonicity.
- **Algebra & trig identities:** factoring, $\sin^2+\cos^2=1$, double angles.
- **Index/log laws:** for logarithmic and exponential differentiation.

---

## 3. Core idea — intuition first

The **average rate of change** of $f$ between $a$ and $b$ is the slope of the **secant line**:
$$\frac{f(b)-f(a)}{b-a}.$$
Now slide $b$ toward $a$. The secant pivots into the **tangent line**, and its slope approaches the **instantaneous rate** — the derivative $f'(a)$.

**Analogy (speedometer).** Average speed over a trip is total distance ÷ total time. But your speedometer reads the speed *right now* — the limit of average speeds over shorter and shorter time windows. The derivative is the speedometer.

**Local linearity.** If you zoom into a differentiable curve at a point, it looks like a straight line — its tangent. That's the geometric meaning of differentiability. If the graph has a **corner** (like $|x|$ at $0$), zooming never straightens it: no single tangent slope exists, so it's **not** differentiable there, even though it's perfectly continuous.

---

## 4. Formal treatment

### 4.1 Definition — the derivative as a limit (first principles)

$f$ is **differentiable at $a$** if the following limit exists (finitely). Two equivalent forms:
$$f'(a)=\lim_{h\to0}\frac{f(a+h)-f(a)}{h}=\lim_{x\to a}\frac{f(x)-f(a)}{x-a}.$$

- $f'(a)$ is the **slope of the tangent line** at $\big(a,f(a)\big)$.
- **Tangent line:** $\ y-f(a)=f'(a)\,(x-a)$.  **Normal line:** slope $-\tfrac{1}{f'(a)}$ (if $f'(a)\neq0$).

### 4.2 One-sided derivatives (the test for corners & piecewise functions)

$$\text{LHD}=f'(a^-)=\lim_{h\to0^-}\frac{f(a+h)-f(a)}{h},\qquad \text{RHD}=f'(a^+)=\lim_{h\to0^+}\frac{f(a+h)-f(a)}{h}.$$

> **Differentiability criterion:** $f$ is differentiable at $a$ $\iff$ **LHD $=$ RHD** (both finite and equal). This is the workhorse for piecewise definitions and points like $|x|$ at $0$.

### 4.3 Differentiability ⇒ Continuity (and the converse is FALSE)

> **Theorem.** If $f$ is differentiable at $a$, then $f$ is continuous at $a$.

*Proof (GATE-favourite, short).*
$$\lim_{x\to a}\big(f(x)-f(a)\big)=\lim_{x\to a}\frac{f(x)-f(a)}{x-a}\cdot(x-a)=f'(a)\cdot 0=0,$$
so $\lim_{x\to a}f(x)=f(a)$, i.e. $f$ is continuous at $a$. $\blacksquare$

**The converse fails:** continuity does **not** imply differentiability. $f(x)=|x|$ is continuous at $0$ but $\text{LHD}=-1\neq+1=\text{RHD}$ — a corner. The four ways differentiability can fail (even when continuous):

| Failure | Signature | Example at $x=0$ |
|---|---|---|
| **Corner** | one-sided slopes finite but **unequal** | $|x|$ (LHD $-1$, RHD $+1$) |
| **Cusp** | one-sided slopes $\to+\infty$ and $-\infty$ | $x^{2/3}$ |
| **Vertical tangent** | slope $\to\pm\infty$ (**same** sign) | $x^{1/3}$ |
| **Discontinuity** | not continuous ⇒ automatically not differentiable | $\operatorname{sgn}(x)$ |

*(A function continuous everywhere yet differentiable nowhere exists — the Weierstrass function — but that's beyond GATE DA; one line is enough.)*

### 4.4 Rules of differentiation

For $f,g$ differentiable at the relevant point:

$$
(c)'=0,\quad (x^n)'=n x^{n-1},\quad (cf)'=c f',\quad (f\pm g)'=f'\pm g',
$$
$$
\textbf{Product: } (fg)'=f'g+fg',\qquad
\textbf{Quotient: } \left(\frac{f}{g}\right)'=\frac{f'g-fg'}{g^{2}}\ (g\neq0),
$$
$$
\textbf{Chain rule: } (f\circ g)'(x)=f'\big(g(x)\big)\cdot g'(x)\quad\Longleftrightarrow\quad \frac{dy}{dx}=\frac{dy}{du}\cdot\frac{du}{dx}.
$$

### 4.5 Standard derivatives (memorise, with domains)

| $f(x)$ | $f'(x)$ | Condition |
|---|---|---|
| $x^{n}$ | $n x^{n-1}$ | real $n$, on its domain |
| $e^{x}$ | $e^{x}$ | — |
| $a^{x}$ | $a^{x}\ln a$ | $a>0$ |
| $\ln x$ | $1/x$ | $x>0$ |
| $\log_a x$ | $1/(x\ln a)$ | $x>0,\ a>0,\ a\neq1$ |
| $\sin x,\ \cos x$ | $\cos x,\ -\sin x$ | — |
| $\tan x,\ \cot x$ | $\sec^{2}x,\ -\csc^{2}x$ | on domain |
| $\sec x,\ \csc x$ | $\sec x\tan x,\ -\csc x\cot x$ | on domain |
| $\arcsin x,\ \arccos x$ | $\dfrac{1}{\sqrt{1-x^{2}}},\ \dfrac{-1}{\sqrt{1-x^{2}}}$ | $|x|<1$ |
| $\arctan x,\ \operatorname{arccot} x$ | $\dfrac{1}{1+x^{2}},\ \dfrac{-1}{1+x^{2}}$ | — |
| $|x|$ | $\dfrac{x}{|x|}=\operatorname{sgn}(x)$ | $x\neq0$ |

### 4.6 Special differentiation techniques

**(a) Implicit differentiation.** When $y$ is defined implicitly by $F(x,y)=0$, differentiate both sides w.r.t. $x$ treating $y=y(x)$ (so $\frac{d}{dx}y^2=2y\,y'$ by chain rule), then solve for $y'$.
*Example:* $x^2+y^2=r^2 \Rightarrow 2x+2y\,y'=0 \Rightarrow y'=-\dfrac{x}{y}.$

**(b) Logarithmic differentiation.** Use when $y$ is a product/quotient of many factors, or has the form $y=f(x)^{g(x)}$ (variable base **and** exponent). Take $\ln$, then differentiate.
*General power-tower rule:* for $y=f^{\,g}$, $\ \ln y=g\ln f \Rightarrow \dfrac{y'}{y}=g'\ln f+g\dfrac{f'}{f}$, so
$$y'=f^{\,g}\!\left(g'\ln f+\frac{g f'}{f}\right).$$
*Example:* $y=x^{x}\Rightarrow y'=x^{x}\big(\ln x+1\big).$

**(c) Parametric differentiation.** If $x=x(t),\ y=y(t)$,
$$\frac{dy}{dx}=\frac{\,dy/dt\,}{\,dx/dt\,}\ \ (dx/dt\neq0),\qquad \frac{d^{2}y}{dx^{2}}=\frac{\dfrac{d}{dt}\!\left(\dfrac{dy}{dx}\right)}{\,dx/dt\,}.$$
*(Common slip: $\dfrac{d^2y}{dx^2}\neq \dfrac{d^2y/dt^2}{d^2x/dt^2}$.)*

**(d) Derivative of an inverse function.** If $f$ is bijective and differentiable with $f'\neq0$, and $y=f^{-1}(x)$, then
$$\big(f^{-1}\big)'(x)=\frac{1}{f'\big(f^{-1}(x)\big)}.$$
Geometric meaning: slopes of $f$ and $f^{-1}$ at corresponding points are **reciprocals** (graphs are reflections across $y=x$).

### 4.7 Higher-order derivatives

$f''=\dfrac{d^{2}y}{dx^{2}},\ f''',\dots,f^{(n)}=\dfrac{d^{n}y}{dx^{n}}$ — differentiate repeatedly. A function can be differentiable once but not twice (see Problem 2(ii)).

**Leibniz rule** for the $n$-th derivative of a product (rarely needed, but it exists):
$$(uv)^{(n)}=\sum_{k=0}^{n}\binom{n}{k}u^{(k)}\,v^{(n-k)}.$$

### 4.8 Mean Value Theorems (heavily tested in GATE)

> **Rolle's Theorem.** If $f$ is **continuous on $[a,b]$**, **differentiable on $(a,b)$**, and $f(a)=f(b)$, then $\exists\,c\in(a,b)$ with $f'(c)=0$. *(Geometrically: a horizontal tangent somewhere.)*

> **Lagrange's MVT (LMVT).** If $f$ is continuous on $[a,b]$ and differentiable on $(a,b)$, then $\exists\,c\in(a,b)$ with
> $$f'(c)=\frac{f(b)-f(a)}{b-a}.$$
> *(A tangent parallel to the secant. Rolle is the special case $f(a)=f(b)$.)*

> **Cauchy's MVT (generalised).** If $f,g$ are continuous on $[a,b]$, differentiable on $(a,b)$, and $g'\neq0$ on $(a,b)$, then $\exists\,c$ with
> $$\frac{f(b)-f(a)}{g(b)-g(a)}=\frac{f'(c)}{g'(c)}.$$
> *(This is the engine behind L'Hôpital's rule from Topic 2.)*

**Hypotheses are non-negotiable:** continuity on the **closed** interval, differentiability on the **open** one. A corner *inside* $(a,b)$ breaks the guarantee. The point $c$ is **guaranteed to exist** but is generally **not unique** and usually not expressible in closed form.

**Typical uses:** prove inequalities (e.g. $|\cos x-\cos y|\le|x-y|$), bound function values, show a function is constant ($f'\equiv0\Rightarrow f$ constant), or count roots.

### 4.9 Monotonicity from the derivative's sign (bridge to optimization)

On an interval $I$:
$$f'>0\ \Rightarrow\ f \text{ strictly increasing},\qquad f'<0\ \Rightarrow\ f \text{ strictly decreasing},\qquad f'\equiv0\ \Rightarrow\ f \text{ constant}.$$
Points where $f'(x)=0$ **or** $f'$ is undefined are **critical points** — the candidates for maxima/minima in Topic 5. (This makes precise the monotonicity tags introduced loosely in Topic 1.)

---

## 5. Worked problems (GATE-style)

### Problem 1 — Piecewise: make it differentiable *(NAT)*

Let
$$f(x)=\begin{cases}x^{2}, & x\le 1\\[2pt] ax+b, & x>1.\end{cases}$$
Find $a,b$ so that $f$ is differentiable at $x=1$.

**Solution.** Two conditions are required — **continuity first**, then **matching slopes**.

*Continuity at $1$:* LHL $=1^{2}=1=f(1)$; RHL $=a(1)+b=a+b$. Need
$$a+b=1.\qquad(\text{i})$$

*Differentiability at $1$ (LHD $=$ RHD):* left piece $x^2$ has slope $2x\big|_{1}=2$; right piece $ax+b$ has slope $a$. Need
$$a=2.\qquad(\text{ii})$$

From (ii) $a=2$, then (i) gives $b=1-a=-1$.
$$\boxed{a=2,\ \ b=-1.}$$

> **Why both equations?** Differentiability *requires* continuity (Theorem 4.3). If you only match slopes ($a=2$) but ignore continuity, the graph could jump — then no derivative exists regardless of slopes. Always solve continuity **and** slope conditions together.

---

### Problem 2 — Classify differentiability at $x=0$ *(MSQ)*

For each function, decide whether it is differentiable at $0$, and name the obstruction if not:
**(i)** $|x|$  **(ii)** $x|x|$  **(iii)** $x^{1/3}$  **(iv)** $x^{2/3}$.

**Solution.**

**(i) $|x|$.** $\text{LHD}=\lim_{h\to0^-}\frac{|h|}{h}=-1$, $\text{RHD}=\lim_{h\to0^+}\frac{|h|}{h}=+1$. Unequal ⇒ **not differentiable (corner)**.

**(ii) $x|x|$.** Here $f(h)=h|h|$, so $\frac{f(h)-f(0)}{h}=\frac{h|h|}{h}=|h|\to0$ from **both** sides. So $f'(0)=0$ ⇒ **differentiable!** *(Subtle: the extra $x$ smooths the corner. In fact $f'(x)=2|x|$, which is continuous but itself not differentiable at $0$ — so $f$ is once-differentiable but not twice.)*

**(iii) $x^{1/3}$.** $f'(x)=\tfrac13 x^{-2/3}=\dfrac{1}{3x^{2/3}}\to+\infty$ as $x\to0$ from both sides. Slope infinite (same sign) ⇒ **not differentiable (vertical tangent)**.

**(iv) $x^{2/3}$.** $f'(x)=\tfrac23 x^{-1/3}=\dfrac{2}{3x^{1/3}}$: $\to+\infty$ as $x\to0^+$, $\to-\infty$ as $x\to0^-$ ⇒ **not differentiable (cusp)**.

**Verdict:** only **(ii)** is differentiable at $0$.

> **Lesson / trap:** a modulus does **not** automatically create a corner — $x|x|$ is smooth at $0$. And fractional powers give vertical-tangent vs cusp depending on the sign behaviour of the one-sided slopes.

---

### Problem 3 — Rolle (find $c$) and LMVT (prove an inequality) *(MCQ + reasoning)*

**(a)** Verify Rolle's theorem for $f(x)=x^{2}-5x+6$ on $[2,3]$ and find $c$.

$f$ is a polynomial ⇒ continuous on $[2,3]$ and differentiable on $(2,3)$. Endpoints: $f(2)=4-10+6=0$, $f(3)=9-15+6=0$, so $f(2)=f(3)$. ✔ All hypotheses hold. Then
$$f'(x)=2x-5=0\ \Rightarrow\ x=\tfrac52=2.5\in(2,3).$$
$$\boxed{c=2.5.}$$

**(b)** Prove that $\ |\cos x-\cos y|\le|x-y|\ $ for all real $x,y$.

$\cos$ is differentiable everywhere, so LMVT on the interval with endpoints $x,y$ gives some $c$ between them with
$$\cos x-\cos y=-\sin(c)\,(x-y)\ \Rightarrow\ |\cos x-\cos y|=|\sin c|\,|x-y|\le 1\cdot|x-y|.\quad\blacksquare$$

> **Trap:** the MVT $c$ exists but you can't (and needn't) solve for it in part (b) — you only use $|\sin c|\le1$. Part (a) is the *computational* flavour (nice polynomial), part (b) the *existence-for-a-bound* flavour. GATE asks both.

---

## 6. Common GATE question types on this topic

- **Compute derivatives** via product/quotient/chain; first-principles computation.
- **Piecewise constants** for continuity & differentiability — the #1 NAT type (two equations).
- **Count points of non-differentiability** of $|g(x)|$, sums of moduli, fractional powers.
- **Implicit / parametric / logarithmic** differentiation; find $\dfrac{dy}{dx}$.
- **Inverse-function / inverse-trig** derivatives.
- **Higher-order derivatives**; find $f^{(n)}$ or a specific value.
- **Rolle / LMVT:** verify hypotheses, find $c$, or use existence to prove inequalities / bound values / count roots.
- **Monotonicity intervals** from the sign of $f'$ (gateway to optimization).
- **Tangent / normal line** equations at a point.

---

## 7. Traps, pitfalls & common mistakes

- **Differentiable ⇒ continuous, never the reverse.** In piecewise problems, **check continuity first** — if it's discontinuous, it's automatically non-differentiable and slope-matching is pointless. You need **both** continuity (LHL $=$ RHL $=$ value) **and** LHD $=$ RHD.
- LHD/RHD compare **slopes**, not function values, and each must use the correct piece.
- **A modulus need not create a corner:** $|g|$ has a corner only where $g$ **crosses** zero (simple root); at a *double* root $g$ touches without sign change and $|g|$ can stay smooth. And $x|x|$ is smooth at $0$.
- **Distinguish the failures:** corner (finite unequal slopes), cusp (slopes $+\infty$ and $-\infty$), vertical tangent (slope $\pm\infty$, same sign). All three are non-differentiable.
- **Chain rule — don't drop the inner derivative:** $\frac{d}{dx}\sin(x^2)=\cos(x^2)\cdot2x$, not $\cos(x^2)$.
- $\frac{d}{dx}a^{x}=a^{x}\ln a$, **not** $x a^{x-1}$ (power rule is for fixed exponents). For $x^{x}$ or any $f^{g}$, neither simple rule works — use **logarithmic differentiation**.
- **Sign discipline:** product rule is $f'g+fg'$ (**plus**); quotient rule numerator is $f'g-fg'$ (**minus**, order matters).
- **Implicit differentiation:** $\frac{d}{dx}(y^2)=2y\,y'$ (chain rule on $y$), not $2y$.
- **Parametric second derivative:** $\frac{d^2y}{dx^2}=\dfrac{\frac{d}{dt}(dy/dx)}{dx/dt}$, **not** $\dfrac{d^2y/dt^2}{d^2x/dt^2}$.
- **Rolle/LMVT need their hypotheses:** continuous on closed $[a,b]$, differentiable on open $(a,b)$ (and Rolle needs $f(a)=f(b)$). A corner inside the interval voids the conclusion.
- The MVT point $c$ is **guaranteed but not unique** and usually not in closed form — match the question to the right flavour (find $c$ vs use existence).
- **Average vs instantaneous:** the secant slope (average rate) is not the derivative (instantaneous rate) — though LMVT says they coincide at some interior $c$.
- Higher derivative of a product $\neq$ product of derivatives — use the product/Leibniz rule.

---

## 8. Connection forward

The derivative is the seed of both remaining topics. **Taylor Series** (Topic 4) takes $f(a),f'(a),f''(a),\dots$ and builds the best polynomial approximation of $f$ near $a$:
$$f(x)\approx f(a)+f'(a)(x-a)+\frac{f''(a)}{2!}(x-a)^2+\cdots$$
The first two terms are exactly the **tangent line** — so Taylor is "local linearity extended to all orders." Then **Maxima, Minima & Optimization** (Topic 5) locates candidate extrema where $f'(x)=0$ (or $f'$ is undefined) and classifies them with the sign of $f''$ — both pure consequences of differentiability and the MVTs developed here.

---

## 9. One-line cheat-sheet summary

> $f'(a)=\lim_{h\to0}\dfrac{f(a+h)-f(a)}{h}$ is the tangent slope; it exists $\iff$ **LHD $=$ RHD** (finite). **Differentiable $\Rightarrow$ continuous, not conversely** (corners/cusps/vertical tangents fail). Know the rules (product $+$, quotient $-$, chain $\times$ inner), the standard-derivative table, and implicit/log/parametric/inverse techniques. **Rolle/LMVT** (continuous on $[a,b]$, differentiable on $(a,b)$) give an interior $c$ — for finding slopes or proving bounds. Piecewise problems need **continuity AND slope-matching** (two equations).

---

### ▶ Next topic
**Topic 4 — Taylor Series** (polynomial approximation from higher-order derivatives; the tangent line is the order-1 case). Ask for it when ready.

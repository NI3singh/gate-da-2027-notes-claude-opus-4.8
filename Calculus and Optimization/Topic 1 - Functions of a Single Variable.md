# Topic 1 — Functions of a Single Variable

> **Subject:** Calculus and Optimization
> **Section:** Calculus & Single-Variable Optimization (Topic 1 of 5)
> **GATE DA syllabus line covered:** *"Functions of a single variable, …"*

---

## 0. Topic header & syllabus map

- **Topic:** Functions of a Single Variable
- **Belongs to:** Calculus and Optimization → the *Calculus & single-variable optimization* cluster.
- **Where it sits:** This is the **first** topic of the whole calculus stream. Nothing in this section comes before it. It defines the *object* — a function — that every later topic operates on. It leads directly into **Limits and Continuity**, which studies how these functions behave near a point.

In one line: *limits, derivatives, integrals, and optimization are all verbs; a function is the noun they act on.* Get the noun right first.

---

## 1. Why this topic exists (motivation + connection)

**The real problem it solves.** We constantly deal with quantities that depend on other quantities: cost depends on units produced, a model's loss depends on a parameter, position depends on time. A **function** is the precise mathematical language for "output is determined by input." Without it we cannot even *state* questions like "where is the loss smallest?" — let alone answer them.

**What plain algebra couldn't do (the gap that forces this topic).** School algebra solves equations for *fixed* unknowns: "find $x$ such that $2x+3=7$." That is a static, one-shot question. It has no vocabulary for "as $x$ varies continuously, how does the output move, and where does it peak?" To even ask about change and optima, we must first treat the rule $x \mapsto f(x)$ as a single object we can compose, invert, classify, and later differentiate. That object is the function. So this topic is the *bridge from algebra (solving for a value) to calculus (studying a whole relationship).*

**What the next topic builds on it.** **Limits and Continuity** asks: as the input $x$ slides toward a point $a$, what value does $f(x)$ approach? That question only makes sense once $f$ is defined and we know its domain and behaviour — exactly what we set up here.

*(This section is the one most online notes skip — it is mandatory here.)*

---

## 2. Prerequisites

Revise these first; everything below assumes fluency with them.

- **Real numbers & intervals:** open/closed intervals, $(-\infty,\infty)$, set notation $\{x : \dots\}$, union $\cup$, intersection $\cap$.
- **Algebra:** factoring quadratics, solving inequalities (especially products/quotients of linear factors), sign charts.
- **Coordinate geometry:** plotting points, reading a graph, slope of a line.
- **School-level standard functions:** polynomials, $\sqrt{\,}$, $1/x$, $a^x$, $\log_a x$, $\sin/\cos/\tan$, $|x|$ — at least their basic shapes.
- **Set maps (light):** the idea of assigning elements of one set to another.

---

## 3. Core idea — intuition first

Think of a function as a **machine** (or a vending machine): you feed in **one** input, it returns **exactly one** output. Press button `B4` and you always get the *same one* item — never two items, never "sometimes a soda, sometimes a chocolate."

That "exactly one output per input" rule is the whole definition. Three vocabulary words:

- **Domain** = the set of inputs the machine *accepts* (which buttons exist).
- **Codomain** = the *declared* universe of possible outputs (what the manufacturer says it could dispense).
- **Range** = the outputs that *actually come out* for some input (what items are genuinely stocked).

"**Single variable**" simply means the input is **one real number** $x$, not a vector $(x_1,\dots,x_n)$. (Multivariable functions are a different subject; GATE handles them under Linear Algebra / Probability, not here.)

**Graphical intuition.** Plot all points $(x, f(x))$. The "exactly one output" rule becomes the **vertical line test**: *every vertical line hits the graph at most once.* A circle $x^2+y^2=1$ fails this (a vertical line cuts it twice) — so a circle is **not** a function of $x$.

---

## 4. Formal treatment

### 4.1 Definition of a function

A **function** $f : A \to B$ is a rule that assigns to **each** $x \in A$ **exactly one** element $f(x) \in B$.

- $A$ = **domain**, $B$ = **codomain**, and **range** $= f(A) = \{\, f(x) : x \in A \,\} \subseteq B$.
- Two conditions hidden in "each … exactly one": (i) *every* domain element must be assigned (totality); (ii) *no* domain element maps to two values (well-defined / single-valued).

> **Vertical line test (graphical):** a curve in the plane is the graph of a function of $x$ iff no vertical line meets it more than once.

### 4.2 Finding the domain (natural domain)

When a formula is given without a stated domain, the **natural domain** is the largest set of real $x$ for which the formula gives a *real* number. Apply these restrictions simultaneously:

| Operation in the formula | Restriction on the domain |
|---|---|
| Denominator $q(x)$ | $q(x) \neq 0$ |
| Even root $\sqrt[2k]{\,g(x)\,}$ | $g(x) \ge 0$ |
| Logarithm $\log_a g(x)$ ($a>0,\ a\neq1$) | $g(x) > 0$ |
| $\arcsin g(x)$, $\arccos g(x)$ | $-1 \le g(x) \le 1$ |
| $\tan g(x)$ | $g(x) \neq (2k+1)\tfrac{\pi}{2}$ |

The natural domain is the **intersection** of all such conditions.

### 4.3 The standard library of single-variable functions

Memorise domain, range, and shape — these are the building blocks of every GATE problem.

| Function | Domain | Range | Notes / shape |
|---|---|---|---|
| Polynomial $a_nx^n+\dots+a_0$ | $\mathbb{R}$ | depends on degree | continuous, smooth |
| Rational $p(x)/q(x)$ | $\mathbb{R}\setminus\{q=0\}$ | varies | vertical asymptotes at zeros of $q$ |
| $\sqrt{x}$ | $[0,\infty)$ | $[0,\infty)$ | half-parabola on its side |
| $1/x$ | $\mathbb{R}\setminus\{0\}$ | $\mathbb{R}\setminus\{0\}$ | rectangular hyperbola |
| $a^x$ ($a>0$) | $\mathbb{R}$ | $(0,\infty)$ | growth if $a>1$, decay if $0<a<1$ |
| $\log_a x$ ($a>0,a\neq1$) | $(0,\infty)$ | $\mathbb{R}$ | inverse of $a^x$ |
| $\sin x,\ \cos x$ | $\mathbb{R}$ | $[-1,1]$ | period $2\pi$ |
| $\tan x$ | $\mathbb{R}\setminus\{(2k+1)\pi/2\}$ | $\mathbb{R}$ | period $\pi$ |
| $|x|$ | $\mathbb{R}$ | $[0,\infty)$ | V-shape; key identity $\sqrt{x^2}=|x|$ |
| $\lfloor x \rfloor$ (floor / greatest integer) | $\mathbb{R}$ | $\mathbb{Z}$ | staircase, jumps at integers |
| $\{x\}=x-\lfloor x\rfloor$ (fractional part) | $\mathbb{R}$ | $[0,1)$ | period $1$ |
| $\operatorname{sgn}(x)$ | $\mathbb{R}$ | $\{-1,0,1\}$ | $-1$ if $x<0$, $0$ at $0$, $+1$ if $x>0$ |

**Inverse trigonometric principal-value ranges** (needed whenever GATE asks for a *principal* value):

$$\arcsin: [-1,1]\to[-\tfrac{\pi}{2},\tfrac{\pi}{2}], \qquad \arccos: [-1,1]\to[0,\pi], \qquad \arctan: \mathbb{R}\to(-\tfrac{\pi}{2},\tfrac{\pi}{2}).$$

*(Hyperbolic functions $\sinh, \cosh, \tanh$ are standard but only rarely surface in GATE DA; know that $\cosh x \ge 1$ and $\sinh$ is odd with range $\mathbb{R}$ — that depth is enough.)*

### 4.4 Classifying functions by their properties

**(a) Even / odd (symmetry).** With a domain symmetric about $0$:

$$f \text{ is even} \iff f(-x)=f(x) \ (\text{symmetric about the } y\text{-axis}), \qquad f \text{ is odd} \iff f(-x)=-f(x) \ (\text{symmetric about the origin}).$$

Parity arithmetic: even·even = even, odd·odd = even, even·odd = odd; even+even = even, odd+odd = odd, even+odd = (generally) neither. Examples: $x^2,\cos x,|x|$ even; $x^3,\sin x,x$ odd; $e^x$ is neither.

**(b) Monotonic.** On an interval $I$:
- **strictly increasing:** $x_1<x_2 \Rightarrow f(x_1)<f(x_2)$;
- **strictly decreasing:** $x_1<x_2 \Rightarrow f(x_1)>f(x_2)$;
- replace $<,>$ by $\le,\ge$ for *non-strict* (monotonic, allowing flat stretches).

> **Key consequence (used constantly):** a **strictly monotonic** function is **one-one (injective)**. (We'll later test monotonicity via the derivative — that's the bridge to Topic 3.)

**(c) Bounded.** $f$ is **bounded above** if $\exists M: f(x)\le M\ \forall x$; **bounded below** if $\exists m: f(x)\ge m$; **bounded** if both, equivalently $\exists M\ge0: |f(x)|\le M\ \forall x$. E.g. $\sin x$ is bounded; $x^3$ is not; $x/(1+|x|)$ is bounded (by $1$).

**(d) Periodic.** $f$ is **periodic** with period $T>0$ if $f(x+T)=f(x)$ for all $x$ in the domain. The smallest such $T$ is the **fundamental period**.

> **Scaling rule:** if $f$ has fundamental period $T$, then $f(ax+b)$ has fundamental period $T/|a|$.
> **Sum rule (with a warning):** if $f,g$ have fundamental periods $T_1,T_2$ with $T_1/T_2$ rational, then $f+g$ is periodic and its fundamental period **divides** $\operatorname{lcm}(T_1,T_2)$ — but it can be **strictly smaller** (see the trap in §7).

**(e) Injective / Surjective / Bijective.** For $f:A\to B$:

$$
\begin{aligned}
\textbf{Injective (one-one):}\ & f(x_1)=f(x_2)\Rightarrow x_1=x_2 \quad(\text{distinct inputs} \to \text{distinct outputs}).\\
\textbf{Surjective (onto):}\ & \text{range} = \text{codomain } B \quad(\text{every } b\in B \text{ is hit}).\\
\textbf{Bijective:}\ & \text{both injective and surjective}.
\end{aligned}
$$

> **Horizontal line test:** $f$ is injective iff no horizontal line meets the graph more than once.
> **Surjectivity depends on the stated codomain $B$.** The same formula can be onto $[0,\infty)$ but not onto $\mathbb{R}$. *Always read the codomain.*

### 4.5 Operations on functions

**Algebra of functions** (combine pointwise):

$$(f\pm g)(x)=f(x)\pm g(x),\quad (fg)(x)=f(x)g(x),\quad \Big(\tfrac{f}{g}\Big)(x)=\tfrac{f(x)}{g(x)}.$$

- Domain of $f\pm g,\ fg$: $\operatorname{dom}(f)\cap\operatorname{dom}(g)$.
- Domain of $f/g$: $\operatorname{dom}(f)\cap\operatorname{dom}(g)\setminus\{x:g(x)=0\}$.

**Composition** $(f\circ g)(x)=f\big(g(x)\big)$ — "do $g$ first, then $f$."

$$\operatorname{dom}(f\circ g)=\{\,x\in\operatorname{dom}(g): g(x)\in\operatorname{dom}(f)\,\}.$$

> **Composition is NOT commutative:** in general $f\circ g \neq g\circ f$.

**Inverse function.** $f:A\to B$ has an inverse $f^{-1}:B\to A$ **iff $f$ is bijective.** Then:

$$(f^{-1}\circ f)(x)=x\ \ \forall x\in A,\qquad (f\circ f^{-1})(y)=y\ \ \forall y\in B,$$

$$\operatorname{dom}(f^{-1})=\text{range}(f),\qquad \text{range}(f^{-1})=\operatorname{dom}(f).$$

The graph of $f^{-1}$ is the reflection of the graph of $f$ across the line $y=x$. **To find $f^{-1}$:** write $y=f(x)$, solve for $x$ in terms of $y$, then swap names.

### 4.6 A note on derivations

This is a *definitional* topic — GATE never asks you to "derive what a function is." So there are **no proofs to memorise here**; the real derivations (e.g., for limits, the Mean Value Theorem, Taylor's theorem) arrive in Topics 2–4 and rest on these definitions. *We deliberately skip formal proofs in this topic for that reason.*

---

## 5. Worked problems (GATE-style)

### Problem 1 — Domain, with the $\sqrt{\ }$ vs $\ln$ trap *(NAT / MCQ)*

Find the natural domain of
**(a)** $f(x)=\sqrt{(x-1)(x-3)}$ and **(b)** $g(x)=\ln\big((x-1)(x-3)\big)$.

**Solution.** Both depend on the sign of $h(x)=(x-1)(x-3)$, an upward parabola with roots $1$ and $3$. Sign chart: $h>0$ for $x<1$, $h<0$ for $1<x<3$, $h>0$ for $x>3$; $h=0$ at $x=1,3$.

- **(a) $\sqrt{\ }$ needs $h(x)\ge 0$** (zero is allowed under a square root):
$$\boxed{\operatorname{dom}(f)=(-\infty,1]\cup[3,\infty).}$$
*(Range note: on this domain $h$ runs over $[0,\infty)$, so $\text{range}(f)=[0,\infty)$.)*

- **(b) $\ln$ needs $h(x)> 0$** (zero is **not** allowed — $\ln 0$ is undefined):
$$\boxed{\operatorname{dom}(g)=(-\infty,1)\cup(3,\infty).}$$

**The trap:** the only difference is the **endpoints** $x=1,3$. They belong to $\operatorname{dom}(f)$ but are *excluded* from $\operatorname{dom}(g)$. Mixing up $\ge$ and $>$ here is the single most common domain error.

---

### Problem 2 — Injective / Surjective classification *(MSQ)*

For each $f:\mathbb{R}\to\mathbb{R}$ below, decide whether it is injective and whether it is surjective.
**(i)** $x^3$  **(ii)** $x^2$  **(iii)** $\dfrac{x}{1+|x|}$  **(iv)** $e^x$.

**Solution.**

**(i) $f(x)=x^3$.** Strictly increasing on all of $\mathbb{R}$ (bigger input ⇒ bigger cube) ⇒ **injective**. As $x\to\pm\infty$, $x^3\to\pm\infty$ and it's continuous, so range $=\mathbb{R}$ ⇒ **surjective**. → **Bijective.**

**(ii) $f(x)=x^2$.** $f(-2)=f(2)=4$ ⇒ **not injective**. Range $=[0,\infty)\neq\mathbb{R}$ ⇒ **not surjective**. → **Neither.**

**(iii) $f(x)=\dfrac{x}{1+|x|}$.** For $x\ge0$, $f=\dfrac{x}{1+x}$ increases from $0$ toward (but never reaching) $1$; for $x<0$, $f=\dfrac{x}{1-x}$ increases from above $-1$ up to $0$. Strictly increasing throughout ⇒ **injective**. Its range is $(-1,1)\neq\mathbb{R}$ ⇒ **not surjective** (it's bounded!). → **Injective only.**

**(iv) $f(x)=e^x$.** Strictly increasing ⇒ **injective**. Range $=(0,\infty)\neq\mathbb{R}$ ⇒ **not surjective** onto $\mathbb{R}$. → **Injective only.**

**Summary:** injective = (i),(iii),(iv); surjective = (i) only; bijective = (i) only.

> **Lesson:** "strictly monotonic ⇒ injective" handled (i),(iii),(iv) instantly. Surjectivity then reduced to "does the **range** equal the codomain $\mathbb{R}$?" — and three of them are bounded or one-sided, so they fail.

---

### Problem 3 — Composition / iteration *(MCQ, trap variety)*

Let $f(x)=\dfrac{x-1}{x+1}$ (for $x\neq -1$). Compute $f(f(x))$, and hence find $f^{(2026)}(x)$, the $2026$-fold composition.

**Solution.** Let $u=f(x)=\dfrac{x-1}{x+1}$. Then

$$
u-1=\frac{x-1}{x+1}-1=\frac{(x-1)-(x+1)}{x+1}=\frac{-2}{x+1},\qquad
u+1=\frac{x-1}{x+1}+1=\frac{(x-1)+(x+1)}{x+1}=\frac{2x}{x+1}.
$$

$$
f(f(x))=\frac{u-1}{u+1}=\frac{-2/(x+1)}{\,2x/(x+1)\,}=\frac{-2}{2x}=\boxed{-\frac{1}{x}}.
$$

Now iterate. With $f^{(2)}(x)=-1/x$:
$$
f^{(4)}(x)=f^{(2)}\big(f^{(2)}(x)\big)=f^{(2)}\!\left(-\tfrac1x\right)=-\frac{1}{-1/x}=x.
$$

So $f^{(4)}=\text{identity}$: composing $f$ four times returns $x$. The iterates are periodic with period $4$. Since $2026=4\cdot506+2$,
$$
f^{(2026)}(x)=f^{(2)}(x)=\boxed{-\frac{1}{x}}.
$$

> **Trap & honesty note:** these identities hold only where every intermediate value is defined — i.e., excluding $x=-1$ (kills $f$), $x=0$ (kills $f^{(2)}=-1/x$), and $x=1$ (since $f(1)=0$ then feeds $f^{(2)}$). The *clean* approach is to find the period of the iteration ($4$ here) and reduce the exponent mod $4$; never multiply out $2026$ compositions by hand.

---

## 6. Common GATE question types on this topic

- **Find the domain (or range)** of a function built from $\sqrt{\ }$, $\ln$, rationals, or inverse trig — usually NAT.
- **One-one / onto / bijective?** Classify one or several $f:\mathbb{R}\to\mathbb{R}$ — classic MSQ; watch the codomain.
- **Composition values / iteration:** compute $f\circ g$, evaluate $f(f(\dots))$, or use the period of the iteration.
- **Inverse function:** find $f^{-1}$, or the value $f^{-1}(c)$, or which functions are invertible (need bijection / domain restriction).
- **Even / odd:** identify parity and exploit it in symmetric definite integrals, $\int_{-a}^{a}$.
- **Fundamental period** of a combination of periodic functions (the $\operatorname{lcm}$-with-a-catch type).
- **Monotonicity intervals** (often solved later via the derivative — links to Topic 3).
- **Functional equations:** given a relation $f(\text{stuff})=\dots$, deduce $f$ or a specific value.

---

## 7. Traps, pitfalls & common mistakes

- $\sqrt{x^2}=|x|$, **not** $x$. (e.g. $\sqrt{(-3)^2}=3$.) The single most punished simplification error.
- **$\sqrt{\ }$ allows $0$, $\ln$ does not.** Endpoints flip between $[\,]$ and $(\,)$ — see Problem 1.
- **"Onto" depends on the codomain.** $f(x)=x^2$ is onto $[0,\infty)$ but not onto $\mathbb{R}$. Don't declare "not surjective" without checking the stated $B$.
- **Codomain ≠ range.** The range is what's actually hit; the codomain is what's declared.
- $f\circ g \neq g\circ f$ in general — order matters in composition.
- **Domain of a composition is restrictive:** you need $g(x)\in\operatorname{dom}(f)$, not just $x\in\operatorname{dom}(g)$.
- **Inverse needs a bijection.** $\sin x$ has no inverse on $\mathbb{R}$; restrict to $[-\tfrac{\pi}{2},\tfrac{\pi}{2}]$ first. Use principal-value ranges for $\arcsin/\arccos/\arctan$.
- **Period of $f(ax+b)$ is $T/|a|$**, not $T$. Forgetting the $|a|$ scaling is common.
- **Period-of-a-sum trap (worked):** find the fundamental period of $f(x)=|\sin x|+|\cos x|$.
  Each of $|\sin x|,\,|\cos x|$ has period $\pi$, so the lazy answer is "$\pi$." Check the smaller candidate $T=\tfrac{\pi}{2}$:
  $$f\!\left(x+\tfrac{\pi}{2}\right)=\big|\sin(x+\tfrac{\pi}{2})\big|+\big|\cos(x+\tfrac{\pi}{2})\big|=|\cos x|+|{-}\sin x|=|\cos x|+|\sin x|=f(x).$$
  So the **fundamental period is $\tfrac{\pi}{2}\approx 1.5708$**, *smaller* than $\operatorname{lcm}(\pi,\pi)=\pi$. Moral: the lcm is only an upper bound; always test smaller divisors when there's hidden phase symmetry.
- **Strict vs non-strict monotonic.** Only *strict* monotonicity guarantees injectivity; a flat stretch breaks one-one-ness.
- **Parity arithmetic slips:** odd·odd = even (not odd); even+odd is usually neither (not "mixed").

---

## 8. Connection forward

We now have the object and its vocabulary: domain, the standard library, parity, monotonicity, boundedness, periodicity, composition, and inverse. **Topic 2 — Limits and Continuity** takes the next step: *as $x$ approaches a point $a$, what value does $f(x)$ head toward, and does the graph join up there with no break?* That local behaviour is the foundation for the derivative (Topic 3), and the monotonicity/extrema language we sketched here becomes computable once we can differentiate — feeding straight into **Maxima, Minima & single-variable optimization** (Topic 5).

---

## 9. One-line cheat-sheet summary

> A function sends each input to **exactly one** output; master its **domain/range**, the **standard library**, the **even/odd–monotone–bounded–periodic** tags, and **composition & inverse** (inverse needs a bijection) — these are the objects every later calculus tool acts on. *Remember:* $\sqrt{x^2}=|x|$, $\ln$ excludes endpoints that $\sqrt{\ }$ keeps, "onto" depends on the codomain, and the period of a sum can beat the lcm.

---

### ▶ Next topic
**Topic 2 — Limits and Continuity** (how a function behaves as $x\to a$; the gateway to the derivative). Ask for it when ready.

# Topic 2 — Limits and Continuity

> **Subject:** Calculus and Optimization
> **Section:** Calculus & Single-Variable Optimization (Topic 2 of 5)
> **GATE DA syllabus line covered:** *"…limit, continuity and…"*

---

## 0. Topic header & syllabus map

- **Topic:** Limits and Continuity
- **Belongs to:** Calculus and Optimization → the *Calculus & single-variable optimization* cluster.
- **Where it sits:** **Topic 2 of 5.** It comes **after Functions of a Single Variable** (which gave us the object) and leads directly into **Differentiability** (Topic 3), whose very definition is a limit. This topic is the *behaviour-near-a-point* layer: it tells us where a function is heading and whether its graph is unbroken.

In one line: *a function tells you the value AT a point; a limit tells you the value the function is HEADING TOWARD near a point — even where the point itself is missing.*

---

## 1. Why this topic exists (motivation + connection)

**The real problem it solves.** A function gives an output at each input — but the questions calculus cares about are about *approach*: What does $f(x)$ do as $x$ creeps toward $a$? What happens at a point where the formula collapses to $\tfrac{0}{0}$? Does the graph join up smoothly or jump? "Limit" is the precise tool for "tendency near a point," and "continuity" is the precise statement of "no break here."

**What the previous topic (Functions) couldn't do.** Functions of a single variable handed us a static input→output rule. But that rule says **nothing** about behaviour *near* a point we cannot plug in. Consider $\frac{x^2-1}{x-1}$ at $x=1$: the function value simply *doesn't exist* there ($\tfrac{0}{0}$), yet the graph clearly heads toward $2$. Functions alone can't express "heads toward $2$." We needed a new idea — the limit — to talk about trend, holes, jumps, and asymptotes that the bare function rule is silent about.

**What the next topic builds on it.** **Differentiability** is *defined* as a limit:
$$f'(a)=\lim_{h\to 0}\frac{f(a+h)-f(a)}{h}.$$
And continuity is a *necessary condition* for differentiability (differentiable ⇒ continuous). So the chain is: **limits → continuity → derivative**. Every tool we build here (one-sided limits, indeterminate-form techniques, L'Hôpital, Taylor) is exactly what computes derivatives in Topic 3 — and later defines integrals and series convergence (Topic 4).

---

## 2. Prerequisites

- **Topic 1 — Functions:** domain/range, the standard-function library, composition, even/odd, piecewise functions.
- **Algebra:** factoring, cancelling, rationalising with conjugates, simplifying rational expressions.
- **Trig identities:** $\sin^2x+\cos^2x=1$, double-angle $1-\cos 2x=2\sin^2x$, etc.
- **Exp/log basics:** $e^x$, $\ln x$, laws of indices and logs.
- **Inequalities** (light) — for the squeeze theorem and the $\varepsilon$–$\delta$ idea.

---

## 3. Core idea — intuition first

A **limit** is the value $f(x)$ *heads toward* as $x$ gets arbitrarily close to $a$ — **ignoring** what happens *at* $a$ itself.

**Analogy.** Driving toward a town: the question "what town are you heading to?" is about your **direction of approach**, not about whether you actually arrive, or what's at the exact town-centre pixel. The limit cares about the trend, not the single value at the destination.

**The hole example.** Let $f(x)=\dfrac{x^2-1}{x-1}$. At $x=1$ this is $\tfrac00$ — undefined. But for every $x\neq1$, $f(x)=\dfrac{(x-1)(x+1)}{x-1}=x+1$. So as $x\to1$, $f(x)\to2$. We write $\lim_{x\to1}f(x)=2$, even though $f(1)$ doesn't exist. The graph is the line $y=x+1$ with a single **hole** punched at $(1,2)$.

**Left and right.** You can approach $a$ from below ($x\to a^-$) or above ($x\to a^+$). These are the **one-sided limits**. The two-sided limit exists only when both agree.

**Continuity** is then the simple statement: *the function actually arrives where it's heading*, with no hole and no jump — you can trace the graph through $x=a$ **without lifting your pen**.

---

## 4. Formal treatment

### 4.1 Definition of a limit (intuitive + formal $\varepsilon$–$\delta$)

**Intuitive.** $\displaystyle\lim_{x\to a}f(x)=L$ means: $f(x)$ can be made as close to $L$ as we please by taking $x$ sufficiently close to $a$ (with $x\neq a$).

**Formal ($\varepsilon$–$\delta$).**
$$\lim_{x\to a}f(x)=L \iff \forall\,\varepsilon>0\ \exists\,\delta>0 \ \text{such that}\ \ 0<|x-a|<\delta \implies |f(x)-L|<\varepsilon.$$
The $0<|x-a|$ part is what **excludes $x=a$** — the limit never looks at the point itself.

> *GATE depth note:* GATE DA rarely demands a full $\varepsilon$–$\delta$ proof, but it does test the *concept* (that the value at $a$ is irrelevant, that "$x\neq a$"). Know the definition; you won't usually have to push $\varepsilon$'s around.

### 4.2 One-sided limits (the workhorse for piecewise functions)

$$\text{LHL}=\lim_{x\to a^-}f(x)\quad(\text{approach from below}),\qquad \text{RHL}=\lim_{x\to a^+}f(x)\quad(\text{approach from above}).$$

> **Existence criterion:** $\displaystyle\lim_{x\to a}f(x)=L \iff \text{LHL}=\text{RHL}=L.$
> If LHL $\neq$ RHL, the two-sided limit **does not exist (DNE)**.

This is the single most-used tool for $|x|$, $\lfloor x\rfloor$, $\operatorname{sgn}(x)$, $e^{1/x}$, and any piecewise definition.

### 4.3 Algebra of limits (limit laws)

If $\displaystyle\lim_{x\to a}f(x)=L$ and $\displaystyle\lim_{x\to a}g(x)=M$ exist **and are finite**, then:

$$
\lim (f\pm g)=L\pm M,\qquad \lim(cf)=cL,\qquad \lim(fg)=LM,\qquad \lim\frac{f}{g}=\frac{L}{M}\ \ (M\neq 0).
$$

**Composition law (mind the condition):** if $\displaystyle\lim_{x\to a}g(x)=M$ **and $f$ is continuous at $M$**, then
$$\lim_{x\to a}f\big(g(x)\big)=f(M).$$
*Without continuity of $f$ at $M$, this can fail* — see Traps.

> **Caveat:** these laws need each individual limit to **exist finitely**. They do **not** license you to "split" indeterminate forms like $\infty-\infty$ or $\tfrac00$.

### 4.4 Standard limits — memorise these (all the constants come from here)

| Limit | Value | Condition |
|---|---|---|
| $\displaystyle\lim_{x\to0}\frac{\sin x}{x}$ | $1$ | $x$ in **radians** |
| $\displaystyle\lim_{x\to0}\frac{\tan x}{x}$ | $1$ | radians |
| $\displaystyle\lim_{x\to0}\frac{1-\cos x}{x^2}$ | $\tfrac12$ | radians |
| $\displaystyle\lim_{x\to0}\frac{1-\cos x}{x}$ | $0$ | radians |
| $\displaystyle\lim_{x\to0}\frac{e^x-1}{x}$ | $1$ | — |
| $\displaystyle\lim_{x\to0}\frac{a^x-1}{x}$ | $\ln a$ | $a>0$ |
| $\displaystyle\lim_{x\to0}\frac{\ln(1+x)}{x}$ | $1$ | — |
| $\displaystyle\lim_{x\to0}(1+x)^{1/x}=\lim_{x\to\infty}\Big(1+\tfrac1x\Big)^{x}$ | $e$ | — |
| $\displaystyle\lim_{x\to\infty}\Big(1+\tfrac kx\Big)^{x}$ | $e^{k}$ | — |
| $\displaystyle\lim_{x\to0}\frac{(1+x)^n-1}{x}$ | $n$ | — |
| $\displaystyle\lim_{x\to a}\frac{x^n-a^n}{x-a}$ | $n\,a^{\,n-1}$ | — |
| $\displaystyle\lim_{x\to\infty}\frac{\ln x}{x^{p}}\ (p>0)$ | $0$ | logs grow slowest |
| $\displaystyle\lim_{x\to\infty}\frac{x^{n}}{e^{x}}$ | $0$ | exponentials dominate |

**Growth hierarchy** (as $x\to\infty$): $\ \ln x \prec x^{p}\ (p>0)\prec a^{x}\ (a>1)\prec x!\,$. Whatever is "bigger" wins a $\tfrac{\infty}{\infty}$ race.

### 4.5 Indeterminate forms and how to break them

The **seven indeterminate forms** (plugging in gives no answer):
$$\frac00,\quad \frac{\infty}{\infty},\quad 0\cdot\infty,\quad \infty-\infty,\quad 1^{\infty},\quad 0^{0},\quad \infty^{0}.$$
*(Forms like $\infty+\infty=\infty$, $0^{\infty}=0$, $\infty\cdot\infty=\infty$ are **determinate** — no work needed.)*

**Techniques (try in this order):**

1. **Factor & cancel** the offending $(x-a)$ — for rational $\tfrac00$.
2. **Rationalise** (multiply by the conjugate) — for surds.
3. **Use a standard limit** from §4.4 — fastest when the form matches.
4. **L'Hôpital's Rule** — for $\tfrac00$ or $\tfrac{\infty}{\infty}$ **only**:
   $$\lim_{x\to a}\frac{f(x)}{g(x)}=\lim_{x\to a}\frac{f'(x)}{g'(x)}$$
   *Conditions:* $f,g$ differentiable near $a$ (except maybe at $a$), $g'(x)\neq0$ near $a$, the form is genuinely $\tfrac00$ or $\tfrac{\infty}{\infty}$, **and** the right-hand limit exists (finite or $\pm\infty$). You differentiate top and bottom **separately**, not as a quotient. Re-apply if still indeterminate.
5. **Taylor / series expansion** — extremely powerful for GATE: substitute $\sin x=x-\tfrac{x^3}{6}+\cdots$, $\cos x=1-\tfrac{x^2}{2}+\cdots$, $e^x=1+x+\tfrac{x^2}{2}+\cdots$, $\ln(1+x)=x-\tfrac{x^2}{2}+\cdots$ and read off the leading behaviour.
6. **Squeeze (Sandwich) theorem:** if $g(x)\le f(x)\le h(x)$ near $a$ and $\lim g=\lim h=L$, then $\lim f=L$.
   *Canonical use:* $-|x|\le x\sin\tfrac1x\le|x|\Rightarrow \displaystyle\lim_{x\to0}x\sin\tfrac1x=0$ (even though $\sin\tfrac1x$ oscillates wildly).

**Handling the exponent forms:**
- $\mathbf{0\cdot\infty}$ and $\mathbf{\infty-\infty}$: algebraically rewrite as $\tfrac00$ or $\tfrac{\infty}{\infty}$, then use L'Hôpital/Taylor.
- $\mathbf{1^{\infty},\,0^{0},\,\infty^{0}}$: take logs, or use the **shortcut for $1^{\infty}$**:
  $$\text{if } f\to1,\ g\to\infty,\quad \lim f^{\,g}=\exp\!\Big(\lim\, g\,(f-1)\Big)\quad(\text{since }\ln f\approx f-1).$$

### 4.6 Limits at infinity & infinite limits (asymptotes)

**At infinity**, for a rational function $\dfrac{p(x)}{q(x)}$, compare degrees:
$$
\lim_{x\to\infty}\frac{p}{q}=
\begin{cases}
0, & \deg p<\deg q\quad(\text{horizontal asymptote } y=0)\\[2pt]
\dfrac{\text{leading coeff }p}{\text{leading coeff }q}, & \deg p=\deg q\\[6pt]
\pm\infty, & \deg p>\deg q.
\end{cases}
$$
**Only the leading terms matter** at infinity.

**Infinite limits** ($\lim_{x\to a}f=\pm\infty$) signal a **vertical asymptote** at $x=a$ (e.g. $\lim_{x\to0^+}\tfrac1x=+\infty$). Note "$=\infty$" is shorthand for "grows without bound" — strictly, no finite limit exists.

### 4.7 Continuity — the three-condition definition

$f$ is **continuous at $a$** iff **all three** hold:
$$\textbf{(1) } f(a)\text{ is defined.}\qquad \textbf{(2) } \lim_{x\to a}f(x)\text{ exists.}\qquad \textbf{(3) } \lim_{x\to a}f(x)=f(a).$$

Equivalently ($\varepsilon$–$\delta$): $\ \forall\varepsilon>0\ \exists\delta>0:\ |x-a|<\delta\Rightarrow|f(x)-f(a)|<\varepsilon.$ (Here $x=a$ is **allowed** — note no "$0<$".)

- **On an interval $[a,b]$:** continuous at every interior point, **right-continuous** at $a$ ($\lim_{x\to a^+}f=f(a)$) and **left-continuous** at $b$.
- Standard functions — polynomials, rationals (on their domain), $\sqrt{\ }$, $e^x$, $\ln$ (on domain), $\sin,\cos,\tan$ (on domain), $|x|$ — are **continuous on their domains**.

### 4.8 Types of discontinuity

| Type | Condition | Example |
|---|---|---|
| **Removable** | limit exists but $\neq f(a)$, or $f(a)$ undefined ("hole") | $\dfrac{x^2-1}{x-1}$ at $x=1$ (limit $2$, value missing) |
| **Jump** (1st kind) | LHL and RHL exist but are **unequal** | $\lfloor x\rfloor$ at integers; $\operatorname{sgn}(x)$ at $0$ |
| **Infinite / essential** (2nd kind) | a one-sided limit is $\pm\infty$ or fails to exist (oscillation) | $\dfrac1x$ at $0$; $e^{1/x}$ at $0$; $\sin\tfrac1x$ at $0$ |

A removable discontinuity can be "patched" by **redefining $f(a)$** to equal the (two-sided) limit; the other two cannot.

### 4.9 Theorems about continuous functions

**Algebra & composition:** sums, differences, products, and quotients (denominator $\neq0$) of continuous functions are continuous; **composition** of continuous functions is continuous (if $g$ is continuous at $a$ and $f$ is continuous at $g(a)$, then $f\circ g$ is continuous at $a$).

**Intermediate Value Theorem (IVT).** If $f$ is continuous on $[a,b]$ and $k$ lies between $f(a)$ and $f(b)$, then $\exists\,c\in(a,b)$ with $f(c)=k$.
*Bolzano corollary (root existence):* if $f$ is continuous on $[a,b]$ and $f(a)f(b)<0$, then $f$ has a root in $(a,b)$.
*Illustration:* does $x=\cos x$ have a solution? Let $g(x)=x-\cos x$; $g(0)=-1<0$, $g(1)=1-\cos1>0$, $g$ continuous ⇒ a root exists in $(0,1)$. ✔

**Extreme Value Theorem (EVT).** If $f$ is continuous on a **closed, bounded** interval $[a,b]$, then $f$ is bounded and **attains** its maximum and minimum on $[a,b]$.
> All three words matter — **continuous**, **closed**, **bounded**. Drop any one and it can fail (e.g. $\tfrac1x$ is continuous on the *non-closed* $(0,1]$ but unbounded). This theorem is the foundation of optimization in Topic 5.

*(Uniform continuity is beyond GATE DA scope — one line is enough: it strengthens continuity to hold "at the same rate" across an interval.)*

---

## 5. Worked problems (GATE-style)

### Problem 1 — A $\tfrac00$ limit, two ways *(NAT)*

Evaluate $\displaystyle\lim_{x\to0}\frac{e^{x}-1-x}{x^{2}}.$

**Method A — Taylor.** $e^{x}=1+x+\dfrac{x^{2}}{2}+\dfrac{x^{3}}{6}+\cdots$, so the numerator is $\dfrac{x^{2}}{2}+\dfrac{x^{3}}{6}+\cdots$. Divide by $x^{2}$:
$$\frac{e^{x}-1-x}{x^{2}}=\frac12+\frac{x}{6}+\cdots\ \xrightarrow{\,x\to0\,}\ \boxed{\tfrac12}.$$

**Method B — L'Hôpital (twice).** Form is $\tfrac00$:
$$\lim_{x\to0}\frac{e^{x}-1-x}{x^{2}}\overset{\text{L'H}}{=}\lim_{x\to0}\frac{e^{x}-1}{2x}\ (\text{still }\tfrac00)\overset{\text{L'H}}{=}\lim_{x\to0}\frac{e^{x}}{2}=\frac12.\ \checkmark$$

Both give $\tfrac12$. *(Taylor is usually faster and avoids L'Hôpital bookkeeping.)*

---

### Problem 2 — Continuity, find the constant (with a trap) *(NAT)*

Let
$$f(x)=\begin{cases}\dfrac{1-\cos 2x}{x^{2}}, & x\neq0\\[6pt] k, & x=0.\end{cases}$$
Find $k$ so that $f$ is continuous at $x=0$.

**Solution.** Continuity at $0$ requires $k=\displaystyle\lim_{x\to0}\frac{1-\cos 2x}{x^{2}}$. Use $1-\cos 2x=2\sin^{2}x$:
$$\frac{1-\cos2x}{x^{2}}=\frac{2\sin^{2}x}{x^{2}}=2\left(\frac{\sin x}{x}\right)^{2}\ \xrightarrow{\,x\to0\,}\ 2(1)^{2}=2.$$
$$\boxed{k=2.}$$

> **The trap:** it is $\cos\mathbf{2}x$, not $\cos x$. If it were $\dfrac{1-\cos x}{x^{2}}$ the answer would be $\tfrac12$. Many students "remember $\tfrac12$" and never read the argument. Equivalently, $1-\cos2x\approx \tfrac{(2x)^2}{2}=2x^2$, so the ratio $\to2$.

---

### Problem 3 — A $1^{\infty}$ form, and a limit that does NOT exist *(MCQ + trap)*

**(a)** Evaluate $\displaystyle\lim_{x\to\infty}\left(\frac{x+3}{x-1}\right)^{x}.$

The base $\to1$ and the exponent $\to\infty$: form $1^{\infty}$. Use the shortcut $\lim f^{\,g}=\exp\!\big(\lim g\,(f-1)\big)$:
$$f-1=\frac{x+3}{x-1}-1=\frac{(x+3)-(x-1)}{x-1}=\frac{4}{x-1},\qquad g\,(f-1)=x\cdot\frac{4}{x-1}=\frac{4x}{x-1}\to4.$$
$$\Rightarrow\ \lim_{x\to\infty}\left(\frac{x+3}{x-1}\right)^{x}=\boxed{e^{4}}.$$

**(b)** Does $\displaystyle\lim_{x\to0}e^{1/x}$ exist? Classify the discontinuity of $e^{1/x}$ at $x=0$.

Check one-sided limits:
$$x\to0^{+}:\ \tfrac1x\to+\infty\Rightarrow e^{1/x}\to+\infty,\qquad x\to0^{-}:\ \tfrac1x\to-\infty\Rightarrow e^{1/x}\to0.$$
LHL $=0$, RHL $=+\infty$, so they **disagree** ⇒ the limit **does not exist**. Because a one-sided limit is infinite, the point $x=0$ is an **infinite / essential (second-kind) discontinuity**.

> **The trap:** writing $\frac10=\infty$ "from both sides." The sign of the approach flips the behaviour entirely. **Always split into LHL and RHL for $\tfrac1x$-type exponents, $|x|$, and floor.**

---

## 6. Common GATE question types on this topic

- **Evaluate a $\tfrac00$ or $\tfrac{\infty}{\infty}$ limit** — by factoring, rationalising, standard limit, L'Hôpital, or Taylor.
- **Evaluate $1^{\infty}$ / $0^{0}$ / $\infty^{0}$** limits (logs / shortcut).
- **Limits at infinity** of rational functions (degree comparison); find horizontal asymptotes.
- **Find a parameter** making a piecewise function continuous (set LHL $=$ RHL $=$ value).
- **Does the limit exist?** — check LHL vs RHL for $|x|$, $\lfloor x\rfloor$, $e^{1/x}$, $\sin\tfrac1x$.
- **Classify a discontinuity** (removable / jump / infinite).
- **Apply IVT / Bolzano** to prove a root exists in an interval, or count sign changes.
- **Squeeze theorem** for oscillatory products like $x^{2}\cos\tfrac1x$.
- **Continuity of composite or piecewise functions** — locate every point of discontinuity.

---

## 7. Traps, pitfalls & common mistakes

- $\dfrac{\sin x}{x}\to1$ only as $x\to0$ **and in radians**. (In degrees the limit is $\tfrac{\pi}{180}$.)
- $\dfrac{1-\cos x}{x^{2}}=\tfrac12$ but $\dfrac{1-\cos x}{x}=0$ — different powers, different answers. And $\cos 2x$ vs $\cos x$ changes the constant (Problem 2).
- **L'Hôpital only for $\tfrac00$ or $\tfrac{\infty}{\infty}$.** Applying it to $\tfrac20$ or $\tfrac53$ gives nonsense — **check the form first**. Differentiate top and bottom *separately* (not the quotient). If it loops or won't simplify, switch to Taylor.
- **Limit exists $\iff$ LHL $=$ RHL** (both finite and equal). Many "limits" actually DNE: $\tfrac{|x|}{x}$, $e^{1/x}$, $\sin\tfrac1x$ at $0$.
- A limit can exist where $f$ is **undefined**; and $f(a)$ can exist while the **limit** doesn't. Continuity needs **all three** conditions, not just $f(a)$.
- $\infty$ is **not a number**: "$=\infty$" means unbounded growth; you cannot do arithmetic like $\infty-\infty=0$.
- $0\cdot\infty,\ \infty-\infty,\ 1^{\infty},\ 0^{0},\ \infty^{0}$ are **indeterminate** — never just "plug in." For $1^{\infty}$ do **not** say "$1$ to anything is $1$"; use $\exp\!\big(\lim g(f-1)\big)$ or logs.
- **Composition-limit condition:** $\lim f(g(x))=f(\lim g)$ needs $f$ **continuous** at the inner limit. Without it the equality can fail.
- A **removable** discontinuity is patchable only because the **two-sided** limit exists; jumps and infinities are not.
- **IVT / EVT need their hypotheses.** IVT needs continuity on $[a,b]$; EVT needs **continuous + closed + bounded**. Drop "closed" and a max may not be attained ($\tfrac1x$ on $(0,1]$).
- **Squeeze** needs the two bounding limits to be **equal**.
- At $\infty$, only the **leading terms** of a rational function decide the limit — don't expand the whole thing.

---

## 8. Connection forward

Everything here was built so we can define the **derivative**. Topic 3, **Differentiability**, sets
$$f'(a)=\lim_{h\to0}\frac{f(a+h)-f(a)}{h},$$
a limit of exactly the $\tfrac00$ flavour we just learned to handle. The derivative exists only if this two-sided limit exists — so the **LHL $=$ RHL** machinery becomes the test for differentiability (left- vs right-derivative), and L'Hôpital/Taylor become our calculators. The key bridge: **differentiable $\Rightarrow$ continuous, but not conversely** — $|x|$ is continuous at $0$ yet has a corner (no derivative there). Continuity is the gateway; the derivative is what walks through it.

---

## 9. One-line cheat-sheet summary

> A limit is the value $f$ **heads toward** near $a$ (ignoring $f(a)$); it exists iff **LHL $=$ RHL**. $f$ is **continuous at $a$** iff $f(a)$ exists, the limit exists, and they're equal. Kill $\tfrac00$ / $\tfrac{\infty}{\infty}$ with factoring, standard limits, **L'Hôpital** (only those forms!), or **Taylor**; use **squeeze** for oscillations and **IVT/EVT** (needs a **closed, bounded** interval) for roots and extrema. Watch: radians, $\cos2x\neq\cos x$, $1^{\infty}\neq1$, and split one-sided limits for $\tfrac1x$, $|x|$, $\lfloor x\rfloor$.

---

### ▶ Next topic
**Topic 3 — Differentiability** (the derivative as a limit; differentiable ⇒ continuous; rules and first-principles). Ask for it when ready.

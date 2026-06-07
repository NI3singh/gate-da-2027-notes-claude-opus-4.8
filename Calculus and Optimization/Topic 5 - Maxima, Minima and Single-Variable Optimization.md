# Topic 5 — Maxima, Minima & Single-Variable Optimization

> **Subject:** Calculus and Optimization
> **Section:** Calculus & Single-Variable Optimization (Topic 5 of 5 — **final topic**)
> **GATE DA syllabus line covered:** *"…Taylor series, maxima and minima, optimization involving a single variable."* (the closing clause — this completes the syllabus line)

---

## 0. Topic header & syllabus map

- **Topic:** Maxima, Minima & Single-Variable Optimization
- **Belongs to:** Calculus and Optimization → the *Calculus & single-variable optimization* cluster.
- **Where it sits:** **Topic 5 of 5 — the capstone.** It comes **after Taylor Series** (Topic 4), whose quadratic expansion *is* the second-derivative test, and it **completes** the Calculus & Single-Variable Optimization section. It is the payoff: everything earlier was machinery for finding a function's best value.

In one line: *the whole section was building to this — use the slope to find candidate peaks/valleys, use the curvature to classify them, and check the boundaries to find the single best value.*

---

## 1. Why this topic exists (motivation + connection)

**The real problem it solves.** Almost every practical question is an optimization: minimize cost, loss, error, or material; maximize area, volume, profit, or accuracy. In data science, *training a model is literally minimizing a loss function.* This topic gives the calculus tools to **locate** a function's extreme points and **certify** whether each is a maximum, a minimum, or neither — locally and globally.

**What the previous topic (Taylor Series) couldn't do on its own.** Taylor handed us the *local* polynomial picture — in particular, the quadratic term told us about curvature. But Taylor by itself doesn't *find* the best point or tell us if a point is the best *overall*. We need a full procedure: (1) locate candidates where the tangent is flat or undefined, plus the boundaries; (2) classify each using the curvature/sign information; (3) compare to find the global best.

**The bridge, stated plainly:** *Differentiability gave us $f'=0$ as the flat-tangent condition; Taylor's quadratic expansion justified that the sign of $f''$ classifies a flat point. This topic assembles those into a complete optimization method and extends it to global extrema and applied problems.*

**What this enables (since it closes the section).** The single-variable theory here is the conceptual seed of the rest of GATE DA: multivariable optimization ($\nabla f=0$, the Hessian test), constrained optimization (Lagrange multipliers), and convex optimization — the backbone of machine learning. **Gradient descent** is just the iterative version of "step opposite the slope toward the minimum."

---

## 2. Prerequisites

- **Topic 3 — Differentiability:** $f',f''$, critical points, and monotonicity from the sign of $f'$.
- **Topic 4 — Taylor Series:** the quadratic / higher-order expansion that *justifies* the derivative tests.
- **Topic 2 — Limits & Continuity:** the **Extreme Value Theorem** (guarantees global extrema on closed, bounded intervals) and boundary/$\infty$ limit behaviour.
- **Topic 1 — Functions:** domain, intervals, monotonicity, convex-shaped standard functions.

---

## 3. Core idea — intuition first

Picture the graph as a **hilly landscape**. Local maxima are hilltops, local minima are valley bottoms. The single highest hilltop over the whole allowed region is the **global maximum**; the lowest valley is the **global minimum**.

**Where can a highest/lowest point sit?** Three kinds of places:
1. **Flat spots** where the tangent is horizontal ($f'=0$) — smooth peaks and valleys.
2. **Kinks/corners** where the slope is undefined ($|x|$ has a valley at its corner).
3. **The boundary** — the edges/endpoints of the allowed region (a trail can be highest at its endpoint).

**Analogy (a hike).** To find the highest elevation you'll reach, check every momentarily-flat point, every sharp peak, and both trailheads — then compare heights. You can't skip the trailheads.

**The crucial subtlety.** A flat spot is **not automatically** a peak or valley. The curve $y=x^3$ has $f'(0)=0$ yet just keeps rising through $0$ — a *horizontal inflection*, no extremum. So $f'=0$ is a **necessary** clue, never a **sufficient** verdict: you must classify.

---

## 4. Formal treatment

### 4.1 Definitions: local vs global extrema

- **Local (relative) maximum** at $c$: $f(c)\ge f(x)$ for all $x$ in some neighbourhood of $c$. **Local minimum:** $f(c)\le f(x)$ nearby.
- **Global (absolute) maximum** at $c$: $f(c)\ge f(x)$ for **all** $x$ in the domain. **Global minimum** likewise.
- *Strict* if the inequality is strict for $x\neq c$.
- Global extrema **may or may not exist**, depending on the domain (EVT guarantees them for a continuous $f$ on a closed, bounded interval).

### 4.2 Critical points and Fermat's theorem (the necessary condition)

> A **critical point** of $f$ is a point $c$ in the domain where $f'(c)=0$ **or** $f'(c)$ does not exist. (A point with $f'(c)=0$ is also called a **stationary point**.)

> **Fermat's Theorem (necessary condition).** If $f$ has a local extremum at an *interior* point $c$ and $f'(c)$ exists, then $f'(c)=0$.

Two warnings packed into this:
- **Necessary, not sufficient:** $f'(c)=0$ does not force an extremum ($y=x^3$ at $0$).
- Extrema can occur where **$f'$ is undefined** (corner: $|x|$ at $0$ is a minimum) or at **endpoints**.

$$\boxed{\textbf{Candidates for extrema}=\{f'=0\}\ \cup\ \{f' \text{ undefined}\}\ \cup\ \{\text{endpoints}\}.}$$

### 4.3 First-derivative test (classify by the sign change of $f'$)

At a critical point $c$, look at the sign of $f'$ just to the left and right:

| $f'$ pattern around $c$ | Verdict at $c$ |
|---|---|
| $+\ \to\ -$ | local **maximum** |
| $-\ \to\ +$ | local **minimum** |
| same sign (no change) | **neither** (e.g. horizontal inflection) |

This is the **most robust** test — it works even when $f''(c)=0$ or $f''(c)$ doesn't exist.

### 4.4 Second-derivative test (classify by curvature)

Suppose $f'(c)=0$ and $f''(c)$ exists:
$$f''(c)>0\ \Rightarrow\ \text{local \textbf{minimum} (concave up)},\qquad f''(c)<0\ \Rightarrow\ \text{local \textbf{maximum} (concave down)},$$
$$f''(c)=0\ \Rightarrow\ \textbf{inconclusive}\ (\text{fall back to the first-derivative or higher-order test}).$$

**Why it works (Taylor, Topic 4):** near $c$ with $f'(c)=0$, $\ f(x)-f(c)\approx\tfrac12 f''(c)(x-c)^2$; since $(x-c)^2\ge0$, the sign of $f''(c)$ alone fixes max vs min.

### 4.5 Higher-order derivative test (when $f''(c)=0$)

From Taylor: if $f'(c)=f''(c)=\cdots=f^{(n-1)}(c)=0$ but $f^{(n)}(c)\neq0$, then $f(x)-f(c)\approx\dfrac{f^{(n)}(c)}{n!}(x-c)^n$, so:

- **$n$ even:** extremum — local **min** if $f^{(n)}(c)>0$, local **max** if $f^{(n)}(c)<0$.
- **$n$ odd:** **not** an extremum (a horizontal inflection, like $x^3$).

*Example:* $f(x)=x^4$ at $0$: $f'=f''=f'''=0$, $f^{(4)}(0)=24>0$, $n=4$ even ⇒ local **minimum** (even though $f''(0)=0$).

### 4.6 Concavity, convexity, and inflection points

- **Concave up / convex:** $f''>0$ — bends upward, lies *above* its tangents, chords lie above the graph.
- **Concave down / concave:** $f''<0$ — bends downward.
- **Inflection point:** a point where concavity **changes** (i.e., $f''$ **changes sign**). There $f''=0$ or $f''$ is undefined **and** the sign flips.

> **Trap:** $f''=0$ alone is **not** an inflection — the sign must change. $x^4$ has $f''(0)=0$ but stays concave up, so no inflection.

**Convex function (formal definition):** $f$ is convex on an interval $I$ if for all $x,y\in I$ and $\lambda\in[0,1]$,
$$f\big(\lambda x+(1-\lambda)y\big)\le \lambda f(x)+(1-\lambda)f(y).$$
For twice-differentiable $f$: **convex $\iff f''\ge0$ on $I$** (strictly convex if $f''>0$). Concave reverses the inequalities.

### 4.7 Global (absolute) extrema

**(a) On a closed, bounded interval $[a,b]$ with $f$ continuous — the Closed-Interval Method.**
By **EVT** (Topic 2), the global max and min are **guaranteed to exist**. Procedure:
1. Find all **critical points** in $(a,b)$.
2. Evaluate $f$ at those critical points **and at both endpoints** $a,b$.
3. The **largest** value is the global maximum; the **smallest** is the global minimum.

*(No classification of each critical point is needed — you just compare function values.)* This is the standard GATE method for "find the max/min of $f$ on $[a,b]$."

**(b) On open or unbounded intervals.**
EVT does **not** apply — a global extremum may **fail to exist**. Examine the behaviour at the open ends / as $x\to\pm\infty$ via limits. If $f$ approaches a value it never attains, there is no extremum there. With a single interior critical point and the function rising to $+\infty$ at both ends, that point is the global minimum (use monotonicity / first-derivative reasoning to confirm).

### 4.8 Convexity ⇒ global optimality (the machine-learning result)

> **Key theorem.** If $f$ is **convex** on an interval, then **every local minimum is a global minimum**. If $f$ is **strictly convex**, the global minimizer (when it exists) is **unique**. (Dually: for **concave** $f$, every local max is a global max.)

This is exactly why **convex loss functions** are prized in ML: any local minimum a gradient method lands on is automatically the global minimum — no spurious local minima. *(Related: **Jensen's inequality** — for convex $f$, $\ f(\mathbb{E}[X])\le \mathbb{E}[f(X)]$ — links to probability and information theory elsewhere in DA.)*

### 4.9 Setting up applied optimization problems

A reliable five-step recipe:
1. Identify the quantity to optimize and write it as a function of **one** variable, using the constraint(s) to eliminate the others.
2. Determine the **domain** from physical constraints (lengths $>0$, etc.).
3. Find the critical points.
4. Classify and **compare** (include endpoints / boundary behaviour).
5. Interpret the answer (with units) and confirm it's a max/min as required.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Global extrema on a closed interval *(NAT)*

Find the absolute maximum and minimum of $f(x)=x^{3}-3x+2$ on $[0,3]$.

**Solution (Closed-Interval Method).**
$$f'(x)=3x^{2}-3=0\ \Rightarrow\ x=\pm1;\quad \text{only } x=1\in(0,3).$$
Evaluate $f$ at the critical point and **both endpoints**:
$$f(0)=2,\qquad f(1)=1-3+2=0,\qquad f(3)=27-9+2=20.$$
$$\boxed{\text{Global max}=20\ \text{at } x=3\ (\text{endpoint});\quad \text{global min}=0\ \text{at } x=1.}$$

> **Trap:** the global maximum is **at the endpoint $x=3$**, not at the interior critical point. A student who only solves $f'=0$ (getting $x=1$, a local *minimum*) would completely miss the maximum. **Always evaluate the endpoints.**

---

### Problem 2 — Classify critical points; the second-derivative test fails *(MCQ / MSQ)*

Find and classify all critical points of $f(x)=3x^{5}-5x^{3}$.

**Solution.**
$$f'(x)=15x^{4}-15x^{2}=15x^{2}(x^{2}-1)=0\ \Rightarrow\ x=0,\ \pm1.$$
$$f''(x)=60x^{3}-30x=30x(2x^{2}-1).$$

- **$x=1$:** $f''(1)=30(1)(2-1)=30>0$ ⇒ **local minimum**, value $f(1)=3-5=-2$.
- **$x=-1$:** $f''(-1)=30(-1)(2-1)=-30<0$ ⇒ **local maximum**, value $f(-1)=-3+5=2$.
- **$x=0$:** $f''(0)=0$ ⇒ **inconclusive**. Use the **first-derivative test**: $f'(x)=15x^{2}(x^{2}-1)$; for $|x|<1$, $x^{2}\ge0$ and $(x^{2}-1)<0$, so $f'<0$ on **both** sides of $0$ (no sign change) ⇒ $x=0$ is **not an extremum** (a horizontal inflection). *(Higher-order check: $f'''(0)=-30\neq0$, $n=3$ odd ⇒ no extremum — consistent.)*

> **Trap:** when $f''(c)=0$ the second-derivative test says **nothing** — you must switch to the first-derivative (or higher-order) test. Concluding "$x=0$ is an extremum because $f'(0)=0$" is the classic error.

---

### Problem 3 — Applied optimization (max area) *(NAT)*

A rectangular field is fenced along **three** sides; the fourth side is a straight river needing no fence. With $100$ m of fencing, find the **maximum** area.

**Solution.** Let $x$ = each side perpendicular to the river, $y$ = the side parallel to it. Fence used: $2x+y=100\Rightarrow y=100-2x$. Area:
$$A(x)=x\,y=x(100-2x)=100x-2x^{2},\qquad 0<x<50.$$
$$A'(x)=100-4x=0\ \Rightarrow\ x=25,\qquad A''(x)=-4<0\ \Rightarrow\ \text{maximum}.$$
Then $y=100-2(25)=50$, and
$$\boxed{A_{\max}=25\times50=1250\ \text{m}^{2}.}$$

> **Setup lesson:** the constraint reduced the problem to **one** variable before differentiating, the domain $0<x<50$ was stated, and $A''<0$ certified a maximum. *(General fact worth knowing: among rectangles of fixed perimeter the **square** maximizes area; here the asymmetry comes from the free river side.)*

---

## 6. Common GATE question types on this topic

- **Absolute max/min on $[a,b]$** via the Closed-Interval Method.
- **Find and classify local extrema** (first- / second-derivative test).
- **Spot a stationary point that is NOT an extremum** (horizontal inflection / saddle).
- **Inflection points & intervals of concavity** ($f''$ sign changes).
- **Convexity** ($f''\ge0$) and using it to claim global optimality.
- **Applied / word optimization** (max area or volume, min cost, material, or distance).
- **Parameter problems:** find a constant for which $f$ has an extremum, or a stated extremum value.
- **Monotonicity intervals** (increasing/decreasing from the sign of $f'$).

---

## 7. Traps, pitfalls & common mistakes

- **$f'(c)=0$ is necessary, not sufficient.** Stationary $\neq$ extremum ($x^3$, $3x^5-5x^3$ at $0$). Always classify.
- **Never skip endpoints** on a closed interval — the global extremum is frequently *at* an endpoint (Problem 1).
- **Include points where $f'$ is undefined.** Critical points are $\{f'=0\}\cup\{f' \text{ undefined}\}$ — extrema can live at corners ($|x|$ at $0$ is a global min).
- **Second-derivative test is silent when $f''(c)=0$** — switch to the first-derivative / higher-order test. Don't declare "no extremum" just because $f''=0$ ($x^4$ at $0$ *is* a minimum).
- **Inflection needs $f''$ to change sign**, not merely $f''=0$ ($x^4$ at $0$: $f''=0$ but no inflection).
- **Global extrema may not exist** on open/unbounded intervals or for discontinuous $f$ — EVT needs **continuous + closed + bounded**. Check boundary/$\infty$ behaviour.
- **Local $\neq$ global.** Only special structure (e.g. convexity for a minimum) makes a local extremum global.
- **Sign convention:** $f''>0\Rightarrow$ **min** (concave up); $f''<0\Rightarrow$ **max** (concave down). Swapping these is a frequent slip.
- **In word problems:** reduce to **one** variable using the constraint *before* differentiating, and **respect the domain** — a critical point outside the feasible range is irrelevant.
- **Maximizing $f$ = minimizing $-f$** (same critical points) — don't duplicate work.
- For convexity-based global claims, verify $f''\ge0$ on the **whole** interval, not just at one point.
- **"Critical point" (broader) vs "stationary point" ($f'=0$ only)** — keep the distinction straight.

---

## 8. Connection forward — and section wrap-up

This topic is the **destination** of the whole arc:
**Functions** (the object) → **Limits & Continuity** (local behaviour + EVT) → **Differentiability** (slopes, $f'=0$) → **Taylor** (curvature, the second-derivative test) → **Optimization** (finding and certifying the best value).

From here it generalises directly into the rest of GATE DA:
- **Multivariable optimization:** stationary points become $\nabla f=0$; the second-derivative test becomes the **Hessian** test (positive-definite ⇒ min, negative-definite ⇒ max, indefinite ⇒ saddle).
- **Constrained optimization:** the **Lagrange-multiplier** method optimizes subject to constraints.
- **Convex optimization & machine learning:** convex losses guarantee local-min $=$ global-min; **gradient descent** iterates "step opposite the slope," the discrete analogue of the $f'=0$ idea developed here.

*(Those multivariable/constrained tools appear in other parts of the GATE DA syllabus — Linear Algebra, Probability, ML — but rest squarely on the single-variable foundations of this section.)* **The Calculus & Single-Variable Optimization section is now complete.**

---

## 9. One-line cheat-sheet summary

> **Candidates** $=\{f'=0\}\cup\{f' \text{ undefined}\}\cup\{\text{endpoints}\}$. **Classify** with the first-derivative sign change ($+\!\to\!-$ max, $-\!\to\!+$ min) or the second-derivative test ($f''>0$ min, $f''<0$ max, $f''=0$ inconclusive → first/higher-order test). **Global extrema on $[a,b]$:** compare $f$ at all critical points **and both endpoints** (Closed-Interval Method, guaranteed by EVT). **Convex $f$ ($f''\ge0$):** every local min is the global min. $f'=0$ is necessary, never sufficient — always certify, and never forget the boundary.

---

### ▶ Section complete
This finishes **all five topics** of *Calculus & Single-Variable Optimization* (Functions → Limits & Continuity → Differentiability → Taylor Series → Maxima, Minima & Optimization). When you're ready, start the next subject and I'll continue in the same format. If useful, I can also produce a one-page consolidated revision sheet spanning all five topics — just ask.

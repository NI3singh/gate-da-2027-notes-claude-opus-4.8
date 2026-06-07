# Topic 6 — Discrete Random Variables and Probability Mass Functions (PMF)

*Subject: Probability and Statistics (GATE DA) · Cluster: Random Variables & Expectation (begins here)*

---

## 0. Topic header & syllabus map

- **Topic:** Discrete Random Variables and Probability Mass Functions (PMF)
- **Subject:** Probability and Statistics
- **Where it sits:** This is the **first** topic of the *Random Variables & Expectation* block. It comes right after **Bayes' theorem** (which closed the *Probability foundations* block, Topics 1–5) and it re-expresses everything you learned there in **numerical** form. It leads directly into **Topic 7 — Continuous Random Variables, PDF & CDF**, which keeps the same machinery but generalizes from a *countable* set of values to a *continuum*.

---

## 1. Why this topic exists (motivation + connection)

- **The real problem.** Topics 1–5 let you compute the probability of *events* — "at least one head," "the card is a king." But most useful questions are about *numbers*: *How many* heads in 10 tosses? What is the *sum* of two dice? How many *defective* items in a batch? You cannot average an event, but you can average a number.
- **What the previous topic couldn't do.** Bayes' theorem and the rest of the foundations only manipulate probabilities of *sets*. They give you no native way to talk about "the typical value," "the spread," or to feed an outcome into arithmetic — there is no operation called "expected value of the event {at least one head}."
- **The fix.** A **random variable** is a rule that attaches a real number to every outcome. Once outcomes are numbers, all of arithmetic, averaging, and (later) calculus becomes available. The **PMF** is a discrete random variable's complete probabilistic fingerprint: it states exactly how likely each value is.
- **Forward.** Topic 7 keeps this idea but for variables whose values fill an interval (e.g., a measured length), where listing every value one-by-one is impossible — so the PMF must be replaced by a density.

---

## 2. Prerequisites

- **Sample space, events, probability axioms** (Topic 2) — a random variable is a function *on* the sample space.
- **Joint, marginal & conditional probability** (Topic 4) — reused directly for **joint PMFs** and marginals.
- **Independence of events** (Topic 3) — generalized to **independence of random variables**.
- **Series:** finite arithmetic sums, and the **infinite geometric series** $\sum_{k\ge 0} r^k = \dfrac{1}{1-r}$ for $|r|<1$ — needed whenever the support is countably infinite.

---

## 3. Core idea — intuition first

Think of a random variable as a **measuring instrument** pointed at the experiment. The experiment produces some outcome ω in the sample space; the instrument reads off a number X(ω). The dial's reading looks "random" only because the outcome feeding it is random — **the instrument itself is a fixed function**, not random and not a variable in the algebra sense.

**Example.** Toss a fair coin twice. Sample space Ω = {HH, HT, TH, TT}. Let X = number of heads. Then

- X(HH) = 2, X(HT) = 1, X(TH) = 1, X(TT) = 0.

X collapses four outcomes into three numbers {0, 1, 2}. The **PMF** just records how much probability lands on each number:

- P(X = 0) = 1/4, P(X = 1) = 2/4, P(X = 2) = 1/4.

That little table **is** everything probabilistic about X. Notice that two outcomes (HT and TH) both map to 1, so their probabilities **add**. This "**pile up the probability over everything that maps to the same value**" idea is the single most important mechanical habit in the whole topic.

**Discrete vs continuous, intuitively.** A *discrete* RV's possible readings can be **listed** — 0, 1, 2, … — i.e., put in a sequence (finite or infinite). A *continuous* RV's readings fill a whole interval (1.7 cm, 1.7001 cm, …) and **cannot** be listed. Topic 6 is the listable case.

---

## 4. Formal treatment

### Definitions

- **Random variable.** Given a sample space Ω, a random variable is a function X : Ω → ℝ. *(Formally it must be "measurable," but GATE never tests measurability — every function you will meet qualifies.)*
- **Discrete RV.** X is **discrete** if its range — the set of values it actually takes — is **countable** (finite or countably infinite).
- **Support.** $S_X = \{\, x : P(X=x) > 0 \,\}$. Outside the support the PMF is 0.
- **PMF.** The probability mass function of a discrete RV X is
$$p_X(x) = P(X = x) = P\big(\{\,\omega \in \Omega : X(\omega) = x\,\}\big).$$

### Properties of a valid PMF

| | Property | Condition / meaning |
|---|---|---|
| **P1** | $p_X(x) \ge 0$ for every $x$ | probabilities are non‑negative |
| **P2** | $\displaystyle\sum_{x \in S_X} p_X(x) = 1$ | total probability; **sum over the whole support** |
| **P3** | $\displaystyle P(X \in A) = \sum_{x \in A} p_X(x)$ | probability of any value‑set $A \subseteq \mathbb{R}$ |

*Conditions:* $S_X$ is countable; the sum in P2/P3 is an ordinary (possibly infinite) series. **A function that fails P1 or P2 is not a PMF.**

### Cumulative distribution function (CDF)

$$F_X(x) = P(X \le x) = \sum_{\substack{t \in S_X \\ t \le x}} p_X(t).$$

**Properties of the CDF** — these hold for **any** random variable, discrete *or* continuous. This is exactly the bridge to Topic 7.

- **C1.** Non-decreasing.
- **C2.** **Right-continuous:** $F_X(x) = \lim_{h\to 0^+} F_X(x+h)$.
- **C3.** $\lim_{x\to -\infty} F_X(x) = 0$ and $\lim_{x\to +\infty} F_X(x) = 1$.
- **C4.** For a discrete RV, $F_X$ is a **step function**: flat between support points, with an upward **jump at each support point $a$ of size** $p_X(a) = F_X(a) - F_X(a^-)$.
- **C5.** $P(a < X \le b) = F_X(b) - F_X(a)$.

**Derived probability formulas (discrete).** Endpoints matter here because positive mass sits *at* the points:

$$
\begin{aligned}
P(X = a) &= F_X(a) - F_X(a^-) &&\text{(jump size at }a\text{)}\\
P(X < a) &= F_X(a^-) = F_X(a) - p_X(a)\\
P(X > a) &= 1 - F_X(a)\\
P(X \ge a) &= 1 - F_X(a^-) = 1 - F_X(a) + p_X(a)\\
P(a \le X \le b) &= F_X(b) - F_X(a^-)
\end{aligned}
$$

### Function of a discrete RV

If $Y = g(X)$, then $Y$ is also discrete and
$$p_Y(y) = \sum_{x \,:\, g(x) = y} p_X(x).$$
*Condition:* **sum over all preimages** of $y$. If $g$ is one‑to‑one this is a single term; if $g$ is many‑to‑one (e.g., $g(x) = x^2$), several PMF values **merge**.

### Expectation — *preview only* (fully developed in Topic 9)

So that PMF problems are self-contained, the two formulas you may need are:
$$E[X] = \sum_x x\,p_X(x), \qquad E[g(X)] = \sum_x g(x)\,p_X(x)\ \text{ (LOTUS)},$$
each valid **provided the series converges absolutely**. Variance, conditional expectation, and the algebraic properties of expectation (linearity, etc.) belong to **Topic 9** — do not reach for them here.

### Joint PMF (two discrete variables)

For discrete X, Y:
$$p_{X,Y}(x,y) = P(X = x,\; Y = y), \qquad \sum_x \sum_y p_{X,Y}(x,y) = 1.$$

- **Marginal PMF** (sum a row / a column — this is Topic 4's marginalization in RV clothing):
$$p_X(x) = \sum_y p_{X,Y}(x,y), \qquad p_Y(y) = \sum_x p_{X,Y}(x,y).$$
- **Independence:** X and Y are independent **iff** $p_{X,Y}(x,y) = p_X(x)\,p_Y(y)$ for **every** $(x,y)$. One violating cell makes them dependent.

> **Derivations GATE does not ask (skipped on purpose):** the measure-theoretic construction of a random variable, and the proof that a step function with properties C1–C3 corresponds to a unique distribution. GATE never requires these.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Normalizing constant + probabilities (NAT/MCQ)

A discrete RV X has PMF $p_X(x) = c\,x$ for $x \in \{1,2,3,4\}$ and 0 otherwise.
**(a)** Find $c$. **(b)** $P(X \ge 3)$. **(c)** $P(X \text{ is even})$. **(d)** Write $F_X$ and evaluate $F_X(2.5)$.

**Solution.**
(a) $\sum p = c(1+2+3+4) = 10c = 1 \Rightarrow \boxed{c = 1/10}$. So $p(1)=.1,\ p(2)=.2,\ p(3)=.3,\ p(4)=.4$.
(b) $P(X\ge 3) = p(3)+p(4) = .3 + .4 = \mathbf{0.7}$.
(c) Even values 2, 4: $.2 + .4 = \mathbf{0.6}$.
(d) Step function:
$$F_X(x)=\begin{cases}0 & x<1\\ 0.1 & 1\le x<2\\ 0.3 & 2\le x<3\\ 0.6 & 3\le x<4\\ 1 & x\ge 4.\end{cases}$$
$F_X(2.5) = \mathbf{0.3}$.

### Problem 2 — Reading a CDF: the strict / non-strict trap (MSQ-style)

A discrete RV has CDF
$$F(x)=\begin{cases}0 & x<0\\ 0.2 & 0\le x<1\\ 0.5 & 1\le x<3\\ 0.9 & 3\le x<5\\ 1 & x\ge 5.\end{cases}$$
Find: **(a)** the PMF, **(b)** $P(X=3)$, **(c)** $P(X<3)$, **(d)** $P(1 \le X \le 3)$, **(e)** $P(X>1)$.

**Solution.**
Support = jump points {0, 1, 3, 5}. Jump sizes:
$$p(0)=0.2,\quad p(1)=0.5-0.2=0.3,\quad p(3)=0.9-0.5=0.4,\quad p(5)=1-0.9=0.1.$$
(b) $P(X=3) = $ jump at 3 $= \mathbf{0.4}$.
(c) $P(X<3) = F(3^-) = \mathbf{0.5}$. **Not** $F(3)=0.9$ — that value already *includes* the mass sitting at 3.
(d) $P(1\le X\le 3) = F(3) - F(1^-) = 0.9 - 0.2 = \mathbf{0.7}$.   *(here $F(1^-)=0.2$, the value just below 1.)*
(e) $P(X>1) = 1 - F(1) = 1 - 0.5 = \mathbf{0.5}$.

> **Trap highlighted:** $P(X<3)=0.5$ but $P(X\le 3)=0.9$; they differ by exactly $p(3)=0.4$. Forgetting this is the single most common discrete‑RV error.

### Problem 3 — Function of an RV, many-to-one (classic trap, NAT)

X is uniform on $\{-2,-1,0,1,2\}$ (each value probability 1/5). Let $Y = X^2$. Find the PMF of Y and $P(Y \ge 1)$.

**Solution.**
$Y$ values: $(-2)^2=4,\ (-1)^2=1,\ 0^2=0,\ 1^2=1,\ 2^2=4 \Rightarrow Y\in\{0,1,4\}$. Sum probabilities over each preimage:
$$
p_Y(0)=P(X=0)=\tfrac15,\quad
p_Y(1)=P(X=-1)+P(X=1)=\tfrac25,\quad
p_Y(4)=P(X=-2)+P(X=2)=\tfrac25.
$$
Check: $\tfrac15+\tfrac25+\tfrac25 = 1$ ✓.  $P(Y\ge 1) = p_Y(1)+p_Y(4) = \tfrac45 = \mathbf{0.8}$.

> **Trap:** writing $p_Y(y)=p_X(\sqrt y)$ treats $g$ as injective and drops the negative root — wrong. Always collect **every** $x$ with $g(x)=y$.

### Problem 4 — Countably infinite support: geometric series (NAT)

X takes values $k = 1,2,3,\dots$ with $p_X(k) = c\,(1/3)^k$.
**(a)** Find $c$. **(b)** $P(X \text{ is odd})$.

**Solution.**
(a) $\displaystyle\sum_{k=1}^{\infty} c\,(1/3)^k = c\cdot\frac{1/3}{1-1/3} = c\cdot\tfrac12 = 1 \Rightarrow \boxed{c = 2}$.  So $p(k)=2\,(1/3)^k$ (e.g. $p(1)=2/3,\ p(2)=2/9,\dots$).
(b) Odd $k = 1,3,5,\dots$:
$$\sum_{j=0}^{\infty} 2\,(1/3)^{2j+1} = 2\cdot\tfrac13\sum_{j=0}^{\infty}(1/9)^j = \tfrac23\cdot\frac{1}{1-1/9} = \tfrac23\cdot\tfrac98 = \mathbf{0.75}.$$
Sanity check: $P(\text{even}) = 1 - 3/4 = 1/4$; direct sum $2(1/9)/(1-1/9)=2/8=1/4$ ✓.

### Problem 5 — Joint PMF: marginals, independence, conditional (MSQ)

Joint PMF of $(X,Y)$, $X\in\{0,1\}$, $Y\in\{0,1\}$:

| $p(x,y)$ | $Y=0$ | $Y=1$ |
|---|---|---|
| $X=0$ | 0.1 | 0.2 |
| $X=1$ | 0.3 | 0.4 |

**(a)** Marginals of X and Y. **(b)** Are X, Y independent? **(c)** $P(X=1 \mid Y=1)$.

**Solution.**
(a) $p_X(0)=0.1+0.2=0.3,\ p_X(1)=0.7$;  $p_Y(0)=0.1+0.3=0.4,\ p_Y(1)=0.6$.
(b) Test one cell: $p(0,0)=0.1$ vs $p_X(0)\,p_Y(0)=0.3\times0.4=0.12$. Since $0.1\ne0.12$, **X and Y are dependent.**
(c) $P(X=1\mid Y=1) = \dfrac{p(1,1)}{p_Y(1)} = \dfrac{0.4}{0.6} = \mathbf{2/3}$.  *(Topic 4 conditional probability, now on RVs.)*

---

## 6. Common GATE question types on this topic

- Find the **normalizing constant $c$** so a given function is a valid PMF (finite support **or** geometric‑series support).
- Compute $P(X\in A)$, $P(X\ge a)$, $P(X\text{ even/odd})$, etc. from a PMF.
- **Build a CDF** from a PMF, or **recover a PMF** from a step‑function CDF; evaluate $F$ at a point.
- Probabilities from a CDF using **strict vs non‑strict** endpoints ($P(X<a)$ vs $P(X\le a)$).
- **PMF of $Y=g(X)$**, especially many‑to‑one $g$ ($X^2$, $|X|$).
- **Joint PMF tables:** marginals, independence test, conditional probability, probability of a region.
- Decide whether a given table/function is a **legitimate PMF** (check P1 and P2).
- Combined with Topic 9: compute $E[X]$, $\mathrm{Var}(X)$ from a PMF.

---

## 7. Traps, pitfalls & common mistakes

- **Strict vs non-strict endpoints.** For discrete RVs $P(X<a)\ne P(X\le a)$; they differ by $p(a)$. This is the **#1 trap**. (Continuous RVs in Topic 7 will not have it.)
- **Many-to-one functions.** For $Y=g(X)$ you must sum over **all** $x$ with $g(x)=y$. Dropping a branch (the negative root in $X^2$) is the classic mistake.
- **Reading the CDF at a jump.** $F$ is **right‑continuous**: at a jump point $a$, $F(a)$ is the *upper* value and equals $P(X\le a)$; the lower value is $F(a^-)=P(X<a)$. Pick the wrong one and every endpoint probability is off.
- **Forgetting normalization / non-negativity.** A "PMF" that sums to $\ne 1$ or has a negative entry is **invalid**. After solving for $c$, *check* that $c$ keeps all values non‑negative.
- **Support discipline.** $p(x)=0$ off the support — don't slip in extra values, and don't forget the support may be **infinite**.
- **PMF ≠ PDF.** PMF values **are** probabilities, so $0\le p(x)\le 1$ always. (In Topic 7 a density $f(x)$ can *exceed* 1 — it is **not** a probability.) Mixing these up is a frequent conceptual error.
- **$P(X=x)$ is not always 0.** For discrete RVs, point masses are real and usually nonzero — the *opposite* of the continuous case. Carrying over the continuous "$P(X=x)=0$" intuition here is wrong.
- **Independence needs every cell.** $X,Y$ independent requires $p(x,y)=p_X(x)p_Y(y)$ for **all** $(x,y)$: checking one matching cell does **not** prove independence, but one mismatching cell **disproves** it.
- **Absolute convergence for $E[X]$.** On infinite supports, $E[X]$ exists only if $\sum |x|\,p(x)$ converges. GATE occasionally exploits a divergent case (full treatment in Topic 9, but be aware).

---

## 8. Connection forward

The **CDF** built here — non‑decreasing, right‑continuous, rising from 0 to 1 — is the one object that survives intact into **Topic 7**. When X's values fill an interval, $P(X=x)$ collapses to 0 and the PMF can no longer carry the information; but the CDF stays meaningful, and its **derivative becomes the probability density function (PDF)**. Sums turn into integrals, and the "pile up over the preimage" habit turns into a change‑of‑variables / Jacobian rule. Everything else — support, normalization to 1, marginals, independence — carries over almost word‑for‑word.

---

## 9. One-line cheat-sheet summary

A discrete RV maps outcomes to a **countable** set of numbers; its **PMF** $p(x)=P(X=x)$ is non‑negative and sums to 1, its **CDF** $F(x)=P(X\le x)$ is a **right‑continuous step function** whose jump at $a$ equals $p(a)$ — and for $Y=g(X)$ you **add probabilities over every $x$ that maps to $y$**.

---

**Next topic (in order): Topic 7 — Continuous Random Variables, PDF & CDF.** It takes the CDF you just learned and, for value‑sets that fill an interval, replaces the PMF with a density obtained by differentiating $F$.

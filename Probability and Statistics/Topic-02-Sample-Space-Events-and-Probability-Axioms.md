# Topic 2 — Sample Space, Events and Probability Axioms

**Subject:** Probability and Statistics (GATE DA)
**Section:** Probability Foundations (Topic 2 of 16)

---

## 0. Topic header & syllabus map

- **Topic:** Random experiments, **sample space**, **events** and the set-algebra on them, and the **Kolmogorov axioms** of probability — together with every theorem that follows directly from those axioms (complement rule, monotonicity, the addition/inclusion–exclusion rule, union bound, classical and geometric probability).
- **Where it sits:** Comes **immediately after Counting**. Counting told us how to find the *size* of a set of outcomes; this topic builds the formal container (sample space + events) those outcomes live in and lays down the rules for assigning **probabilities** to them. It leads directly into **Independent vs Mutually Exclusive Events**, which is just two special relationships between events expressed in the language built here.

---

## 1. Why this topic exists (motivation + connection)

- **The real problem it solves:** Counting alone can't tell you *how likely* something is — it only gives raw tallies, and only when every outcome is equally likely. We need a framework that (a) precisely says what the possible outcomes are, (b) lets us talk about combinations of outcomes ("at least one six," "an even number *or* a prime"), and (c) assigns consistent numbers — probabilities — even when outcomes are **not** equally likely (a biased coin) or when the space is **continuous** (a random point on a line).
- **What the previous topic couldn't handle:** Counting handles only finite, equally-likely setups and produces *counts*, not probabilities. It has no vocabulary for "the event does not occur," "both events occur," or "the experiment has uncountably many outcomes," and no rules guaranteeing the numbers you assign behave sensibly (stay in $[0,1]$, add up correctly over disjoint cases). Those gaps are exactly what the sample-space model and the axioms fill.
- **What the next topic builds on it:** Two events being **mutually exclusive** ($A\cap B=\varnothing$) is a fact about the *event algebra* introduced here; two events being **independent** ($P(A\cap B)=P(A)P(B)$) is a fact about the *probability* introduced here. The addition rule we derive below is precisely what makes the difference between these two ideas visible — so Topic 3 is a direct sequel.

---

## 2. Prerequisites

- **Counting (Topic 1):** to size finite sample spaces and events for classical probability.
- **Set operations:** union $\cup$, intersection $\cap$, complement $^c$, difference $\setminus$, subset $\subseteq$, and **De Morgan's laws**.
- **Comfort with intervals/areas** (for the continuous, geometric-probability part).

---

## 3. Core idea — intuition first

Think of a **random experiment** (toss a coin, roll dice, pick a card, measure a bulb's lifetime). Before it runs, you don't know the result, but you *can* list all results that are possible. That complete list is the **sample space** $S$. Any question you might ask ("did we get an even number?") corresponds to a *subset* of $S$ — and we call such a subset an **event**. The event "happens" exactly when the true outcome lands inside that subset.

Probability is then a number we attach to each event measuring how likely it is. Intuitively it's the **long-run fraction** of trials in which the event occurs: toss a fair coin many times and heads shows up about half the time, so $P(\text{heads})=\tfrac12$. Three common-sense facts about such fractions become the *axioms*:

1. A fraction is never negative → **$P(A)\ge 0$**.
2. *Something* always happens → **$P(S)=1$**.
3. For outcomes that can't co-occur, fractions just **add** → if $A$ and $B$ are disjoint, $P(A\cup B)=P(A)+P(B)$.

Everything else in the topic — the complement rule, the addition rule, bounds — is squeezed out of these three with set algebra. The beauty is that once the axioms hold, you can compute consistently whether outcomes are equally likely or not, finite or infinite.

---

## 4. Formal treatment

### 4.1 The building blocks

- **Random experiment:** a process whose outcome is uncertain but whose set of possible outcomes is known.
- **Sample space $S$ (or $\Omega$):** the set of *all* possible outcomes. Three kinds:
  - **Finite discrete:** $\{H,T\}$; $\{1,2,3,4,5,6\}$.
  - **Countably infinite discrete:** number of tosses until the first head $=\{1,2,3,\dots\}$.
  - **Continuous (uncountable):** bulb lifetime $\in[0,\infty)$; a random point in $[0,1]$.
- **Outcome / sample point:** an element $\omega\in S$.
- **Event:** any subset $A\subseteq S$. It *occurs* if the realised outcome $\omega\in A$.
  - **Elementary (simple) event:** a singleton $\{\omega\}$.
  - **Sure (certain) event:** $S$ (probability 1).
  - **Impossible event:** $\varnothing$ (probability 0).

> *Technical aside (not tested directly in GATE DA):* the events on which $P$ is defined must form a **$\sigma$-field** — a collection closed under complement and countable unions. For finite or countable $S$ you may safely take *every* subset as an event, so you can ignore this subtlety in GATE problems.

### 4.2 Event algebra (set operations, read as logic)

| Set expression | Meaning ("…occurs") |
|---|---|
| $A\cup B$ | $A$ **or** $B$ (or both) |
| $A\cap B$ (also $AB$) | **both** $A$ and $B$ |
| $A^c$ (also $A',\bar A$) $= S\setminus A$ | $A$ does **not** occur |
| $A\setminus B = A\cap B^c$ | $A$ occurs but $B$ does **not** |
| $A\subseteq B$ | $A$ **implies** $B$ |
| $A\cap B=\varnothing$ | $A,B$ **mutually exclusive** (disjoint) |
| $A_1\cup A_2\cup\cdots=S$ | events are **exhaustive** |
| mutually exclusive **and** exhaustive | a **partition** of $S$ |

Useful compound events (memorise the translations — GATE tests them):
- **At least one of $A,B$:** $A\cup B$.
- **None of $A,B$:** $(A\cup B)^c = A^c\cap B^c$  *(De Morgan)*.
- **Exactly one of $A,B$:** $(A\cap B^c)\cup(A^c\cap B) = (A\cup B)\setminus(A\cap B)$.
- **At most one of $A,B$:** $(A\cap B)^c$.

**De Morgan's laws:**
$$(A\cup B)^c=A^c\cap B^c,\qquad (A\cap B)^c=A^c\cup B^c,$$
and generally $\big(\bigcup_i A_i\big)^c=\bigcap_i A_i^c$, $\big(\bigcap_i A_i\big)^c=\bigcup_i A_i^c$.

### 4.3 Kolmogorov's axioms

A **probability measure** $P$ assigns to each event a real number satisfying:

$$
\textbf{(A1) Non-negativity: } P(A)\ge 0.\qquad
\textbf{(A2) Normalization: } P(S)=1.
$$
$$
\textbf{(A3) Countable additivity: } \text{if } A_1,A_2,\dots \text{ are pairwise disjoint, then } P\!\Big(\bigcup_{i} A_i\Big)=\sum_{i} P(A_i).
$$

*Condition on A3:* the events must be **pairwise mutually exclusive** ($A_i\cap A_j=\varnothing$ for $i\ne j$). For finitely many events this is **finite additivity**, a consequence of A3.

### 4.4 Theorems that fall out of the axioms

Each holds for **any** valid probability — no extra assumptions unless stated.

| Result | Statement | One-line proof idea |
|---|---|---|
| Probability of $\varnothing$ | $P(\varnothing)=0$ | $S=S\cup\varnothing$ disjoint $\Rightarrow P(\varnothing)=0$ |
| **Complement rule** | $P(A^c)=1-P(A)$ | $A\cup A^c=S$ disjoint $\Rightarrow P(A)+P(A^c)=1$ |
| Upper bound | $0\le P(A)\le 1$ | $P(A^c)=1-P(A)\ge 0$ |
| **Monotonicity** | $A\subseteq B\Rightarrow P(A)\le P(B)$ | $B=A\cup(B\setminus A)$ disjoint |
| Difference | $P(A\setminus B)=P(A)-P(A\cap B)$ | $A=(A\cap B)\cup(A\cap B^c)$ disjoint |
| **Addition rule (2 events)** | $P(A\cup B)=P(A)+P(B)-P(A\cap B)$ | split $A\cup B$ into disjoint pieces |
| **Inclusion–exclusion (3)** | see below | repeat the addition rule |
| **Boole / union bound** | $P\!\left(\bigcup_i A_i\right)\le\sum_i P(A_i)$ | overlaps are only subtracted, never added |
| **Bonferroni lower bound** | $P(A\cap B)\ge P(A)+P(B)-1$ | rearrange $P(A\cup B)\le 1$ |

**Addition rule — the workhorse (always true):**
$$\boxed{\,P(A\cup B)=P(A)+P(B)-P(A\cap B)\,}$$
You subtract $P(A\cap B)$ because outcomes in the overlap got counted twice. If — and only if — $A,B$ are **mutually exclusive**, $P(A\cap B)=0$ and it simplifies to $P(A\cup B)=P(A)+P(B)$.

**Three-event inclusion–exclusion:**
$$P(A\cup B\cup C)=P(A)+P(B)+P(C)-P(A\cap B)-P(A\cap C)-P(B\cap C)+P(A\cap B\cap C).$$

### 4.5 Assigning probabilities

**(i) Classical (equally likely) model.** If $S$ is finite with $N$ outcomes, all equally likely, and event $A$ contains $m$ of them:
$$\boxed{\,P(A)=\dfrac{|A|}{|S|}=\dfrac{m}{N}\,}\qquad\text{(this is where Topic 1's counting feeds in).}$$
*Condition:* outcomes must genuinely be equally likely — never assume this; it must be implied (fair die, well-shuffled deck, random selection).

**(ii) General discrete model.** Assign each outcome $\omega_i$ a weight $p(\omega_i)\ge0$ with $\sum_i p(\omega_i)=1$; then $P(A)=\sum_{\omega\in A}p(\omega)$. This handles **biased** experiments where outcomes are not equally likely.

**(iii) Geometric (continuous) probability.** For a uniform random point in a region, probability is proportional to **measure** (length / area / volume):
$$\boxed{\,P(A)=\dfrac{\text{measure}(A)}{\text{measure}(S)}\,}\qquad\text{(uniform case).}$$
*Consequences:* a single point has probability $0$ even though it's a possible outcome; an event can have probability $0$ without being impossible, and probability $1$ without being the whole sample space.

*Derivations skipped:* the axioms are taken as given (GATE never asks you to construct a $\sigma$-field or prove countable additivity). The short proofs in §4.4 *are* GATE-relevant because "which statement is always true" MSQs lean on exactly this reasoning.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Classical probability via counting + complement (NAT)
*Two fair dice are rolled. Find (a) $P(\text{sum}=7)$, (b) $P(\text{sum}\ge 10)$, (c) $P(\text{at least one six})$.*

$|S|=6\times6=36$ equally likely outcomes.

(a) Sum $7$: $(1,6),(2,5),(3,4),(4,3),(5,2),(6,1)\Rightarrow 6$ outcomes $\Rightarrow \dfrac{6}{36}=\dfrac16$.

(b) Sum $10,11,12$: $(4,6),(5,5),(6,4)$ + $(5,6),(6,5)$ + $(6,6) = 3+2+1=6\Rightarrow\dfrac{6}{36}=\dfrac16$.

(c) Complement is "no six": $5\times5=25$ outcomes, so
$$P(\text{at least one six})=1-\frac{25}{36}=\boxed{\frac{11}{36}}.$$
*(Using the complement beats summing "exactly one six" + "two sixes.")*

---

### Problem 2 — The event-algebra computations (NAT / MSQ)
*Given $P(A)=0.5,\ P(B)=0.4,\ P(A\cap B)=0.2$, find: (a) $P(A\cup B)$, (b) $P(\text{exactly one of }A,B)$, (c) $P(\text{neither})$, (d) $P(A\text{ but not }B)$.*

(a) $P(A\cup B)=0.5+0.4-0.2=\boxed{0.7}$.

(b) Exactly one $=P(A\cup B)-P(A\cap B)=0.7-0.2=\boxed{0.5}$  *(equivalently $P(A)+P(B)-2P(A\cap B)$).*

(c) Neither $=(A\cup B)^c$: $1-0.7=\boxed{0.3}$.

(d) $A\setminus B$: $P(A)-P(A\cap B)=0.5-0.2=\boxed{0.3}$.

---

### Problem 3 — Bounding an unknown intersection (MSQ / conceptual trap)
*Given $P(A)=0.6,\ P(B)=0.7$, with **no** independence assumed. What are the tightest possible bounds on $P(A\cap B)$?*

- **Upper:** $A\cap B\subseteq A$ and $\subseteq B\Rightarrow P(A\cap B)\le\min(0.6,0.7)=0.6$.
- **Lower (Bonferroni):** since $P(A\cup B)\le1$,
$$P(A\cap B)=P(A)+P(B)-P(A\cup B)\ge 0.6+0.7-1=0.3.$$

$$\boxed{0.3\le P(A\cap B)\le 0.6.}$$
**Trap defused:** with only $P(A)$ and $P(B)$ you **cannot** pin down $P(A\cap B)$. Multiplying $0.6\times0.7=0.42$ would silently assume *independence*, which was never given (that's Topic 3's condition, not a default).

---

### Problem 4 — Three-event inclusion–exclusion (NAT)
*$P(A)=0.5,\ P(B)=0.4,\ P(C)=0.3,\ P(A\cap B)=0.2,\ P(B\cap C)=0.15,\ P(A\cap C)=0.1,\ P(A\cap B\cap C)=0.05.$ Find $P$ (at least one) and $P$ (none).*

$$P(A\cup B\cup C)=0.5+0.4+0.3-0.2-0.15-0.1+0.05=\boxed{0.80}.$$
$$P(\text{none})=(A\cup B\cup C)^c=1-0.80=\boxed{0.20}.$$

---

### Problem 5 — Geometric probability: the "meeting" problem (NAT)
*Two friends each arrive at a café at an independent, uniformly random time between 5:00 and 6:00 pm, and each waits 15 minutes. What is the probability that they meet?*

Let $x,y\in[0,60]$ minutes be the arrival times; $S$ is the $60\times60$ square (area $3600$). They meet iff $|x-y|\le 15$.

The region where they **miss** is $|x-y|>15$: two right triangles, each with legs $60-15=45$, so each has area $\tfrac12(45)(45)=1012.5$; together $2025$.
$$\text{meeting area}=3600-2025=1575\ \Rightarrow\ P(\text{meet})=\frac{1575}{3600}=\boxed{\frac{7}{16}=0.4375}.$$
*(This is the canonical continuous-sample-space question — answer by comparing areas, not by counting.)*

---

### Problem 6 — "Which statements are always true?" (MSQ, axiom check)
*For arbitrary events $A,B$, which are always correct?*
1. $P(A\cup B)=P(A)+P(B)$  
2. $P(A^c\cap B^c)=1-P(A\cup B)$  
3. $A\subseteq B\Rightarrow P(A)\le P(B)$  
4. $P(A\cap B)=P(A)\,P(B)$

- (1) **False** in general — only if $A,B$ disjoint.
- (2) **True** — it's De Morgan + complement: $A^c\cap B^c=(A\cup B)^c$.
- (3) **True** — monotonicity.
- (4) **False** in general — only under independence (Topic 3).

**Answer: 2 and 3.**

---

## 6. Common GATE question types on this topic

- **Classical probability from a described experiment** (dice, cards, balls from an urn, random digits) — *counting in disguise*; uses Topic 1 directly.
- **Event-algebra computations:** given a few probabilities, find $P(A\cup B)$, $P(A\cap B)$, $P(\text{exactly one})$, $P(\text{neither})$, $P(\text{at least one})$.
- **Bounding probabilities** when data is insufficient (Boole's inequality for unions, Bonferroni $P(A\cap B)\ge P(A)+P(B)-1$).
- **Consistency / "which is always true" MSQs** — test whether a set of given probabilities can coexist and which axiomatic consequences hold.
- **Translating English to set events** ("at least two of three," "none," "exactly one") and computing.
- **Geometric probability** for uniform continuous spaces (meeting times, random points, dartboards, breaking a stick).
- **Complementary counting** ("probability of *at least one*…" $=1-P(\text{none})$).

---

## 7. Traps, pitfalls & common mistakes

- **Dropping the overlap term.** $P(A\cup B)=P(A)+P(B)$ is wrong unless $A,B$ are mutually exclusive — always subtract $P(A\cap B)$.
- **Mistaking "mutually exclusive" for "independent."** They're different *and* (for events of positive probability) incompatible: disjoint events with $P>0$ are *dependent*, since $A$ occurring forces $B$ not to. (Full treatment in Topic 3 — don't conflate them.)
- **Assuming equal likelihood.** Only use $|A|/|S|$ when outcomes are genuinely equally likely; a biased coin or a non-uniform selection breaks it.
- **Assuming independence to get $P(A\cap B)=P(A)P(B)$** when it wasn't stated — a frequent silent error; without it you can only *bound* the intersection.
- **Continuous-space confusions:** a single point has probability $0$ yet is possible; $P(A)=0$ does **not** mean $A=\varnothing$, and $P(A)=1$ does **not** mean $A=S$.
- **Additivity needs disjointness.** Axiom A3 applies only to pairwise mutually exclusive events; you can't just add probabilities of overlapping events.
- **De Morgan slips** when translating "neither/none": $(A\cup B)^c=A^c\cap B^c$, not $A^c\cup B^c$.
- **Inconsistent given data.** If a problem hands you, say, $P(A\cup B)<\max(P(A),P(B))$ or any probability outside $[0,1]$, the data is impossible — flag it rather than computing.
- **Forgetting "at least one" ⇒ complement.** Summing many overlapping sub-cases invites double counting; $1-P(\text{none})$ is cleaner and trap-free.

---

## 8. Connection forward

We now have a sample space, an event algebra, and axioms that let us compute and combine probabilities — including the all-purpose addition rule $P(A\cup B)=P(A)+P(B)-P(A\cap B)$. The **next topic, Independent vs Mutually Exclusive Events**, drills into two special relationships hiding inside that rule: *mutually exclusive* ($A\cap B=\varnothing$, so the overlap term vanishes) versus *independent* ($P(A\cap B)=P(A)P(B)$, a multiplicative relationship). Distinguishing them — and seeing why disjoint $\ne$ independent — is the immediate sequel, and it sets up conditional and joint probability right after.

---

## 9. One-line cheat-sheet summary

**Sample space $S$ = all outcomes; events = subsets; Kolmogorov axioms ($P\ge0$, $P(S)=1$, disjoint events add) give $P(A^c)=1-P(A)$, monotonicity, and the master rule $P(A\cup B)=P(A)+P(B)-P(A\cap B)$ — with $P(A)=|A|/|S|$ when equally likely and $P(A)=\text{measure}(A)/\text{measure}(S)$ when uniform-continuous.**

---

### ➡️ Next topic (in order): **Topic 3 — Independent vs Mutually Exclusive Events.**
*Ask for it when you're ready, and I'll keep the same structure.*

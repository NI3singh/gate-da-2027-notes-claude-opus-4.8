# Topic 3 — Independent vs Mutually Exclusive Events

**Subject:** Probability and Statistics (GATE DA)
**Section:** Probability Foundations (Topic 3 of 16)

---

## 0. Topic header & syllabus map

- **Topic:** Two special relationships between events — **mutually exclusive** (disjoint, a *set* property) and **independent** (a *probability* property) — the multiplication rule for independent events, independence of complements, the difference between **pairwise** and **mutual** independence, and why disjoint $\ne$ independent.
- **Where it sits:** Comes **right after Sample Space, Events & Axioms**, where we met the addition rule $P(A\cup B)=P(A)+P(B)-P(A\cap B)$ and the definition of mutually exclusive. This topic is the natural next step because that addition rule has *two* special cases hiding in it, and it gives us our **first tool to actually compute an intersection** $P(A\cap B)$. It leads straight into **Marginal, Conditional & Joint Probability**, where independence becomes the special case "conditioning changes nothing."

---

## 1. Why this topic exists (motivation + connection)

- **The real problem it solves:** We constantly need $P(A\cap B)$ — "both happen." The previous topic could only *bound* it ($P(A)+P(B)-1\le P(A\cap B)\le\min(P(A),P(B))$). **Independence** is the first situation where we can compute it *exactly*: $P(A\cap B)=P(A)P(B)$. Separately, **mutual exclusivity** is the situation where the intersection is *empty*, which simplifies the addition rule. The topic exists to nail these two extremes precisely — and to stop students from confusing them.
- **What the previous topic couldn't handle:** Topic 2 defined "mutually exclusive" ($A\cap B=\varnothing$) but gave **no general method** for the overlap $P(A\cap B)$ — only inequalities. So joint events stayed un-computable. This topic supplies the multiplicative shortcut (under independence) and clarifies what disjointness *does* and *does not* give you.
- **What the next topic builds on it:** Independence is exactly the case where $P(A\mid B)=P(A)$ — knowing $B$ tells you nothing about $A$. The general (dependent) case needs **conditional probability**, and the general formula for an intersection is the **multiplication rule** $P(A\cap B)=P(A)\,P(B\mid A)$, of which independence is the special case $P(B\mid A)=P(B)$. So Topic 4 generalises exactly what we set up here.

---

## 2. Prerequisites

- **Event algebra & axioms (Topic 2):** $\cup,\cap,{}^c$, mutually exclusive, the addition rule, complement rule $P(A^c)=1-P(A)$.
- **Counting (Topic 1):** to compute the underlying probabilities in concrete problems.
- The bounds on $P(A\cap B)$ from Topic 2 (to appreciate what independence *adds*).

---

## 3. Core idea — intuition first

These two phrases sound similar in English but mean opposite things mathematically — keep them apart from day one.

**Mutually exclusive ("can't both happen").** A coin lands Heads *or* Tails, never both — so {Heads} and {Tails} are mutually exclusive. This is a statement about the *events themselves*: their intersection is empty. No probabilities are needed to check it. Its effect: in $P(A\cup B)$, the overlap term is zero.

**Independent ("knowing one tells you nothing about the other").** Roll two separate dice: whether the first shows a 6 has no bearing on the second. This is a statement about the *probability measure*: the chance of both equals the product of the chances. Its effect: $P(A\cap B)=P(A)P(B)$.

Here's the punchline that GATE loves to test: **disjoint events are about as far from independent as you can get.** If $A$ and $B$ can't both happen and both have positive probability, then learning that $A$ occurred tells you a *lot* about $B$ — namely, $B$ definitely did *not* occur. That's strong dependence, not independence. So "mutually exclusive" and "independent" are not just different; for ordinary (positive-probability) events they are **incompatible**.

Mental model:
- *Mutually exclusive* = two non-overlapping regions on a map.
- *Independent* = two regions whose overlap is exactly the "expected" product of their sizes — they **must** overlap (if both are nonempty in probability).

---

## 4. Formal treatment

### 4.1 Definitions

$$\textbf{Mutually exclusive (disjoint): } A\cap B=\varnothing \ \Rightarrow\ P(A\cap B)=0.$$
$$\boxed{\textbf{Independent: } P(A\cap B)=P(A)\,P(B).}$$

**Conditions/notes:**
- Mutual exclusivity is a **structural** (set-theoretic) condition; it does not depend on $P$.
- Independence is a **numerical** condition; it depends entirely on $P$. The *same* two events can be independent under one probability assignment and dependent under another.
- Independence is **symmetric**: $A\perp B \iff B\perp A$.
- $A\cap B=\varnothing$ forces $P(A\cap B)=0$, but in a *continuous* space $P(A\cap B)=0$ need not mean $A\cap B=\varnothing$. For GATE, treat "mutually exclusive" as $A\cap B=\varnothing$.

### 4.2 Effect on the addition rule (the two simplifications)

Starting from the always-true $P(A\cup B)=P(A)+P(B)-P(A\cap B)$:
- **If mutually exclusive:** $P(A\cap B)=0\Rightarrow \boxed{P(A\cup B)=P(A)+P(B)}$.
- **If independent:** $P(A\cap B)=P(A)P(B)\Rightarrow \boxed{P(A\cup B)=P(A)+P(B)-P(A)P(B)}$.

> Don't swap these. *Mutually exclusive* simplifies the **union**; *independent* gives you the **intersection**.

### 4.3 The incompatibility theorem (must know)

**Claim.** If $A,B$ are mutually exclusive with $P(A)>0$ and $P(B)>0$, then $A$ and $B$ are **dependent**.
**Proof.** Mutual exclusivity gives $P(A\cap B)=0$, but independence would require $P(A\cap B)=P(A)P(B)>0$. Contradiction. $\square$

**Corollary.** Two independent events with positive probability **must intersect** ($P(A\cap B)=P(A)P(B)>0$, so $A\cap B\ne\varnothing$).
**When can an event pair be *both* disjoint and independent?** Only if at least one of them has probability $0$ (e.g. $A$ with $P(A)=0$, or $B=\varnothing$).

### 4.4 Independence of complements and derived events

If $A\perp B$, then $A\perp B^c$, $A^c\perp B$, and $A^c\perp B^c$.
**Proof of one case:**
$$P(A\cap B^c)=P(A)-P(A\cap B)=P(A)-P(A)P(B)=P(A)\big(1-P(B)\big)=P(A)\,P(B^c).\ \square$$
*Consequence (used everywhere):* for independent events,
$$P(\text{none of }A_1,\dots,A_n)=\prod_{i}P(A_i^{\,c}),\qquad P(\text{at least one})=1-\prod_i P(A_i^{\,c}).$$

**Self-independence:** $A\perp A \iff P(A)=P(A)^2 \iff P(A)\in\{0,1\}$. (Only sure/impossible-type events are independent of themselves.)

**$S$ and $\varnothing$:** the sure event $S$ and the impossible event $\varnothing$ are independent of *every* event.

### 4.5 Multiplication rule for independent events

If $A_1,\dots,A_n$ are **mutually independent**,
$$\boxed{\,P\!\Big(\bigcap_{i=1}^n A_i\Big)=\prod_{i=1}^n P(A_i)\,.}$$
This is the engine for **repeated independent trials** (coin tosses, dice rolls, with-replacement draws) and for **reliability** of independent components.

### 4.6 Pairwise vs mutual independence (a classic GATE trap)

For $n\ge 3$ events, two different strengths exist:

- **Pairwise independent:** every *pair* satisfies the product rule: $P(A_i\cap A_j)=P(A_i)P(A_j)$ for all $i\ne j$.
- **Mutually (jointly) independent:** **every** sub-collection satisfies it — all pairs, all triples, …, up to the full set. For three events $A,B,C$ this means *all four* hold:
$$P(A\cap B)=P(A)P(B),\ P(A\cap C)=P(A)P(C),\ P(B\cap C)=P(B)P(C),\ \text{and } P(A\cap B\cap C)=P(A)P(B)P(C).$$

**Pairwise independence does NOT imply mutual independence.** (Counterexample worked in Problem 4.) Mutual independence is the condition needed for the full multiplication rule.

### 4.7 Connection to conditional probability (preview)

When $P(B)>0$, independence is equivalent to
$$A\perp B \iff P(A\mid B)=P(A)\iff P(B\mid A)=P(B).$$
This is the cleanest *intuition* ("$B$ gives no information about $A$") and is the bridge to the next topic. The formal definition of $P(A\mid B)$ belongs to Topic 4; here just note the equivalence.

*Derivations skipped:* none essential are omitted. The short proofs in §4.3–§4.4 *are* GATE-relevant (they power the conceptual MSQs and the "at least one" computations).

---

## 5. Worked problems (GATE-style)

### Problem 1 — The head-to-head contrast (NAT / conceptual)
*Let $P(A)=0.3,\ P(B)=0.5.$ Compute $P(A\cap B)$ and $P(A\cup B)$ (a) if $A,B$ are mutually exclusive, (b) if $A,B$ are independent. (c) Can both hold simultaneously here?*

**(a) Mutually exclusive:** $P(A\cap B)=0$, so $P(A\cup B)=0.3+0.5=\boxed{0.8}$.

**(b) Independent:** $P(A\cap B)=0.3\times0.5=\boxed{0.15}$, so
$$P(A\cup B)=0.3+0.5-0.15=\boxed{0.65}.$$

**(c)** No. Both positive probabilities, so mutual exclusivity needs $P(A\cap B)=0$ while independence needs $P(A\cap B)=0.15$ — they can't both be true. (Possible only if some event had probability $0$.)

*Takeaway:* the two assumptions give **different** answers from the **same** $P(A),P(B)$ — proof they are distinct concepts.

---

### Problem 2 — Independent events: exactly one / neither / at least one (NAT)
*$A,B$ are independent with $P(A)=0.4,\ P(B)=0.7$. Find $P(\text{at least one})$, $P(\text{neither})$, $P(\text{exactly one})$.*

Use independence of complements.
- $P(\text{neither})=P(A^c)P(B^c)=(0.6)(0.3)=\boxed{0.18}$.
- $P(\text{at least one})=1-0.18=\boxed{0.82}$.
- $P(\text{exactly one})=P(A)P(B^c)+P(A^c)P(B)=(0.4)(0.3)+(0.6)(0.7)=0.12+0.42=\boxed{0.54}$.

*Check:* exactly one $+$ both $+$ neither $=0.54+0.28+0.18=1.00$. ✓

---

### Problem 3 — System reliability: series vs parallel (NAT)
*Three components work independently, each with reliability $0.9$. Find the system reliability if the components are connected (a) in series (all must work) and (b) in parallel (at least one must work).*

**(a) Series** — needs the intersection of "all work":
$$P=\,0.9\times0.9\times0.9=0.9^3=\boxed{0.729}.$$

**(b) Parallel** — fails only if all fail; use the complement with independence:
$$P=1-(0.1)^3=1-0.001=\boxed{0.999}.$$

*Lesson:* parallel redundancy uses $1-\prod P(\text{fail}_i)$, **not** $1-\prod P(\text{work}_i)$ — a common slip.

---

### Problem 4 — Pairwise but NOT mutually independent (MSQ / trap)
*Toss two fair coins. Let $A=$ "first coin is H," $B=$ "second coin is H," $C=$ "both coins show the same face." Are $A,B,C$ pairwise independent? Mutually independent?*

Sample space $\{HH,HT,TH,TT\}$, each with probability $\tfrac14$.
$$P(A)=P(B)=P(C)=\tfrac12.$$
**Pairwise:**
$$P(A\cap B)=P(HH)=\tfrac14=P(A)P(B),\quad P(A\cap C)=P(HH)=\tfrac14=P(A)P(C),\quad P(B\cap C)=P(HH)=\tfrac14=P(B)P(C).$$
So **pairwise independent. ✓**
**Triple:**
$$P(A\cap B\cap C)=P(HH)=\tfrac14,\qquad P(A)P(B)P(C)=\tfrac18.$$
Since $\tfrac14\ne\tfrac18$, they are **NOT mutually independent.** ✗

*Trap defused:* checking all pairs is **not** enough for three or more events — the triple (and higher) conditions can still fail.

---

### Problem 5 — "Which statements are correct?" (MSQ, conceptual)
*$A,B$ are events with $P(A),P(B)\in(0,1)$. Which are always true?*
1. If $A,B$ are mutually exclusive, they are independent.  
2. If $A,B$ are independent, $A^c$ and $B^c$ are independent.  
3. If $A,B$ are mutually exclusive, $P(A\cup B)=P(A)+P(B)$.  
4. If $A,B$ are independent, $P(A\cup B)=P(A)+P(B)$.

- (1) **False** — disjoint positive-probability events are *dependent* (§4.3).
- (2) **True** — independence of complements (§4.4).
- (3) **True** — overlap term vanishes.
- (4) **False** — independence simplifies the *intersection*, giving $P(A\cup B)=P(A)+P(B)-P(A)P(B)$.

**Answer: 2 and 3.**

---

## 6. Common GATE question types on this topic

- **Distinguish the two notions** ("which statement is always true," "are these events independent / mutually exclusive").
- **Compute $P(A\cap B)$ and $P(A\cup B)$** under (i) mutual exclusivity, (ii) independence — often the *same* $P(A),P(B)$ to highlight the contrast.
- **"At least one" of independent events** via the complement $1-\prod P(A_i^c)$.
- **System reliability:** series $=\prod p_i$; parallel $=1-\prod(1-p_i)$.
- **Repeated independent trials:** probability of a specific sequence of outcomes, or of $k$ successes happening in fixed positions (full distribution comes later in Binomial).
- **Pairwise vs mutual independence:** identify or build a counterexample.
- **With- vs without-replacement** framing: with replacement $\Rightarrow$ independent draws; without $\Rightarrow$ dependent (sets up conditional probability).
- **Independence of derived events** ($A^c, B^c, A\cap B$, etc.).

---

## 7. Traps, pitfalls & common mistakes

- **Confusing disjoint with independent — the #1 error.** Mutually exclusive (positive-prob) events are *maximally dependent*, never independent.
- **Swapping the two simplifications.** Mutual exclusivity ⇒ $P(A\cup B)=P(A)+P(B)$; independence ⇒ $P(A\cap B)=P(A)P(B)$. Using the wrong one is an instant wrong answer.
- **"At least one" with complements done wrong.** $P(\text{none})=\prod P(A_i^c)$, so $P(\text{at least one})=1-\prod P(A_i^c)$ — **not** $1-\prod P(A_i)$.
- **Assuming independence without justification.** Only use the product rule when independence is stated or physically clear (separate fair coins, with-replacement draws). Otherwise you can only bound the intersection.
- **Pairwise ≠ mutual.** For $\ge3$ events you must verify *every* subset's product condition, including the all-events one.
- **Parallel-reliability slip:** failure side multiplies, so use $1-\prod(1-p_i)$.
- **Without-replacement = dependent.** Draws without replacement change the remaining composition, so they are *not* independent (a frequent trap in urn problems).
- **Forgetting that a probability-0 (or 1) event is independent of everything**, and is the *only* way an event can be both disjoint from and independent of another.
- **Independence is relative to $P$:** the same events can flip between independent and dependent if the probability assignment changes — don't treat it as a fixed physical property.

---

## 8. Connection forward

Independence is the clean special case where extra information is useless: $P(A\mid B)=P(A)$. Real problems are usually **not** that clean — knowing one event *does* change the odds of another. The **next topic, Marginal, Conditional & Joint Probability**, builds the machinery for that general case: the conditional probability $P(A\mid B)=\dfrac{P(A\cap B)}{P(B)}$, the general multiplication rule $P(A\cap B)=P(A)\,P(B\mid A)$ (with independence as the special case $P(B\mid A)=P(B)$), and how to recover a single-event "marginal" probability from a joint distribution. That topic, in turn, leads to Bayes' theorem. Independence will keep reappearing — and its generalisation, **conditional independence** ($P(A\cap B\mid C)=P(A\mid C)P(B\mid C)$), underpins later ML topics like naive Bayes.

---

## 9. One-line cheat-sheet summary

**Mutually exclusive = $A\cap B=\varnothing$ ⇒ $P(A\cup B)=P(A)+P(B)$ (a *set* fact); Independent = $P(A\cap B)=P(A)P(B)$ ⇔ $P(A\mid B)=P(A)$ (a *probability* fact) — and for positive-probability events the two are incompatible (disjoint ⇒ dependent); "at least one" $=1-\prod P(A_i^c)$, and pairwise independence does NOT imply mutual independence.**

---

### ➡️ Next topic (in order): **Topic 4 — Marginal, Conditional and Joint Probability.**
*Ask for it when you're ready, and I'll keep the same structure.*

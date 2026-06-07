# Topic 7 — Exact Inference: Variable Elimination

*Subject: Artificial Intelligence · GATE DA · Section: Reasoning Under Uncertainty*

---

## 0. Topic header & syllabus map

- **Topic:** Exact Inference in Bayesian networks via **Variable Elimination (VE)** — and its naive baseline, **inference by enumeration.**
- **Subject:** Artificial Intelligence.
- **Where it sits:** Second topic of "Reasoning Under Uncertainty." It comes **right after Bayesian Networks (Topic 6)** — that topic gave you the compact data structure (a DAG + CPTs encoding the joint as $\prod_i P(X_i\mid\mathrm{Pa}(X_i))$); **this topic gives the algorithm that answers queries** $P(\text{query}\mid\text{evidence})$ on it *exactly*. It **leads into Topic 8 (Approximate Inference — Sampling)**, which is needed precisely because exact inference becomes intractable on large/dense networks.

---

## 1. Why this topic exists (motivation + connection)

**The problem it solves.** Topic 6 let us *store* a joint distribution compactly and *read off* independencies. But the actual goal of a probabilistic agent is to **answer questions**: "Given that both neighbours called, what's the probability of a burglary?" i.e. compute a **posterior** $P(\text{Burglary}\mid \text{JohnCalls}, \text{MaryCalls})$. The network alone doesn't answer this — we need an inference procedure.

**What the previous topic couldn't do.** Building the network and reading d-separation tells you *which* variables matter, but not *the number* $P(B\mid j,m)$. To get that number you must combine the CPTs and sum out the hidden variables.

**Why not "just enumerate the joint"?** You *can* answer any query by summing over the full joint (inference by enumeration). It's correct but **exponential** — for $n$ Boolean variables it touches $\Theta(2^n)$ joint entries and, worse, **recomputes the same sub-sums over and over.** Variable Elimination is enumeration made smart: it applies the **distributive law** to push summations inward and **caches** intermediate results (dynamic programming), so each subexpression is computed once.

**What the next topic builds on it.** VE is *exact*, but its cost is exponential in the network's **treewidth**. Dense networks (large treewidth) defeat it. **Topic 8 (Sampling)** trades exactness for tractability when VE blows up. So VE is the gold-standard answer when affordable; sampling is the fallback when it isn't.

---

## 2. Prerequisites

- **Bayesian networks (Topic 6):** the factorization $P(X_{1..n})=\prod_i P(X_i\mid\mathrm{Pa}(X_i))$, CPTs, and **d-separation** (used to spot irrelevant variables).
- **Marginalization** ($\sum$ out a variable), **conditional probability**, **Bayes' rule**, **normalization constant $\alpha$**.
- **Distributive law** of arithmetic: $\sum_x (a\cdot b) = a\sum_x b$ when $a$ does not depend on $x$. (This single fact *is* VE.)
- DAG vocabulary: parents, ancestors, descendants, leaves; topological order.

---

## 3. Core idea — intuition first

Suppose you must compute, over Boolean $A,B,C$:

```
Σ_A Σ_B Σ_C  f(A) · g(A,B) · h(B,C)
```

**Naive way (enumeration):** expand all $2^3=8$ combinations, multiply three numbers each, add up. Worse, in a real query the same partial products reappear many times.

**Smart way (variable elimination):** notice $h(B,C)$ is the *only* term mentioning $C$. So push $\sum_C$ inward:

```
= Σ_A Σ_B  f(A)·g(A,B)·(Σ_C h(B,C))
= Σ_A Σ_B  f(A)·g(A,B)· h'(B)         ← computed Σ_C once, stored as h'(B)
```

Now $\sum_B$ touches only $g(A,B)\,h'(B)$:

```
= Σ_A f(A)·(Σ_B g(A,B)·h'(B))
= Σ_A f(A)· m(A)                       ← stored Σ_B as m(A)
```

We replaced one giant sum-of-products with a sequence of **small** sums, each producing a reusable table. That sequence — *"multiply everything that mentions the variable, then sum the variable out, repeat"* — is the **Variable Elimination algorithm.** The tables we pass around are called **factors.**

The two cost-shaping decisions: **which variables we can throw away entirely** (irrelevant ones), and **the order in which we eliminate the rest** (a bad order builds huge intermediate tables).

---

## 4. Formal treatment

### 4.1 The query, precisely

Partition the variables into three groups:

- **Query variables $Q$** — what we want a distribution over.
- **Evidence variables $E$** — observed, fixed to values $E=e$.
- **Hidden variables $Y$** — everything else (to be summed out).

We want the posterior:

```
P(Q | E = e) = α · P(Q, e) = α · Σ_y P(Q, E=e, Y=y)
```

```
where the summand P(Q, e, y) is the BN joint = Π_i P(Xi | Parents(Xi)),
      Y = y ranges over ALL assignments of the hidden variables,
      α = 1 / Σ_q P(q, e)   is the normalisation constant,  valid when P(e) > 0.
```

This formula is **inference by enumeration**. It's exact but its sum has $\prod_{Y_i}|Y_i|$ terms — exponential in the number of hidden variables.

---

### 4.2 Factors — the working objects

> A **factor** $f(X_1,\dots,X_k)$ is a function from each assignment of variables $X_1,\dots,X_k$ to a non-negative real number — basically a multi-dimensional table. Its variables are its **scope**.

- Every **CPT** $P(X_i\mid \mathrm{Pa}(X_i))$ is a factor over $\{X_i\}\cup\mathrm{Pa}(X_i)$.
- **A factor is NOT necessarily a probability distribution** — its entries need not sum to 1. (Only full CPT *rows* per parent-config do.) This is a top GATE trap.

Three operations are all you need:

**(a) Restriction / reduce by evidence.** Fix an evidence variable to its observed value and drop it from the scope.
```
Given E observed to value e:  f(A, E)  →  f_e(A) = f(A, E=e)   (keep only the E=e slice)
```

**(b) Pointwise product (factor multiplication).** Combine two factors; the result's scope is the **union** of scopes; multiply entries that **agree on shared variables**.
```
f(A,B) × g(B,C) = h(A,B,C),   where  h(a,b,c) = f(a,b) · g(b,c)
```
Cost/size: a product of factors with $m$ distinct variables (all Boolean) has $2^m$ entries — this is what blows up with a bad ordering.

**(c) Summing out (marginalising) a variable.** Remove a variable from a factor by adding over its values.
```
Σ_B f(A,B,C) = g(A,C),   where  g(a,c) = Σ_b f(a,b,c)
```

---

### 4.3 The Variable Elimination algorithm

> **Input:** BN factors (the CPTs), query $Q$, evidence $E=e$, an elimination ordering of the hidden variables $Y$.
>
> 1. **Make factors.** Start with one factor per CPT.
> 2. **Apply evidence.** Restrict every factor that mentions an evidence variable to the observed value (operation a).
> 3. **(Optional but important) Prune irrelevant variables** (§4.5).
> 4. **Eliminate each hidden variable $Z$, in the chosen order:**
>    - Collect **all** factors whose scope contains $Z$.
>    - **Multiply** them together (operation b) into one factor.
>    - **Sum out** $Z$ from that product (operation c).
>    - Replace the collected factors with this new factor.
> 5. **Finish.** Multiply all remaining factors → a factor over $Q$ (and possibly the constant). This is $P(Q,e)$.
> 6. **Normalise:** $P(Q\mid e) = \alpha\,P(Q,e)$ with $\alpha = 1/\sum_q P(q,e)$.

**Why it's correct.** It is exactly the enumeration sum $\sum_y\prod_i f_i$ with summations *pushed inward* by the distributive law. Multiplying *all* factors that contain $Z$ before summing $Z$ out is the algebraic move "factor out everything independent of $Z$." Doing it variable-by-variable just caches the partial results.

> **Iron rule (most common mistake):** before you sum out $Z$, you must have multiplied **every** remaining factor that still contains $Z$. Summing $Z$ out while it lingers in another factor gives a *wrong* answer.

---

### 4.4 Relationship to inference by enumeration (the bridge)

```
Enumeration:   one big sum-of-products,  Θ(2^n) terms, recomputes sub-sums
      │  apply distributive law + cache intermediate factors  (dynamic programming)
      ▼
Variable Elimination:  nested small sums, each factor computed once
```

VE never changes the *answer* — only the *amount of work*. Both are **exact**.

---

### 4.5 Irrelevant variables (a free speed-up — and a GATE favourite)

> **Pruning rule:** Any variable that is **not an ancestor of any query or evidence variable** is irrelevant to the query and can be deleted from the network before inference.
>
> Operationally: a **leaf** node that is **neither a query nor an evidence variable** is *barren* and can be removed; repeat until no such leaf remains.

**Why:** summing a barren leaf out of its CPT gives $\sum_{X} P(X\mid \mathrm{Pa}(X)) = 1$, contributing a harmless factor of $1$. So it never affects the result — computing with it is pure waste.

(Connection to Topic 6: this is d-separation in action — barren leaves are d-separated from the query given the evidence, hence carry no information.)

---

### 4.6 Elimination ordering & complexity

- **Order changes cost, not correctness.** The dominant cost is the size of the **largest factor created**: with all-Boolean variables, a factor over $w{+}1$ variables has $2^{w+1}$ entries.
- **Induced width $w$ of an ordering** = (size of the largest factor produced) − 1. **Treewidth** = the *minimum* induced width over all orderings.

> **Time and space of VE $\approx \Theta(n\cdot 2^{w})$** for $n$ Boolean variables and induced width $w$ of the chosen ordering — **exponential in the width, not in $n$.**

- **Finding the optimal ordering is NP-hard.** Practical **heuristics:**
  - **Min-degree:** next eliminate the variable with the fewest neighbours in the current (moral) interaction graph.
  - **Min-fill:** next eliminate the variable whose elimination adds the fewest new edges ("fill-in").
- **Polytrees (singly connected networks** — at most one undirected path between any two nodes**):** there is an ordering giving VE **linear** time and space in the network size (with bounded number of parents per node). Multiply-connected networks can have large treewidth and so be intractable for VE — *this is exactly why Topic 8 exists.*

**Rule of thumb for picking an order in exam problems:** eliminate **leaves first / high-degree hub nodes last**, to keep intermediate factors small.

---

## 5. Worked problems (GATE-style)

Problems 1, 3 reuse the **Alarm network** from Topic 6 (all Boolean): $B\to A$, $E\to A$, $A\to J$, $A\to M$.

```
P(B=t)=0.001   P(E=t)=0.002
P(A=t|B,E):  (t,t)=0.95  (t,f)=0.94  (f,t)=0.29  (f,f)=0.001
P(J=t|A=t)=0.90   P(J=t|A=f)=0.05
P(M=t|A=t)=0.70   P(M=t|A=f)=0.01
```

---

### Problem 1 (NAT) — full VE computation of a posterior

**Q.** Compute $P(\text{Burglary}\mid \text{JohnCalls}=t,\ \text{MaryCalls}=t)$, i.e. $P(B\mid j,m)$.

**Setup.** Query $Q=B$; evidence $J=t,M=t$; hidden $Y=\{E,A\}$. We need
$P(B,j,m)=\sum_E\sum_A P(B)\,P(E)\,P(A\mid B,E)\,P(j\mid A)\,P(m\mid A).$

**Factors after applying evidence** ($J=t,M=t$ fix the call CPTs to single-column factors over $A$):
```
f_B(B)=P(B)        f_E(E)=P(E)        f_A(A,B,E)=P(A|B,E)
f_J(A)=P(j|A):  A=t→0.90, A=f→0.05
f_M(A)=P(m|A):  A=t→0.70, A=f→0.01
```

**Step 1 — eliminate $A$.** Factors containing $A$: $f_A, f_J, f_M$. Multiply and sum out $A$:
```
τ_A(B,E) = Σ_A  P(A|B,E)·f_J(A)·f_M(A)
         = P(A=t|B,E)·(0.90·0.70) + P(A=f|B,E)·(0.05·0.01)
         = P(A=t|B,E)·0.63 + (1−P(A=t|B,E))·0.0005
```
Evaluate for each $(B,E)$:
```
(B=t,E=t): 0.95·0.63 + 0.05·0.0005 = 0.598525
(B=t,E=f): 0.94·0.63 + 0.06·0.0005 = 0.592230
(B=f,E=t): 0.29·0.63 + 0.71·0.0005 = 0.183055
(B=f,E=f): 0.001·0.63 + 0.999·0.0005 = 0.0011295
```

**Step 2 — eliminate $E$.** Factors containing $E$: $f_E, τ_A$. Multiply and sum out $E$:
```
τ_E(B) = Σ_E P(E)·τ_A(B,E) = 0.002·τ_A(B,E=t) + 0.998·τ_A(B,E=f)

B=t: 0.002·0.598525 + 0.998·0.592230 = 0.59224259
B=f: 0.002·0.183055 + 0.998·0.0011295 = 0.00149335
```

**Step 3 — multiply remaining factors** ($f_B$ and $τ_E$) → $P(B,j,m)$:
```
P(B=t, j, m) = 0.001 · 0.59224259 = 0.00059224
P(B=f, j, m) = 0.999 · 0.00149335 = 0.00149186
```

**Step 4 — normalise.** $\alpha=1/(0.00059224+0.00149186)=1/0.00208410$.
```
P(B=t | j,m) = 0.00059224 / 0.00208410 ≈ 0.284
P(B=f | j,m) = 0.00149186 / 0.00208410 ≈ 0.716
```

**Answer: $P(B=t\mid j,m)\approx 0.284$.**

*Reads sensibly:* both neighbours calling raises burglary belief from the prior $0.001$ to $\approx 0.28$ — yet it's still under 50%, because false alarms and innocent calls are common. (As a byproduct, $P(j,m)=1/\alpha\approx 0.00208$.)

---

### Problem 2 (NAT) — factor product and sum-out (the mechanics)

**Q.** Given factors
```
f1(A,B):  (a,b)=0.3  (a,¬b)=0.5  (¬a,b)=0.8  (¬a,¬b)=0.1
f2(B,C):  (b,c)=0.2  (b,¬c)=0.9  (¬b,c)=0.6  (¬b,¬c)=0.4
```
compute $g(A,C)=\sum_B \big(f_1(A,B)\cdot f_2(B,C)\big)$.

**Solution.** First the **pointwise product** $h(A,B,C)=f_1(A,B)\,f_2(B,C)$ (entries agree on $B$):
```
(a,b,c)=0.3·0.2=0.06    (a,b,¬c)=0.3·0.9=0.27
(a,¬b,c)=0.5·0.6=0.30   (a,¬b,¬c)=0.5·0.4=0.20
(¬a,b,c)=0.8·0.2=0.16   (¬a,b,¬c)=0.8·0.9=0.72
(¬a,¬b,c)=0.1·0.6=0.06  (¬a,¬b,¬c)=0.1·0.4=0.04
```
Now **sum out $B$** (add the $b$ and $\lnot b$ rows that share $(A,C)$):
```
g(a,c)   = 0.06 + 0.30 = 0.36
g(a,¬c)  = 0.27 + 0.20 = 0.47
g(¬a,c)  = 0.16 + 0.06 = 0.22
g(¬a,¬c) = 0.72 + 0.04 = 0.76
```
**Answer:** $g(a,c){=}0.36,\ g(a,\lnot c){=}0.47,\ g(\lnot a,c){=}0.22,\ g(\lnot a,\lnot c){=}0.76.$
(Note these do **not** sum to 1 — $g$ is a factor, not a distribution.)

---

### Problem 3 (MSQ / short reasoning) — irrelevant variables & elimination ordering

**Q.** We want $P(\text{JohnCalls})$, i.e. $P(J)$ with **no evidence**, on the alarm net.
(a) Which variable(s) are irrelevant and can be pruned? (b) Compare the cost of elimination order $\sigma_1=[M,B,E,A]$ vs $\sigma_2=[A,B,E,M]$ by the size of the largest factor created.

**Solution.**

**(a) Pruning.** $J$ is the query; there is no evidence. $M$ is a **leaf** and is neither query nor evidence ⇒ **barren ⇒ remove $M$**. ($B,E,A$ are all ancestors of $J$, hence relevant.) After pruning, $\sum_M P(M\mid A)=1$, so $M$ never mattered.

**(b) Ordering (using the full set $\{B,E,A,M\}$ to see the effect):**

*Order $\sigma_1=[M,B,E,A]$ — leaves first:*
```
Elim M: factors{P(M|A)} → Σ_M = 1                       largest factor scope {M,A} → size 2
Elim B: factors{P(B), P(A|B,E)} → τ(A,E)                largest scope {A,B,E}    → size 3
Elim E: factors{P(E), τ(A,E)} → τ(A)                    largest scope {A,E}      → size 2
Elim A: factors{τ(A), P(J|A), (1)} → τ(J)               largest scope {A,J}      → size 2
```
Largest factor = **3 variables** ($\Rightarrow$ induced width 2, $\le 2^3=8$ entries).

*Order $\sigma_2=[A,B,E,M]$ — hub $A$ first:*
```
Elim A: factors{P(A|B,E), P(J|A), P(M|A)} → τ(B,E,J,M)  largest scope {A,B,E,J,M} → size 5
... subsequent factors stay ~4 variables ...
```
Largest factor = **5 variables** ($\Rightarrow$ induced width 4, up to $2^5=32$ entries).

**Conclusion:** $\sigma_1$ is far cheaper ($\sim 2^3$ vs $\sim 2^5$ in the dominant step). Eliminating the **hub $A$ first** forces a giant factor over all of $A$'s neighbours; eliminating **leaves first** keeps factors small. Pruning $M$ first makes it cheaper still.

---

### Problem 4 (MCQ) — complexity statement

**Q.** Which is TRUE of exact inference by Variable Elimination?
(A) Its running time is always polynomial in the number of variables.
(B) Its running time is exponential in the induced width of the elimination ordering, and the answer it returns is exact.
(C) Different elimination orderings can yield different final probabilities.
(D) It is generally faster than inference by enumeration only because it uses a different (incorrect) approximation.

**Solution.** (A) false — only polytrees (bounded width) are efficient; multiply-connected networks can be exponential. (C) false — ordering changes cost, never the answer. (D) false — VE is **exact**, not an approximation; its speed-up comes from caching via the distributive law. **(B) is correct.**

---

## 6. Common GATE question types on this topic

1. **Compute a posterior** $P(Q\mid e)$ end-to-end (set up factors, eliminate, normalise). *(Problem 1.)*
2. **Factor algebra**: perform a pointwise product and/or sum-out on given factors, report the resulting table. *(Problem 2.)*
3. **Identify irrelevant/barren variables** for a given query–evidence pair. *(Problem 3a.)*
4. **Compare elimination orderings** by largest-factor size / induced width / number of multiplications. *(Problem 3b.)*
5. **Complexity / properties MCQ**: exactness, exponential-in-treewidth, ordering affects cost not answer, polytrees are linear. *(Problem 4.)*
6. **Enumeration vs VE** conceptual contrast (same answer; VE = distributive law + caching).
7. **Count operations** of inference by enumeration (number of multiplications/additions for a stated query) — sometimes asked as a NAT.

---

## 7. Traps, pitfalls & common mistakes

- **Summing a variable out too early.** You must first multiply **all** factors that still contain $Z$. Forgetting one → wrong answer. *(The iron rule of §4.3.)*
- **Treating a factor as a probability distribution.** Intermediate factors generally do **not** sum to 1; do **not** "normalise" them mid-algorithm. Normalisation happens **once, at the very end**, over the query. *(Problem 2 shows entries summing to >1.)*
- **Forgetting to normalise / mishandling evidence.** $P(Q,e)$ is unnormalised; the posterior needs $\alpha=1/\sum_q P(q,e)$. Also: evidence variables are **fixed, not summed over** — restrict their factors instead of marginalising them.
- **Wrong product scope.** The product of two factors has scope = **union** of their scopes; you multiply only entries that **agree on shared variables**. Mixing up which rows multiply is a frequent arithmetic slip.
- **Bad elimination order.** Eliminating a high-degree hub early creates an enormous factor. Prefer leaves first; hubs last. *(Problem 3.)*
- **Not pruning barren variables.** Carrying irrelevant leaves wastes work and invites errors. Remove any non-query, non-evidence leaf first.
- **Off-by-one in width.** Induced width = (largest factor scope) **− 1**, not the scope itself.
- **Believing VE is always efficient.** It's only linear for **polytrees**; for high-treewidth networks it's exponential — that's the whole motivation for sampling.
- **Confusing exact with approximate.** VE returns the **exact** posterior. If a question says "VE approximates," it's false.
- **Zero-probability evidence.** $P(Q\mid e)$ is undefined if $P(e)=0$ (the normaliser $\alpha$ diverges).

---

## 8. Connection forward

Variable Elimination is the **exact** answer to "what does the network believe given the evidence" — but it pays in cost that is **exponential in the network's treewidth**. For densely connected networks (many overlapping causes and effects), even the best elimination order produces unmanageably large factors, and exact inference becomes infeasible. **Topic 8 — Approximate Inference: Sampling** answers the *same* queries by drawing random samples consistent with the network (and the evidence) and **estimating** probabilities from sample frequencies — giving answers that get more accurate with more samples, at a cost we control. VE is the benchmark of correctness against which those approximations are judged.

---

## 9. One-line cheat-sheet summary

**Variable Elimination = inference-by-enumeration sped up by the distributive law: repeatedly "multiply all factors containing $Z$, then sum $Z$ out," after restricting evidence and pruning non-ancestor (barren) variables; normalise once at the end. The answer is *exact*; cost is $\Theta(n\,2^{w})$ — exponential in the elimination order's induced width $w$ (linear for polytrees) — so ordering changes cost, never the result.**

---

*Next topic (in order): **Topic 8 — Approximate Inference: Sampling.***

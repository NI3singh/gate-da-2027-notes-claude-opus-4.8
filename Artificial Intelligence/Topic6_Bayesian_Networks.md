# Topic 6 — Conditional Independence Representation (Bayesian Networks)

*Subject: Artificial Intelligence · GATE DA · Section: Reasoning Under Uncertainty*

---

## 0. Topic header & syllabus map

- **Topic:** Conditional Independence Representation — **Bayesian Networks (BNs)**, also called *belief networks* or *probabilistic graphical models (directed)*.
- **Subject:** Artificial Intelligence.
- **Where it sits:** This is the **first topic of "Reasoning Under Uncertainty"**. It comes right after **Logic (Propositional + Predicate)**, which could only express things that are *definitely true or false*. Bayesian networks let us represent **degrees of belief** and the **(conditional) independence structure** among many random variables compactly. It directly **leads into Topic 7 (Exact Inference — Variable Elimination)** and Topic 8 (Approximate Inference — Sampling), both of which *answer queries* on the network this topic teaches you to *build and read*.

---

## 1. Why this topic exists (motivation + connection)

**The problem it solves.** Real agents act under uncertainty: a doctor doesn't *know* you have flu, a robot doesn't *know* a door is open. We need to represent and combine **probabilities**, not just true/false facts.

**What the previous topic (Logic) couldn't do.** First-order and propositional logic are **monotone and categorical** — a sentence is true, false, or unknown. Logic cannot say "given a cough and fever, flu has probability 0.7," nor combine many such partial beliefs coherently. It also has no clean way to revise belief when new evidence arrives. Probability does all of this, and Bayesian networks make it *tractable*.

**Why not "just use the full joint distribution"?** A joint distribution over $n$ binary variables has $2^n - 1$ independent numbers. For $n = 30$ that's over a billion — impossible to store, learn, or reason with. **The full joint is correct but unusable.** Bayesian networks exploit the fact that, in the real world, most variables only *directly* depend on a few others. Encoding these **conditional independences** as a graph shrinks $2^n-1$ numbers down to something linear-ish, while still representing the *exact same joint distribution*.

**What the next topic builds on it.** Once the network compactly stores the joint, **Variable Elimination (Topic 7)** computes any query $P(\text{query} \mid \text{evidence})$ from it *without* ever rebuilding the full joint. So this topic is the data structure; the next is the algorithm that runs on it.

---

## 2. Prerequisites

- **Basic probability:** sample space, events, $P(A) \in [0,1]$, $\sum P = 1$.
- **Random variables** and **distributions** (here mostly discrete/Boolean for GATE).
- **Conditional probability, product rule, Bayes' rule, chain rule** (recapped in §4.1 — revise these first if rusty).
- **Marginalization** (summing out variables).
- **Directed Acyclic Graphs (DAGs):** parents, children, descendants, ancestors, topological order. (You met DAGs in Search/graph topics.)
- Comfort with **summation $\sum$ and product $\prod$ notation.**

---

## 3. Core idea — intuition first

Imagine variables: **Burglary (B), Earthquake (E), Alarm (A), JohnCalls (J), MaryCalls (M).**

Common sense tells you:
- A burglary or an earthquake can set off the **alarm**.
- John and Mary call **only because they heard the alarm** — not because they directly sense burglars or quakes.

So the *direct* causal arrows are:

```
   B        E
    \      /
     v    v
       A           (A's direct causes are B and E)
      / \
     v   v
     J    M        (J and M are caused only by A)
```

The key realisation: **once you know whether the alarm rang, learning about a burglary tells you nothing extra about whether John calls.** John depends on the alarm *and nothing else directly*. Formally $J \perp B \mid A$. The graph **encodes exactly these "given my parents, I'm independent of the rest" statements.**

That single idea does two jobs at once:

1. **Compression.** Instead of one giant joint table, each node stores only $P(\text{node} \mid \text{its parents})$ — a small **Conditional Probability Table (CPT)**.
2. **Reasoning structure.** The arrows tell us *how information flows*: which observations make two variables relevant to each other and which make them irrelevant.

A Bayesian network is just this picture **plus the small CPTs**. From it you can rebuild the full joint exactly, and you can *read off* independence facts by looking at paths in the graph.

---

## 4. Formal treatment

### 4.1 Probability machinery you will actually use

All variables below are discrete. For Boolean $X$, write $x$ for $X=\text{true}$ and $\lnot x$ for $X=\text{false}$.

```
Product rule:        P(A, B) = P(A | B) · P(B) = P(B | A) · P(A)
Conditional prob.:   P(A | B) = P(A, B) / P(B),           valid only if P(B) > 0
Bayes' rule:         P(A | B) = P(B | A) · P(A) / P(B),    valid only if P(B) > 0
Marginalisation:     P(A) = Σ_b P(A, B=b)         (sum over ALL values of B)
Chain rule (always): P(X1,...,Xn) = Π_{i=1}^n P(Xi | X1, ..., X_{i-1})
Normalisation:       P(A | B) = α · P(A, B),  α = 1 / Σ_a P(A=a, B)
```

**Conditions to remember:** conditioning on a zero-probability event is undefined; the chain rule above holds for **any** ordering of the variables (no assumptions needed — that's important for §4.4).

**Size of the full joint.** For $n$ Boolean variables the joint has $2^n$ cells; because they sum to 1, only $\mathbf{2^n - 1}$ are independent. This number is the benchmark BNs beat.

---

### 4.2 (Conditional) independence — precise definitions

**Marginal independence** $A \perp B$:

```
A ⊥ B   ⇔   P(A, B) = P(A) · P(B)
        ⇔   P(A | B) = P(A)        (whenever P(B) > 0)
        ⇔   P(B | A) = P(B)        (whenever P(A) > 0)
```

**Conditional independence** $A \perp B \mid C$ (read: "A and B are independent *given* C"):

```
A ⊥ B | C   ⇔   P(A, B | C) = P(A | C) · P(B | C)
            ⇔   P(A | B, C) = P(A | C)     (whenever P(B, C) > 0)
```

**Crucial GATE warning:** marginal and conditional independence are **independent properties** — neither implies the other.
- $A \perp B$ can be **true** while $A \perp B \mid C$ is **false** (e.g., $B$ and $E$ given $A$ in the alarm net — see "explaining away").
- $A \perp B$ can be **false** while $A \perp B \mid C$ is **true** (e.g., $J$ and $M$, dependent in general but independent given $A$).

This is the most-tested subtlety in the whole topic.

---

### 4.3 What a Bayesian network is (definition)

A **Bayesian network** over variables $X_1,\dots,X_n$ is:

1. A **directed acyclic graph (DAG)** whose nodes are the variables. An edge $X \to Y$ means "$X$ is a **parent** (direct influence) of $Y$." *No directed cycles allowed.*
2. For **each** node $X_i$, a **CPT** giving $P(X_i \mid \mathrm{Parents}(X_i))$.

Notation: $\mathrm{Pa}(X_i)$ = parents of $X_i$.

A node with **no parents** (a *root*) stores an unconditional prior $P(X_i)$.

---

### 4.4 The factorization theorem — *global semantics* (and why it holds)

> **Chain rule for Bayesian networks.** A BN represents the joint distribution
>
> ```
> P(X1, X2, ..., Xn) = Π_{i=1}^n  P(Xi | Parents(Xi))
> ```
>
> **Conditions:** the variables are numbered in a **topological order** (every parent comes before its children), and the network's local independence assumptions hold.

**Why this is true (short derivation — GATE may ask "justify the factorization").**
Start from the chain rule, which is *always* valid for a topological ordering $X_1,\dots,X_n$:

```
P(X1,...,Xn) = Π_i P(Xi | X1, ..., X_{i-1})
```

Because the ordering is topological, every parent of $X_i$ appears among $X_1,\dots,X_{i-1}$. The BN's **Markov assumption** (§4.5) says $X_i$ is independent of its other predecessors once we condition on its parents:

```
P(Xi | X1, ..., X_{i-1}) = P(Xi | Parents(Xi))
```

Substituting gives the product form. ∎

**Reading direction (used in problems).** To compute the probability of a **full assignment** (every variable given a value), just multiply one number per node:

```
P(x1, ..., xn) = Π_i  CPT-entry for (Xi=xi given its parents' values)
```

---

### 4.5 Local semantics (the Markov condition)

> **Each variable is conditionally independent of its non-descendants, given its parents.**
>
> ```
> Xi ⊥ NonDescendants(Xi) | Parents(Xi)
> ```

"Non-descendants" = everything except $X_i$, its children, grandchildren, … This local statement is **equivalent** to the global factorization in §4.4 (each implies the other). It's the precise version of "given my parents, the rest of the past is irrelevant."

---

### 4.6 Markov blanket

> The **Markov blanket** $\mathrm{MB}(X)$ of a node $X$ is:
>
> ```
> MB(X) = Parents(X) ∪ Children(X) ∪ Parents-of-Children(X)
> ```
> (the "other parents" of $X$'s children — sometimes called **co-parents** or **spouses**).
>
> **Property:** given its Markov blanket, $X$ is conditionally independent of *all* other variables:
> ```
> X ⊥ (everything else) | MB(X)
> ```

This is the smallest set that "shields" $X$. (It matters in Topic 8: Gibbs sampling only needs a node's Markov blanket.)

**Why co-parents are included:** $X$ and a co-parent share a child (a collider). Conditioning on that child makes them dependent (§4.7), so the co-parent must be inside the blanket to restore independence.

---

### 4.7 d-separation — the master tool for reading independence off the graph

This answers: **"Does the network guarantee $X \perp Y \mid Z$?"** purely from the graph, without any numbers. $Z$ is the set of **observed** (evidence) variables.

Examine every **undirected path** between $X$ and $Y$. Along a path, look at each **triple** of consecutive nodes $\;\cdot \to \mathbf{m} \to \cdot\;$ (the middle node $m$). There are exactly three shapes:

| # | Shape (along the path) | Name | Path through $m$ is **BLOCKED** when… | **ACTIVE** (open) when… |
|---|---|---|---|---|
| 1 | $\;A \to m \to C\;$ | **Chain / serial** | $m \in Z$ (observed) | $m \notin Z$ |
| 2 | $\;A \leftarrow m \to C\;$ | **Fork / common cause** | $m \in Z$ (observed) | $m \notin Z$ |
| 3 | $\;A \to m \leftarrow C\;$ | **Collider / v-structure / common effect** | $m \notin Z$ **and no descendant of $m$ is in $Z$** | $m \in Z$ **or some descendant of $m$ is in $Z$** |

**Reading the table:** chains and forks behave *normally* — observing the middle node cuts the flow. **Colliders are the reverse:** they are blocked by default and **open up when you observe the collider (or any of its descendants).**

**Combining triples into a verdict.**
- A **path is ACTIVE** iff **every** triple on it is active.
- A **path is BLOCKED** iff **at least one** triple on it is blocked.
- $X$ and $Y$ are **d-separated given $Z$** iff **all** paths between them are blocked. Then the network guarantees $\;X \perp Y \mid Z$.
- If even one active path exists, $X,Y$ are **d-connected** — the network does **not** assert independence (they're generally dependent).

> **Soundness/correctness (state it precisely):** d-separation is **sound** — if $X$ and $Y$ are d-separated given $Z$, then $X \perp Y \mid Z$ in **every** distribution that factorizes over the DAG. The converse holds *structurally*: if they are d-connected, there exists *some* compatible distribution in which they are dependent. (A specific numerical CPT could still be independent by coincidence, but GATE tests the structural reading.)

**"Explaining away" — the intuition behind colliders.** Two independent causes ($B$, $E$) of a common effect ($A$) become **dependent once the effect is observed.** If the alarm rang and you then learn it was an earthquake, the burglary explanation becomes *less* needed — the causes compete. Hence $B \perp E$ but $B \not\perp E \mid A$.

---

### 4.8 Parameter counting — how much the BN saves

> **CPT size for one node.** If $X$ has $r_X$ possible values and its parents have $r_{j}$ values each, its CPT needs
>
> ```
> (r_X − 1) · Π_{j ∈ Parents(X)} r_j      independent numbers
> ```
> The "$-1$" is because each row (one parent configuration) must sum to 1, so one entry is determined.
>
> **All-Boolean special case:** node with $k$ Boolean parents needs $\;2^k\;$ independent numbers.
>
> **Total for the network:**
> ```
> Σ_X  (r_X − 1) · Π_{j ∈ Parents(X)} r_j
> ```

Compare with the **full joint** ($2^n - 1$ for $n$ Boolean variables). The BN is far smaller **whenever each node has few parents.** If every node has at most $k$ parents, total parameters $\le n \cdot 2^k$ — *linear in $n$* instead of exponential.

---

### 4.9 Variable ordering & compactness (why the graph can be good or bad)

You can build a BN for the *same* distribution using **any** variable ordering — but the **wrong order produces a dense, useless graph.**

- **Good order = causes before effects.** Each effect needs only its few direct causes as parents → sparse graph, tiny CPTs.
- **Bad order** (e.g., symptoms before diseases, $M, J, A, B, E$ in the alarm net) forces nodes to take many parents to recover the right dependencies → almost-complete graph, exponential CPTs.

**Takeaway for GATE:** a Bayesian network is **not unique**; only the *full joint* it encodes is fixed. Adding a few "extra" correct edges never makes the represented distribution wrong — it only adds independence *assumptions you fail to exploit* (more parameters). Removing a *needed* edge **can** make it wrong.

---

## 5. Worked problems (GATE-style)

We use the classic **Alarm network** for Problems 1–4. All variables Boolean.

**Structure:** $B \to A$, $E \to A$, $A \to J$, $A \to M$. So $\mathrm{Pa}(A)=\{B,E\}$, $\mathrm{Pa}(J)=\{A\}$, $\mathrm{Pa}(M)=\{A\}$; $B,E$ are roots.

**CPTs:**

```
P(B=t) = 0.001                 P(E=t) = 0.002

P(A=t | B, E):                 P(J=t | A):          P(M=t | A):
  B=t, E=t : 0.95                A=t : 0.90           A=t : 0.70
  B=t, E=f : 0.94                A=f : 0.05           A=f : 0.01
  B=f, E=t : 0.29
  B=f, E=f : 0.001
```

---

### Problem 1 (NAT) — probability of a full assignment

**Q.** Compute $P(J=t,\ M=t,\ A=t,\ B=f,\ E=f)$ — "the alarm rang, both called, but there was no burglary and no earthquake."

**Solution.** It's a full assignment, so apply the factorization (one CPT entry per node):

```
P(j, m, a, ¬b, ¬e)
  = P(¬b) · P(¬e) · P(a | ¬b, ¬e) · P(j | a) · P(m | a)
  = (1−0.001) · (1−0.002) · 0.001 · 0.90 · 0.70
  = 0.999 · 0.998 · 0.001 · 0.90 · 0.70
```

Step the multiplication:
```
0.999 · 0.998      = 0.997002
× 0.001            = 0.000997002
× 0.90             = 0.0008973018
× 0.70             = 0.00062811126
```

**Answer ≈ 0.000628** (i.e., $6.28\times10^{-4}$).

*Lesson:* full-assignment queries need **no summation** — just multiply the right CPT cells. The "no burglary/quake but alarm rang" path is rare (uses the 0.001 false-alarm rate), which is why the number is tiny.

---

### Problem 2 (NAT) — counting independent parameters; BN vs full joint

**Q.** (a) How many independent numbers fully specify this network? (b) How many would the full joint over the 5 variables need? 

**Solution.** Use $2^{(\#\text{parents})}$ per Boolean node:

```
Node   Parents   Params
 B       0        2^0 = 1
 E       0        2^0 = 1
 A      {B,E}     2^2 = 4
 J      {A}       2^1 = 2
 M      {A}       2^1 = 2
                 -------
        Total      10
```

(a) **10 independent parameters.**
(b) Full joint over 5 Boolean variables: $2^5 - 1 = \mathbf{31}$.

So the BN uses **10 vs 31** numbers — and the gap explodes as the network grows.

---

### Problem 3 (MSQ) — d-separation on the alarm net

**Q.** Using only the graph structure, which of the following hold?
(i) $J \perp M \mid A$  (ii) $J \perp M$  (iii) $B \perp E$  (iv) $B \perp E \mid A$  (v) $B \perp E \mid J$

**Solution.** Check paths and triples.

- **(i) $J \perp M \mid A$.** Only path $J \leftarrow A \to M$ = **fork at $A$**; $A$ observed ⇒ blocked. All paths blocked ⇒ **TRUE.**
- **(ii) $J \perp M$ (no evidence).** Same fork $J \leftarrow A \to M$; $A$ *not* observed ⇒ active ⇒ d-connected ⇒ **FALSE** (they're dependent: both reflect the alarm).
- **(iii) $B \perp E$ (no evidence).** Only path $B \to A \leftarrow E$ = **collider at $A$**; $A$ not observed, no descendant observed ⇒ blocked ⇒ **TRUE.**
- **(iv) $B \perp E \mid A$.** Collider at $A$ with $A$ observed ⇒ active ⇒ **FALSE.** (This is *explaining away*.)
- **(v) $B \perp E \mid J$.** $J$ is a **descendant** of the collider $A$ ($A \to J$). Observing a descendant of a collider also opens it ⇒ active ⇒ **FALSE.**

**Correct independencies: (i) and (iii) only.** (iv) and (v) are the classic traps.

---

### Problem 4 (NAT, short reasoning) — Markov blanket

**Q.** Write the Markov blanket of (a) $A$, (b) $B$, (c) $J$.

**Solution.** $\mathrm{MB}(X) = \mathrm{Pa}(X)\cup\mathrm{Children}(X)\cup\text{co-parents}$.

```
(a) MB(A): parents {B,E} ∪ children {J,M} ∪ co-parents {none}  = {B, E, J, M}
(b) MB(B): parents {}    ∪ children {A}   ∪ co-parents of A {E} = {A, E}
(c) MB(J): parents {A}   ∪ children {}     ∪ co-parents {none}  = {A}
```

Note $E$ enters $\mathrm{MB}(B)$ only because $B$ and $E$ are **co-parents of $A$** (a collider) — directly reflecting "explaining away."

---

### Problem 5 (MSQ, trap-heavy) — diamond network & conditioning on a common effect

**Network 2 (Boolean):** $A \to B$, $A \to C$, $B \to D$, $C \to D$. ($A$ = common cause of $B,C$; $D$ = common effect of $B,C$.)

```
        A
       / \
      v   v
      B    C
       \  /
        vv
        D
```

**Q.** Which hold? (i) $B \perp C$  (ii) $B \perp C \mid A$  (iii) $B \perp C \mid \{A, D\}$

**Solution.** Two undirected paths between $B$ and $C$: **Path-1** $B \leftarrow A \to C$ (fork at $A$); **Path-2** $B \to D \leftarrow C$ (collider at $D$).

- **(i) $B \perp C$ (no evidence).** Path-1 fork, $A$ unobserved ⇒ **active** ⇒ d-connected ⇒ **FALSE.** (One active path is enough to break independence.)
- **(ii) $B \perp C \mid A$.** Path-1 fork, $A$ observed ⇒ blocked. Path-2 collider at $D$, $D$ not observed (and no descendant observed) ⇒ blocked. Both paths blocked ⇒ **TRUE.**
- **(iii) $B \perp C \mid \{A, D\}$.** Path-1 still blocked (A observed). **But** Path-2 collider at $D$ now has $D$ observed ⇒ **active** ⇒ d-connected ⇒ **FALSE.**

**Correct: (ii) only.** The jump from (ii) TRUE to (iii) FALSE — caused purely by *adding* the common effect $D$ to the evidence — is the quintessential GATE trap.

---

## 6. Common GATE question types on this topic

1. **Compute a full-joint probability** from the graph + CPTs (multiply one entry per node). *(Problem 1.)*
2. **Count independent parameters** of a BN, and/or compare with the full joint $2^n-1$. *(Problem 2.)*
3. **d-separation / conditional-independence MSQs:** "Which statements are guaranteed?" — especially fork vs collider, and **conditioning on a collider or its descendant.** *(Problems 3, 5.)*
4. **Identify the Markov blanket** of a node. *(Problem 4.)*
5. **Pick the correct factorization** of the joint that matches a given DAG (or vice versa: which DAG encodes a given factorization).
6. **Validity / minimality of edges:** which edge can be added/removed without changing the represented distribution; recognizing that the BN is not unique.
7. **Small one-step inference** (a tiny marginalization or Bayes update) — full machinery comes in Topic 7, but compact 2–3 variable versions appear here.

---

## 7. Traps, pitfalls & common mistakes

- **Collider direction reversed.** The #1 error. For $A \to m \leftarrow C$: the path is **OPEN when $m$ is observed**, blocked when it isn't — the *opposite* of chains/forks. Memorize: *"observing a collider connects its parents (explaining away)."*
- **Forgetting collider descendants.** Observing **any descendant** of a collider also opens it. (Problem 3(v): conditioning on $J$ opens collider $A$.)
- **Confusing $A\perp B$ with $A\perp B\mid C$.** They are *separate*; neither implies the other. Always re-check whether evidence is present.
- **"One blocked triple blocks the path; one active path breaks independence."** Mixing the AND/OR logic: a path needs **all** triples active to carry information; d-separation needs **all** paths blocked.
- **Using non-topological order in the factorization.** $P(X_i\mid \mathrm{Pa}(X_i))$ is only the right factor when parents precede $X_i$. With the chain rule you may use any order, but then the conditioning set is *all predecessors*, not just parents.
- **Parameter miscount.** Forget the "$-1$" per row (rows sum to 1). A Boolean node with $k$ Boolean parents has $2^k$ params, **not** $2^{k+1}$. Roots have $1$ param, not $2$.
- **Assuming the BN is unique.** Same joint ⇒ many valid DAGs (depending on ordering). Don't reject a correct-but-denser network as "wrong."
- **Reading edges as logical implication.** $X\to Y$ is a *probabilistic* dependence, **not** "X causes Y with certainty" and not a logical rule. Absence of an edge asserts a (conditional) independence; presence asserts nothing numeric until you read the CPT.
- **Treating absent independence as guaranteed dependence.** d-separation *guarantees* independence; d-connection only means independence is *not guaranteed* — a specific CPT could still be independent numerically (not usually asked, but don't over-claim).
- **Zero-probability conditioning.** $P(\cdot\mid B)$ is undefined if $P(B)=0$ — watch degenerate CPTs.

---

## 8. Connection forward

You can now **build** a Bayesian network, **read its independencies** via d-separation, and **count its parameters.** What you *cannot yet do efficiently* is answer a general query like $P(\text{Burglary}\mid \text{JohnCalls}=t,\ \text{MaryCalls}=t)$, which needs summing out the hidden variables. Doing that naively (rebuilding the joint) defeats the whole point of the compact graph. **Topic 7 — Variable Elimination** is the algorithm that exploits the very factorization in §4.4 to compute such queries cheaply, pushing sums inside products. The network you learned to write here is exactly its input.

---

## 9. One-line cheat-sheet summary

**A Bayesian network = DAG + CPTs encoding $P(X_{1..n})=\prod_i P(X_i\mid \mathrm{Pa}(X_i))$; read independence by d-separation (chains/forks block when the middle node is observed, colliders *open* when the collider or a descendant is observed — "explaining away"), and a Boolean node with $k$ Boolean parents costs $2^k$ parameters.**

---

*Next topic (in order): **Topic 7 — Exact Inference: Variable Elimination.***

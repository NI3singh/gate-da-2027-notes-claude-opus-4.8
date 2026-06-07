# Topic 5 — Vector Spaces and Subspaces

*GATE DA · Linear Algebra · "Vector Spaces and Rank" section (Topic 1 of 3 in this block)*

---

## 0. Topic header & syllabus map

- **Topic:** Vector Spaces and Subspaces
- **Subject:** Linear Algebra (GATE Data Science & AI)
- **Where it sits:** This is **Topic 5**, the opening topic of the *Vector Spaces and Rank* section. It comes **after** the *Matrices & Systems* block — Matrices (1) → Determinant (2) → Systems of Linear Equations & Gaussian Elimination (3) → LU Decomposition (4) — and it **leads into** Linear Dependence & Independence (Topic 6), which then feeds Rank & Nullity (Topic 7).

---

## 1. Why this topic exists (motivation + connection)

In Topics 1–4 we treated matrices as **calculation devices**: we multiplied them, took determinants, solved $A\mathbf{x}=\mathbf{b}$ by Gaussian elimination, and factored $A = LU$. But every one of those computations was secretly a question about **collections of vectors**:

- *"Is $A\mathbf{x}=\mathbf{b}$ solvable?"* is really *"Can $\mathbf{b}$ be built as a combination of $A$'s columns?"*
- *"How many solutions does $A\mathbf{x}=\mathbf{0}$ have?"* is really *"How large is the set of vectors that $A$ sends to zero?"*

Gaussian elimination gave us the **procedure** but not the **structure**. It told us whether a particular system had $0$, $1$, or $\infty$ solutions, but it never explained *why* solution sets always come out as clean geometric objects — points, lines, and planes **through the origin**. The **vector space** is the abstraction that captures this structure: it is the minimal set of rules ("you may add vectors and scale them, and you never leave the set") under which all of linear algebra operates. **Subspaces** are exactly the "flat objects through the origin" that keep showing up as solution sets and column collections. This is the vocabulary we must fix *before* we can rigorously define rank, nullity, basis, and dimension in the next two topics.

---

## 2. Prerequisites

- Vector and matrix operations: addition, scalar multiplication, the matrix–vector product (Topic 1).
- The key viewpoint that $A\mathbf{x}$ is a **linear combination of the columns** of $A$: $A\mathbf{x} = x_1(\text{col}_1) + x_2(\text{col}_2) + \cdots$ (Topic 1/3).
- Homogeneous vs. non-homogeneous systems $A\mathbf{x}=\mathbf{0}$ and $A\mathbf{x}=\mathbf{b}$, and what a "solution set" is (Topic 3).
- Arithmetic over the real field $\mathbb{R}$ (the default field in GATE).

---

## 3. Core idea — intuition first

Think of a vector space as a **"playground with two toys."** You can **add** any two things in the playground, and you can **stretch or shrink** anything by a number — and crucially, *neither action ever throws you out of the playground*. That "you stay inside" property is **closure**, and it is the entire game.

The most familiar playground is $\mathbb{R}^3$ (arrows in 3D). But the very same rules govern:

- the set of all $2\times 2$ matrices,
- the set of all polynomials of degree $\le 3$,
- the set of all real-valued functions.

These feel like different worlds, but **algebraically they obey identical rules**, so anything proven "for a vector space" instantly applies to all of them. That reuse is the entire payoff of the abstraction.

A **subspace** is a *smaller playground living inside a bigger one*, using the **same two toys**. In $\mathbb{R}^3$, the only subspaces are:

1. the origin alone $\{\mathbf{0}\}$,
2. any **line through the origin**,
3. any **plane through the origin**,
4. all of $\mathbb{R}^3$.

Notice the repeated phrase: ***through the origin***. A line that misses the origin is **not** a subspace — scaling any of its vectors by $0$ produces $\mathbf{0}$, which is not on that line, so you got thrown out. This one observation ("**must contain $\mathbf{0}$**") is the single most-tested idea in this topic.

> **Analogy.** A vector space is a language with grammar rules; a subspace is a smaller vocabulary that is still *grammatically complete on its own* — you can combine its words endlessly and never produce something outside the vocabulary.

---

## 4. Formal treatment

### 4.1 Definition — Vector space

A **vector space over a field $F$** (for GATE, $F=\mathbb{R}$ unless stated) is a set $V$ equipped with two operations — vector addition $(+)$ and scalar multiplication $(\cdot)$ — such that, for all $\mathbf{u},\mathbf{v},\mathbf{w}\in V$ and all scalars $a,b\in F$, **all** of the following axioms hold:

**Closure**

| | |
|---|---|
| (A0) | $\mathbf{u}+\mathbf{v}\in V$ — closed under addition |
| (M0) | $a\cdot\mathbf{u}\in V$ — closed under scalar multiplication |

**Addition axioms**

| | |
|---|---|
| (A1) | $\mathbf{u}+\mathbf{v}=\mathbf{v}+\mathbf{u}$ (commutative) |
| (A2) | $(\mathbf{u}+\mathbf{v})+\mathbf{w}=\mathbf{u}+(\mathbf{v}+\mathbf{w})$ (associative) |
| (A3) | $\exists\,\mathbf{0}\in V$ with $\mathbf{u}+\mathbf{0}=\mathbf{u}$ (zero / additive identity) |
| (A4) | $\forall\mathbf{u}\;\exists(-\mathbf{u})\in V$ with $\mathbf{u}+(-\mathbf{u})=\mathbf{0}$ (additive inverse) |

**Scalar-multiplication axioms**

| | |
|---|---|
| (M1) | $a\cdot(\mathbf{u}+\mathbf{v})=a\mathbf{u}+a\mathbf{v}$ |
| (M2) | $(a+b)\cdot\mathbf{u}=a\mathbf{u}+b\mathbf{u}$ |
| (M3) | $a\cdot(b\cdot\mathbf{u})=(ab)\cdot\mathbf{u}$ |
| (M4) | $1\cdot\mathbf{u}=\mathbf{u}$ |

> **Conditions / assumptions.** The field is fixed (here $\mathbb{R}$). **All** axioms must hold; failing even one means it is *not* a vector space. (Some textbooks list "8 axioms" by folding closure into the definition of the operations; the content is identical.) From the axioms it follows that the zero vector is **unique** and each inverse is **unique** — GATE may state these as facts.

**Standard examples (memorize):**

- $\mathbb{R}^n$ with componentwise $+$ and $\cdot$ — the prototype.
- $M_{m\times n}(\mathbb{R})$: all $m\times n$ real matrices.
- $P_n$: all polynomials of degree $\le n$ (this **includes** the zero polynomial).
- $P$ (all polynomials), $C[a,b]$ (continuous functions on $[a,b]$), and the space of all functions $\mathbb{R}\to\mathbb{R}$.
- $\{\mathbf{0}\}$: the **trivial / zero** vector space (the smallest possible one).

**Non-examples (classic traps):**

- Polynomials of degree **exactly** $n$ — **not** a vector space (no zero polynomial; not closed: $x^n + (-x^n + x)$ has degree $<n$).
- The first quadrant $\{(x,y):x\ge 0,\,y\ge 0\}$ — **not** a vector space (no additive inverses; not closed under negative scalars).
- $\mathbb{R}^2$ with a **redefined** $+$ or $\cdot$ — GATE sometimes changes the operations and asks you to verify the axioms from scratch.

### 4.2 Definition — Subspace

A subset $W\subseteq V$ is a **subspace** of $V$ if $W$ is itself a vector space under the operations inherited from $V$.

> **Subspace Test — the only thing you ever compute.** $W\subseteq V$ is a subspace **iff** all three hold:
>
> - **(S1)** $W$ is non-empty — equivalently, $\mathbf{0}\in W$.
> - **(S2)** Closed under addition: $\mathbf{u},\mathbf{v}\in W \Rightarrow \mathbf{u}+\mathbf{v}\in W$.
> - **(S3)** Closed under scalar multiplication: $\mathbf{u}\in W,\ a$ scalar $\Rightarrow a\mathbf{u}\in W$.

**Why only three checks?** Because $W$ sits inside $V$ with the *same* operations, the other seven axioms (commutativity, associativity, distributivity, etc.) are **inherited automatically** from $V$. You never re-verify them.

> **Equivalent one-line test.** $W$ is a subspace iff $W\ne\varnothing$ and it is **closed under linear combinations**:
> $$a\mathbf{u}+b\mathbf{v}\in W \quad\text{for all scalars } a,b \text{ and all } \mathbf{u},\mathbf{v}\in W.$$
> (This merges S2 and S3 into one statement.)

> **⚡ Quick-rejection rule.** If $\mathbf{0}\notin W$, **stop** — $W$ is not a subspace. This instantly kills almost every "translated" / affine set (anything defined by "$=$ a nonzero constant").

### 4.3 Span

For vectors $\mathbf{v}_1,\dots,\mathbf{v}_k\in V$,
$$\operatorname{span}\{\mathbf{v}_1,\dots,\mathbf{v}_k\}=\{\,c_1\mathbf{v}_1+\cdots+c_k\mathbf{v}_k : c_i\in F\,\}.$$

**Theorem.** $\operatorname{span}(S)$ is **always** a subspace of $V$ — in fact the *smallest* subspace containing $S$.
*Condition:* holds for any subset $S$, including $S=\varnothing$, where by convention $\operatorname{span}(\varnothing)=\{\mathbf{0}\}$.

### 4.4 The two matrix subspaces introduced here

*(The complete "four fundamental subspaces" picture is developed in Topic 7; we introduce two of them now because they are the most important examples of subspaces and they close the loop with Topic 3.)*

- **Column space** $C(A)=\operatorname{span}\{\text{columns of }A\}=\{A\mathbf{x}:\mathbf{x}\in\mathbb{R}^n\}\subseteq\mathbb{R}^m$.
  - **Key fact (bridge to Topic 3):** $\boxed{A\mathbf{x}=\mathbf{b}\text{ is solvable}\iff \mathbf{b}\in C(A).}$

- **Null space** $N(A)=\{\mathbf{x}\in\mathbb{R}^n : A\mathbf{x}=\mathbf{0}\}\subseteq\mathbb{R}^n$.
  - $N(A)$ is **always** a subspace (it is the solution set of the *homogeneous* system).
  - *Verification:* $A\mathbf{0}=\mathbf{0}$ so $\mathbf{0}\in N(A)$; if $A\mathbf{x}_1=\mathbf{0}$ and $A\mathbf{x}_2=\mathbf{0}$, then $A(\mathbf{x}_1+\mathbf{x}_2)=\mathbf{0}$ and $A(c\mathbf{x}_1)=\mathbf{0}$. ✔
  - **Contrast:** the solution set of $A\mathbf{x}=\mathbf{b}$ with $\mathbf{b}\ne\mathbf{0}$ is **not** a subspace — it does not contain $\mathbf{0}$.

> Note that an $m\times n$ matrix has its **column space inside $\mathbb{R}^m$** but its **null space inside $\mathbb{R}^n$** — different ambient spaces. Mixing these up is a frequent error (see Traps).

### 4.5 Operations on subspaces

Let $U,W$ be subspaces of $V$.

| Operation | Definition | Subspace? |
|---|---|---|
| **Intersection** | $U\cap W$ | **Always** ✔ |
| **Union** | $U\cup W$ | **Only if** $U\subseteq W$ or $W\subseteq U$ (almost never) ✘ |
| **Sum** | $U+W=\{\mathbf{u}+\mathbf{w}:\mathbf{u}\in U,\,\mathbf{w}\in W\}$ | **Always** ✔ (smallest subspace containing both) |

- **Direct sum:** we write $U\oplus W$ when $U\cap W=\{\mathbf{0}\}$; then every vector of $U+W$ has a **unique** decomposition $\mathbf{u}+\mathbf{w}$.

**Why the union usually fails** (worth knowing, since it's a favourite trap): pick $\mathbf{u}\in U\setminus W$ and $\mathbf{w}\in W\setminus U$. If $U\cup W$ were closed, then $\mathbf{u}+\mathbf{w}$ would lie in $U$ or in $W$. If $\mathbf{u}+\mathbf{w}\in U$, then $\mathbf{w}=(\mathbf{u}+\mathbf{w})-\mathbf{u}\in U$ — contradiction; symmetrically for $W$. So $\mathbf{u}+\mathbf{w}$ escapes the union.

> **Dimension formula (Grassmann's identity).**
> $$\dim(U+W)=\dim U+\dim W-\dim(U\cap W).$$
> *Conditions:* $U,W$ finite-dimensional subspaces of the **same** $V$. For a direct sum, $\dim(U\oplus W)=\dim U+\dim W$ (since $\dim(U\cap W)=\dim\{\mathbf 0\}=0$).
> *(We define "dimension" rigorously in Topic 6; here read it as "number of independent directions.")*

> **On derivations:** GATE asks you to **apply** the subspace test and the dimension formula, never to derive them from the axioms. So we skip those formal derivations and keep only the short, instructive arguments above (null space is a subspace; union usually fails).

---

## 5. Worked problems (GATE-style)

### Problem 1 — Subspace identification (MSQ)

Which of the following subsets of $\mathbb{R}^3$ are subspaces?

- (P) $W_1=\{(x,y,z):x+2y-z=0\}$
- (Q) $W_2=\{(x,y,z):x+2y-z=1\}$
- (R) $W_3=\{(x,y,z):xyz=0\}$
- (S) $W_4=\{(x,y,z):x=y=z\}$

**Solution.**

- **(P)** Defined by a *homogeneous linear* equation. $\mathbf{0}$ satisfies it ($0+0-0=0$ ✔). Any linear combination of solutions of a homogeneous linear equation is again a solution, so it is closed under $+$ and scalar $\cdot$. **Subspace ✔.** (Geometrically: a plane through the origin.)
- **(Q)** Does $\mathbf{0}$ satisfy it? $0+2(0)-0=0\ne 1$. **No.** **Not a subspace ✘** (an affine plane that misses the origin).
- **(R)** Contains $\mathbf{0}$ ✔, so the quick-rejection rule does *not* dismiss it — we must check closure. Take $(1,1,0)\in W_3$ (product $0$) and $(0,1,1)\in W_3$ (product $0$). Their sum $(1,2,1)$ has product $2\ne 0$, so it is **not** in $W_3$. **Not a subspace ✘** (nonlinear defining condition).
- **(S)** This is $\operatorname{span}\{(1,1,1)\}$ — a line through the origin. Contains $\mathbf{0}$, closed under combinations. **Subspace ✔.**

**Answer:** $W_1$ and $W_4$ (i.e., **P and S**).

---

### Problem 2 — Dimension of a sum (NAT)

In $\mathbb{R}^4$, let
$$U=\operatorname{span}\{(1,0,0,0),(0,1,0,0)\},\qquad W=\operatorname{span}\{(0,1,0,0),(0,0,1,0)\}.$$
Find $\dim(U+W)$.

**Solution.**

$\dim U=2$, $\dim W=2$. A vector in $U$ has the form $(a,b,0,0)$; a vector in $W$ has the form $(0,c,d,0)$. For a vector to lie in **both**, matching components forces $a=0$ and $d=0$, leaving $\{(0,b,0,0)\}=\operatorname{span}\{(0,1,0,0)\}$, so $\dim(U\cap W)=1$.

By Grassmann's identity:
$$\dim(U+W)=\dim U+\dim W-\dim(U\cap W)=2+2-1=3.$$

*(Sanity check: $U+W=\operatorname{span}\{e_1,e_2,e_3\}$, which is clearly $3$-dimensional.)*

**Answer:** $\boxed{3}$.

---

### Problem 3 — Matrix subspaces & the union trap (MSQ, with traps)

Let $A$ be a fixed $3\times 4$ real matrix. Which statements are **TRUE**?

- (I) The solution set of $A\mathbf{x}=\mathbf{0}$ is a subspace of $\mathbb{R}^4$.
- (II) The solution set of $A\mathbf{x}=\mathbf{b}$ (for some fixed $\mathbf{b}\ne\mathbf{0}$ making the system consistent) is a subspace of $\mathbb{R}^4$.
- (III) If $U,W$ are subspaces of $\mathbb{R}^4$, then $U\cup W$ is always a subspace of $\mathbb{R}^4$.
- (IV) The column space $C(A)$ is a subspace of $\mathbb{R}^3$.

**Solution.**

- **(I) TRUE.** This is $N(A)\subseteq\mathbb{R}^4$ — always a subspace (contains $\mathbf{0}$, closed).
- **(II) FALSE.** A consistent inhomogeneous system has solution set $\mathbf{x}_p+N(A)$, an **affine** set. It cannot contain $\mathbf{0}$, because $A\mathbf{0}=\mathbf{0}\ne\mathbf{b}$. *(Trap: "solutions of a linear system" are a subspace only when the system is homogeneous.)*
- **(III) FALSE.** A union of subspaces is a subspace only if one contains the other. Counterexample: two distinct lines through the origin in $\mathbb{R}^4$ — their union is not closed under addition.
- **(IV) TRUE.** $C(A)=\operatorname{span}\{\text{columns of }A\}$, and each column lies in $\mathbb{R}^3$, so $C(A)\subseteq\mathbb{R}^3$ is a subspace. *(Note the dimensions: column space in $\mathbb{R}^3=\mathbb{R}^m$, null space in $\mathbb{R}^4=\mathbb{R}^n$.)*

**Answer:** **(I) and (IV)** are true.

---

## 6. Common GATE question types on this topic

- **"Which of the following is a subspace of $\mathbb{R}^n$?"** — classify several given sets. (Always: check $\mathbf{0}\in$ set first, then closure.)
- **Subspaces of matrix spaces** $M_n$. Quick verdicts (each because the defining condition is linear/homogeneous, or fails closure):
  - symmetric matrices — ✔ · skew-symmetric — ✔ · diagonal — ✔ · upper-triangular — ✔
  - trace-zero matrices — ✔ (trace is linear) · matrices commuting with a fixed $B$ — ✔
  - matrices with $\det=0$ — ✘ (not closed under $+$) · **invertible** matrices — ✘ (no zero matrix)
- **Subspaces of polynomial spaces** $P_n$: $\{p:p(0)=0\}$ — ✔ · $\{p:p(1)=0\}$ — ✔ · $\{p:\deg p=n\}$ — ✘ · $\{p:p(2)=1\}$ — ✘.
- **Span membership:** *"Is $\mathbf{v}\in\operatorname{span}\{\dots\}$?"* — set up and solve a linear system (straight back to Gaussian elimination, Topic 3).
- **Dimension of $U+W$ or $U\cap W$** via Grassmann's identity.
- **Column-space / null-space facts**, especially "$A\mathbf{x}=\mathbf{b}$ solvable $\iff \mathbf{b}\in C(A)$."
- **True/false MSQ bundles** mixing: union vs. sum, homogeneous vs. inhomogeneous solution sets, ambient-space ($\mathbb{R}^m$ vs $\mathbb{R}^n$) confusions.

---

## 7. Traps, pitfalls & common mistakes

- **Forgetting the origin.** Any set defined by "$=$ a nonzero constant" (e.g. $x+y=1$) fails because $\mathbf{0}\notin$ set. Check this *first*.
- **Nonlinear defining conditions** ($xy=0$, $x^2=y$, $|x|=|y|$, $\det=0$) almost always break closure **even when $\mathbf{0}$ is in the set**. Containing $\mathbf{0}$ is necessary, *not* sufficient — you must still test closure.
- **"Solutions of a linear system form a subspace" — only if homogeneous.** For $\mathbf{b}\ne\mathbf{0}$ you get an affine set ($\mathbf{x}_p+N(A)$), not a subspace.
- **Union $\ne$ subspace.** $U\cup W$ is a subspace only when one is contained in the other. If you need a subspace from two, use $U+W$.
- **$\deg=n$ vs. $\deg\le n$.** $P_n$ ("$\le n$") is a vector space; "exactly $n$" is not (no zero polynomial, not closed).
- **Closure must hold for *all* elements and *all* scalars** — including the scalar $0$ and **negative** scalars. The first quadrant fails on negative scalars; a half-line/ray fails on scalar $0$.
- **Column space vs. null space ambient confusion.** For an $m\times n$ matrix, $C(A)\subseteq\mathbb{R}^m$ but $N(A)\subseteq\mathbb{R}^n$ — easy to swap the dimensions under time pressure.
- **Don't conflate "span" with "basis."** $\operatorname{span}(S)$ is a subspace whether or not $S$ is independent; independence/minimality is Topic 6.
- **The empty set is *not* a subspace** (it has no $\mathbf{0}$). But $\operatorname{span}(\varnothing)=\{\mathbf{0}\}$ *is* a subspace.
- **Redefined operations.** If the question changes $+$ or $\cdot$, re-check the axioms from scratch — do **not** assume the usual behaviour (this is a deliberate GATE trick).

---

## 8. Connection forward

A subspace can be infinite, yet it is pinned down completely by a handful of "direction" vectors through **span**. That immediately raises the next two questions: *What is the **smallest** spanning set?* and *When does a spanning set carry **no redundancy**?* Answering them is exactly **Linear Dependence and Independence (Topic 6)**, which delivers the notions of **basis** and **dimension** — turning the loose word "size" used in the dimension formula above into a rigorous integer. That count then powers **Rank and Nullity (Topic 7)**, where the column space and null space we met here are measured precisely and linked by the rank–nullity theorem.

---

## 9. One-line cheat-sheet summary

> A **subspace** is a non-empty subset closed under addition and scalar multiplication (equivalently: contains $\mathbf{0}$ and **all** linear combinations); **spans, column spaces, and null spaces are always subspaces**, but affine sets ("$=$ nonzero const"), unions, and nonlinearly-defined sets usually are **not**.

---

**Next topic (in order):** **Topic 6 — Linear Dependence and Independence** (basis, dimension, span), which formalizes the "directions" idea and gives the exact dimension count this topic kept gesturing at.

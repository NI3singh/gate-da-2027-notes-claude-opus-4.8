# Topic 6 — Linear Dependence and Independence (Basis, Dimension and Span)

*GATE DA · Linear Algebra · "Vector Spaces and Rank" section (Topic 2 of 3 in this block)*

---

## 0. Topic header & syllabus map

- **Topic:** Linear Dependence and Independence — together with **Span, Basis, and Dimension**.
- **Subject:** Linear Algebra (GATE Data Science & AI).
- **Where it sits:** This is **Topic 6**, the middle topic of the *Vector Spaces and Rank* section. It comes **after** Vector Spaces and Subspaces (Topic 5) — which gave us subspaces and the loose word "dimension" — and it **leads into** Rank and Nullity (Topic 7), where these ideas are applied directly to a matrix's column space and null space.

---

## 1. Why this topic exists (motivation + connection)

Topic 5 taught us that a few vectors can generate an entire (possibly infinite) subspace through **span**. That immediately raises an efficiency question it left unanswered: *which spanning sets carry no waste?* If $W=\operatorname{span}\{\mathbf{v}_1,\mathbf{v}_2,\mathbf{v}_3\}$ but $\mathbf{v}_3$ is already a combination of $\mathbf{v}_1,\mathbf{v}_2$, then $\mathbf{v}_3$ is **redundant** — dropping it changes nothing. **Linear independence** is exactly the precise statement of "no redundancy."

The previous topic could describe a subspace but could not *measure* it: it used phrases like "number of independent directions" and wrote a dimension formula without ever defining "dimension." This topic fixes that. A **basis** is a spanning set with zero redundancy (independent *and* spanning), and **dimension** is the number of vectors in a basis — a single integer that turns "size" into something rigorous and computable. Without this machinery, "rank" and "nullity" in the next topic would have no meaning, because rank is *defined* as the dimension of a subspace.

---

## 2. Prerequisites

- **Span**, subspaces, and "$\operatorname{span}(S)$ is always a subspace" (Topic 5).
- **Linear combination** $c_1\mathbf{v}_1+\cdots+c_k\mathbf{v}_k$ and the column view $A\mathbf{c}=c_1(\text{col}_1)+\cdots$ (Topic 1/5).
- **Homogeneous systems** $A\mathbf{c}=\mathbf{0}$, free variables, and row reduction to echelon form (Topic 3).
- **Determinant** of a square matrix and the fact "$\det\ne 0\iff$ invertible" (Topic 2).
- The **null space** $N(A)$ as the solution set of $A\mathbf{c}=\mathbf{0}$ (Topic 5) — the literal bridge into independence.

---

## 3. Core idea — intuition first

**Independence = "nobody is redundant."** A set of vectors is **linearly independent** when no vector in it can be built from the others. The instant *one* of them is expressible through the rest, the whole set is **dependent**.

Picture arrows in the plane. Two arrows pointing in genuinely different directions — say East and North — are independent: you cannot make North out of East alone. But East, North, and Northeast are **dependent**, because Northeast $=$ East $+$ North; the third arrow adds no new direction. In $\mathbb{R}^2$ you can have at most $2$ independent directions; a third must be redundant.

Now the three headline words:

- **Span** asks *"do these vectors reach everywhere in $V$?"* (a coverage question).
- **Independence** asks *"do these vectors waste nothing?"* (an efficiency question).
- A **basis** is the sweet spot that answers **both yes** at once: it reaches everything (spans) using no redundant vectors (independent). It is a complete, non-redundant set of **coordinate axes** for the space.
- **Dimension** is simply *how many axes you need* — the number of vectors in a basis. $\mathbb{R}^2$ needs $2$, $\mathbb{R}^3$ needs $3$, a plane through the origin needs $2$, a line needs $1$.

> **Why a basis matters most:** relative to a basis, **every vector has exactly one** recipe (one list of coordinates). Too few vectors and some recipes don't exist (can't span); too many and recipes aren't unique (dependent). A basis is the unique balance point.

---

## 4. Formal treatment

### 4.1 Linear independence and dependence

Let $\mathbf{v}_1,\dots,\mathbf{v}_k\in V$.

> **Definition.** The vectors are **linearly independent** if the *only* solution of
> $$c_1\mathbf{v}_1+c_2\mathbf{v}_2+\cdots+c_k\mathbf{v}_k=\mathbf{0}$$
> is the **trivial** one $c_1=c_2=\cdots=c_k=0$.
> They are **linearly dependent** if there exist scalars **not all zero** making the combination $\mathbf{0}$.

> **Equivalent characterization.** The set is **dependent** $\iff$ at least one vector is a linear combination of the others. (For an *ordered* set, equivalently: some $\mathbf{v}_j$ is a combination of the **preceding** ones.)

**Facts you must know (each with its condition):**

| Fact | Condition |
|---|---|
| Any set containing $\mathbf{0}$ is **dependent** | always (take coefficient $1$ on $\mathbf{0}$) |
| $\{\mathbf{v}\}$ is independent | $\iff \mathbf{v}\ne\mathbf{0}$ |
| $\{\mathbf{u},\mathbf{v}\}$ is dependent | $\iff$ one is a scalar multiple of the other |
| A **superset** of a dependent set is dependent; a **subset** of an independent set is independent | always |
| Any $k$ vectors in $\mathbb{R}^n$ with $k>n$ are **dependent** | always (more vectors than the dimension) |

> **The bridge to matrices (memorize).** Vectors $\mathbf{v}_1,\dots,\mathbf{v}_k\in\mathbb{R}^n$ are **independent**
> $$\iff \operatorname{rank}\big[\,\mathbf{v}_1\ \mathbf{v}_2\ \cdots\ \mathbf{v}_k\,\big]=k \iff A\mathbf{c}=\mathbf{0}\text{ has only the trivial solution} \iff N(A)=\{\mathbf{0}\}.$$
> This links independence directly to the **null space** from Topic 5: *independent columns $\Leftrightarrow$ trivial null space.* It is also the seed of "rank" in Topic 7.

### 4.2 Spanning set

> **Definition.** A set $S\subseteq V$ **spans** $V$ if $\operatorname{span}(S)=V$ — i.e., every vector of $V$ is a linear combination of vectors in $S$. We call $S$ a **spanning (generating) set**.

(Recall from Topic 5: $\operatorname{span}(S)$ is always a subspace, so the only question is *whether it fills up all of* $V$.)

### 4.3 Basis

> **Definition.** A set $B\subseteq V$ is a **basis** of $V$ if it is **(i) linearly independent** and **(ii) spans $V$**.

Equivalent characterizations (all the same object):

- a **minimal** spanning set (remove any vector and it no longer spans), and
- a **maximal** independent set (add any vector and it becomes dependent).

> **Uniqueness of representation (the key property).** $B=\{\mathbf{b}_1,\dots,\mathbf{b}_n\}$ is a basis $\iff$ **every** $\mathbf{v}\in V$ has a **unique** expression $\mathbf{v}=c_1\mathbf{b}_1+\cdots+c_n\mathbf{b}_n$.
> *Short proof of uniqueness (GATE-relevant logic):* if $\mathbf{v}=\sum c_i\mathbf{b}_i=\sum d_i\mathbf{b}_i$, subtract to get $\sum (c_i-d_i)\mathbf{b}_i=\mathbf{0}$; independence forces every $c_i-d_i=0$, so $c_i=d_i$.

**Standard bases (memorize):**

- $\mathbb{R}^n$: the standard basis $\{\mathbf{e}_1,\dots,\mathbf{e}_n\}$.
- $P_n$ (polynomials of degree $\le n$): $\{1,x,x^2,\dots,x^n\}$.
- $M_{m\times n}$: the $mn$ matrices $E_{ij}$ with a single $1$.

### 4.4 Dimension

> **Definition.** $\dim(V)$ is the number of vectors in **any** basis of $V$.

This is well-defined because of:

> **Theorem (all bases have equal size).** In a finite-dimensional space, any two bases contain the same number of vectors.
> *(This rests on the Steinitz exchange/replacement lemma. GATE never asks for the proof, only the consequence — so we skip the derivation.)*

**Standard dimensions (memorize):**

| Space | Dimension |
|---|---|
| $\mathbb{R}^n$ | $n$ |
| $P_n$ (degree $\le n$) | $n+1$ &nbsp; *(note the $+1$ — the constant term)* |
| $M_{m\times n}(\mathbb{R})$ | $mn$ |
| symmetric $n\times n$ | $\dfrac{n(n+1)}{2}$ |
| skew-symmetric $n\times n$ | $\dfrac{n(n-1)}{2}$ |
| diagonal / upper-triangular $n\times n$ | $n$ / $\dfrac{n(n+1)}{2}$ |
| $\{\mathbf{0}\}$ | $0$ &nbsp; *(basis $=\varnothing$)* |

For a subspace $W\subseteq V$: $\dim W\le\dim V$, with **equality iff $W=V$**.

### 4.5 Counting theorems — the exam shortcuts

Let $\dim V=n$. Then:

- Any **independent** set in $V$ has **at most $n$** vectors.
- Any **spanning** set of $V$ has **at least $n$** vectors.
- **Half-test (huge time-saver):** for a set of **exactly $n$** vectors,
  $$\text{independent} \iff \text{spanning} \iff \text{basis}.$$
  So if the count already equals the dimension, you only need to verify **one** of the two properties.
- Any **independent** set **can be extended** to a basis; any **spanning** set **can be reduced** to a basis.

### 4.6 How to actually test independence (computational toolbox)

1. **Square case — determinant.** For $n$ vectors in $\mathbb{R}^n$, form the $n\times n$ matrix with them as rows (or columns). $\det\ne 0\Rightarrow$ **independent**; $\det=0\Rightarrow$ **dependent**.
2. **General case — rank / row reduction.** Stack the vectors into a matrix and row-reduce. $\operatorname{rank}=$ number of vectors $\Rightarrow$ independent; $\operatorname{rank}<$ number of vectors $\Rightarrow$ dependent (and rank $=$ size of the largest independent subset).
3. **Homogeneous-system view.** Solve $A\mathbf{c}=\mathbf{0}$ (columns $=$ the vectors). Only trivial solution (no free variables) $\Rightarrow$ independent.
4. **Functions — Wronskian** (uncommon in DA, included for completeness). For differentiable $f_1,\dots,f_n$, if the Wronskian determinant $W(x)\ne 0$ at some point, the functions are independent. *(Caution: $W\equiv 0$ does **not** by itself prove dependence.)*

> **Equivalence chain for an $n\times n$ matrix $A$ (ties Topics 2–7 together).**
> $$\text{columns independent}\iff \det A\ne 0\iff A\text{ invertible}\iff \operatorname{rank}A=n\iff A\mathbf{x}=\mathbf{b}\text{ has a unique solution }\iff \text{columns form a basis of }\mathbb{R}^n.$$

### 4.7 Coordinates relative to a basis (brief)

Given an ordered basis $B=\{\mathbf{b}_1,\dots,\mathbf{b}_n\}$, the **unique** scalars in $\mathbf{v}=\sum c_i\mathbf{b}_i$ form the **coordinate vector** $[\mathbf{v}]_B=(c_1,\dots,c_n)$. Changing the basis re-expresses these coordinates via an invertible **change-of-basis matrix**. GATE rarely asks deep change-of-basis computations in DA; the testable nugget is simply that **coordinates relative to a basis are unique** (Section 4.3).

---

## 5. Worked problems (GATE-style)

### Problem 1 — Test independence; find rank and a dependency (NAT/MCQ)

Are $\mathbf{v}_1=(1,2,3),\ \mathbf{v}_2=(4,5,6),\ \mathbf{v}_3=(7,8,9)$ linearly independent? If not, give a dependency relation and the rank.

**Solution.** Form the matrix with these as rows and take the determinant:
$$\det\begin{pmatrix}1&2&3\\4&5&6\\7&8&9\end{pmatrix}=1(45-48)-2(36-42)+3(32-35)=-3+12-9=0.$$
Since $\det=0$, the set is **linearly dependent**. To find the relation, note $\mathbf{v}_2-\mathbf{v}_1=(3,3,3)$ and $\mathbf{v}_3-\mathbf{v}_2=(3,3,3)$, so $\mathbf{v}_3-\mathbf{v}_2=\mathbf{v}_2-\mathbf{v}_1$, giving
$$\boxed{\mathbf{v}_1-2\mathbf{v}_2+\mathbf{v}_3=\mathbf{0}.}$$
Two of the vectors (say $\mathbf{v}_1,\mathbf{v}_2$) are independent, so **rank $=2$** and the dimension of their span is $2$ (a plane through the origin).

---

### Problem 2 — Basis and dimension of a subspace (MCQ/NAT)

Find a basis and the dimension of $W=\{(a,b,c,d)\in\mathbb{R}^4 : a=d \text{ and } b=c\}$.

**Solution.** The constraints leave $a$ and $b$ free, with $c=b$ and $d=a$. A general element is
$$(a,b,b,a)=a\,(1,0,0,1)+b\,(0,1,1,0).$$
So $W=\operatorname{span}\{(1,0,0,1),(0,1,1,0)\}$. These two vectors are **independent** (neither is a scalar multiple of the other), and they span $W$ — hence they form a basis.
$$\boxed{\text{Basis}=\{(1,0,0,1),(0,1,1,0)\},\qquad \dim W=2.}$$
*(General principle: for a subspace cut out by independent homogeneous equations, $\dim W = (\text{number of variables}) - (\text{number of independent constraints}) = 4-2 = 2$ — a direct preview of rank–nullity.)*

---

### Problem 3 — Parameter-dependent dependence (the double-root trap)

For which real values of $\lambda$ are the vectors $(1,\lambda,1),\ (1,1,\lambda),\ (\lambda,1,1)$ linearly **dependent**?

**Solution.** Three vectors in $\mathbb{R}^3$ are dependent $\iff$ the determinant of the matrix formed by them is $0$:
$$\det\begin{pmatrix}1&\lambda&1\\1&1&\lambda\\\lambda&1&1\end{pmatrix}
=1(1-\lambda)-\lambda(1-\lambda^2)+1(1-\lambda)=\lambda^3-3\lambda+2.$$
Factor: $\lambda^3-3\lambda+2=(\lambda-1)(\lambda^2+\lambda-2)=(\lambda-1)^2(\lambda+2).$
Setting it to $0$:
$$\boxed{\lambda=1\ \text{(double root)}\quad\text{or}\quad\lambda=-2.}$$

**Checks (and why the trap bites):** at $\lambda=1$ all three vectors collapse to $(1,1,1)$ — obviously dependent. At $\lambda=-2$ the vectors $(1,-2,1),(1,1,-2),(-2,1,1)$ **sum to $\mathbf{0}$** — dependent. The trap is the repeated root: a hasty solver who finds $\lambda=1$ and stops misses $\lambda=-2$ entirely. For **all other** $\lambda$ the vectors are independent (and thus a basis of $\mathbb{R}^3$, by the half-test).

---

## 6. Common GATE question types on this topic

- **Test independence of given vectors** via determinant (square) or rank/row reduction (general).
- **Find the dimension / a basis of a subspace** defined by homogeneous equations or as a span — then read off $\dim$ from free variables or from the number of pivots.
- **"For what value of the parameter are these dependent?"** — set the determinant to $0$ and solve (watch for repeated roots — Problem 3).
- **Half-test questions:** *"Do these $n$ vectors form a basis of $\mathbb{R}^n$?"* — since the count equals $\dim$, just check $\det\ne 0$. Example: $\{(1,1,0),(0,1,1),(1,0,1)\}$ has $\det=2\ne0$, so it **is** a basis of $\mathbb{R}^3$.
- **Dimension of standard spaces:** $\dim P_n=n+1$, $\dim M_{m\times n}=mn$, $\dim(\text{symmetric }n\times n)=n(n+1)/2$ — pure recall, frequently the whole question.
- **Counting/inequality reasoning:** "Can $4$ vectors be independent in $\mathbb{R}^3$?" (No — more than $n$.) "Can $2$ vectors span $\mathbb{R}^3$?" (No — fewer than $n$.)
- **Uniqueness-of-coordinates** statements as true/false items (a basis $\Leftrightarrow$ unique representation).
- **Independence of polynomials/matrices/functions** — convert to coordinates in the standard basis, then it's an ordinary $\mathbb{R}^n$ rank check (Wronskian only if explicitly about functions).

---

## 7. Traps, pitfalls & common mistakes

- **Spanning $\ne$ independent.** A set can span without being independent (too many vectors) or be independent without spanning (too few). A **basis needs both** — checking only one is enough *only* when the count already equals $\dim V$ (the half-test).
- **Zero vector ⇒ automatically dependent.** Any list containing $\mathbf{0}$ is dependent, no computation needed.
- **More than $n$ ⇒ dependent; fewer than $n$ ⇒ can't span $\mathbb{R}^n$.** Use these to eliminate options instantly without arithmetic.
- **$\dim P_n=n+1$, not $n$.** The constant polynomial $1$ is a basis vector — forgetting it is the classic off-by-one error.
- **Determinant test is for square only.** For $k$ vectors in $\mathbb{R}^n$ with $k\ne n$ there is no determinant; use **rank**.
- **Repeated roots in parameter problems.** $\det=0$ may have multiple or repeated solutions — find them all (Problem 3).
- **Row vs column confusion.** You may stack the vectors as rows *or* as columns — the rank (hence the independence verdict) is identical (row rank $=$ column rank, formalized in Topic 7), but be consistent and don't mix orientations mid-calculation.
- **"$W\equiv 0$ Wronskian ⇒ dependent" is false.** A zero Wronskian does not prove dependence; only a nonzero value proves independence. (Edge case, but GATE-math has tested it.)
- **Empty set / zero space.** $\varnothing$ is the basis of $\{\mathbf{0}\}$ and $\dim\{\mathbf{0}\}=0$ — not undefined, not $1$.
- **Field matters (rarely in DA).** Independence can change between $\mathbb{R}$ and $\mathbb{C}$; GATE DA uses $\mathbb{R}$ unless stated.

---

## 8. Connection forward

Independence, basis, and dimension are the *general* tools; the next topic simply points them at the two subspaces a matrix carries. The **rank** of $A$ is defined as the dimension of its column space — equivalently, the maximum number of independent columns (which, by the bridge in §4.1, is the number of pivots). The **nullity** is the dimension of the null space $N(A)$ — the number of free variables. Topic 7's centerpiece, the **rank–nullity theorem** $\operatorname{rank}(A)+\operatorname{nullity}(A)=n$, is nothing more than the bookkeeping identity "pivot columns $+$ free columns $=$ total columns," now expressed in the precise language of dimension we built here. (Notice Problem 2 already computed a nullity in disguise.)

---

## 9. One-line cheat-sheet summary

> Vectors are **independent** iff the only combination equal to $\mathbf{0}$ is the trivial one (equivalently, no vector is a combination of the others / the matrix of columns has trivial null space); a **basis** is an independent spanning set (= minimal spanning = maximal independent), giving every vector a **unique** coordinate list; **dimension** is the basis size, and in an $n$-dimensional space *any $n$ independent vectors — or any $n$ spanning vectors — automatically form a basis.*

---

**Next topic (in order):** **Topic 7 — Rank and Nullity** (rank–nullity theorem, the four fundamental subspaces), which turns the column/null spaces and the dimension count of this topic into the single most useful pair of numbers for a matrix.

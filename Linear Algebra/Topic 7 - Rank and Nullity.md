# Topic 7 — Rank and Nullity (Rank–Nullity Theorem, Four Fundamental Subspaces)

*GATE DA · Linear Algebra · "Vector Spaces and Rank" section (Topic 3 of 3 — the capstone)*

---

## 0. Topic header & syllabus map

- **Topic:** Rank and Nullity — the **rank–nullity theorem** and the **four fundamental subspaces**.
- **Subject:** Linear Algebra (GATE Data Science & AI).
- **Where it sits:** This is **Topic 7**, the **closing** topic of the *Vector Spaces and Rank* section. It comes **after** Linear Dependence, Basis, Dimension, and Span (Topic 6) — which gave us the tools to *measure* a subspace — and applies them directly to a matrix. It is the natural endpoint of Topics 1–6 and the launch pad for the next chat, **Eigenvalues and Eigenvectors (Topic 8)**, where the central object (the eigenspace) is literally a null space.

---

## 1. Why this topic exists (motivation + connection)

Topic 6 gave us **dimension** — a single number measuring "how many independent directions" a subspace has. The obvious move is to point that ruler at the two subspaces every matrix already carries (from Topic 5): its **column space** $C(A)$ and its **null space** $N(A)$. Measuring them produces the two most useful numbers in all of matrix theory:

- **rank** $=\dim C(A)$ — how much the matrix actually "reaches" (its true output dimension), and
- **nullity** $=\dim N(A)$ — how much the matrix "collapses" (the dimension of inputs sent to zero).

The previous topics could test independence of *given* vectors but had no single statistic summarizing a whole matrix's behaviour, and Topic 3's Gaussian elimination told us a system had "$0$, $1$, or $\infty$ solutions" without a *theory* of why. Rank supplies that theory: the **rank–nullity theorem** $\operatorname{rank}(A)+\operatorname{nullity}(A)=n$ is just the bookkeeping fact "pivot columns $+$ free columns $=$ all columns," now written in the language of dimension. Rank also decides **consistency** of any linear system in one stroke (Rouché–Capelli), unifying everything from Topic 3 into a single criterion.

---

## 2. Prerequisites

- **Column space $C(A)$ and null space $N(A)$** as subspaces; "$A\mathbf{x}=\mathbf{b}$ solvable $\iff \mathbf{b}\in C(A)$" (Topic 5).
- **Independence, basis, dimension**, and the half-test; "independent columns $\iff N(A)=\{\mathbf 0\}$" (Topic 6).
- **Gaussian elimination**, pivots, free variables, row echelon form (REF/RREF) (Topic 3).
- **Determinant** and "$\det\ne0\iff$ invertible" for square matrices (Topic 2).
- The **transpose** and basic matrix multiplication (Topic 1).

---

## 3. Core idea — intuition first

Think of an $m\times n$ matrix $A$ as a machine that eats vectors from $\mathbb{R}^n$ (inputs) and outputs vectors in $\mathbb{R}^m$. Two numbers describe everything it does:

- **Rank $r$** — the dimension of the set of *reachable outputs* $C(A)$. If $A$ maps 3D inputs but every output lands on a single plane, the rank is $2$: the machine genuinely produces only a $2$-dimensional spread of results.
- **Nullity** — the dimension of the *inputs crushed to zero*, $N(A)$. These are the "wasted" input directions; along them the machine sees nothing.

Here is the conservation law that makes it all click. Each input direction must do exactly one of two jobs: either it **survives** (contributes to an output direction) or it **dies** (gets crushed to zero). It cannot do both and cannot do neither. So:

$$\underbrace{(\text{surviving directions})}_{\text{rank}}+\underbrace{(\text{crushed directions})}_{\text{nullity}}=\underbrace{(\text{all input directions})}_{n}.$$

That is the **rank–nullity theorem** in one sentence — a statement about the *inputs* ($n$ columns), not the outputs.

**The four fundamental subspaces** are simply the four natural subspaces you get from $A$ and its transpose $A^T$: two living in input space $\mathbb{R}^n$ (row space and null space) and two in output space $\mathbb{R}^m$ (column space and left null space). Strang calls their relationship the "big picture" of linear algebra because, beautifully, the two in $\mathbb{R}^n$ are **perpendicular complements** of each other, and so are the two in $\mathbb{R}^m$.

---

## 4. Formal treatment

### 4.1 Rank — six equivalent definitions

> **Definition.** For $A\in\mathbb{R}^{m\times n}$, the **rank** $r=\operatorname{rank}(A)$ is any of the following (all equal):
> 1. $\dim C(A)$ — the **column rank** (max number of independent **columns**);
> 2. $\dim C(A^T)$ — the **row rank** (max number of independent **rows**);
> 3. the number of **pivots** in any row echelon form of $A$ (= number of nonzero rows in REF);
> 4. the order of the **largest non-vanishing minor** (determinantal rank);
> 5. for $A=U\Sigma V^T$, the number of **nonzero singular values** (preview of SVD, Topic 12);
> 6. (linear-map view) $\dim(\operatorname{Im} T_A)$, the dimension of the image.

> **Fundamental fact — row rank $=$ column rank.** Definitions 1 and 2 always agree. *(GATE assumes this; it is not asked to be proved.)*

### 4.2 Nullity

> **Definition.** $\operatorname{nullity}(A)=\dim N(A)$, where $N(A)=\{\mathbf{x}\in\mathbb{R}^n:A\mathbf{x}=\mathbf{0}\}$.
> Operationally, **nullity $=$ number of free variables $=$ number of non-pivot columns** when $A$ is row-reduced.

### 4.3 The Rank–Nullity Theorem

> **Theorem (Rank–Nullity / Dimension Theorem).** For any $A\in\mathbb{R}^{m\times n}$,
> $$\boxed{\operatorname{rank}(A)+\operatorname{nullity}(A)=n}\qquad (n=\text{number of columns}).$$
> *Condition:* $n$ is the number of **columns** (the input/domain dimension) — **not** $m$. This is the single most common mistake.
>
> **Linear-map form.** For a linear map $T:V\to W$, $\dim(\operatorname{Im}T)+\dim(\ker T)=\dim V$. The matrix statement is the case $V=\mathbb{R}^n$.

*Why it's "obvious" once stated:* every column is either a **pivot column** (its direction survives — counts toward rank) or a **free column** (its variable is free — counts toward nullity). The two groups partition all $n$ columns. No derivation beyond this is expected by GATE.

### 4.4 The Four Fundamental Subspaces

For $A\in\mathbb{R}^{m\times n}$ with rank $r$:

| Subspace | Symbol | Lives in | Dimension | How to find it |
|---|---|---|---|---|
| **Column space** | $C(A)$ | $\mathbb{R}^m$ | $r$ | pivot columns **of original** $A$ |
| **Left null space** | $N(A^{T})$ | $\mathbb{R}^m$ | $m-r$ | solve $A^{T}\mathbf{y}=\mathbf{0}$ |
| **Row space** | $C(A^{T})$ | $\mathbb{R}^n$ | $r$ | nonzero rows of REF of $A$ |
| **Null space** | $N(A)$ | $\mathbb{R}^n$ | $n-r$ | solve $A\mathbf{x}=\mathbf{0}$ |

Two instances of rank–nullity sit inside this table:

- In $\mathbb{R}^n$: $\dim C(A^T)+\dim N(A)=r+(n-r)=n$.
- In $\mathbb{R}^m$: $\dim C(A)+\dim N(A^T)=r+(m-r)=m$.

> **Orthogonality (the "second half" of the fundamental theorem).** Under the standard dot product,
> $$C(A^{T})\perp N(A)\ \text{ in }\mathbb{R}^n,\qquad C(A)\perp N(A^{T})\ \text{ in }\mathbb{R}^m,$$
> and each pair are **orthogonal complements** that direct-sum to the whole space:
> $$C(A^{T})\oplus N(A)=\mathbb{R}^n,\qquad C(A)\oplus N(A^{T})=\mathbb{R}^m.$$
> *One-line reason:* if $A\mathbf{x}=\mathbf{0}$ then each **row** of $A$ dotted with $\mathbf{x}$ is $0$, so $\mathbf{x}$ is orthogonal to every row — hence to the entire row space. (The orthogonal-**projection** consequences of this are developed in Topic 10.)

### 4.5 Properties of rank — the GATE toolbox

For conformable real matrices (sizes such that the operations are defined):

| Property | Statement | Condition |
|---|---|---|
| Bounds | $0\le \operatorname{rank}(A)\le\min(m,n)$ | $A$ is $m\times n$ |
| Transpose | $\operatorname{rank}(A)=\operatorname{rank}(A^{T})$ | always |
| Gram | $\operatorname{rank}(A^{T}A)=\operatorname{rank}(AA^{T})=\operatorname{rank}(A)$ | real $A$ |
| Submultiplicative | $\operatorname{rank}(AB)\le\min(\operatorname{rank}A,\operatorname{rank}B)$ | sizes compatible |
| **Sylvester inequality** | $\operatorname{rank}(A)+\operatorname{rank}(B)-n\le\operatorname{rank}(AB)$ | $A_{m\times n},\,B_{n\times p}$ |
| Subadditive | $\operatorname{rank}(A+B)\le\operatorname{rank}(A)+\operatorname{rank}(B)$ | same size |
| Invariance | $\operatorname{rank}(PAQ)=\operatorname{rank}(A)$ | $P,Q$ invertible |
| Rank-1 | $\operatorname{rank}(\mathbf{u}\mathbf{v}^{T})=1$ | $\mathbf{u},\mathbf{v}\ne\mathbf{0}$ |

> **Full rank.** $A$ has full rank when $r=\min(m,n)$.
> - **Full column rank** ($r=n\le m$): independent columns, $N(A)=\{\mathbf 0\}$, nullity $0$, $A^{T}A$ invertible — at most one solution to $A\mathbf{x}=\mathbf{b}$.
> - **Full row rank** ($r=m\le n$): independent rows, columns span $\mathbb{R}^m$, $C(A)=\mathbb{R}^m$, $AA^{T}$ invertible — at least one solution to $A\mathbf{x}=\mathbf{b}$ for **every** $\mathbf{b}$.

> **Square-matrix equivalence chain ($n\times n$):** $\operatorname{rank}=n\iff\det\ne0\iff A^{-1}\text{ exists}\iff\operatorname{nullity}=0\iff$ unique solution to $A\mathbf{x}=\mathbf{b}\iff$ columns are a basis of $\mathbb{R}^n$.

### 4.6 Rank and solvability — Rouché–Capelli

For $A\mathbf{x}=\mathbf{b}$ with $A\in\mathbb{R}^{m\times n}$, let $[A\mid\mathbf{b}]$ be the augmented matrix.

> **Consistency.** The system has a solution $\iff \operatorname{rank}(A)=\operatorname{rank}([A\mid\mathbf{b}])$.
> Then, with $r=\operatorname{rank}(A)$:
> - $r=n\Rightarrow$ **unique** solution;
> - $r<n\Rightarrow$ **infinitely many** solutions with $(n-r)$ free parameters.
>
> **Inconsistent** $\iff \operatorname{rank}(A)<\operatorname{rank}([A\mid\mathbf{b}])$.

> **Homogeneous case $A\mathbf{x}=\mathbf{0}$:** always consistent (the trivial solution exists). A **nontrivial** solution exists $\iff r<n\iff \operatorname{nullity}>0$. The solution set is the subspace $N(A)$ of dimension $n-r$, so it has exactly $n-r$ independent solutions.

### 4.7 How to compute (the routine)

1. Row-reduce $A$ to REF. **rank $=$ number of pivots**; **nullity $=n-$ rank**.
2. For a basis of $N(A)$: assign each free variable the value $1$ (others $0$) in turn and back-substitute — this yields nullity-many independent null vectors.
3. For a basis of $C(A)$: take the **pivot columns of the original $A$** (not of the reduced form — reduction changes the column space).
4. Determinantal alternative (small matrices): rank $=$ size of the largest nonzero minor.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Rank, nullity, and a basis of the null space (NAT)

Find the rank and nullity of
$$A=\begin{pmatrix}1&2&1&3\\2&4&3&7\\3&6&4&10\end{pmatrix},$$
and give a basis for $N(A)$. Verify the rank–nullity theorem.

**Solution.** Row-reduce. $R_2\to R_2-2R_1=(0,0,1,1)$, $R_3\to R_3-3R_1=(0,0,1,1)$, then $R_3\to R_3-R_2=(0,0,0,0)$:
$$\begin{pmatrix}1&2&1&3\\0&0&1&1\\0&0&0&0\end{pmatrix}.$$
Pivots are in columns $1$ and $3$, so $\operatorname{rank}(A)=2$ and $\operatorname{nullity}(A)=4-2=2$. **Check:** $2+2=4=n$. ✔

Free variables: $x_2,x_4$. From the rows, $x_3=-x_4$ and $x_1=-2x_2-x_3-3x_4=-2x_2+x_4-3x_4=-2x_2-2x_4$.
- $x_2=1,\ x_4=0:\ (-2,1,0,0)$
- $x_2=0,\ x_4=1:\ (-2,0,-1,1)$

$$\boxed{\operatorname{rank}=2,\quad \operatorname{nullity}=2,\quad N(A)=\operatorname{span}\{(-2,1,0,0),\,(-2,0,-1,1)\}.}$$

---

### Problem 2 — Consistency via Rouché–Capelli (parametric MSQ)

For the system
$$x+y+z=6,\qquad x+2y+3z=10,\qquad x+2y+\lambda z=\mu,$$
find the conditions on $(\lambda,\mu)$ for (a) **no solution**, (b) a **unique** solution, (c) **infinitely many** solutions.

**Solution.** Row-reduce the augmented matrix. $R_2\to R_2-R_1=(0,1,2\mid 4)$ and $R_3\to R_3-R_1=(0,1,\lambda-1\mid\mu-6)$; then $R_3\to R_3-R_2=(0,0,\lambda-3\mid\mu-10)$:
$$\left[\begin{array}{ccc|c}1&1&1&6\\0&1&2&4\\0&0&\lambda-3&\mu-10\end{array}\right].$$
The bottom row reads $(\lambda-3)z=\mu-10$. Reading off the ranks:

- **(b) Unique:** if $\lambda\ne 3$, then $\operatorname{rank}(A)=\operatorname{rank}([A\mid b])=3=n$ → **unique solution** (any $\mu$).
- If $\lambda=3$, the last row is $(0,0,0\mid \mu-10)$:
  - **(a) No solution:** $\mu\ne 10\Rightarrow \operatorname{rank}(A)=2<3=\operatorname{rank}([A\mid b])$ → **inconsistent**.
  - **(c) Infinitely many:** $\mu=10\Rightarrow \operatorname{rank}(A)=\operatorname{rank}([A\mid b])=2<3$ → **infinite solutions** ($1$ free parameter).

$$\boxed{\text{No sol: }\lambda=3,\mu\ne10;\quad \text{Unique: }\lambda\ne3;\quad \text{Infinite: }\lambda=3,\mu=10.}$$

---

### Problem 3 — Rank of a product (the "full rank but singular" trap)

Let $A$ be $5\times 3$ and $B$ be $3\times 5$, **both of rank $3$**. Consider $AB$ ($5\times5$) and $BA$ ($3\times3$).
(a) Is $AB$ invertible? (b) Find $\operatorname{rank}(AB)$ and $\operatorname{nullity}(AB)$. (c) What can be said about $\operatorname{rank}(BA)$?

**Solution.**

**(a)** By submultiplicativity, $\operatorname{rank}(AB)\le\min(\operatorname{rank}A,\operatorname{rank}B)=\min(3,3)=3$. A $5\times5$ matrix with rank $\le 3<5$ is **singular**, so $AB$ is **not invertible** — even though *both factors have full rank*.

**(b)** Sylvester's inequality with inner dimension $n=3$ gives $\operatorname{rank}(AB)\ge \operatorname{rank}A+\operatorname{rank}B-3=3+3-3=3$. Combined with $\le 3$, we get $\operatorname{rank}(AB)=3$ exactly. Then by rank–nullity on $AB$ (which has $5$ columns), $\operatorname{nullity}(AB)=5-3=2$.
$$\boxed{AB\text{ singular};\quad \operatorname{rank}(AB)=3,\ \operatorname{nullity}(AB)=2.}$$

**(c)** $BA$ is $3\times3$, so $\operatorname{rank}(BA)\le 3$. Sylvester with inner dimension $5$ gives $\operatorname{rank}(BA)\ge 3+3-5=1$. Hence $1\le\operatorname{rank}(BA)\le 3$ — **not determined** by the given data; $BA$ may or may not be invertible. *(For instance, if $B=A^{T}$ with $A$ of full column rank, then $BA=A^{T}A$ is invertible with rank $3$.)*

**Lesson:** a "fat" product $AB$ of full-rank rectangular factors is forced to be rank-deficient (rank capped by the small inner dimension), while the "thin" product $BA$ is left undetermined.

---

## 6. Common GATE question types on this topic

- **Compute rank and nullity** by row reduction; report a basis of $N(A)$ or $C(A)$.
- **Consistency / number of solutions** of a parametric system via Rouché–Capelli (Problem 2 is the canonical form).
- **Rank of products and sums**: apply submultiplicativity / Sylvester / subadditivity; spot when a product *must* be singular.
- **Property MSQs from full rank.** *Example:* "$A$ is $4\times6$ with rank $4$ — which hold?" Quick answers: $A\mathbf{x}=\mathbf{b}$ solvable for **all** $\mathbf{b}$ (full row rank → $C(A)=\mathbb{R}^4$) ✔; nullity $=6-4=2$ ✔; columns **dependent** ($6>4$, or nullity $>0$) so *not* independent; $\operatorname{rank}(A^{T}A)=4$ ✔.
- **Dimensions of the four subspaces** given $m,n,r$ (pure plug-in: $r,\ n-r,\ r,\ m-r$).
- **Rank of structured matrices**: rank-$1$ outer products $\mathbf{u}\mathbf{v}^{T}$ (rank $1$); $A^{T}A$ vs $A$ (equal rank); effect of an invertible multiplier (rank unchanged).
- **Homogeneous systems**: "number of independent solutions" $=$ nullity $=n-r$; "nontrivial solution exists?" $\iff r<n$.

---

## 7. Traps, pitfalls & common mistakes

- **Rank–nullity uses $n$ (columns), not $m$ (rows).** $\operatorname{rank}+\operatorname{nullity}=n$ always refers to the domain dimension. Mixing up $m$ and $n$ is the #1 error here.
- **Column space basis: use pivot columns of the *original* matrix**, not of the reduced one — row operations change $C(A)$ (though they preserve rank and the row space).
- **Full rank of factors does not survive multiplication.** $A,B$ full rank $\not\Rightarrow AB$ invertible; the rank is capped by the smaller inner dimension (Problem 3).
- **Consistency needs the *augmented* matrix.** Compare $\operatorname{rank}(A)$ with $\operatorname{rank}([A\mid\mathbf b])$ — equal ⇒ consistent, strictly less ⇒ no solution. Don't judge from $A$ alone.
- **"Infinitely many" requires consistency first.** $r<n$ alone is not enough; you must also have $\operatorname{rank}(A)=\operatorname{rank}([A\mid\mathbf b])$. If they differ, it's *no solution*, not infinite.
- **Counting free parameters:** the solution family has $n-r$ free parameters, **not** $n-r-1$ or $m-r$. (Homogeneous solution space has dimension exactly $n-r$.)
- **Sylvester direction.** It is a *lower* bound: $\operatorname{rank}(AB)\ge\operatorname{rank}A+\operatorname{rank}B-n$. The *upper* bound is the separate submultiplicative one. Don't swap them.
- **$\operatorname{rank}(A^{T}A)=\operatorname{rank}(A)$ holds over $\mathbb{R}$** (real entries). It is *not* generally $=n$ unless $A$ has full column rank.
- **Largest-minor method:** the rank is the largest order with *some* nonzero minor; a single zero minor of that order does not lower the rank — you must check whether **any** minor of that size is nonzero.
- **Rank of the zero matrix is $0$** (and its nullity is $n$) — a boundary case GATE sometimes hides in an MSQ.

---

## 8. Connection forward

This topic closes the *Vector Spaces and Rank* arc and quietly sets up the **entire next chapter on eigenvalues** (Topic 8). The eigenvalue equation $A\mathbf{x}=\lambda\mathbf{x}$ rearranges to $(A-\lambda I)\mathbf{x}=\mathbf{0}$ — a **homogeneous system**. So:

- $\lambda$ is an **eigenvalue** $\iff (A-\lambda I)$ is singular $\iff \operatorname{rank}(A-\lambda I)<n \iff \operatorname{nullity}(A-\lambda I)\ge 1 \iff \det(A-\lambda I)=0$;
- the **eigenspace** for $\lambda$ is exactly the null space $N(A-\lambda I)$, and its dimension — the **geometric multiplicity** — is $\operatorname{nullity}(A-\lambda I)=n-\operatorname{rank}(A-\lambda I)$.

In other words, *finding eigenvectors is just computing null spaces of $A-\lambda I$*, so every technique from this topic carries straight over. Looking further in the subject, rank reappears as **the number of nonzero singular values** in SVD (Topic 12), and the orthogonality of the four subspaces underpins **orthogonal projections and least squares** (Topic 10, via $A^{T}A$).

---

## 9. One-line cheat-sheet summary

> **Rank $=\dim C(A)=$ pivots $=$ row rank $=$ column rank;** **nullity $=\dim N(A)=$ free variables**; they satisfy $\operatorname{rank}+\operatorname{nullity}=n$ (columns). The four subspaces split $\mathbb{R}^n$ into $C(A^{T})\!\oplus\!N(A)$ and $\mathbb{R}^m$ into $C(A)\!\oplus\!N(A^{T})$; a system $A\mathbf{x}=\mathbf{b}$ is solvable iff $\operatorname{rank}A=\operatorname{rank}[A\mid\mathbf b]$, with a unique solution when that rank equals $n$ and $\infty$-many (with $n-r$ free parameters) when it is less.

---

**Next topic (new chat):** **Topic 8 — Eigenvalues and Eigenvectors.** It begins by solving $(A-\lambda I)\mathbf{x}=\mathbf 0$ — the very null-space computation built here — so keep the rank/nullity machinery fresh.

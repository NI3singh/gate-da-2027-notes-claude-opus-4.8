# Determinant
*GATE DA · Linear Algebra · Notes Topic 2*

---

## 0. Topic header & syllabus map

- **Topic:** Determinant — definition, cofactor/Laplace expansion, the full property toolbox, the adjoint (adjugate) and its identities, Cramer's rule, and the singularity test.
- **Subject:** Linear Algebra.
- **Where it sits:** Second topic of *Matrices & Systems*. It **comes right after Matrices** — it is the single number we promised there to decide when a square matrix is invertible — and it **leads into Systems of Linear Equations & Gaussian Elimination**, because the row operations that leave a determinant unchanged are exactly the operations of elimination, and `det = 0` is precisely the signal that a system fails to have a unique solution.

---

## 1. Why this topic exists (motivation + connection)

In Topic 1 we defined the inverse and even wrote a formula for it, but every statement came with the fine print "**provided $\det A\neq 0$**" — a quantity we hadn't yet defined. We could *say* a square matrix is invertible iff it's nonsingular, but we had no way to **look at a matrix and decide**. The determinant fixes exactly that gap.

- **What the previous topic couldn't do:** matrix operations let you build, transpose and (sometimes) invert matrices, but give you no test for *whether* an inverse exists, and no formula for the inverse's denominator. The adjoint route $A^{-1}=\tfrac{1}{\det A}\operatorname{adj}A$ literally needs this number.
- **The new idea:** the determinant is **one scalar** $\det A$ that captures, all at once, whether $A$ is invertible (nonzero ⇔ invertible), how much $A$ scales volume, and — later — the product of its eigenvalues.
- **What it leads into:** because adding a multiple of one row to another **does not change** the determinant, elimination to triangular form gives a fast determinant (product of pivots), and `det = 0` flags the systems that have no unique solution. That is the whole launchpad for **Gaussian elimination** (next topic).

---

## 2. Prerequisites

- Matrix notation, square matrices, identity, transpose, triangular matrices (Topic 1).
- The adjoint/2×2-inverse formula previewed in Topic 1.
- Comfort expanding sums; basic factoring of polynomials (for "find the parameter" questions).

---

## 3. Core idea — intuition first

A determinant answers one geometric question: **by what factor does the linear map $A$ scale volume, and does it flip orientation?**

Picture the columns of a $2\times2$ matrix as two arrows. They span a parallelogram. **$|\det A|$ is the area of that parallelogram** (for $3\times3$, the volume of the parallelepiped spanned by the three columns). The **sign** of $\det A$ records orientation (whether the map flips space, like a mirror).

The punchline for all of linear algebra:

> If $\det A = 0$, the columns **collapse** into a lower dimension (the parallelogram is flattened to zero area). The map crushes space, so it **cannot be undone** → $A$ is **singular / not invertible**.

That's the entire intuition behind "$\det A\neq 0 \iff$ invertible." Everything else — the formulas, the adjoint, Cramer's rule — is machinery for *computing* this number and *exploiting* it.

---

## 4. Formal treatment

### 4.1 What the determinant is

The determinant is a function $\det:\ \{n\times n\ \text{matrices}\}\to \mathbb{R}$ (or $\mathbb{C}$). **Defined only for square matrices.** Notations: $\det A$ or $|A|$.

**Small cases (memorize):**

$$
[a]\ \Rightarrow\ \det=a;\qquad
\begin{vmatrix} a & b\\ c & d\end{vmatrix}=ad-bc.
$$

$$
\begin{vmatrix} a & b & c\\ d & e & f\\ g & h & i\end{vmatrix}
= a(ei-fh)-b(di-fg)+c(dh-eg)\quad\text{(Sarrus / cofactor along row 1).}
$$

**General definition (two equivalent views):**

- **Recursive (Laplace/cofactor):** expand along any row or column using minors and cofactors (Section 4.2). This is what you actually use.
- **Leibniz (for completeness, rarely needed for GATE):** $\displaystyle \det A=\sum_{\sigma}\operatorname{sgn}(\sigma)\prod_{i=1}^{n}a_{i,\sigma(i)}$, a signed sum over all $n!$ permutations $\sigma$. This is why the determinant is **expensive** ($n!$ terms) — a fact that later motivates elimination over direct expansion.

### 4.2 Minors, cofactors & Laplace expansion (the computational engine)

- **Minor** $M_{ij}$ = determinant of the $(n-1)\times(n-1)$ matrix left after deleting **row $i$ and column $j$**.
- **Cofactor** $C_{ij}=(-1)^{\,i+j}M_{ij}$. The $(-1)^{i+j}$ is the **checkerboard sign pattern**:

$$
\begin{pmatrix} + & - & +\\ - & + & -\\ + & - & +\end{pmatrix}
$$

- **Laplace expansion** — along **row $i$**, or along **column $j$** (both give the same answer):

$$
\det A=\sum_{j=1}^{n} a_{ij}\,C_{ij}\qquad\text{or}\qquad \det A=\sum_{i=1}^{n} a_{ij}\,C_{ij}.
$$

> **Practical rule:** expand along the row or column with the **most zeros** — each zero kills a whole cofactor term.

### 4.3 The property toolbox (each with its condition)

These properties are the **most tested** part of the topic. Let $A,B$ be $n\times n$.

> **P1 (Transpose):** $\det(A^{T})=\det A$. *(So every row property is also a column property.)*
>
> **P2 (Row/column swap):** swapping two rows (or two columns) **multiplies the determinant by $-1$**. $k$ swaps → factor $(-1)^k$.
>
> **P3 (Equal/proportional rows):** if two rows (or columns) are equal — or one is a scalar multiple of another — then $\det A=0$.
>
> **P4 (Zero line):** if any row or column is all zeros, $\det A=0$.
>
> **P5 (Scale one row):** multiplying a **single** row (or column) by $k$ multiplies the determinant by $k$.
>  → **Whole-matrix scaling:** $\boxed{\det(kA)=k^{\,n}\det A}$ (all $n$ rows scaled). *The exponent $n$ is a classic trap.*
>
> **P6 (Replacement — the elimination move):** adding a scalar multiple of one row to **another** row **does not change** the determinant. (Same for columns.) *This is why Gaussian elimination preserves the determinant.*
>
> **P7 (Multilinearity):** the determinant is linear in each row separately (holding the others fixed). P5 and the row-splitting rule are special cases.
>
> **P8 (Triangular/diagonal):** for an upper- or lower-triangular (hence also diagonal) matrix, $\det = $ **product of the diagonal entries**. In particular $\det I_n=1$.
>
> **P9 (Product rule):** $\boxed{\det(AB)=\det A\,\det B}$ (square, same order). Hence $\det(A^k)=(\det A)^k$.
>
> **P10 (Inverse):** if $A$ is invertible, $\det(A^{-1})=\dfrac{1}{\det A}$ (follows from P9 with $AA^{-1}=I$).

> ⚠️ **There is NO sum rule:** $\det(A+B)\neq \det A+\det B$ in general.

**Why P8 holds (short):** expand repeatedly along the first column of a lower-triangular matrix — only the $a_{11}$ term survives each time, peeling off diagonal entries one by one.

### 4.4 Singularity and the `det = 0` equivalences

A square matrix is **singular** when $\det A=0$ and **nonsingular (invertible)** when $\det A\neq 0$. The following are **all equivalent** (the determinant slot of the Invertible Matrix Theorem):

$$
\det A\neq0\iff A^{-1}\text{ exists}\iff \operatorname{rank}A=n\iff \text{rows/cols independent}\iff Ax=0\text{ has only }x=0\iff 0\text{ is not an eigenvalue.}
$$

**Two structured consequences worth memorizing:**

- **Odd-order skew-symmetric ⇒ singular** (this is the promise from Topic 1, now provable):
$$
\det A=\det(A^{T})=\det(-A)=(-1)^{n}\det A.
$$
For **odd** $n$, this says $\det A=-\det A$, so $\det A=0$.

- **Orthogonal matrix ⇒ $\det=\pm1$:** from $A^{T}A=I$ and P1/P9, $(\det A)^2=1$. *(Orthogonal matrices get a full topic later; this is the one-line determinant fact.)*

### 4.5 The adjoint (adjugate) and the inverse

Let $C=[C_{ij}]$ be the **cofactor matrix**. The **adjoint** (adjugate) is its transpose:

$$
\operatorname{adj}(A)=C^{T},\qquad (\operatorname{adj}A)_{ij}=C_{ji}.
$$

**The master identity (holds for every square $A$, singular or not):**

$$
\boxed{A\,\operatorname{adj}(A)=\operatorname{adj}(A)\,A=(\det A)\,I_n.}
$$

Taking $\det$ of both sides (and using P9, P5-whole) is the **one move that generates every adjoint formula** — you rarely need to memorize them if you can reproduce this:

$$
\det A\cdot\det(\operatorname{adj}A)=(\det A)^{n}\ \Rightarrow\ \boxed{\det(\operatorname{adj}A)=(\det A)^{\,n-1}}\quad(n\ge 2).
$$

**The full identity list** (all for $n\times n$, $n\ge 2$):

$$
A^{-1}=\frac{1}{\det A}\operatorname{adj}(A)\ \ (\det A\neq0),\qquad
\operatorname{adj}(\operatorname{adj}A)=(\det A)^{\,n-2}A,
$$
$$
\operatorname{adj}(AB)=\operatorname{adj}(B)\,\operatorname{adj}(A)\ \text{(order reverses)},\qquad
\operatorname{adj}(kA)=k^{\,n-1}\operatorname{adj}(A),
$$
$$
\operatorname{adj}(A^{T})=(\operatorname{adj}A)^{T},\qquad
\operatorname{adj}(A^{-1})=(\operatorname{adj}A)^{-1}=\frac{A}{\det A}.
$$

**For a 2×2** specifically, $\operatorname{adj}\!\begin{pmatrix}a&b\\c&d\end{pmatrix}=\begin{pmatrix}d&-b\\-c&a\end{pmatrix}$ (swap the diagonal, negate the off-diagonal).

### 4.6 Cramer's rule (the determinant-based solver)

For a **square** system $Ax=b$ with $\det A\neq 0$ (so a **unique** solution exists),

$$
x_i=\frac{\det(A_i)}{\det A},\qquad i=1,\dots,n,
$$

where $A_i$ is $A$ with its **$i$-th column replaced by $b$**. *Condition:* $\det A\neq0$. If $\det A=0$, Cramer's rule does **not** apply (the system has either no solution or infinitely many — settled in the next topic).

> **Cost note:** Cramer's rule needs $n{+}1$ determinants; computing determinants by expansion is $\sim n!$ work. For anything beyond $3\times3$ it is hopelessly slow — which is precisely why **Gaussian elimination** (next topic) is the real-world method.

### 4.7 Determinant ↔ eigenvalues, and special determinants

- **Eigenvalue link (forward):** $\det A=\prod_{i}\lambda_i$ (**product of all eigenvalues**, with multiplicity), and the eigenvalues are the roots of the **characteristic equation** $\det(A-\lambda I)=0$. *(Full treatment in the eigenvalue topic; pairs with $\operatorname{tr}A=\sum\lambda_i$ from Topic 1.)*
- **Block-triangular determinant:** if $M=\begin{pmatrix}A&B\\O&D\end{pmatrix}$ with $A,D$ square, then $\det M=\det A\cdot\det D$ (same if the *lower-left* block is $O$). *(General block determinants are studied with partition matrices later.)*
- **Vandermonde (one-line awareness):** the matrix with rows/cols of powers $x_i^{\,j}$ has $\det=\prod_{i<j}(x_j-x_i)$ — nonzero iff all $x_i$ are distinct. Occasionally appears; no derivation needed.

---

## 5. Worked problems (GATE-style)

### Problem 1 (NAT) — compute a 3×3 efficiently, two ways
Find $\det A$ for $A=\begin{pmatrix}1&2&3\\2&5&3\\1&0&8\end{pmatrix}$.

**Method A — cofactor along row 1:**
$$
\det A=1(5\cdot8-3\cdot0)-2(2\cdot8-3\cdot1)+3(2\cdot0-5\cdot1)=40-2(13)+3(-5)=40-26-15=\mathbf{-1}.
$$
**Method B — elimination to triangular (bridge to next topic):**
$R_2\to R_2-2R_1=(0,1,-3)$, $R_3\to R_3-R_1=(0,-2,5)$ (P6, det unchanged); then $R_3\to R_3+2R_2=(0,0,-1)$.
Triangular ⇒ $\det=1\cdot1\cdot(-1)=\mathbf{-1}$ ✓. Same answer, and elimination never touched the determinant because every step was a P6 "replacement" move.

### Problem 2 (MSQ/NAT) — property scaling on a 4×4
$A$ is $4\times4$ with $\det A=5$. Evaluate:
**(a)** $\det(2A)$, **(b)** $\det(A^{-1})$, **(c)** $\det(A^{T}A)$, **(d)** $\det(\operatorname{adj}A)$.

**Solution** ($n=4$):
- (a) $\det(2A)=2^{4}\det A=16\cdot5=\mathbf{80}$ — *not* $2\cdot5$ (the $k^n$ trap).
- (b) $\det(A^{-1})=1/\det A=\mathbf{1/5}$.
- (c) $\det(A^{T}A)=\det(A^{T})\det(A)=5\cdot5=\mathbf{25}$.
- (d) $\det(\operatorname{adj}A)=(\det A)^{n-1}=5^{3}=\mathbf{125}$.

### Problem 3 (NAT, trap variety) — nested adjoint
$A$ is $3\times3$ with $\det A=2$. Find $\det\big(\operatorname{adj}(\operatorname{adj}A)\big)$.

**Solution — peel one layer at a time (robust method):**
Layer 1: $\det(\operatorname{adj}A)=(\det A)^{n-1}=2^{2}=4$.
Layer 2: let $B=\operatorname{adj}A$ (a $3\times3$ matrix with $\det B=4$). Then $\det(\operatorname{adj}B)=(\det B)^{n-1}=4^{2}=\mathbf{16}$.

*Check with the closed form* $\det(\operatorname{adj}(\operatorname{adj}A))=(\det A)^{(n-1)^2}=2^{(2)^2}=2^{4}=16$ ✓. (Memorizing the closed form is optional; the layer-by-layer method never fails.)

### Problem 4 (MCQ, trap + bridge) — find the parameter that makes a matrix singular
For which real $k$ is $A=\begin{pmatrix}k&1&1\\1&k&1\\1&1&k\end{pmatrix}$ singular?

**Solution.** Use P6: $R_1\to R_1+R_2+R_3$ makes the first row $(k+2,\,k+2,\,k+2)$, so factor out $(k+2)$:
$$
\det A=(k+2)\begin{vmatrix}1&1&1\\1&k&1\\1&1&k\end{vmatrix}.
$$
Now $C_2\to C_2-C_1$ and $C_3\to C_3-C_1$ (P6) give $\begin{vmatrix}1&0&0\\1&k-1&0\\1&0&k-1\end{vmatrix}=(k-1)^2$ (triangular). Hence
$$
\det A=(k+2)(k-1)^2.
$$
Singular ⇔ $\det A=0$ ⇔ $\boxed{k=1\ \text{or}\ k=-2}$.
*Bridge:* at $k=1$ all three rows are identical (rank 1); at $k=-2$ the rows sum to zero (dependent). In both cases $Ax=0$ has nontrivial solutions — exactly the "no unique solution" situation the next topic analyzes.

---

## 6. Common GATE question types on this topic

- **Compute** a $3\times3$ or $4\times4$ determinant efficiently (cofactor along a sparse line, or reduce to triangular).
- **"If $\det A=d$, find…"** $\det(kA)$, $\det(A^{T})$, $\det(A^{-1})$, $\det(A^{k})$, $\det(\operatorname{adj}A)$ — pure property recall.
- **Adjoint identities**, especially nested $\operatorname{adj}(\operatorname{adj}A)$ and $\det(\operatorname{adj}A)$.
- **Find a parameter** making a matrix singular / a system have nontrivial or non-unique solutions.
- **Cramer's rule** to solve a small system or find one unknown.
- **Determinant via eigenvalues** ($\det=\prod\lambda_i$) — often embedded in eigenvalue problems.
- **Effect of row/column operations** on the determinant (track signs and scalings).
- **Block / triangular** determinant shortcuts.

---

## 7. Traps, pitfalls & common mistakes

- **$\det(kA)=k^{n}\det A$, not $k\det A$.** Forgetting the power $n$ is the single most common slip.
- **No sum rule:** $\det(A+B)\neq\det A+\det B$.
- **Sign on swaps:** each row (or column) swap flips the sign once; an odd number of swaps changes the sign.
- **Cofactor sign:** the factor is $(-1)^{i+j}$ (checkerboard), not the bare minor.
- **$\det(\operatorname{adj}A)=(\det A)^{n-1}$**, not $\det A$. Likewise the exponents in $\operatorname{adj}(\operatorname{adj}A)=(\det A)^{n-2}A$ are easy to misremember — rebuild from $A\,\operatorname{adj}A=(\det A)I$ if unsure.
- **Order reversal** in $\operatorname{adj}(AB)=\operatorname{adj}(B)\operatorname{adj}(A)$ (same reversal as transpose and inverse).
- **Determinant is square-only.** A rectangular matrix has no determinant.
- **Cramer's rule needs $\det A\neq0$.** If $\det A=0$ it tells you *nothing* directly (don't divide by zero).
- **Odd-order skew-symmetric is automatically singular** ($\det=0$); even-order need not be.
- **Triangular shortcut only after you've reached triangular form** — and only "replacement" (P6) moves keep the determinant; swaps and scalings must be compensated.
- **Proportional rows ⇒ $\det=0$**, not just identical rows.

---

## 8. Connection forward

The determinant hands us three things that drive the next topic. First, **P6**: since adding a multiple of one row to another never changes the determinant, we can freely reduce $A$ to triangular form — the basis of **Gaussian elimination**, with the determinant falling out as the product of pivots. Second, **`det = 0` is the alarm**: it marks exactly the systems $Ax=b$ that *lack* a unique solution, splitting them into "no solution" vs "infinitely many." Third, **Cramer's rule** gives a determinant formula for the solution but is far too slow in practice — motivating elimination as the real algorithm. All of this is **Systems of Linear Equations & Gaussian Elimination** (next).

---

## 9. One-line cheat-sheet summary

**The determinant is one scalar (square matrices only) equal to the signed volume-scaling of $A$: it is $0$ exactly when $A$ is singular/non-invertible; it satisfies $\det(A^T)=\det A$, $\det(kA)=k^n\det A$, $\det(AB)=\det A\det B$, $\det(A^{-1})=1/\det A$, $\det(\operatorname{adj}A)=(\det A)^{n-1}$; row-replacement leaves it unchanged, a swap flips its sign, and $A^{-1}=\operatorname{adj}(A)/\det A$.**

---

### ▶ Next topic (in order): **Systems of Linear Equations & Gaussian Elimination**
*(using `det = 0` to classify solutions, and row-replacement to actually solve $Ax=b$ efficiently — the practical alternative to Cramer's rule).*

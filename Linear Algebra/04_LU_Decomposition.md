# LU Decomposition
*GATE DA · Linear Algebra · Notes Topic 4*

---

## 0. Topic header & syllabus map

- **Topic:** LU decomposition — factoring a square matrix as $A=LU$ (lower × upper triangular), how Gaussian elimination *produces* it, the existence/uniqueness conditions, the $PA=LU$ form with pivoting, solving $Ax=b$ by forward/back substitution, computing $\det A$ from it, and the variants (Doolittle, Crout, LDU, **Cholesky**).
- **Subject:** Linear Algebra.
- **Where it sits:** Fourth and final topic of *Matrices & Systems*. It **comes right after Gaussian Elimination** — LU is literally elimination *recorded as a matrix factorization*. It is the **first "matrix factorization"** you meet, a theme that recurs and culminates in **SVD**. The next chat steps back from algorithms to **structure (Vector Spaces & Subspaces)**, which supplies the language — independence, basis, column/null space — explaining *why* pivots count the way they do and why factorizations behave as they do.

---

## 1. Why this topic exists (motivation + connection)

In practice you rarely solve $Ax=b$ for a single $b$. You solve it for **many right-hand sides** with the *same* $A$ — e.g. inverting a matrix (solve $AX=I$, one column of $I$ at a time), time-stepping a simulation, or re-fitting a model as data arrives.

- **What the previous topic couldn't do:** plain Gaussian elimination throws away its work. Each new $b$ forces a fresh $O(n^3)$ elimination, even though the expensive part — reducing $A$ to triangular form — depends only on $A$, not on $b$.
- **The new idea:** **separate the $A$-only work from the $b$-dependent work.** Do elimination *once* and store its result as two triangular matrices, $A=LU$. A triangular system is trivial to solve by substitution, so every later solve costs only $O(n^2)$: solve $Ly=b$, then $Ux=y$.
- **What it leads into (and across the chat boundary):** LU is the gateway to the recurring idea that *matrices are best understood through factorizations* (LU now, eigen-decomposition and **SVD** later). And the structural questions LU raises — when does it exist? why are there exactly $r$ pivots? — are answered by the **vector-space** concepts (rank, independence, basis) that open the next chat.

---

## 2. Prerequisites

- **Gaussian elimination**: forward elimination, multipliers, pivots, REF (Topic 3).
- **Triangular matrices** and **forward/back substitution**.
- From determinants (Topic 2): $\det$ of a triangular matrix $=$ product of its diagonal; $\det(AB)=\det A\det B$; effect of a row swap on $\det$ (sign $-1$).
- **Permutation matrices** $P$ (identity with rows reordered; $P^{-1}=P^{T}$, $\det P=\pm1$).
- **Positive definiteness** (used only for Cholesky) — defined fully with quadratic forms in Topic 11; we borrow the term here.

---

## 3. Core idea — intuition first

Think of LU as **"elimination, but saved."**

When you run forward elimination, two things happen: you *subtract multiples of rows* (those multipliers are a record of what you did), and you *arrive at a staircase* (upper-triangular) matrix. LU just keeps both:

- **$U$** = the upper-triangular result of forward elimination (the pivots sit on its diagonal).
- **$L$** = a lower-triangular bookkeeping matrix holding the **multipliers** you used, with $1$s on the diagonal.

Why bother? Because **triangular systems are easy**: a lower-triangular $Ly=b$ solves top-to-bottom by *forward substitution*; an upper-triangular $Ux=y$ solves bottom-to-top by *back substitution*. Splitting $A$ into two triangular factors converts one hard solve into two trivial ones — and the split is computed only once. It's the matrix analogue of factoring a number once ($360=2^3\cdot3^2\cdot5$) and reusing the factorization for many later divisions.

---

## 4. Formal treatment

### 4.1 The factorization

For a square matrix $A$ ($n\times n$), an **LU factorization** writes

$$
A = LU,
$$

with $L$ **lower triangular** and $U$ **upper triangular**. The diagonals are not fixed by this alone, so a **convention** is chosen:

- **Doolittle** (the one elimination gives, and the GATE default): $L$ has **unit diagonal** ($\ell_{ii}=1$); $U$ carries the pivots on its diagonal.
- **Crout:** $U$ has unit diagonal; $L$ carries the pivots.

$$
A=\underbrace{\begin{pmatrix}1&0&0\\ \ell_{21}&1&0\\ \ell_{31}&\ell_{32}&1\end{pmatrix}}_{L\ (\text{unit lower})}\;\underbrace{\begin{pmatrix}u_{11}&u_{12}&u_{13}\\0&u_{22}&u_{23}\\0&0&u_{33}\end{pmatrix}}_{U\ (\text{upper})}
$$

### 4.2 How Gaussian elimination produces $L$ and $U$ (the key insight)

Run forward elimination on $A$ **with no row swaps**. The result is $U$. The **multiplier** used to eliminate the entry in row $i$, column $j$ (for $i>j$) is

$$
\ell_{ij}=\frac{\text{entry being eliminated}}{\text{current pivot in column }j}.
$$

> **The "magic":** these very multipliers, placed below the diagonal, **are exactly $L$** — no separate computation. You eliminate to get $U$, and you write down each multiplier as you go; together they reconstruct $A=LU$.

### 4.3 Existence, uniqueness, and the $PA=LU$ form (honesty block)

LU **without** row swaps does not always exist — elimination can hit a **zero pivot**.

> **Existence theorem.** $A$ has an LU factorization with **unit lower-triangular** $L$ **iff** all leading principal minors $\det(A_k)$ (top-left $k\times k$ blocks) are **nonzero for $k=1,\dots,n-1$**. If, in addition, $\det(A_n)=\det A\neq0$, the factorization is **unique**.
>
> Equivalently: LU (no pivoting) succeeds **iff no zero pivot is encountered** during elimination.

When a zero pivot does appear, you **swap rows** (partial pivoting). Recording the swaps in a **permutation matrix** $P$ gives the always-available form

$$
\boxed{PA = LU.}
$$

> **Every square matrix has a $PA=LU$ factorization** (with partial pivoting). The need for $P$ signals that a *leading principal minor vanished*, not that $A$ is singular — a nonsingular $A$ can still require pivoting.

**On "how many LU factorizations":** without fixing a diagonal, there are infinitely many (a diagonal scaling can move between $L$ and $U$). Fixing $L$'s diagonal to $1$ (Doolittle) makes it **unique** for a nonsingular $A$ with the minor condition satisfied.

### 4.4 Solving $Ax=b$ with an LU factorization

Replace $Ax=b$ by $L(Ux)=b$. Let $y=Ux$. Then **two triangular solves**:

**Step 1 — forward substitution** ($Ly=b$, top to bottom):
$$
y_i=\frac{1}{\ell_{ii}}\Big(b_i-\sum_{j=1}^{i-1}\ell_{ij}\,y_j\Big)\quad\Big(\text{Doolittle: }\ell_{ii}=1\Rightarrow y_i=b_i-\sum_{j<i}\ell_{ij}y_j\Big).
$$

**Step 2 — back substitution** ($Ux=y$, bottom to top):
$$
x_i=\frac{1}{u_{ii}}\Big(y_i-\sum_{j=i+1}^{n}u_{ij}\,x_j\Big).
$$

With pivoting, solve $Ly=Pb$ then $Ux=y$ (permute $b$ the same way the rows were swapped).

### 4.5 Determinant and inverse from LU

Since $\det(AB)=\det A\det B$ and triangular determinants are products of diagonals:

$$
\det A=\det L\cdot\det U.
$$
- **Doolittle** ($\det L=1$): $\boxed{\det A=\prod_{i}u_{ii}}$ — the **product of the pivots** (matching Topics 2–3).
- **With pivoting:** $\det(PA)=\det L\det U$, and $\det P=(-1)^{(\#\text{swaps})}$, so $\det A=(-1)^{\#\text{swaps}}\prod_i u_{ii}$.

**Inverse via LU:** solve $AX=I$ one column at a time — each column $x^{(k)}$ comes from $Ly^{(k)}=e_k$, $Ux^{(k)}=y^{(k)}$. Cost $\approx O(n^3)$ overall, but the factorization is shared across all $n$ columns.

### 4.6 Variants: LDU and Cholesky

- **LDU:** factor out the pivots from $U$: $A=LDU'$ with $L,U'$ **unit** triangular and $D$ the diagonal of pivots. For a **symmetric** $A$, this becomes $A=LDL^{T}$.
- **Cholesky** (symmetric **positive definite** $A$ only): $A=LL^{T}$, with $L$ lower triangular and **positive** diagonal (or $A=R^{T}R$ with $R$ upper). It exists **and is unique iff $A$ is symmetric positive definite**, and costs about **half** of general LU. Formulas:
$$
\ell_{jj}=\sqrt{a_{jj}-\sum_{k=1}^{j-1}\ell_{jk}^{2}},\qquad
\ell_{ij}=\frac{1}{\ell_{jj}}\Big(a_{ij}-\sum_{k=1}^{j-1}\ell_{ik}\ell_{jk}\Big)\ \ (i>j).
$$
*(Positive definiteness is exactly what guarantees every square root above is of a positive number — full definition in Topic 11. Cholesky is the natural factorization for covariance matrices and least-squares normal equations.)*

### 4.7 Cost — the whole point

| Task | Cost |
|---|---|
| LU factorization (once) | $\approx \tfrac13 n^3$ |
| Each solve afterward ($Ly=b$, $Ux=y$) | $\approx n^2$ |
| $k$ right-hand sides | $\tfrac13n^3 + k\,n^2$ (vs $\tfrac13kn^3$ re-running elimination) |
| Cramer's rule (Topic 2) | $\sim n\cdot n!$ — unusable |
| Cholesky factorization (SPD) | $\approx \tfrac16 n^3$ (half of LU) |

The takeaway: **factor once, solve cheaply forever.**

---

## 5. Worked problems (GATE-style)

### Problem 1 (NAT) — compute the Doolittle LU and the determinant
Factor $A=\begin{pmatrix}2&1&1\\4&3&3\\8&7&9\end{pmatrix}=LU$ and find $\det A$.

**Solution.** Eliminate (pivot $a_{11}=2$): multipliers $\ell_{21}=\tfrac{4}{2}=2,\ \ell_{31}=\tfrac{8}{2}=4$.
$R_2\to R_2-2R_1=(0,1,1)$, $R_3\to R_3-4R_1=(0,3,5)$.
Next pivot $=1$: multiplier $\ell_{32}=\tfrac{3}{1}=3$, $R_3\to R_3-3R_2=(0,0,2)$. Hence

$$
L=\begin{pmatrix}1&0&0\\2&1&0\\4&3&1\end{pmatrix},\qquad U=\begin{pmatrix}2&1&1\\0&1&1\\0&0&2\end{pmatrix}.
$$

(Quick check: row 3 of $LU$ is $4(2,1,1)+3(0,1,1)+1(0,0,2)=(8,7,9)$ ✓.)
$$
\det A=\prod u_{ii}=2\cdot1\cdot2=\boxed{4}.
$$

### Problem 2 (NAT) — solve $Ax=b$ reusing the factorization from Problem 1
With the same $A=LU$, solve $Ax=b$ for $b=(4,10,24)^{T}$.

**Solution.** **Forward** ($Ly=b$, $\ell_{ii}=1$): $y_1=4$; $2y_1+y_2=10\Rightarrow y_2=2$; $4y_1+3y_2+y_3=24\Rightarrow16+6+y_3=24\Rightarrow y_3=2$. So $y=(4,2,2)^{T}$.
**Back** ($Ux=y$): $2x_3=2\Rightarrow x_3=1$; $x_2+x_3=2\Rightarrow x_2=1$; $2x_1+x_2+x_3=4\Rightarrow x_1=1$.
$$
\boxed{x=(1,1,1)^{T}.}
$$
Note we **never re-eliminated $A$** — exactly the payoff: a second right-hand side would again cost only the two cheap substitution sweeps.

### Problem 3 (trap, NAT) — a matrix that needs pivoting ($PA=LU$) and the sign in $\det$
Try to factor $A=\begin{pmatrix}1&2&3\\2&4&7\\3&5&8\end{pmatrix}$ as $LU$, then find $\det A$.

**Solution.** Eliminate: $\ell_{21}=2,\ \ell_{31}=3$; $R_2\to R_2-2R_1=(0,0,1)$, $R_3\to R_3-3R_1=(0,-1,-1)$. The $(2,2)$ pivot is now **$0$** — plain LU **fails**. (Indeed the 2nd leading principal minor $\det\!\begin{pmatrix}1&2\\2&4\end{pmatrix}=0$, violating the existence condition.) **Swap** $R_2\leftrightarrow R_3$:
$$
P=\begin{pmatrix}1&0&0\\0&0&1\\0&1&0\end{pmatrix},\qquad PA=\begin{pmatrix}1&2&3\\3&5&8\\2&4&7\end{pmatrix}.
$$
Now eliminate $PA$: $\ell_{21}=3,\ \ell_{31}=2$; $R_2\to R_2-3R_1=(0,-1,-1)$, $R_3\to R_3-2R_1=(0,0,1)$; pivot $(2,2)=-1$, $\ell_{32}=0$. Thus
$$
L=\begin{pmatrix}1&0&0\\3&1&0\\2&0&1\end{pmatrix},\quad U=\begin{pmatrix}1&2&3\\0&-1&-1\\0&0&1\end{pmatrix}.
$$
**Determinant:** $\det(PA)=\prod u_{ii}=1\cdot(-1)\cdot1=-1$, and $\det P=-1$ (one swap), so
$$
\det A=\frac{\det(PA)}{\det P}=\frac{-1}{-1}=\boxed{1}.
$$
(Direct check: $\det A = 1(32-35)-2(16-21)+3(10-12)=-3+10-6=1$ ✓.)

### Problem 4 (NAT) — Cholesky of a small SPD matrix
Find the Cholesky factor $L$ (so $A=LL^{T}$) of $A=\begin{pmatrix}4&2\\2&5\end{pmatrix}$.

**Solution.** $\ell_{11}=\sqrt{4}=2$; $\ell_{21}=a_{21}/\ell_{11}=2/2=1$; $\ell_{22}=\sqrt{a_{22}-\ell_{21}^{2}}=\sqrt{5-1}=2$.
$$
L=\begin{pmatrix}2&0\\1&2\end{pmatrix},\qquad LL^{T}=\begin{pmatrix}4&2\\2&1+4\end{pmatrix}=\begin{pmatrix}4&2\\2&5\end{pmatrix}\ \checkmark.
$$
($A$ is symmetric with positive pivots/eigenvalues, i.e. positive definite, so Cholesky exists and is unique.)

---

## 6. Common GATE question types on this topic

- **Compute $L$ and $U$** (Doolittle) for a $3\times3$ matrix; read off entries of $L$ or $U$.
- **Find $\det A$** from $U$ (product of pivots), including the **sign** when a permutation is used.
- **Solve $Ax=b$** given (or after computing) $A=LU$, via forward + back substitution.
- **Existence questions:** does $A$ have an LU (no pivoting)? — check leading principal minors / look for a zero pivot. "Find $PA=LU$."
- **How many LU factorizations** exist (role of the diagonal convention / uniqueness).
- **Cholesky** of a symmetric positive-definite matrix; recognizing when Cholesky applies.
- **Cost/conceptual:** why LU beats re-running elimination or Cramer's rule for multiple RHS.

---

## 7. Traps, pitfalls & common mistakes

- **Zero pivot ⇒ no plain LU.** A vanishing leading principal minor blocks $A=LU$; you need $PA=LU$. Don't force elimination through a zero pivot.
- **Pivoting is not a sign of singularity.** A perfectly nonsingular matrix can require row swaps (Problem 3 had $\det A=1$).
- **Forgetting the permutation sign in the determinant:** $\det A=(-1)^{\#\text{swaps}}\prod u_{ii}$. An odd number of swaps flips the sign.
- **Mixing conventions.** Doolittle puts $1$s on $L$'s diagonal and pivots on $U$'s; Crout is the opposite. State which you're using before reading off entries.
- **$L$'s entries are the multipliers, with the sign as used.** $\ell_{ij}=\frac{\text{eliminated entry}}{\text{pivot}}$ (positive when you *subtract* a positive multiple). A common slip is to negate them.
- **Cholesky only for symmetric positive definite $A$.** If $A$ isn't symmetric, or a diagonal term under the square root goes $\le0$, Cholesky does not exist — use LU instead.
- **Solve order matters:** $Ly=b$ **first** (forward), then $Ux=y$ (back). Reversing them is wrong.
- **With pivoting, permute $b$:** solve $Ly=Pb$, not $Ly=b$.
- **LU is for square matrices** (the standard GATE setting); rectangular generalizations are out of scope here.

---

## 8. Connection forward

LU is the **first factorization** in a story that runs through the rest of the syllabus: just as $A=LU$ splits a matrix into simple (triangular) pieces tailored to *solving systems*, later factorizations split a matrix into pieces tailored to other goals — diagonalization $A=PDP^{-1}$ for understanding repeated application, and **singular value decomposition** $A=U\Sigma V^{T}$ for understanding *any* matrix geometrically. Just as importantly, the questions LU forces us to ask — *why exactly $r$ pivots? when are rows/columns independent? what is the space the columns span?* — are the subject of the next chat. **Vector Spaces & Subspaces** (next) builds the structural vocabulary (span, independence, basis, dimension) that turns the bookkeeping of pivots and ranks into a theory.

---

## 9. One-line cheat-sheet summary

**LU records Gaussian elimination as $A=LU$ (Doolittle: unit-lower $L$ of multipliers, upper $U$ of pivots), exists without row swaps iff every leading principal minor is nonzero (else use $PA=LU$); solve $Ax=b$ by forward $Ly=b$ then back $Ux=y$, get $\det A=(-1)^{\#\text{swaps}}\prod u_{ii}$, and for symmetric positive-definite $A$ use Cholesky $A=LL^{T}$ at half the cost.**

---

### ▶ Next topic — new chat: **Vector Spaces & Subspaces**
*(the structural foundation — span, independence, basis, dimension — that explains why pivots, rank, and factorizations behave as they do).*

*This completes the **Matrices & Systems** chat (Topics 1–4: Matrices → Determinant → Gaussian Elimination → LU).*

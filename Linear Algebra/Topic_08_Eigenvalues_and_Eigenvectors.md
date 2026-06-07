# Topic 8 — Eigenvalues and Eigenvectors
*Subject: Linear Algebra · GATE Data Science & AI (DA)*

---

## 0. Topic header & syllabus map

- **Topic:** Eigenvalues and Eigenvectors
- **Subject:** Linear Algebra
- **Where it sits:** This is the **first** topic of the cluster *"Eigenvalues, special matrices, forms & SVD."* It comes **right after Rank & Nullity (Topic 7)**, which closed the "vector spaces & rank" cluster, and it **leads into Special Matrices (Topic 9)**. Almost everything left in the syllabus — special matrices, projections, quadratic forms, and SVD — is built on the language of eigenvalues. If you understand this topic well, the rest of linear algebra becomes one connected story.

---

## 1. Why this topic exists (motivation + connection)

A matrix $A$ is a **linear transformation**: feed it a vector, it spits out another vector, generally *rotated, stretched, and sheared* all at once. That mixed behaviour is hard to reason about. But for almost every transformation there exist a few **special directions** that the matrix only **scales** — it does not rotate them at all. Along such a direction $v$, the action of $A$ collapses to multiplication by a single number $\lambda$:
$$Av = \lambda v.$$
Those special directions are **eigenvectors**, and their scaling factors are **eigenvalues**. They are the "natural axes" of the transformation.

**What the previous topics couldn't do.** Rank & nullity (Topic 7) told us about the *static* anatomy of $A$ — what its column space and null space are, what collapses to zero. Solving $Ax=b$ (Topic 3) handled one fixed right-hand side. But none of them told us how $A$ **behaves dynamically** — what happens when you apply it *repeatedly*, $A^2x, A^3x, \dots$, which is exactly what shows up in ML and applied math (PCA, covariance structure, Markov chains, power iteration, stability of iterative solvers). Eigen-analysis answers this by **breaking $A$ into independent 1-D scaling actions**: along an eigenvector, applying $A$ once multiplies by $\lambda$, twice by $\lambda^2$, $k$ times by $\lambda^k$. The hard "repeated transformation" problem becomes trivial arithmetic.

**Forward.** Once we can find eigenvectors, we can often **diagonalize** $A$, which makes powers, exponentials, and classification effortless — and that machinery powers special-matrix theory, projection matrices, quadratic-form classification, and SVD.

---

## 2. Prerequisites

- **Determinants (Topic 2):** the eigenvalue condition is $\det(A-\lambda I)=0$.
- **Systems of linear equations & Gaussian elimination (Topic 3):** eigenvectors solve the homogeneous system $(A-\lambda I)v=0$.
- **Rank & nullity (Topic 7):** geometric multiplicity $= \text{nullity}(A-\lambda I)=n-\text{rank}(A-\lambda I)$.
- **Linear independence, basis, dimension, span (Topic 6):** needed for the eigenbasis and the diagonalizability test.
- **Matrix operations & trace (Topic 1):** sum of eigenvalues $=$ trace; products/powers of matrices.

---

## 3. Core idea — intuition first

Think of $A$ as a machine that takes an arrow and returns another arrow. For a *typical* input arrow, both its **length and direction** change. But for a few **special arrows**, the output points along the **same line** as the input (same direction, or exactly reversed) — only the **length** changes. Those are eigenvectors; the length-change factor is the eigenvalue (it can be positive, negative, zero, or even complex).

**Diagonal matrices wear their eigenvalues on their sleeve.** Take
$$A=\begin{bmatrix}2&0\\0&3\end{bmatrix}.$$
The $x$-axis arrow $\begin{bmatrix}1\\0\end{bmatrix}\mapsto\begin{bmatrix}2\\0\end{bmatrix}$ — stretched by **2**, no turn. The $y$-axis arrow $\begin{bmatrix}0\\1\end{bmatrix}\mapsto\begin{bmatrix}0\\3\end{bmatrix}$ — stretched by **3**, no turn. So the eigenvalues are $2,3$ and the eigenvectors lie along the axes.

**Not everything has obvious eigenvectors.** A pure rotation in the plane (say by $90^\circ$) turns *every* real arrow, so it has **no real eigenvectors** — its eigenvalues are complex ($\pm i$). A *shear* like $\begin{bmatrix}1&1\\0&1\end{bmatrix}$ has only **one** independent eigenvector even though it's $2\times2$. These "defective" cases are exactly what GATE likes to test.

**Why "scaling only" is powerful.** If $Av=\lambda v$, then $A^2v = A(\lambda v)=\lambda(Av)=\lambda^2 v$, and so on. Along an eigenvector, the messy repeated transformation $A^k$ becomes just $\lambda^k$. If we can write *every* vector as a combination of eigenvectors, then $A^k$ on *any* vector becomes easy — that is the whole point of diagonalization.

---

## 4. Formal treatment

### 4.1 Definitions

Let $A$ be a **square** $n\times n$ matrix (eigenvalues are defined only for square matrices).

- **Eigenvalue / eigenvector.** A scalar $\lambda$ is an **eigenvalue** of $A$ if there exists a **nonzero** vector $v$ with
$$Av=\lambda v,\qquad v\neq 0.$$
That $v$ is an **eigenvector** of $A$ for $\lambda$. *(The vector must be nonzero; the scalar $\lambda$ may be $0$.)*

- **Characteristic equation.** Rewrite as $(A-\lambda I)v=0$. A nonzero solution exists **iff** $A-\lambda I$ is **singular**, i.e.
$$\boxed{\det(A-\lambda I)=0.}$$
The left side, $p(\lambda)=\det(A-\lambda I)$, is the **characteristic polynomial**, of degree $n$ in $\lambda$. Its $n$ roots (real or complex, with repetition) are exactly the eigenvalues.

- **Eigenspace.** For a fixed eigenvalue $\lambda$,
$$E_\lambda=\text{Null}(A-\lambda I)=\{v : Av=\lambda v\}.$$
This is a **subspace** (it contains $0$ and is closed under linear combinations). *Note:* $0\in E_\lambda$, but $0$ is **not** called an eigenvector.

- **Spectrum:** the set of all eigenvalues. **Spectral radius** $\rho(A)=\max_i|\lambda_i|$ (largest eigenvalue in magnitude — the "dominant" eigenvalue that governs $A^k$ for large $k$).

- **Algebraic multiplicity** $\text{AM}(\lambda)$: the multiplicity of $\lambda$ as a root of $p(\lambda)$.
- **Geometric multiplicity** $\text{GM}(\lambda)=\dim E_\lambda=\text{nullity}(A-\lambda I)=n-\text{rank}(A-\lambda I)$: the number of *independent* eigenvectors for $\lambda$.

- **Fundamental inequality** (memorize — heavily tested):
$$\boxed{1\le \text{GM}(\lambda)\le \text{AM}(\lambda).}$$

### 4.2 Key properties (each with its conditions)

For an $n\times n$ matrix $A$ with eigenvalues $\lambda_1,\dots,\lambda_n$ (listed with algebraic multiplicity, **complex ones included**):

| # | Property | Condition / note |
|---|----------|------------------|
| 1 | $\sum_i \lambda_i = \text{trace}(A)$ (sum of diagonal entries) | always (count AM; include complex) |
| 2 | $\prod_i \lambda_i = \det(A)$ | always (count AM; include complex) |
| 3 | Triangular/diagonal $\Rightarrow$ eigenvalues are the **diagonal entries** | $A$ upper- or lower-triangular |
| 4 | $A$ and $A^{T}$ have the **same eigenvalues** | same char. polynomial (eigenvectors usually differ) |
| 5 | $A^k$ has eigenvalues $\lambda^k$ (same eigenvectors) | $k\in\mathbb{Z}^+$; any integer $k$ if $A$ invertible |
| 6 | $A^{-1}$ has eigenvalues $1/\lambda$ (same eigenvectors) | requires $A$ invertible (all $\lambda\neq0$) |
| 7 | $\alpha A+\beta I$ has eigenvalues $\alpha\lambda+\beta$ (same eigenvectors) | always; e.g. $A+cI \to \lambda+c$ |
| 8 | For a polynomial $q$, $q(A)$ has eigenvalues $q(\lambda)$ (same eigenvectors) | always; e.g. $A^2-3A+2I\to\lambda^2-3\lambda+2$ |
| 9 | $A$ is **singular** $\iff$ $0$ is an eigenvalue | from property 2 |
| 10 | Eigenvectors for **distinct** eigenvalues are linearly independent | always |
| 11 | **Similar** matrices $B=P^{-1}AP$ share char. poly $\Rightarrow$ same eigenvalues, same AM **and** GM | $P$ invertible (eigenvectors related by $P$) |
| 12 | Real **symmetric** $A$: eigenvalues all **real**; eigenvectors of distinct $\lambda$ are **orthogonal**; **always diagonalizable** | $A=A^T$ (spectral theorem — detailed next topic) |
| 13 | Complex eigenvalues of a **real** matrix occur in **conjugate pairs** $a\pm bi$ | $A$ real |

Two facts worth stating as their own boxes because they hide a trap (see §7):

- $\text{rank}(A)=$ (number of nonzero eigenvalues) **only if $A$ is diagonalizable.** Not true in general.
- Eigenvalues do **not** add or multiply across matrices in general: $\lambda(A+B)\neq\lambda(A)+\lambda(B)$ and $\lambda(AB)\neq\lambda(A)\lambda(B)$ in general. However, $AB$ and $BA$ have the **same nonzero eigenvalues**.

### 4.3 Where the sum/product rules come from (short derivation — GATE uses these constantly)

Factor the characteristic polynomial over its roots:
$$\det(A-\lambda I)=(\lambda_1-\lambda)(\lambda_2-\lambda)\cdots(\lambda_n-\lambda).$$
Put $\lambda=0$: $\det(A)=\lambda_1\lambda_2\cdots\lambda_n$ (Property 2). Matching the coefficient of $\lambda^{n-1}$ gives $\sum\lambda_i=\text{trace}(A)$ (Property 1).

For the **$2\times2$** case this is just:
$$\det(A-\lambda I)=\lambda^2-(\text{tr }A)\,\lambda+\det A=(\lambda-\lambda_1)(\lambda-\lambda_2),$$
$$\Rightarrow\quad \lambda_1+\lambda_2=\text{tr }A,\qquad \lambda_1\lambda_2=\det A.$$

For the **$3\times3$** case the characteristic equation (written monic) is
$$\lambda^3-(\text{tr }A)\,\lambda^2+M\,\lambda-\det A=0,$$
where $M$ is the **sum of the three principal $2\times2$ minors** of $A$ ($M=\sum_{i<j}\lambda_i\lambda_j$). *(Sign convention: $\det(A-\lambda I)$ and $\det(\lambda I-A)$ differ by $(-1)^n$ but have the same roots — use whichever is cleaner; the monic form above is convenient for reading off the invariants.)*

### 4.4 Diagonalization

$A$ is **diagonalizable** if
$$A=PDP^{-1},$$
where $D$ is **diagonal** (the eigenvalues on the diagonal) and the **columns of $P$ are the corresponding eigenvectors** (so $P$ must be invertible).

- **Diagonalizability test (exact):**
$$A\text{ is diagonalizable} \iff A\text{ has }n\text{ linearly independent eigenvectors} \iff \text{GM}(\lambda)=\text{AM}(\lambda)\ \text{for every eigenvalue }\lambda.$$
- **Sufficient (not necessary) shortcut:** $n$ **distinct** eigenvalues $\Rightarrow$ diagonalizable. The converse fails — e.g. $I$ has all equal eigenvalues yet is (already) diagonal.
- **Defective matrix:** a matrix that is **not** diagonalizable (some eigenvalue has $\text{GM}<\text{AM}$).
- **Why we care — easy powers:**
$$A^k=PD^kP^{-1},\qquad D^k=\operatorname{diag}(\lambda_1^k,\dots,\lambda_n^k).$$
Raising a diagonal matrix to a power is just raising each entry — this is how powers, matrix exponentials, and steady states get computed.

### 4.5 Cayley–Hamilton Theorem

> **Every square matrix satisfies its own characteristic equation.** If $p(\lambda)=\det(A-\lambda I)$, then $p(A)=\mathbf 0$ (the zero matrix).

*(We use the theorem; GATE never asks for its proof — so we skip the proof deliberately.)*

Two workhorse uses:

1. **Express $A^{-1}$ via powers of $A$** (when $\det A\neq0$). For $2\times2$, $p(\lambda)=\lambda^2-(\text{tr }A)\lambda+\det A$, so
$$A^2-(\text{tr }A)A+(\det A)I=0\ \Rightarrow\ A^{-1}=\frac{1}{\det A}\big((\text{tr }A)I-A\big).$$
2. **Reduce high powers** of $A$ to a low-degree polynomial in $A$ (replace $A^2$ by the relation from $p(A)=0$, then $A^3,A^4,\dots$ collapse to combinations of $A$ and $I$).

---

## 5. Worked problems (GATE-style)

### Problem 1 (NAT) — eigenvalues + transformed matrices + sanity checks
Let $A=\begin{bmatrix}4&1\\2&3\end{bmatrix}$. Find the eigenvalues and one eigenvector for each. Then state the eigenvalues of $A^{-1}$, $A^2$, and $A+2I$.

**Solution.**
$$\det(A-\lambda I)=\det\begin{bmatrix}4-\lambda&1\\2&3-\lambda\end{bmatrix}=(4-\lambda)(3-\lambda)-2=\lambda^2-7\lambda+10=(\lambda-5)(\lambda-2).$$
So $\boxed{\lambda=5,\ 2}$.

*Checks:* $\text{tr }A=4+3=7=5+2$ ✔ and $\det A=12-2=10=5\cdot2$ ✔.

Eigenvectors:
- $\lambda=5$: $(A-5I)=\begin{bmatrix}-1&1\\2&-2\end{bmatrix}\Rightarrow -x+y=0\Rightarrow v=\begin{bmatrix}1\\1\end{bmatrix}$.
- $\lambda=2$: $(A-2I)=\begin{bmatrix}2&1\\2&1\end{bmatrix}\Rightarrow 2x+y=0\Rightarrow v=\begin{bmatrix}1\\-2\end{bmatrix}$.

Transformed matrices (same eigenvectors; transform the eigenvalues):
- $A^{-1}$: $\tfrac15,\ \tfrac12$  (Property 6)
- $A^{2}$: $25,\ 4$  (Property 5)
- $A+2I$: $7,\ 4$  (Property 7)

> **Mini-variant (very common NAT):** "A $2\times2$ matrix has trace $5$ and determinant $6$; find its eigenvalues." Just solve $\lambda^2-5\lambda+6=0\Rightarrow\lambda=2,3$. No need to know the entries.

### Problem 2 (MCQ/trap) — algebraic vs geometric multiplicity & diagonalizability
Let $A=\begin{bmatrix}3&1\\0&3\end{bmatrix}$. Is $A$ diagonalizable?

**Solution.** $\det(A-\lambda I)=(3-\lambda)^2$, so $\lambda=3$ with $\text{AM}=2$.
$$A-3I=\begin{bmatrix}0&1\\0&0\end{bmatrix},\quad \text{rank}=1\ \Rightarrow\ \text{GM}=2-1=1.$$
Since $\text{GM}(3)=1<\text{AM}(3)=2$, the test fails: **$A$ is NOT diagonalizable (defective).** It has only one independent eigenvector, $v=\begin{bmatrix}1\\0\end{bmatrix}$.

*Lesson:* a repeated eigenvalue does **not** automatically give enough eigenvectors. Always check GM against AM.

### Problem 3 (MCQ) — Cayley–Hamilton for $A^{-1}$ and a power
Let $A=\begin{bmatrix}1&2\\2&1\end{bmatrix}$. Use Cayley–Hamilton to find $A^{-1}$ and $A^3$.

**Solution.** $\det(A-\lambda I)=(1-\lambda)^2-4=\lambda^2-2\lambda-3$ (eigenvalues $3,-1$).
By Cayley–Hamilton, $A^2-2A-3I=0$, i.e. $A^2=2A+3I$.

- **Inverse:** $A^2-2A=3I\Rightarrow A(A-2I)=3I\Rightarrow A^{-1}=\tfrac13(A-2I)=\tfrac13\begin{bmatrix}-1&2\\2&-1\end{bmatrix}=\begin{bmatrix}-\tfrac13&\tfrac23\\[2pt]\tfrac23&-\tfrac13\end{bmatrix}.$
 *(Check: $\det A=1-4=-3$, and $\tfrac{1}{-3}\begin{bmatrix}1&-2\\-2&1\end{bmatrix}$ gives the same matrix ✔.)*
- **Cube:** $A^3=A\cdot A^2=A(2A+3I)=2A^2+3A=2(2A+3I)+3A=7A+6I=\begin{bmatrix}13&14\\14&13\end{bmatrix}.$
 *(Check via eigenvalues: $\text{tr}=3^3+(-1)^3=26=13+13$ ✔; $\det=3^3\cdot(-1)^3=-27=13^2-14^2$ ✔.)*

### Problem 4 (NAT) — $3\times3$ characteristic polynomial + diagonalizability of a repeated eigenvalue
Let $A=\begin{bmatrix}2&1&1\\1&2&1\\1&1&2\end{bmatrix}$. Find all eigenvalues and state whether $A$ is diagonalizable.

**Solution (mechanical).** With $a=2-\lambda$,
$$\det(A-\lambda I)=\det\begin{bmatrix}a&1&1\\1&a&1\\1&1&a\end{bmatrix}=a(a^2-1)-(a-1)+(1-a)=a^3-3a+2.$$
Set $a^3-3a+2=0\Rightarrow(a-1)^2(a+2)=0\Rightarrow a=1\,(\text{double}),\,a=-2.$
Since $a=2-\lambda$: $a=1\Rightarrow\lambda=1$; $a=-2\Rightarrow\lambda=4$. So $\boxed{\lambda=1,1,4}$.

*Checks:* $\text{tr}=6=1+1+4$ ✔; $\det=4=1\cdot1\cdot4$ ✔.

**Shortcut worth recognizing:** $A=I+J$ where $J$ is the all-ones matrix. $J$ (rank 1) has eigenvalues $3,0,0$, so $A$ has $1+\{3,0,0\}=\{4,1,1\}$ — same answer instantly.

**Diagonalizable?** For $\lambda=1$ ($\text{AM}=2$): $A-I=J$ has rank $1$, so $\text{GM}=3-1=2=\text{AM}$ ✔. For $\lambda=4$, $\text{AM}=\text{GM}=1$. All match $\Rightarrow$ **diagonalizable**. (Also guaranteed because $A$ is symmetric — Property 12.) Eigenvectors: for $\lambda=1$, solve $x+y+z=0$, e.g. $\begin{bmatrix}1\\-1\\0\end{bmatrix},\begin{bmatrix}1\\0\\-1\end{bmatrix}$; for $\lambda=4$, $\begin{bmatrix}1\\1\\1\end{bmatrix}$.

### Problem 5 (NAT) — eigenvalues of a function of $A$
The eigenvalues of a $3\times3$ matrix $A$ are $1,2,3$. Find $\det(A^2+A+I)$ and $\text{trace}(A^2+A+I)$.

**Solution.** By Property 8, $A^2+A+I$ has eigenvalues $q(\lambda)=\lambda^2+\lambda+1$:
$$q(1)=3,\quad q(2)=7,\quad q(3)=13.$$
$$\det=3\cdot7\cdot13=\boxed{273},\qquad \text{trace}=3+7+13=\boxed{23}.$$
*(You never needed the actual entries of $A$ — only its eigenvalues.)*

### Problem 6 (MSQ) — conceptual / trap-spotting
Which of the following are **always true**?
(i) A real symmetric matrix has only real eigenvalues.
(ii) The rank of a square matrix equals the number of its nonzero eigenvalues.
(iii) $A$ and $A^T$ have the same eigenvalues.
(iv) If $0$ is an eigenvalue of $A$, then $A$ is singular.
(v) Eigenvectors for distinct eigenvalues are linearly independent.
(vi) Every square matrix is diagonalizable.

**Solution.**
- **(i) True** — spectral theorem (Property 12).
- **(ii) FALSE** — only true for *diagonalizable* matrices. Counterexample: $\begin{bmatrix}0&1\\0&0\end{bmatrix}$ has rank $1$ but both eigenvalues $0$ (zero nonzero eigenvalues).
- **(iii) True** — Property 4.
- **(iv) True** — Property 9.
- **(v) True** — Property 10.
- **(vi) FALSE** — defective matrices exist (Problem 2).

**Correct answers: (i), (iii), (iv), (v).**

---

## 6. Common GATE question types on this topic

- **Compute eigenvalues** from the characteristic equation ($2\times2$ and $3\times3$).
- **NAT shortcuts:** find the **sum** (trace) or **product** (det) of eigenvalues — often *without* solving for individual eigenvalues.
- **Eigenvalues of a transformed matrix:** $A^{-1}$, $A^T$, $A^k$, $\alpha A+\beta I$, and polynomials $q(A)$.
- **Find an eigenvector / a basis of an eigenspace.**
- **Algebraic vs geometric multiplicity; is $A$ diagonalizable?**
- **Cayley–Hamilton:** find $A^{-1}$, find a high power $A^k$, or evaluate a matrix polynomial.
- **Conceptual MSQ:** which listed properties hold (symmetric $\to$ real, similar matrices, $A$ vs $A^T$, rank-vs-nonzero-eigenvalues trap).
- **Partial-information puzzles:** given trace, det, and one eigenvalue, recover the rest.
- **Determinant/trace of a matrix expression** computed through eigenvalues.

---

## 7. Traps, pitfalls & common mistakes

- **$0$ is a legitimate eigenvalue.** A singular matrix has $0$ as an eigenvalue and still has eigenvectors. "Eigenvalue $0$" $\neq$ "no eigenvalue."
- **Eigenvectors are nonzero.** $0$ lives in every eigenspace but is never called an eigenvector.
- **$\text{GM}\le\text{AM}$ always**, never the reverse. To be **diagonalizable you need $\text{GM}=\text{AM}$ for *every* eigenvalue**, not just one.
- **Distinct eigenvalues $\Rightarrow$ diagonalizable, but diagonalizable does NOT require distinct eigenvalues** ($I$ is diagonalizable with all equal eigenvalues).
- **Rank $\neq$ number of nonzero eigenvalues in general.** Equality holds *only* for diagonalizable matrices. Classic trap: $\begin{bmatrix}0&1\\0&0\end{bmatrix}$ has rank $1$ but eigenvalues $0,0$.
- **$A$ and $A^T$ share eigenvalues but generally have *different eigenvectors*.**
- **Eigenvalues don't combine additively/multiplicatively:** $\lambda(A+B)\neq\lambda(A)+\lambda(B)$ and $\lambda(AB)\neq\lambda(A)\lambda(B)$ in general. (Only $AB$ and $BA$ share nonzero eigenvalues.)
- **Real matrices can have complex eigenvalues** (e.g. a rotation). Don't assume real unless $A$ is symmetric.
- **Sum/product rules need *all* eigenvalues**, counted with algebraic multiplicity and including complex ones.
- **$\text{trace}(A^k)=\sum\lambda_i^k$**, *not* $(\text{trace }A)^k$. Common slip.
- **Sign discipline:** $\det(A-\lambda I)$ vs $\det(\lambda I-A)$ differ by $(-1)^n$; the *roots* are identical, but keep your characteristic-polynomial signs consistent (and get the constant term $=\pm\det A$ right) when applying Cayley–Hamilton.

---

## 8. Connection forward

Eigen-analysis is the **lens** for the entire rest of the syllabus:

- **Special matrices (Topic 9, next):** each special class is defined by its eigenvalue/eigenvector behaviour — **orthogonal** matrices have eigenvalues on the unit circle ($|\lambda|=1$); **idempotent** matrices ($A^2=A$) have eigenvalues only $0$ or $1$; **symmetric** matrices have real eigenvalues and orthogonal eigenvectors (spectral theorem). Diagonalization is exactly the tool that makes these classifications precise.
- **Projections (Topic 10):** projection matrices are the idempotent case; their eigenvalues $\{0,1\}$ literally encode "kept vs killed" directions.
- **Quadratic forms (Topic 11):** the **signs of the eigenvalues** of a symmetric matrix classify a form as positive/negative (semi)definite or indefinite.
- **SVD (Topic 12):** generalizes eigendecomposition to **non-square** matrices via the eigenvalues of $A^TA$ and $AA^T$ (the squared singular values).

---

## 9. One-line cheat-sheet summary

**$Av=\lambda v$:** solve $\det(A-\lambda I)=0$ for $\lambda$ (sum $=$ trace, product $=$ det); eigenvectors span $\text{Null}(A-\lambda I)$; **diagonalizable $\iff$ GM $=$ AM for every $\lambda$**; Cayley–Hamilton: $A$ satisfies its own characteristic equation.

---

### ➡️ Next topic
**Topic 9 — Special Matrices** (orthogonal, idempotent, and partition/block matrices, and their properties), which uses the eigenvalue language built here to classify these matrices.

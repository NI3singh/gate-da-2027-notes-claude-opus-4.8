# Topic 10 — Projections and Projection Matrices
*Subject: Linear Algebra · GATE Data Science & AI (DA)*

---

## 0. Topic header & syllabus map

- **Topic:** Projections and Projection Matrices
- **Subject:** Linear Algebra
- **Where it sits:** Comes **right after Special Matrices (Topic 9)** and **leads into Quadratic Forms (Topic 11)**. This topic is the **geometric meaning of the idempotent matrices** you just met: a projection matrix is exactly an idempotent matrix, and the *symmetric* idempotent ones are **orthogonal projections**. It is also the mathematical heart of **least squares / linear regression**, which makes it one of the most exam- and ML-relevant topics in the whole syllabus.

---

## 1. Why this topic exists (motivation + connection)

Very often a linear system $Ax=b$ has **no exact solution** — there are more equations than unknowns (an *overdetermined*, noisy system), so $b$ does not lie in the set of reachable outputs $\text{Col}(A)$. This is exactly the situation in **data fitting / regression**: you can't pass a line *exactly* through a cloud of points. When you cannot solve $Ax=b$, the next best thing is to find the **closest reachable vector** to $b$ — the point in $\text{Col}(A)$ nearest to $b$. That nearest point is the **orthogonal projection** of $b$ onto $\text{Col}(A)$, and the matrix that produces it is the **projection matrix**.

**What the previous topic couldn't do / what this adds.** Topic 9 told us idempotent matrices have eigenvalues $0,1$ and satisfy $\text{trace}=\text{rank}$ — the *algebra*. But it never said what they **do geometrically**, nor how to **construct** the specific projection you need. This topic supplies both: the geometric picture (every vector splits into a *part inside the subspace* plus a *perpendicular part*) and the explicit construction $P=A(A^TA)^{-1}A^T$ for projecting onto **any** subspace. It also extends Topic 3 (which solved *consistent* systems $Ax=b$) to **inconsistent** systems via least squares.

**Forward.** Projections are the building blocks of the **spectral theorem** (Topic 11/12): a symmetric matrix is a weighted sum of orthogonal projections onto its eigenvectors. The least-squares idea here is the seed of the **pseudoinverse** in SVD (Topic 12).

---

## 2. Prerequisites

- **Idempotent & symmetric matrices (Topic 9):** orthogonal projection $=$ symmetric idempotent ($P^2=P$, $P^T=P$).
- **Eigenvalues (Topic 8):** projections have eigenvalues only $0,1$ and $\text{trace}=\text{rank}$.
- **Column space, null space, orthogonal complement, rank (Topic 7):** $\text{Col}(A)$, and crucially $\text{Col}(A)^{\perp}=\text{Null}(A^{T})$.
- **Matrix inverse & transpose (Topic 1):** plus when $A^TA$ is invertible.
- **Dot product / orthogonality** of vectors; solving linear systems (Topic 3) for the normal equations.

---

## 3. Core idea — intuition first

Imagine the sun **directly overhead** casting shadows onto a flat floor (the floor $=$ the subspace). A point $b$ floating in space casts a shadow $Pb$ on the floor. Two facts capture everything:

1. The shadow $Pb$ is the **closest floor point** to $b$.
2. The straight drop from $b$ to its shadow is **perpendicular to the floor**. That drop, $b-Pb$, is the **error**, and it is orthogonal to the subspace.

So an orthogonal projection **splits every vector into two perpendicular pieces**: the part lying *inside* the subspace ($Pb$) and the part *sticking out perpendicular* to it ($b-Pb$).

**Start with the simplest case — a line.** To project $b$ onto the line through a vector $a$, drop a perpendicular from $b$ to the line; the foot of that perpendicular is the projection $p$. It is some multiple $\hat x\,a$ of $a$, and you pick $\hat x$ so the leftover $b-\hat x a$ is perpendicular to $a$. Then you scale up to planes and higher-dimensional subspaces by the same "make the error perpendicular" rule.

---

## 4. Formal treatment

### 4A. Projection onto a line (1-D subspace through $a$)

Make the error $b-\hat x a$ perpendicular to $a$: $a^T(b-\hat x a)=0\Rightarrow \hat x=\dfrac{a^Tb}{a^Ta}$. Hence

$$\hat x=\frac{a^{T}b}{a^{T}a},\qquad p=\hat x\,a=\frac{a^{T}b}{a^{T}a}\,a,\qquad \boxed{P=\frac{a a^{T}}{a^{T}a}}.$$

- The error $e=b-p$ satisfies $a^Te=0$ (perpendicular to the line).
- $P$ is $n\times n$, **rank 1**, **symmetric**, **idempotent**, with $\text{trace}(P)=1$ ($=\text{rank}$).
- $a a^{T}$ is an **outer product** (a matrix); $a^{T}a$ is a **scalar** (the squared length). Don't swap them.

### 4B. Projection onto a subspace $\text{Col}(A)$

Let the columns of $A$ ($m\times n$) be a **basis** of the subspace (so $A$ has full column rank). We want $p=A\hat x\in\text{Col}(A)$ closest to $b$, i.e. error $e=b-A\hat x$ perpendicular to **every** column of $A$:
$$A^{T}(b-A\hat x)=0\quad\Longrightarrow\quad \boxed{A^{T}A\,\hat x=A^{T}b}\quad(\textbf{normal equations}).$$
If $A^TA$ is invertible,
$$\hat x=(A^{T}A)^{-1}A^{T}b,\qquad p=A\hat x=\underbrace{A(A^{T}A)^{-1}A^{T}}_{=\,P}\,b,\qquad \boxed{P=A(A^{T}A)^{-1}A^{T}}.$$

**Invertibility condition (important).** $A^TA$ is invertible $\iff$ $A$ has **linearly independent columns** (full column rank). Reason: $A^TAx=0\Rightarrow x^TA^TAx=\lVert Ax\rVert^2=0\Rightarrow Ax=0$, so $\text{Null}(A^TA)=\text{Null}(A)$; this is $\{0\}$ exactly when the columns are independent. (Also $\text{rank}(A^TA)=\text{rank}(A)$ always, and $A^TA$ is symmetric positive semidefinite.)

**Properties of $P=A(A^TA)^{-1}A^T$ (with reasons).**

| Property | Why |
|---|---|
| **Symmetric** $P^T=P$ | $A^TA$ symmetric $\Rightarrow (A^TA)^{-1}$ symmetric $\Rightarrow P^T=A(A^TA)^{-1}A^T=P$ |
| **Idempotent** $P^2=P$ | $P^2=A(A^TA)^{-1}\underbrace{(A^TA)(A^TA)^{-1}}_{I}A^T=P$ |
| $Pb\in\text{Col}(A)$, and $Pv=v$ for all $v\in\text{Col}(A)$ | $P$ acts as the identity on its own column space |
| $\text{rank}(P)=\text{rank}(A)=\dim(\text{subspace})$ | |
| $\text{trace}(P)=\text{rank}(P)=\dim(\text{subspace})$ | idempotent ⇒ trace $=$ rank (Topic 9) |
| eigenvalues are $1$ ($\times\dim$ subspace) and $0$ ($\times$ rest) | idempotent |
| **PSD**: $b^TPb=\lVert Pb\rVert^2\ge0$ | symmetric idempotent |

**Orthonormal-columns simplification.** If the columns of $A$ are **orthonormal** ($A^TA=I$), then
$$P=AA^{T}\quad(\text{no inverse needed}).$$
(Note $AA^T$ is the projection, while $A^TA=I$ is the orthonormality statement — and $AA^T\neq I$ unless $A$ is square.)

### 4C. Projection onto the orthogonal complement: $I-P$

If $P$ projects onto $S=\text{Col}(A)$, then $I-P$ projects onto $S^{\perp}=\text{Col}(A)^{\perp}=\text{Null}(A^{T})$.

- $I-P$ is also **symmetric and idempotent**; $P(I-P)=(I-P)P=0$.
- **Orthogonal decomposition:** every $b$ splits as
$$b=\underbrace{Pb}_{\in\, S}+\underbrace{(I-P)b}_{\in\, S^{\perp}},\qquad (Pb)\perp\big((I-P)b\big).$$
- The **residual/error** is $e=(I-P)b=b-p$, which lies in $\text{Null}(A^T)$, i.e. $A^Te=0$.
- **Distance** from $b$ to the subspace $S$ is $\lVert b-Pb\rVert=\lVert(I-P)b\rVert$.

### 4D. Orthogonal vs oblique projections

- A matrix $P$ with $P^2=P$ (idempotent) is **always a projection** — onto its range $\text{Col}(P)$ **along** its null space $\text{Null}(P)$, with $\mathbb R^n=\text{Col}(P)\oplus\text{Null}(P)$.
- It is an **orthogonal projection** $\iff$ additionally $P^{T}=P$. Then $\text{Null}(P)=\text{Col}(P)^{\perp}$ (the drop is perpendicular).
- A projection that is idempotent but **not** symmetric is an **oblique projection** — it still has eigenvalues $0,1$, but the error is *not* perpendicular to the target subspace.

$$\boxed{P\text{ is an orthogonal projection}\iff P^2=P\text{ and }P^T=P.}$$

### 4E. Least squares — the headline application

When $Ax=b$ has no solution, the **least-squares solution** minimizes $\lVert Ax-b\rVert^2$. Two routes give the same answer:

- **Geometric:** the best $A\hat x$ is the projection $Pb$, so the error is perpendicular to $\text{Col}(A)$ → normal equations $A^TA\hat x=A^Tb$.
- **Calculus:** $f(x)=\lVert Ax-b\rVert^2=x^TA^TAx-2b^TAx+b^Tb$, and $\nabla f=2A^TAx-2A^Tb=0$ gives the same normal equations.

$$\hat x=(A^TA)^{-1}A^Tb,\qquad \text{fitted } \hat b=A\hat x=Pb,\qquad \text{residual } e=b-\hat b=(I-P)b\perp\text{Col}(A).$$

**Regression / hat matrix (ML link).** Fitting $y=X\beta$, the least-squares estimate is $\hat\beta=(X^TX)^{-1}X^Ty$ and the fitted values are $\hat y=X\hat\beta=Hy$ where
$$H=X(X^TX)^{-1}X^T$$
is the **hat matrix** ("it puts the hat on $y$") — a projection onto the column space of $X$. The residuals are $(I-H)y$. **Where else projections appear:** Gram–Schmidt orthogonalization (repeatedly subtract the projection onto previous vectors), and the spectral decomposition (next).

### 4F. Reflection (brief, for awareness)

The matrix $R=2P-I$ **reflects** across the subspace $S$ ($R$ is orthogonal and **involutory**, $R^2=I$). A **Householder reflection** $R=I-2\dfrac{vv^T}{v^Tv}$ reflects across the hyperplane perpendicular to $v$ — used to build orthogonal matrices in QR/SVD.

---

## 5. Worked problems (GATE-style)

### Problem 1 (NAT) — project a vector onto a line; error, matrix, distance
Project $b=\begin{bmatrix}1\\2\\3\end{bmatrix}$ onto the line through $a=\begin{bmatrix}1\\1\\1\end{bmatrix}$. Find $\hat x$, $p$, the error $e$, the projection matrix $P$, and the distance from $b$ to the line.

**Solution.** $a^Tb=1+2+3=6$, $a^Ta=3$, so $\hat x=\dfrac{6}{3}=2$.
$$p=2a=\begin{bmatrix}2\\2\\2\end{bmatrix},\qquad e=b-p=\begin{bmatrix}-1\\0\\1\end{bmatrix}.$$
Check $a^Te=-1+0+1=0$ ✔ (error ⟂ line).
$$P=\frac{aa^T}{a^Ta}=\frac13\begin{bmatrix}1&1&1\\1&1&1\\1&1&1\end{bmatrix},\qquad \text{trace}=1=\text{rank}.$$
Distance $=\lVert e\rVert=\sqrt{(-1)^2+0^2+1^2}=\sqrt2$. *(Quick idempotent check: $P^2=P$ since $a^Ta=3$ cancels — consistent with §4A.)*

### Problem 2 (NAT/MCQ) — project onto a 2-D subspace; build $P$ via the normal equations
Let $A=\begin{bmatrix}1&0\\0&1\\1&1\end{bmatrix}$ (columns span a plane in $\mathbb R^3$) and $b=\begin{bmatrix}6\\0\\0\end{bmatrix}$. Find $\hat x$, the projection $p$, the projection matrix $P$, and verify the error is perpendicular to the plane.

**Solution.**
$$A^TA=\begin{bmatrix}2&1\\1&2\end{bmatrix},\quad \det=3,\quad (A^TA)^{-1}=\frac13\begin{bmatrix}2&-1\\-1&2\end{bmatrix},\quad A^Tb=\begin{bmatrix}6\\0\end{bmatrix}.$$
$$\hat x=(A^TA)^{-1}A^Tb=\frac13\begin{bmatrix}2&-1\\-1&2\end{bmatrix}\begin{bmatrix}6\\0\end{bmatrix}=\begin{bmatrix}4\\-2\end{bmatrix},\qquad p=A\hat x=\begin{bmatrix}4\\-2\\2\end{bmatrix}.$$
Error $e=b-p=\begin{bmatrix}2\\2\\-2\end{bmatrix}$; check against the columns: $e\cdot(1,0,1)=2-2=0$ ✔ and $e\cdot(0,1,1)=2-2=0$ ✔.

Projection matrix:
$$P=A(A^TA)^{-1}A^T=\frac13\begin{bmatrix}2&-1&1\\-1&2&1\\1&1&2\end{bmatrix}.$$
Sanity: $\text{trace}=\tfrac{2+2+2}{3}=2=\dim(\text{plane})$ ✔; $P$ symmetric ✔; $Pb=\tfrac13(12,-6,6)^T=(4,-2,2)^T=p$ ✔.

### Problem 3 (NAT) — least-squares best-fit line (regression)
Find the least-squares line $y=c+dx$ through the points $(1,1),(2,2),(3,2)$.

**Solution.** Model $A\beta=b$ with $A=\begin{bmatrix}1&1\\1&2\\1&3\end{bmatrix}$, $\beta=\begin{bmatrix}c\\d\end{bmatrix}$, $b=\begin{bmatrix}1\\2\\2\end{bmatrix}$.
$$A^TA=\begin{bmatrix}3&6\\6&14\end{bmatrix}\ \Big(\substack{n=3,\ \sum x=6,\\ \sum x^2=14}\Big),\qquad A^Tb=\begin{bmatrix}\sum y\\ \sum xy\end{bmatrix}=\begin{bmatrix}5\\11\end{bmatrix}.$$
Normal equations: $3c+6d=5$ and $6c+14d=11$. Solving: $2d=1\Rightarrow d=\tfrac12$, then $c=\tfrac{5-6d}{3}=\tfrac{2}{3}$.
$$\boxed{y=\tfrac23+\tfrac12 x.}$$
*Check (orthogonality of residuals):* fitted values $\tfrac76,\tfrac53,\tfrac{13}{6}$ give residuals $-\tfrac16,+\tfrac13,-\tfrac16$. Sum of residuals $=0$ and $\sum (\text{residual})\cdot x=0$ — the residual vector is perpendicular to both columns of $A$, confirming the orthogonal projection. ✔

### Problem 4 (MSQ) — concepts & traps
Which statements are **always true**?
(i) Every projection matrix is symmetric.
(ii) If $P$ is an orthogonal projection, so is $I-P$.
(iii) A projection matrix has eigenvalues only $0$ and $1$.
(iv) For an orthogonal projection $P$, $\lVert Pb\rVert\le\lVert b\rVert$ for all $b$.
(v) The projection of $b$ onto $\text{Col}(A)$ equals $b$ iff $b\in\text{Col}(A)$.
(vi) For any projection matrix, $\det(P)=1$.

**Solution.**
- **(i) FALSE** — only *orthogonal* projections are symmetric; *oblique* projections are idempotent but not symmetric.
- **(ii) TRUE** — $(I-P)^2=I-P$ and $(I-P)^T=I-P$; it projects onto $\text{Col}(A)^{\perp}$.
- **(iii) TRUE** — projections are idempotent.
- **(iv) TRUE** — $\lVert b\rVert^2=\lVert Pb\rVert^2+\lVert(I-P)b\rVert^2\ge\lVert Pb\rVert^2$ (orthogonal split). Equality iff $b\in\text{Col}(A)$.
- **(v) TRUE** — $P$ fixes its column space and only its column space.
- **(vi) FALSE** — $\det(P)=0$ for any projection onto a *proper* subspace (it has a $0$ eigenvalue); $\det=1$ only for $P=I$.

**Correct: (ii), (iii), (iv), (v).**

---

## 6. Common GATE question types on this topic

- **Project a vector** onto a given line or subspace; report $\hat x$ and $p$.
- **Build the projection matrix** $P=\frac{aa^T}{a^Ta}$ (line) or $A(A^TA)^{-1}A^T$ (subspace); simplify to $AA^T$ for orthonormal columns.
- **Shortest distance** from a point to a line/plane $=\lVert(I-P)b\rVert$.
- **Least-squares / best-fit line**, or "find the best approximate solution of an inconsistent $Ax=b$" via the **normal equations** $A^TA\hat x=A^Tb$.
- **Property checks:** $P$ symmetric & idempotent, eigenvalues $0/1$, $\text{trace}=\text{rank}=\dim(\text{subspace})$.
- **$I-P$** projects onto the orthogonal complement; orthogonal decomposition $b=Pb+(I-P)b$.
- **Orthogonal vs oblique** identification; **hat matrix** / residuals in regression.

---

## 7. Traps, pitfalls & common mistakes

- **Symmetry is not automatic.** Only *orthogonal* projections satisfy $P^T=P$. An idempotent non-symmetric matrix is a valid (oblique) projection.
- **$A^TA$ invertible needs full *column* rank.** If the columns of $A$ are dependent, $A^TA$ is singular and you need the **pseudoinverse** (SVD, Topic 12); $\hat x$ is then not unique.
- **Don't confuse $\hat x$ with $p$.** $\hat x=(A^TA)^{-1}A^Tb$ is the **coefficient vector** (in $\mathbb R^n$); $p=A\hat x$ is the **projection** (in $\mathbb R^m$).
- **Outer vs inner product:** in $\frac{aa^T}{a^Ta}$, the numerator $aa^T$ is a **matrix**, the denominator $a^Ta$ is a **scalar**. Swapping them is a classic slip.
- **Operator order:** it is $A(A^TA)^{-1}A^T$ — keep the order; $(A^TA)^{-1}$ sits in the middle.
- **$AA^T\neq I$** in general for orthonormal columns; $A^TA=I$ is the orthonormality, while $AA^T=P$ is the projection.
- **$\det(P)=0$** for any projection onto a proper subspace — projections (other than $I$) are **singular**, hence not invertible.
- **The residual is perpendicular to the subspace**, $A^Te=0$; the error lives in the **left** null space $\text{Null}(A^T)=\text{Col}(A)^{\perp}$ — *not* $\text{Null}(A)$ in general.
- **Projection never lengthens:** $\lVert Pb\rVert\le\lVert b\rVert$.
- **Projecting twice does nothing new** ($P^2=P$); applying $P$ to a vector already in the subspace returns it unchanged.

---

## 8. Connection forward

- **Quadratic forms (Topic 11, next):** a real symmetric matrix has the **spectral decomposition**
$$A=\sum_i \lambda_i\,q_iq_i^{T}=\sum_i\lambda_i P_i,\qquad P_i=q_iq_i^T,$$
where each $P_i$ is the **orthogonal projection onto the $i$-th eigenvector**. So a symmetric matrix is a *weighted sum of orthogonal projections* — projections are the atoms of the spectral theorem, which classifies quadratic forms by the **signs of $\lambda_i$**.
- **SVD (Topic 12):** the **pseudoinverse** $A^{+}$ generalizes least squares to rank-deficient and non-square $A$; the products $AA^{+}$ and $A^{+}A$ are the orthogonal projections onto the column space and row space. The projection-based least squares here is precisely the special (full-rank) case.

---

## 9. One-line cheat-sheet summary

**Onto a line $a$:** $P=\dfrac{aa^T}{a^Ta}$, $p=\dfrac{a^Tb}{a^Ta}\,a$. **Onto $\text{Col}(A)$:** $P=A(A^TA)^{-1}A^T$, $\hat x=(A^TA)^{-1}A^Tb$ (needs $A$ full column rank; $=AA^T$ if columns orthonormal). $P$ is **symmetric + idempotent**, eigenvalues $\{0,1\}$, $\text{trace}=\text{rank}=\dim$; $I-P$ projects onto $\text{Col}(A)^{\perp}$; residual ⟂ subspace; **least squares $=$ orthogonal projection**.

---

### ➡️ Next topic
**Topic 11 — Quadratic Forms** — expressions $x^TAx$ with $A$ symmetric: definiteness (positive/negative (semi)definite, indefinite) read off the eigenvalue signs, diagonalization by an orthogonal matrix (principal-axis theorem), and tests like leading principal minors. Projections reappear there as the spectral building blocks.

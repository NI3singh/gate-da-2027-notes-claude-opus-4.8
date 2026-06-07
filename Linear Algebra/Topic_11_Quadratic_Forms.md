# Topic 11 — Quadratic Forms
*Subject: Linear Algebra · GATE Data Science & AI (DA)*

---

## 0. Topic header & syllabus map

- **Topic:** Quadratic Forms
- **Subject:** Linear Algebra
- **Where it sits:** Comes **right after Projections & Projection Matrices (Topic 10)** and is the **last topic before Singular Value Decomposition (Topic 12)**. It runs entirely on the **spectral theorem** for symmetric matrices (Topic 9: symmetric ⇒ real eigenvalues, orthogonal eigenvectors, orthogonally diagonalizable), and the **projection** atoms from Topic 10 reappear here as the building blocks of that decomposition. It is also the gateway to SVD, because SVD is the eigen-story of the symmetric **PSD** matrices $A^TA$ and $AA^T$.

---

## 1. Why this topic exists (motivation + connection)

A huge number of quantities in ML and optimization are **scalar functions that are quadratic in a vector**: the variance $x^T\Sigma x$ of a covariance matrix $\Sigma$, the "energy" of a system, the squared-error/loss surface near a minimum, the second-order term in a Taylor expansion. For any such quantity the decisive question is its **sign behaviour**: is the function always positive (a bowl with a unique minimum — i.e. *convex*), always negative (a dome), or does it go both ways (a saddle)? **Quadratic forms and definiteness answer exactly this.**

**What the previous topic couldn't do / what this adds.** Topic 10 *projected* vectors and minimized $\lVert Ax-b\rVert^2$, but it quietly assumed the geometry was well-behaved — it never asked *when* a quadratic objective even has a minimum, or how to classify the shape of a quadratic surface. Topics 8–9 produced eigenvalues but never tied their **signs** to the behaviour of a scalar function. Quadratic forms make that link precise: the **signs of the eigenvalues** of the (symmetric) matrix decide whether $x^TAx$ is a bowl, a dome, or a saddle.

**Forward.** SVD (Topic 12) is built on the fact that $A^TA$ is **symmetric positive semidefinite** for *every* matrix $A$ — its eigenvalues are all $\ge 0$ and are the **squared singular values**. So the PSD slice of quadratic-form theory is precisely what makes SVD work.

---

## 2. Prerequisites

- **Eigenvalues & eigenvectors (Topic 8):** real eigenvalues, diagonalization, $\text{trace}=\sum\lambda_i$, $\det=\prod\lambda_i$.
- **Special matrices (Topic 9):** the **spectral theorem** — a real symmetric matrix satisfies $A=Q\Lambda Q^{T}$ with $Q$ orthogonal and $\Lambda$ real diagonal; orthogonal matrices preserve length/angle.
- **Projections (Topic 10):** the spectral decomposition $A=\sum\lambda_i q_iq_i^{T}$ as a weighted sum of orthogonal projections.
- **Determinant (Topic 2):** for principal minors.
- **Matrix multiplication, transpose, trace (Topic 1).**

---

## 3. Core idea — intuition first

A quadratic form $x^TAx$ is just a **weighted sum of products of coordinates** — the multivariable generalization of $ax^2$. In one variable, $ax^2$ is a parabola: it opens **up** if $a>0$ (minimum at $0$), **down** if $a<0$ (maximum at $0$). In $n$ variables the question "does it open up, down, or both?" is answered by the **eigenvalues**.

The magic of a symmetric $A$ is that you can **rotate the coordinate axes** (an orthogonal change of variables) so that *all the messy cross terms vanish* and the form becomes a clean weighted sum of squares:
$$x^TAx \;\xrightarrow{\text{rotate}}\; \lambda_1 y_1^2+\lambda_2 y_2^2+\dots+\lambda_n y_n^2.$$
In these "principal-axis" coordinates each $\lambda_i$ is simply the **curvature along that axis**. So: all $\lambda_i>0$ → a **bowl** (positive definite); all $<0$ → a **dome** (negative definite); mixed signs → a **saddle** (indefinite); some zero → a **flat valley** (semidefinite). Picture the form as a landscape and the eigenvalues as the up/down curvature in each principal direction.

---

## 4. Formal treatment

### 4A. Definition and the symmetric matrix

A **quadratic form** in $n$ variables is
$$Q(x)=x^{T}A x=\sum_{i=1}^{n}\sum_{j=1}^{n}a_{ij}\,x_ix_j .$$
Since $x^TAx=x^TA^Tx=x^T\big(\tfrac{A+A^T}{2}\big)x$, only the **symmetric part** matters — so **we always take $A$ symmetric.**

**Building $A$ from the polynomial (memorize):**
$$a_{ii}=\text{coefficient of }x_i^2,\qquad a_{ij}=a_{ji}=\tfrac12\,(\text{coefficient of }x_ix_j)\ \ (i\neq j).$$

*Example.* $Q=5x^2+4xy+5y^2\Rightarrow A=\begin{bmatrix}5&2\\2&5\end{bmatrix}$ (diagonal $5,5$; the $4xy$ splits as $2+2$). Check: $x^TAx=5x^2+2xy+2yx+5y^2=5x^2+4xy+5y^2$ ✔.

### 4B. Diagonalization — the Principal-Axis Theorem

Because $A$ is symmetric, the spectral theorem gives $A=Q\Lambda Q^{T}$ with $Q$ orthogonal ($Q^TQ=I$), $\Lambda=\operatorname{diag}(\lambda_1,\dots,\lambda_n)$, all $\lambda_i\in\mathbb R$. Substituting the **orthogonal change of variables** $y=Q^Tx$ (a rotation, so $\lVert y\rVert=\lVert x\rVert$):
$$Q(x)=x^TQ\Lambda Q^Tx=(Q^Tx)^T\Lambda(Q^Tx)=y^T\Lambda y=\boxed{\lambda_1 y_1^2+\cdots+\lambda_n y_n^2.}$$
This is the **principal-axis (canonical) form**: every quadratic form reduces to a pure sum of squares with the **eigenvalues as coefficients**, via an orthogonal (length-preserving) transformation. The principal axes are the **eigenvector directions**.

**Spectral / projection view (ties to Topic 10).** With orthonormal eigenvectors $q_i$,
$$A=\sum_i\lambda_i\,q_iq_i^{T}=\sum_i\lambda_i P_i,\qquad Q(x)=\sum_i\lambda_i\,(q_i^Tx)^2,$$
where each $P_i=q_iq_i^T$ is the **orthogonal projection onto eigenvector $q_i$**. A symmetric matrix is a *weighted sum of orthogonal projections*, weights $=$ eigenvalues.

### 4C. Definiteness classification (eigenvalue criterion)

For symmetric $A$ with (real) eigenvalues $\lambda_i$:

| Class | $Q(x)=x^TAx$ for all $x\neq0$ | Eigenvalue condition |
|---|---|---|
| **Positive definite (PD)** | $>0$ | all $\lambda_i>0$ |
| **Positive semidefinite (PSD)** | $\ge0$ | all $\lambda_i\ge0$ (at least one $=0$) |
| **Negative definite (ND)** | $<0$ | all $\lambda_i<0$ |
| **Negative semidefinite (NSD)** | $\le0$ | all $\lambda_i\le0$ (at least one $=0$) |
| **Indefinite** | both $+$ and $-$ values | eigenvalues of **both** signs |

Useful corollaries (each with its condition):
- $A$ **PD** $\Rightarrow$ invertible ($\det=\prod\lambda_i>0$) and $A^{-1}$ is also PD. **PSD does *not* imply invertible** (a $0$ eigenvalue is allowed).
- $A$ **PD** $\iff$ $A=B^TB$ for some invertible $B$ (Cholesky). More generally, **$A^TA$ is PSD for any $A$, and PD $\iff$ $A$ has full column rank** (links to normal equations, Topic 10, and SVD, Topic 12).
- The diagonal entries of a PD matrix are all positive ($a_{ii}=e_i^TAe_i>0$) — **necessary but not sufficient**.

### 4D. Practical tests

**(1) Eigenvalue test** — the primary, always-valid test (table above).

**(2) Leading principal minors (Sylvester's criterion).** Let $D_k=\det(\text{top-left }k\times k\text{ block})$.
$$\boxed{A\text{ PD}\iff D_1>0,\,D_2>0,\dots,D_n>0.}$$
$$A\text{ ND}\iff (-1)^kD_k>0\ \forall k,\ \text{i.e. signs alternate starting negative: }D_1<0,\,D_2>0,\,D_3<0,\dots\ (\Leftrightarrow -A\text{ PD}).$$
> **Crucial limitation:** the *leading*-minor test certifies **only PD/ND**. For **semidefinite** you must check **all** principal minors (not just the leading ones): $A$ is **PSD $\iff$ every principal minor $\ge0$**. (See the trap in §7.)

**(3) $2\times2$ trace–determinant shortcut** (gold for GATE). For symmetric $2\times2$ ($\text{tr}=\lambda_1+\lambda_2$, $\det=\lambda_1\lambda_2$):
$$\det>0,\ \text{tr}>0\Rightarrow\text{PD};\quad \det>0,\ \text{tr}<0\Rightarrow\text{ND};\quad \det<0\Rightarrow\text{indefinite};\quad \det=0\Rightarrow\text{semidefinite (sign by tr)}.$$

**(4) Pivot signs.** In symmetric elimination ($A=LDL^T$), the number of **positive pivots equals the number of positive eigenvalues** (by inertia, below); all pivots positive $\iff$ PD.

### 4E. Rank, index, signature — Sylvester's Law of Inertia

- **Rank** $r$ = number of nonzero eigenvalues = $\text{rank}(A)$.
- **Index (positive index of inertia)** $p$ = number of **positive** eigenvalues; let $q$ = number of negative eigenvalues ($r=p+q$).
- **Signature** $= p-q$ (sometimes reported as the pair $(p,q)$).
- **Law of inertia:** the triple $(p,q,\text{zeros})$ is **invariant under any congruence** $x=Py$ ($P$ invertible). So however you diagonalize — orthogonally *or* by completing the square — you always get the **same counts** of $+$, $-$, $0$ terms. A **canonical form** by congruence is $\sum_{i=1}^{p}y_i^2-\sum_{j=1}^{q}y_j^2$ ($p$ plus-ones, $q$ minus-ones, $n-r$ zeros).

### 4F. Completing the square (Lagrange's method) — an alternative to eigenvalues

You can diagonalize a form by **completing squares** instead of finding eigenvalues; the **signs** of the resulting squared-term coefficients give the signature directly (matching the eigenvalue signs by the law of inertia — though the *values* differ from the eigenvalues). Handy when eigenvalues are ugly.

### 4G. Optimization connection (the Hessian) — ML relevance

At a critical point ($\nabla f=0$) of a function $f$, the **Hessian** $H=[\partial^2 f/\partial x_i\partial x_j]$ (symmetric) classifies it:
$$H\text{ PD}\Rightarrow\text{local minimum},\quad H\text{ ND}\Rightarrow\text{local maximum},\quad H\text{ indefinite}\Rightarrow\text{saddle},\quad H\text{ semidefinite}\Rightarrow\text{inconclusive}.$$
A function is **convex $\iff$ its Hessian is PSD everywhere** — the backbone of convex optimization in ML.

---

## 5. Worked problems (GATE-style)

### Problem 1 (NAT/MCQ) — build $A$, classify, reduce to canonical form
For $Q(x,y)=5x^2+4xy+5y^2$: write $A$, classify it, and give the canonical (principal-axis) form.

**Solution.** $A=\begin{bmatrix}5&2\\2&5\end{bmatrix}$. $\text{tr}=10>0$, $\det=25-4=21>0$ → **positive definite** (and leading minors $D_1=5>0$, $D_2=21>0$ confirm). Eigenvalues: $\lambda^2-10\lambda+21=(\lambda-3)(\lambda-7)\Rightarrow\lambda=3,7$. Canonical form:
$$Q=3u^2+7v^2\quad(\text{both coefficients }>0\Rightarrow\text{PD}).$$

### Problem 2 (MCQ) — $3\times3$ positive-definiteness via Sylvester
Is $A=\begin{bmatrix}2&-1&0\\-1&2&-1\\0&-1&2\end{bmatrix}$ positive definite?

**Solution.** Leading principal minors:
$$D_1=2>0,\quad D_2=\det\begin{bmatrix}2&-1\\-1&2\end{bmatrix}=3>0,$$
$$D_3=\det A=2(4-1)-(-1)\big((-1)(2)-0\big)+0=6-2=4>0.$$
All three positive $\Rightarrow$ **positive definite**. (Indeed its eigenvalues are $2-\sqrt2,\,2,\,2+\sqrt2$, all $>0$.)

### Problem 3 (NAT) — parameter range for positive definiteness
For which real $k$ is $A=\begin{bmatrix}1&k\\k&4\end{bmatrix}$ positive definite?

**Solution.** $D_1=1>0$ (always); $D_2=4-k^2>0\iff k^2<4$. So $A$ is PD $\iff \boxed{-2<k<2}$. *(Eigenvalue view: $\text{tr}=5>0$ always; $\det=4-k^2>0\iff|k|<2$.)*

### Problem 4 (NAT) — signature via completing the square + law of inertia
Reduce $Q(x_1,x_2,x_3)=x_1^2+x_2^2+x_3^2+2x_1x_2$ to a sum of squares and state its rank, index, and signature.

**Solution.** Complete the square: $x_1^2+2x_1x_2+x_2^2=(x_1+x_2)^2$, so
$$Q=(x_1+x_2)^2+x_3^2.$$
Two squared terms, both with coefficient $+$, and one missing variable. Hence **rank $=2$**, **index $p=2$**, **$q=0$**, **signature $=2$**, one zero — so $Q$ is **PSD (not PD)**.
*Cross-check via eigenvalues:* $A=\begin{bmatrix}1&1&0\\1&1&0\\0&0&1\end{bmatrix}$ has eigenvalues $\{2,1,0\}$ — also $2$ positive, $0$ negative, $1$ zero. The *coefficients* differ ($1,1$ from completing the square vs $2,1$ from eigenvalues), but the **sign counts agree** — Sylvester's law of inertia in action. ✔

### Problem 5 (MCQ) — classify a critical point with the Hessian
Classify the stationary point at the origin of $f(x,y)=x^2+4xy+y^2$.

**Solution.** $\nabla f=(2x+4y,\,4x+2y)=0\Rightarrow x=y=0$ (origin is the only critical point). Hessian
$$H=\begin{bmatrix}f_{xx}&f_{xy}\\f_{yx}&f_{yy}\end{bmatrix}=\begin{bmatrix}2&4\\4&2\end{bmatrix},\qquad \det H=4-16=-12<0\ \Rightarrow\ \text{indefinite}.$$
So the origin is a **saddle point**. *(Remark: if instead $\det H>0$ with $f_{xx}>0$ it would be a local minimum (PD Hessian); $\det H>0$ with $f_{xx}<0$ a local maximum (ND).)*

### Problem 6 (MSQ) — concepts & traps
Which statements are **always true**?
(i) If all *leading* principal minors of a symmetric matrix are $\ge0$, it is PSD.
(ii) A positive definite matrix is invertible.
(iii) For any real matrix $A$, $A^TA$ is positive semidefinite.
(iv) $x^TAx=x^T\big(\tfrac{A+A^T}{2}\big)x$ for every square $A$.
(v) A symmetric $2\times2$ matrix with $\det>0$ and $\text{trace}>0$ is positive definite.
(vi) A positive definite matrix can have a negative diagonal entry.

**Solution.**
- **(i) FALSE** — leading minors are not enough for PSD; need **all** principal minors. Counterexample $\begin{bmatrix}0&0\\0&-1\end{bmatrix}$: leading minors $0,0\ (\ge0)$, yet eigenvalue $-1<0$ (it is NSD, not PSD).
- **(ii) TRUE** — $\det=\prod\lambda_i>0$.
- **(iii) TRUE** — $x^T(A^TA)x=\lVert Ax\rVert^2\ge0$.
- **(iv) TRUE** — only the symmetric part contributes.
- **(v) TRUE** — the $2\times2$ shortcut.
- **(vi) FALSE** — $a_{ii}=e_i^TAe_i>0$ for PD.

**Correct: (ii), (iii), (iv), (v).**

---

## 6. Common GATE question types on this topic

- **Write the symmetric matrix** of a given quadratic form (halve the cross terms).
- **Classify definiteness** (PD/PSD/ND/NSD/indefinite) via eigenvalues, leading principal minors, or the $2\times2$ trace–det shortcut.
- **Parameter range** for which a form is PD/PSD.
- **Reduce to canonical/diagonal form** (orthogonal *or* by congruence); find **rank, index, signature**.
- **Completing the square** (Lagrange's method).
- **Classify a critical point with the Hessian** (min / max / saddle); convexity.
- **Conceptual:** $A^TA$ is PSD; only the symmetric part matters; PD ⇒ invertible.
- **Level-set shape** (ellipse for PD/ND, hyperbola for indefinite).

---

## 7. Traps, pitfalls & common mistakes

- **Halving the cross terms:** the off-diagonal entry is $a_{ij}=\tfrac12(\text{coeff of }x_ix_j)$, *not* the full coefficient.
- **Leading-minor test is only for PD/ND — never use it to certify semidefiniteness.** For PSD you need **all** principal minors $\ge0$. (Counterexample $\begin{bmatrix}0&0\\0&-1\end{bmatrix}$.)
- **ND sign pattern** alternates starting **negative**: $(-1)^kD_k>0$. Easy to misremember.
- **The biggest $n\ge3$ trap:** $\det A>0$ does **not** imply PD for $3\times3$ or larger. Example: eigenvalues $-1,-1,1$ give $\det=1>0$ but the matrix is **indefinite** (two negatives). For $n\ge3$ you must check **all** leading minors $D_1,\dots,D_n>0$ — a single positive determinant is insufficient. (The "$\det<0\Rightarrow$ indefinite, $\det>0\Rightarrow$ definite" reasoning works *only* for $2\times2$.)
- **PSD vs PD boundary:** "$\ge0$" eigenvalues is PSD; "$>0$" is PD. A single zero eigenvalue moves you from PD to PSD (and loses invertibility).
- **Positive diagonal is necessary, not sufficient** for PD. $\begin{bmatrix}1&3\\3&1\end{bmatrix}$ has positive diagonal but $\det=1-9<0$ → indefinite.
- **Definiteness is a property of symmetric matrices** (or the symmetric part). For non-symmetric $A$, $x^TAx$ uses only $\tfrac{A+A^T}{2}$.
- **Canonical-form coefficients from completing the square are *not* the eigenvalues** — only the sign counts (inertia) are invariant.
- **PD ⇒ invertible** (and $A^{-1}$ PD), but **PSD does not** (zero eigenvalue allowed).

---

## 8. Connection forward

- **Singular Value Decomposition (Topic 12, next):** for **any** matrix $A$ (rectangular included), both $A^TA$ and $AA^T$ are **symmetric PSD**. Their nonnegative eigenvalues are the **squared singular values** $\sigma_i^2$; the orthonormal eigenvectors of $A^TA$ are the **right** singular vectors and those of $AA^T$ the **left** singular vectors. Thus SVD is essentially the *eigen-/principal-axis analysis of the PSD quadratic forms $A^TA$, $AA^T$* — the orthogonal diagonalization you mastered here becomes the two orthogonal factors $U,V$ of SVD, and the "all eigenvalues $\ge0$" fact (PSD) is what guarantees real, nonnegative singular values.

---

## 9. One-line cheat-sheet summary

$Q(x)=x^TAx$, $A$ symmetric ($a_{ii}=$ coeff $x_i^2$, $a_{ij}=\tfrac12$ coeff $x_ix_j$). Orthogonally diagonalize $\to\lambda_1y_1^2+\dots+\lambda_n y_n^2$. **PD $\iff$ all $\lambda>0\iff$ all leading minors $>0$** ($2\times2$: $\det>0$ & $\text{tr}>0$); **PSD $\iff$ all $\lambda\ge0$** (needs *all* principal minors $\ge0$, not just leading); **indefinite $\iff$ both signs**. Hessian PD/ND/indefinite $\Rightarrow$ min/max/saddle. Rank, index, signature are invariant (**Sylvester's law of inertia**).

---

### ➡️ Next topic
**Topic 12 — Singular Value Decomposition (SVD)** — factoring *any* $m\times n$ matrix as $A=U\Sigma V^T$ (orthogonal $U,V$; nonnegative diagonal $\Sigma$), built from the PSD eigen-analysis of $A^TA$ and $AA^T$; singular values, rank, the four fundamental subspaces, low-rank approximation, and the pseudoinverse.

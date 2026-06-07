# Topic 9 — Special Matrices: Orthogonal, Idempotent & Partition (Block) Matrices
*Subject: Linear Algebra · GATE Data Science & AI (DA)*

---

## 0. Topic header & syllabus map

- **Topic:** Special Matrices — **Orthogonal**, **Idempotent**, and **Partition / Block** matrices (and their properties)
- **Subject:** Linear Algebra
- **Where it sits:** Comes **right after Eigenvalues & Eigenvectors (Topic 8)** and **leads into Projections & Projection Matrices (Topic 10)**. Topic 8 gave us eigenvalues as a *diagnostic tool*; this topic flips the direction — we **recognize structure** in a matrix and instantly read off its eigenvalues, determinant, rank, and inverse *without grinding through* $\det(A-\lambda I)=0$. Idempotent matrices in particular *are* projection matrices, so this topic is the algebraic doorway into Topic 10.

---

## 1. Why this topic exists (motivation + connection)

A "general" matrix is structureless: to find its inverse you do full elimination, to find its eigenvalues you expand a degree-$n$ polynomial, to find its determinant you do cofactors. But a handful of **structured matrix families** show up again and again in ML and numerical computing — **rotations/reflections** (orthogonal), **projections** (idempotent), and **block-organized systems** (partition matrices). Each family comes with **shortcuts** so powerful that recognizing the structure replaces a page of computation:

- For an **orthogonal** matrix, the inverse is *free* — just transpose ($Q^{-1}=Q^{T}$).
- For an **idempotent** matrix, $\text{trace}=\text{rank}$ and the eigenvalues are *only* $0$ and $1$ — no characteristic polynomial needed.
- For a **block-triangular** matrix, the determinant is just the product of the diagonal blocks' determinants, and the eigenvalues are the union of the blocks' eigenvalues.

**What the previous topic couldn't do / what this adds.** Topic 8 computed eigenvalues *from scratch* for an arbitrary matrix. Here we go the other way: structure $\Rightarrow$ spectrum. Saying "$A$ is idempotent" *immediately* gives eigenvalues $\in\{0,1\}$, diagonalizability, and $\text{trace}=\text{rank}$. Saying "$Q$ is orthogonal" *immediately* gives $|\lambda|=1$ and $\det=\pm1$. The eigenvalue machinery becomes a fast classifier rather than a calculator.

**Forward.** Idempotent matrices are exactly **projection matrices** (Topic 10) — the symmetric ones are *orthogonal* projections. Orthogonal matrices are the $Q$ in QR and the $U,V$ in **SVD** (Topic 12). Block structure underlies covariance matrices and the $A^TA$ / $AA^T$ blocks behind SVD and quadratic forms (Topic 11).

---

## 2. Prerequisites

- **Eigenvalues & eigenvectors (Topic 8):** eigenvalue characterizations, diagonalizability, $\text{trace}=\sum\lambda_i$, $\det=\prod\lambda_i$, $\text{rank}=$ #nonzero eigenvalues *for diagonalizable matrices*.
- **Determinant (Topic 2):** $\det(AB)=\det A\det B$, $\det(A^T)=\det A$, $\det(cA)=c^n\det A$.
- **Matrix operations (Topic 1):** transpose, inverse, trace, products.
- **Rank & nullity (Topic 7):** for $\text{trace}=\text{rank}$ and $\text{rank}(A)+\text{rank}(I-A)=n$.
- **Dot product / orthonormality of vectors:** a set is **orthonormal** if vectors are mutually orthogonal **and** each has unit length.

---

## 3. Core idea — intuition first

- **Orthogonal = rigid motion.** Picture rotating or reflecting a rigid object: distances between points and angles never change. An orthogonal matrix is exactly a transformation that **preserves all lengths and angles** — its columns are a *rotated/reflected copy of the standard axes* (perpendicular unit vectors).
- **Idempotent = "doing it once equals doing it twice."** Think of the shadow a 3-D object casts on the floor. Casting a shadow *of the shadow* gives the same shadow — applying the map again changes nothing. That is $A^2=A$. Such a map keeps whatever already lies in its "target" and flattens everything else once.
- **Partition / block = divide and conquer.** A big matrix is chopped into rectangular sub-blocks. If the blocks are arranged neatly (diagonal or triangular), the big matrix's determinant, eigenvalues, and inverse **decompose into the blocks'** — you solve small problems and assemble.

---

## 4. Formal treatment

### 4A. Orthogonal matrices

**Definition.** A **square** real matrix $Q$ is **orthogonal** if
$$Q^{T}Q = QQ^{T}=I \quad\Longleftrightarrow\quad Q^{-1}=Q^{T}.$$

**Equivalent characterizations.**
- The **columns** of $Q$ are **orthonormal** (mutually orthogonal *and* unit length). By $QQ^T=I$, the **rows** are orthonormal too.
- $Q$ **preserves the dot product**: $(Qx)\cdot(Qy)=x^TQ^TQy=x\cdot y$ for all $x,y$. Hence it **preserves lengths and angles**: $\lVert Qx\rVert=\lVert x\rVert$.

**Properties (with conditions).**

| Property | Note |
|---|---|
| $Q^{-1}=Q^{T}$ | inverse is free — just transpose |
| $\det(Q)=\pm1$ | $+1$: rotation (proper); $-1$: reflection (improper) |
| Every eigenvalue satisfies $\lvert\lambda\rvert=1$ | real eigenvalues are $\pm1$; complex ones come in conjugate pairs $e^{\pm i\theta}$ |
| Product of orthogonals is orthogonal; $Q^T$ is orthogonal | they form the group $O(n)$ |
| Preserves length & angle (isometry) | $\lVert Qx\rVert=\lVert x\rVert$ |
| Columns **and** rows are orthonormal bases of $\mathbb R^n$ | |

**$2\times2$ orthogonal matrices** are exactly
$$\underbrace{\begin{bmatrix}\cos\theta & -\sin\theta\\ \sin\theta & \cos\theta\end{bmatrix}}_{\det=+1\ (\text{rotation})}\qquad\text{or}\qquad\underbrace{\begin{bmatrix}\cos\theta & \sin\theta\\ \sin\theta & -\cos\theta\end{bmatrix}}_{\det=-1\ (\text{reflection})}.$$

**Two short derivations GATE expects.**

- **$\det Q=\pm1$:** from $Q^TQ=I$, $\det(Q^T)\det(Q)=\det I=1$, and $\det(Q^T)=\det(Q)$, so $\det(Q)^2=1\Rightarrow\det Q=\pm1$.
- **$\lvert\lambda\rvert=1$:** if $Qv=\lambda v$ with $v\neq0$, take norms: $\lVert Qv\rVert=\lvert\lambda\rvert\,\lVert v\rVert$. But $\lVert Qv\rVert=\lVert v\rVert$, so $\lvert\lambda\rvert=1$.

> **Caution (terminology trap):** "orthogonal matrix" really means **orthonormal columns**. A matrix whose columns are merely orthogonal but *not unit length* is **not** orthogonal (see §7). Also, "orthogonal" requires the matrix to be **square**; a tall matrix with orthonormal columns satisfies only $Q^TQ=I$ (not $QQ^T=I$) — that's a *matrix with orthonormal columns* (relevant in SVD), not an orthogonal matrix.

### 4B. Idempotent matrices

**Definition.** A **square** matrix $A$ is **idempotent** if
$$A^{2}=A.$$

**Properties (with conditions).**

| Property | Why / condition |
|---|---|
| Eigenvalues are **only $0$ or $1$** | $Av=\lambda v\Rightarrow A^2v=\lambda^2v$, but $A^2v=Av=\lambda v\Rightarrow\lambda^2=\lambda$ |
| $A$ is **diagonalizable** | minimal polynomial divides $x(x-1)$ — distinct linear factors |
| $\boxed{\text{trace}(A)=\text{rank}(A)}$ | $=$ number of eigenvalues equal to $1$ (uses diagonalizability) |
| $I-A$ is idempotent | $(I-A)^2=I-2A+A^2=I-A$ |
| $A(I-A)=(I-A)A=0$ | $A-A^2=A-A=0$ |
| $\text{rank}(A)+\text{rank}(I-A)=n$ | $\text{rank}(I-A)=\text{tr}(I-A)=n-\text{tr}(A)$ |
| If $A$ idempotent **and invertible**, then $A=I$ | multiply $A^2=A$ by $A^{-1}$ |
| $A\neq I$ idempotent $\Rightarrow$ $A$ is **singular** ($\det A=0$) | $0$ is then an eigenvalue |

**Determinant.** Eigenvalues are $0/1$, so $\det A=\prod\lambda_i\in\{0,1\}$, and $\det A=1$ **only** when *every* eigenvalue is $1$, i.e. $A=I$. For any other idempotent, $\det A=0$.

> **Bridge to Topic 10:** an idempotent matrix *is* a **projection** — it projects onto its column space (the $\lambda=1$ eigenspace, the "fixed points" with $Av=v$) along its null space (the $\lambda=0$ eigenspace). A **symmetric** idempotent ($A^2=A$ and $A^T=A$) is an **orthogonal projection**. We develop the geometry next topic.

### 4C. Partition (block) matrices

A matrix written in terms of submatrices (blocks), e.g.
$$M=\left[\begin{array}{cc} A & B\\ C & D\end{array}\right].$$

**Block operations (need conformable partitions).**
- **Addition:** add corresponding blocks (partitions must match).
- **Multiplication:** treat blocks like scalar entries — but the **column partition of the first factor must equal the row partition of the second** so block products are defined. Then $(MN)_{ij}=\sum_k M_{ik}N_{kj}$ with block products.
- **Transpose:** flip block positions across the diagonal **and** transpose each block:
$$\left[\begin{array}{cc}A&B\\C&D\end{array}\right]^{T}=\left[\begin{array}{cc}A^{T}&C^{T}\\B^{T}&D^{T}\end{array}\right].$$

**Block-diagonal** $\operatorname{diag}(A,D)$ (off-diagonal blocks zero):
$$\det=\det(A)\det(D),\quad \text{eigenvalues}=\text{eig}(A)\cup\text{eig}(D),\quad \operatorname{diag}(A,D)^{-1}=\operatorname{diag}(A^{-1},D^{-1}),\quad \operatorname{diag}(A,D)^{k}=\operatorname{diag}(A^{k},D^{k}).$$

**Block-triangular** $\begin{bmatrix}A&B\\0&D\end{bmatrix}$ (or lower): the off-diagonal block $B$ does **not** affect determinant or eigenvalues:
$$\boxed{\det\begin{bmatrix}A&B\\0&D\end{bmatrix}=\det(A)\det(D)},\qquad \text{eigenvalues}=\text{eig}(A)\cup\text{eig}(D),$$
$$\begin{bmatrix}A&B\\0&D\end{bmatrix}^{-1}=\begin{bmatrix}A^{-1}&-A^{-1}BD^{-1}\\0&D^{-1}\end{bmatrix}\quad(A,D\text{ invertible}).$$

**General $2\times2$ block determinant — Schur complement** (when $A$ is invertible):
$$\boxed{\det\begin{bmatrix}A&B\\C&D\end{bmatrix}=\det(A)\,\det\!\big(D-CA^{-1}B\big)}.$$
The matrix $S=D-CA^{-1}B$ is the **Schur complement** of $A$. (If $D$ is invertible instead: $\det M=\det(D)\det(A-BD^{-1}C)$.) *Derivation:* factor $M=\begin{bmatrix}I&0\\CA^{-1}&I\end{bmatrix}\begin{bmatrix}A&B\\0&D-CA^{-1}B\end{bmatrix}$; the first factor has determinant $1$.

*Rarely needed special case:* if the lower blocks commute ($CD=DC$, all blocks square), $\det M=\det(AD-BC)$.

### 4D. Related named matrices (quick awareness — same eigenvalue trick)

These pair naturally with idempotent and have appeared in GATE CS/DA-adjacent questions; recognizing them gives instant eigenvalues:
- **Involutory:** $A^2=I$ $\Rightarrow$ eigenvalues $\pm1$; $A^{-1}=A$. (Reflections are symmetric involutory and orthogonal.)
- **Nilpotent:** $A^{k}=0$ for some $k$ $\Rightarrow$ **all eigenvalues are $0$**; $\det A=0$, $\text{trace}=0$, and such $A$ (if $\neq0$) is **not** diagonalizable.

---

## 5. Worked problems (GATE-style)

### Problem 1 (MCQ/NAT) — verify orthogonality and read off its consequences
Is $Q=\dfrac13\begin{bmatrix}1&2&2\\2&1&-2\\2&-2&1\end{bmatrix}$ orthogonal? If so, state $Q^{-1}$, $\det Q$, $\lVert Qx\rVert$, and the moduli of its eigenvalues.

**Solution.** Check columns are orthonormal (work with the integer columns; the $\tfrac13$ scales each norm$^2$ by $\tfrac19$).
- Norms$^2$: $1^2+2^2+2^2=9$, $2^2+1^2+(-2)^2=9$, $2^2+(-2)^2+1^2=9$ → each column has length $\sqrt{9}/3=1$ ✔.
- Dot products: $c_1\!\cdot\!c_2=2+2-4=0$; $c_1\!\cdot\!c_3=2-4+2=0$; $c_2\!\cdot\!c_3=4-2-2=0$ ✔.

So $Q$ **is orthogonal**. Therefore:
- $Q^{-1}=Q^{T}$ (no inversion work needed).
- $\det Q=\pm1$. Compute: $\det\!\begin{bmatrix}1&2&2\\2&1&-2\\2&-2&1\end{bmatrix}=-27$, and $\det Q=\tfrac{1}{3^3}(-27)=-1$. So $\det Q=-1$ → an **improper** orthogonal matrix (reflection-type).
- $\lVert Qx\rVert=\lVert x\rVert$ for every $x$ (length-preserving).
- All eigenvalues have $\lvert\lambda\rvert=1$. (With $\det=-1$ in $3\times3$, the real spectrum includes $\lambda=-1$, the rest a conjugate pair on the unit circle.)

> **Mini-variant (completion):** For which $a$ is $Q=\begin{bmatrix}a&a\\a&-a\end{bmatrix}$ orthogonal? Need columns of unit length: $2a^2=1\Rightarrow a=\pm\tfrac{1}{\sqrt2}$ (columns are automatically orthogonal: $a^2-a^2=0$). Then $\det Q=-2a^2=-1$ → reflection.

### Problem 2 (NAT) — idempotent: find the parameter, then use trace $=$ rank
**(a)** Find all $k$ for which $A=\begin{bmatrix}k&k\\k&k\end{bmatrix}$ is idempotent, and for the nontrivial $k$ give the eigenvalues, rank, and $\det$.
**(b)** A $5\times5$ matrix $B$ satisfies $B^2=B$ with $\text{trace}(B)=2$. Find $\text{rank}(B)$, $\det(B)$, the eigenvalues with multiplicities, and $\text{rank}(I-B)$.

**Solution (a).** $A^2=\begin{bmatrix}2k^2&2k^2\\2k^2&2k^2\end{bmatrix}$. Setting $A^2=A$ gives $2k^2=k\Rightarrow k(2k-1)=0\Rightarrow k=0$ or $k=\tfrac12$.
For $k=\tfrac12$: $A=\tfrac12\begin{bmatrix}1&1\\1&1\end{bmatrix}$, with $\text{tr}=1$, $\det=\tfrac14-\tfrac14=0$. Eigenvalues $\{1,0\}$; $\text{rank}=1=\text{trace}$ ✔. ($I-A=\tfrac12\begin{bmatrix}1&-1\\-1&1\end{bmatrix}$ is also idempotent.)

**Solution (b).** Idempotent $\Rightarrow$ eigenvalues are $0/1$ and $\text{trace}=\text{rank}$. So $\text{rank}(B)=2$. Eigenvalues: $1$ with multiplicity $2$, $0$ with multiplicity $3$. $\det(B)=\prod\lambda_i=0$. $\text{rank}(I-B)=n-\text{rank}(B)=5-2=3$.

### Problem 3 (MCQ/NAT) — block determinants & eigenvalues
**(a)** Let $M=\begin{bmatrix}A&B\\0&D\end{bmatrix}$ with $A=\begin{bmatrix}2&1\\0&3\end{bmatrix}$, $D=\begin{bmatrix}5&2\\0&7\end{bmatrix}$, and any $B$. Find $\det M$ and the eigenvalues of $M$.
**(b)** Using the Schur complement, find $\det\begin{bmatrix}I_2&B\\ I_2&3I_2\end{bmatrix}$ where $B=\begin{bmatrix}1&1\\1&1\end{bmatrix}$.

**Solution (a).** Block-triangular, so $B$ is irrelevant:
$$\det M=\det(A)\det(D)=(2\cdot3)(5\cdot7)=6\cdot35=210.$$
Eigenvalues $=\text{eig}(A)\cup\text{eig}(D)=\{2,3\}\cup\{5,7\}=\{2,3,5,7\}$.

**Solution (b).** Here $A=I_2$ (so $A^{-1}=I_2$), $C=I_2$, $D=3I_2$. Schur complement:
$$\det=\det(A)\,\det(D-CA^{-1}B)=\det(3I_2-B)=\det\begin{bmatrix}2&-1\\-1&2\end{bmatrix}=4-1=3.$$
*(Check by elimination on the $4\times4$: subtracting the top block-rows from the bottom turns it into block-upper-triangular with Schur block $\begin{bmatrix}2&-1\\-1&2\end{bmatrix}$, determinant $3$ — same answer.)*

### Problem 4 (MSQ) — trap-spotting across all three families
Which statements are **always true**?
(i) If a real square matrix has $\det=\pm1$, it is orthogonal.
(ii) An idempotent matrix other than $I$ is singular.
(iii) If the columns of a square matrix are mutually orthogonal, the matrix is orthogonal.
(iv) For block matrices, $\det\begin{bmatrix}A&B\\C&D\end{bmatrix}=\det(A)\det(D)-\det(B)\det(C)$.
(v) Every idempotent matrix is diagonalizable.
(vi) Every eigenvalue of an orthogonal matrix has absolute value $1$.

**Solution.**
- **(i) FALSE** — necessary, not sufficient. $\begin{bmatrix}1&1\\0&1\end{bmatrix}$ has $\det=1$ but is not orthogonal.
- **(ii) TRUE** — $\det=0$ unless $A=I$.
- **(iii) FALSE** — columns must be **orthonormal** (unit length too). $\begin{bmatrix}2&0\\0&3\end{bmatrix}$ has orthogonal columns but isn't orthogonal.
- **(iv) FALSE** — the "scalar-like" rule fails; only block-triangular gives $\det(A)\det(D)$, and the general case needs the Schur complement.
- **(v) TRUE** — minimal polynomial splits into distinct linear factors $x(x-1)$.
- **(vi) TRUE** — shown by norm preservation.

**Correct: (ii), (v), (vi).**

---

## 6. Common GATE question types on this topic

- **Orthogonal:** verify orthogonality (check $Q^TQ=I$ / orthonormal columns); complete a matrix to orthogonal; use $\det=\pm1$; use $\lvert\lambda\rvert=1$; norm/angle preservation; identify $2\times2$ rotation vs reflection; "is a product of orthogonals orthogonal?".
- **Idempotent:** consequences of $A^2=A$ (eigenvalues $0/1$, **$\text{trace}=\text{rank}$**, $\det$, diagonalizability); find parameter making a matrix idempotent; properties of $I-A$; "invertible idempotent $\Rightarrow I$".
- **Block / partition:** determinant and eigenvalues of block-triangular / block-diagonal matrices; inverse of block-diagonal; Schur-complement determinant; block multiplication conformability; block transpose.
- **Related types:** instantly identify eigenvalues of involutory ($\pm1$) and nilpotent ($0$) matrices.

---

## 7. Traps, pitfalls & common mistakes

- **Orthogonal ≠ "orthogonal columns."** The columns must be **orthonormal** (perpendicular **and** unit length). Orthogonal-but-not-unit columns give $Q^TQ=\text{diagonal}\neq I$.
- **Orthogonal matrices must be square.** A tall matrix with orthonormal columns gives only $Q^TQ=I$, **not** $QQ^T=I$.
- **$\det=\pm1$ is necessary, not sufficient** for orthogonality (e.g. a shear has $\det=1$).
- **Idempotent (other than $I$) is singular**, hence **not invertible** — only $I$ is an invertible idempotent.
- **Eigenvalues of an idempotent are in $\{0,1\}$**, but a given one may be all $1$'s (only $I$), all $0$'s (only the zero matrix), or a mix.
- **Not every idempotent is symmetric.** A symmetric idempotent is an *orthogonal* projection; a general idempotent is an *oblique* projection. Don't assume $A^T=A$.
- **The biggest block trap:** $\det\begin{bmatrix}A&B\\C&D\end{bmatrix}\neq\det(A)\det(D)-\det(B)\det(C)$ in general. Treating blocks like scalar entries is **wrong** unless the matrix is block-triangular/diagonal (then $\det(A)\det(D)$) or the relevant blocks commute. Use the **Schur complement** when $C\neq0$.
- **Block multiplication needs conformable partitions** — you can't multiply arbitrarily partitioned blocks.
- **Block transpose flips positions *and* transposes each block** — $(M^T)_{ij}=(M_{ji})^T$. Forgetting either step is a common error.
- **Schur complement requires the corner block ($A$ or $D$) to be invertible.**
- **$\text{trace}=\text{rank}$ is specific to idempotent matrices** — don't apply it to arbitrary matrices.

---

## 8. Connection forward

- **Projections & projection matrices (Topic 10, next):** "idempotent" is the *algebra* ($A^2=A$); "projection" is the *geometry*. The $\lambda=1$ eigenspace is what gets kept (the range), the $\lambda=0$ eigenspace is what gets killed (the null space). **Symmetric idempotent $=$ orthogonal projection** $P=A(A^TA)^{-1}A^T$, which we build next.
- **Quadratic forms (Topic 11):** symmetric matrices (a structured family) get classified by eigenvalue *signs*; orthogonal matrices diagonalize them (principal-axis theorem).
- **SVD (Topic 12):** the factor matrices $U$ and $V$ are **orthogonal**, and the construction rests on the symmetric blocks $A^TA$, $AA^T$ — the block and orthogonal ideas here are the scaffolding.

---

## 9. One-line cheat-sheet summary

**Orthogonal:** $Q^TQ=I$ (orthonormal cols) ⇒ $Q^{-1}=Q^T$, $\det=\pm1$, $\lvert\lambda\rvert=1$, preserves length/angle. **Idempotent:** $A^2=A$ ⇒ eigenvalues $\in\{0,1\}$, **trace = rank**, diagonalizable, $I-A$ idempotent, $A\neq I$ is singular. **Block:** $\det$ of block-triangular/diagonal $=$ product of diagonal-block dets, eigenvalues $=$ union; general $\det\!\begin{bmatrix}A&B\\C&D\end{bmatrix}=\det(A)\det(D-CA^{-1}B)$.

---

### ➡️ Next topic
**Topic 10 — Projections and Projection Matrices** — the geometric meaning of idempotent matrices: projecting a vector onto a subspace (and onto its orthogonal complement), the orthogonal-projection formula, and least-squares as the headline application.

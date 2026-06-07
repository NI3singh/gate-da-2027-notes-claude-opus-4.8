# Matrices — Operations, Transpose & Inverse
*GATE DA · Linear Algebra · Notes Topic 1*

---

## 0. Topic header & syllabus map

- **Topic:** Matrices — operations (addition, scalar multiplication, multiplication, powers), **transpose**, and **inverse**.
- **Subject:** Linear Algebra.
- **Where it sits:** This is the **first** topic of the *Matrices & Systems* block and the foundation of the entire Linear Algebra syllabus. Determinant, rank, eigenvalues, LU decomposition and SVD are all built on the operations defined here. **It comes first and leads directly into the Determinant** — the single number that decides *when the inverse defined here actually exists*.

---

## 1. Why this topic exists (motivation + connection)

A huge amount of real mathematics is **linear**: systems of equations, transformations of space (rotation, scaling, projection), and tabular data (rows = samples, columns = features). Writing all of that out equation-by-equation is unworkable. A **matrix** is the data structure that packs an entire linear system, an entire linear transformation, or an entire dataset into **one object** you can add, scale and multiply algebraically.

- **What came before couldn't do this:** ordinary numbers (scalars) describe one quantity and one operation at a time. They cannot, in a single symbol, encode "take 3 unknowns, mix them by these 9 coefficients, and read off 3 outputs." Matrices can.
- **The key new idea:** matrix **multiplication is composition of linear maps** — doing one transformation then another. This is why $AB$ usually means something different from $BA$, and why so much of the subject is about products, powers and inverses of matrices.
- **What it leads into:** once we can multiply matrices, the natural question is "can we *undo* a matrix?" That is the **inverse**, and deciding whether an inverse exists forces us to invent the **determinant** (next topic).

---

## 2. Prerequisites

- Basic arithmetic and algebra (real and, lightly, complex numbers).
- Summation notation $\sum$.
- The idea of a function/mapping (helpful for the "matrix = transformation" view).
- Nothing else — this is the genuine starting point of the subject.

---

## 3. Core idea — intuition first

Think of a matrix two ways at once:

1. **A grid of numbers** — a table with $m$ rows and $n$ columns. Good for storing data and coefficients.
2. **A machine (linear transformation)** — feed it a vector $x$, it returns a vector $Ax$. The matrix *is* the rule.

The single most important mental model is **matrix × vector = a recipe for mixing columns**:

$$
A x \;=\; x_1(\text{col}_1 \text{ of } A) + x_2(\text{col}_2 \text{ of } A) + \cdots + x_n(\text{col}_n \text{ of } A)
$$

So $Ax$ is a **linear combination of the columns of $A$**, weighted by the entries of $x$. (This single sentence is the seed of *column space*, *rank* and *solvability* — topics you'll meet later.)

**Multiplication of two matrices** $AB$ then just means: apply $B$ first, then $A$. Because order of operations matters in the real world (rotate-then-stretch ≠ stretch-then-rotate), $AB \ne BA$ in general.

**Transpose** = flip the grid across its main diagonal (rows become columns).
**Inverse** = the machine that perfectly *undoes* $A$, so that doing $A$ then $A^{-1}$ leaves everything unchanged.

---

## 4. Formal treatment

### 4.1 Notation & basic vocabulary

A matrix $A$ of **order** (size) $m \times n$ is written $A = [a_{ij}]$, where $a_{ij}$ is the entry in row $i$, column $j$:

$$
A=\begin{pmatrix} a_{11} & a_{12} & \cdots & a_{1n}\\ a_{21} & a_{22} & \cdots & a_{2n}\\ \vdots & & \ddots & \vdots\\ a_{m1} & a_{m2} & \cdots & a_{mn}\end{pmatrix}
$$

- **Square** matrix: $m=n$. **Rectangular**: $m\ne n$.
- **Row vector**: $1\times n$. **Column vector**: $m\times 1$.
- **Equality:** $A=B$ iff same order *and* $a_{ij}=b_{ij}$ for all $i,j$.
- **Main diagonal** of a square matrix: entries $a_{11},a_{22},\dots,a_{nn}$.

### 4.2 Special matrices (reference)

| Name | Defining condition |
|---|---|
| Zero / null $O$ | every entry $0$ |
| Diagonal $D$ | $a_{ij}=0$ for $i\ne j$ (square) |
| Scalar | diagonal with all diagonal entries equal |
| Identity $I_n$ | diagonal with all diagonal entries $=1$ |
| Upper triangular | $a_{ij}=0$ for $i>j$ |
| Lower triangular | $a_{ij}=0$ for $i<j$ |
| Symmetric | $A^{T}=A$ (i.e. $a_{ij}=a_{ji}$) |
| Skew-symmetric | $A^{T}=-A$ (i.e. $a_{ij}=-a_{ji}$, so $a_{ii}=0$) |

> **Powers / structural families** (definitions only here; some get a full topic later):
> **Idempotent** $A^2=A$ · **Nilpotent** $A^k=O$ for some $k$ · **Involutory** $A^2=I$ · **Orthogonal** $A^TA=I$.
> *(Idempotent, orthogonal and partition matrices are studied in depth in their own later topic; only the definitions are needed now.)*

### 4.3 Operations and their **conditions**

> **Addition / subtraction** — *condition: same order.*
> $(A\pm B)_{ij}=a_{ij}\pm b_{ij}$. Done entrywise.

> **Scalar multiplication** — *always defined.*
> $(kA)_{ij}=k\,a_{ij}$.

> **Matrix multiplication** — *condition: inner dimensions match.*
> If $A$ is $m\times n$ and $B$ is $n\times p$, then $AB$ is $m\times p$ with
> $$(AB)_{ij}=\sum_{k=1}^{n} a_{ik}\,b_{kj}.$$
> **(columns of $A$) = (rows of $B$)** is required. The product $BA$ may not even be defined.

**Properties of these operations** (assume all sizes conformable):

- Addition is commutative and associative; $A+O=A$.
- Scalar mult distributes: $k(A+B)=kA+kB$, $(k+l)A=kA+lA$.
- Multiplication is **associative**: $(AB)C=A(BC)$.
- Multiplication is **distributive**: $A(B+C)=AB+AC$, $(A+B)C=AC+BC$.
- $I_mA=A=AI_n$ (identity acts as "1").
- **Multiplication is NOT commutative:** generally $AB\ne BA$.
- **No zero-product rule:** $AB=O$ does **not** imply $A=O$ or $B=O$.
- **No cancellation:** $AB=AC$ does **not** imply $B=C$ (unless $A$ is invertible).

**Powers:** for square $A$, $A^k=\underbrace{A\cdots A}_{k}$, with $A^0=I$. Standard index laws $A^pA^q=A^{p+q}$, $(A^p)^q=A^{pq}$ hold, but $(AB)^k\ne A^kB^k$ in general (again, because $AB\ne BA$).

> ⚠️ **Binomial fails:** $(A+B)^2 = A^2+AB+BA+B^2 \ne A^2+2AB+B^2$ unless $AB=BA$.

### 4.4 Transpose

$$
(A^{T})_{ij}=a_{ji}\qquad(\text{rows}\leftrightarrow\text{columns});\quad A\ \text{is }m\times n \Rightarrow A^{T}\ \text{is }n\times m.
$$

**Properties (all always true):**

$$
(A^{T})^{T}=A,\qquad (A+B)^{T}=A^{T}+B^{T},\qquad (kA)^{T}=kA^{T},
$$
$$
\boxed{(AB)^{T}=B^{T}A^{T}}\quad\text{(order reverses)},\qquad (ABC)^{T}=C^{T}B^{T}A^{T}.
$$

**Symmetric & skew-symmetric (square matrices):**

- Symmetric: $A^{T}=A$.
- Skew-symmetric: $A^{T}=-A$ ⇒ all diagonal entries are $0$ ⇒ $\operatorname{tr}(A)=0$.
- **Every square matrix splits uniquely** into a symmetric + a skew-symmetric part:
$$
A=\underbrace{\tfrac12\,(A+A^{T})}_{\text{symmetric}}+\underbrace{\tfrac12\,(A-A^{T})}_{\text{skew-symmetric}}.
$$
- For any $A$: $AA^{T}$ and $A^{T}A$ are **symmetric** (and square). *(This fact powers SVD later.)*

**Counting independent entries of an $n\times n$ matrix** (a recurring GATE NAT):

$$
\text{symmetric: } \frac{n(n+1)}{2}, \qquad \text{skew-symmetric: } \frac{n(n-1)}{2}, \qquad \text{diagonal: } n.
$$

**Complex case (brief — light for GATE DA):** the analogue of transpose is the **conjugate transpose** $A^{*}=\overline{A}^{\,T}$. Then **Hermitian** means $A^{*}=A$, **skew-Hermitian** $A^{*}=-A$, **unitary** $U^{*}U=I$. For *real* matrices these reduce exactly to symmetric, skew-symmetric and orthogonal.

### 4.5 Trace (square matrices)

$$
\operatorname{tr}(A)=\sum_{i=1}^{n} a_{ii}\quad(\text{sum of diagonal entries}).
$$

**Properties:**
$$
\operatorname{tr}(A+B)=\operatorname{tr}A+\operatorname{tr}B,\quad \operatorname{tr}(kA)=k\operatorname{tr}A,\quad \operatorname{tr}(A^{T})=\operatorname{tr}A,
$$
$$
\boxed{\operatorname{tr}(AB)=\operatorname{tr}(BA)}\ \text{(cyclic; holds even when } AB\ne BA\text{)},\qquad \operatorname{tr}(ABC)=\operatorname{tr}(BCA)=\operatorname{tr}(CAB).
$$

> **Forward link:** $\operatorname{tr}(A)=$ sum of eigenvalues of $A$. GATE often hides a trace fact inside an eigenvalue question.

### 4.6 Inverse

$A$ (square, $n\times n$) is **invertible / nonsingular** if there is a matrix $A^{-1}$ with
$$
AA^{-1}=A^{-1}A=I_n.
$$

- The inverse, when it exists, is **unique**.
- **Only square matrices** can have a (two-sided) inverse.

**Properties (whenever the inverses exist):**
$$
(A^{-1})^{-1}=A,\qquad \boxed{(AB)^{-1}=B^{-1}A^{-1}}\ \text{(order reverses)},\qquad (kA)^{-1}=\tfrac1k\,A^{-1}\ (k\ne0),
$$
$$
(A^{T})^{-1}=(A^{-1})^{T},\qquad (A^{-1})^{T}=(A^{T})^{-1},\qquad \det(A^{-1})=\frac{1}{\det A}.
$$

*Why the reversal $(AB)^{-1}=B^{-1}A^{-1}$?* Check it: $(AB)(B^{-1}A^{-1})=A(BB^{-1})A^{-1}=AIA^{-1}=I$. Putting $A^{-1}$ first would leave a stray $B^{-1}A$ in the middle. (One-line proof GATE sometimes wants.)

**The 2×2 formula — memorize:**
$$
A=\begin{pmatrix} a & b\\ c & d\end{pmatrix}\ \Rightarrow\ A^{-1}=\frac{1}{ad-bc}\begin{pmatrix} d & -b\\ -c & a\end{pmatrix},\qquad \text{condition: } ad-bc\ne0.
$$
(Swap the diagonal, negate the off-diagonal, divide by the determinant.)

**Two ways to compute a general inverse:**

1. **Adjoint method:** $A^{-1}=\dfrac{1}{\det A}\,\operatorname{adj}(A)$, where $\operatorname{adj}(A)$ is the transpose of the cofactor matrix. *Condition:* $\det A\ne0$. *(Determinant, cofactors and the adjoint are developed fully in the next topic; here just note the route.)*
2. **Gauss–Jordan method:** row-reduce the augmented block $[\,A\mid I\,]$ until the left side is $I$; the right side becomes $A^{-1}$:
$$
[\,A\mid I\,]\ \xrightarrow{\text{row ops}}\ [\,I\mid A^{-1}\,].
$$
*(The elementary row operations used here are the engine of Gaussian elimination — Topic 3.)*

**When does $A^{-1}$ exist? (Invertible Matrix Theorem — preview)**
For a square $A$, the following are **all equivalent**:
$\det A\ne0$ ⇔ $A$ has full rank ($\operatorname{rank}=n$) ⇔ rows (and columns) are linearly independent ⇔ $0$ is **not** an eigenvalue ⇔ $Ax=0$ has only the trivial solution $x=0$ ⇔ $Ax=b$ has a **unique** solution for every $b$.
*(Each of these conditions is a topic later; collect them here so the picture is one connected story.)*

> **Non-square note:** a rectangular matrix has no two-sided inverse, but may have a one-sided (left or right) inverse depending on its rank, and always has a **pseudoinverse** $A^{+}$. *(The pseudoinverse is built from the SVD — final Linear-Algebra topic.)*

---

## 5. Worked problems (GATE-style)

### Problem 1 (NAT) — non-commutativity *and* the trace-cyclic fact
Let $A=\begin{pmatrix}1&2\\3&4\end{pmatrix}$, $B=\begin{pmatrix}0&1\\1&0\end{pmatrix}$. Compute $AB$, $BA$, and $\operatorname{tr}(AB)$, $\operatorname{tr}(BA)$.

**Solution.**
$$
AB=\begin{pmatrix}1\cdot0+2\cdot1 & 1\cdot1+2\cdot0\\ 3\cdot0+4\cdot1 & 3\cdot1+4\cdot0\end{pmatrix}=\begin{pmatrix}2&1\\4&3\end{pmatrix},\quad
BA=\begin{pmatrix}3&4\\1&2\end{pmatrix}.
$$
So $AB\ne BA$ (non-commutative), yet $\operatorname{tr}(AB)=2+3=5=\operatorname{tr}(BA)=3+2$. **Both traces $=5$.**
*Takeaway:* products can differ while their traces are forced equal by $\operatorname{tr}(AB)=\operatorname{tr}(BA)$.

### Problem 2 (NAT/MCQ) — symmetric + skew decomposition
Write $A=\begin{pmatrix}2&5\\1&3\end{pmatrix}$ as symmetric $S$ plus skew-symmetric $K$.

**Solution.** $A^{T}=\begin{pmatrix}2&1\\5&3\end{pmatrix}$.
$$
S=\tfrac12(A+A^{T})=\tfrac12\begin{pmatrix}4&6\\6&6\end{pmatrix}=\begin{pmatrix}2&3\\3&3\end{pmatrix},\qquad
K=\tfrac12(A-A^{T})=\tfrac12\begin{pmatrix}0&4\\-4&0\end{pmatrix}=\begin{pmatrix}0&2\\-2&0\end{pmatrix}.
$$
Check $S+K=\begin{pmatrix}2&5\\1&3\end{pmatrix}=A$ ✓. Note $K$ has zero diagonal, as every skew-symmetric matrix must.

### Problem 3 (NAT) — 2×2 inverse and $(A^{2})^{-1}=(A^{-1})^{2}$
For $A=\begin{pmatrix}2&1\\1&1\end{pmatrix}$ find $A^{-1}$ and verify $(A^{2})^{-1}=(A^{-1})^{2}$.

**Solution.** $\det A=2\cdot1-1\cdot1=1$, so
$$
A^{-1}=\frac{1}{1}\begin{pmatrix}1&-1\\-1&2\end{pmatrix}=\begin{pmatrix}1&-1\\-1&2\end{pmatrix}.
$$
Then $(A^{-1})^{2}=\begin{pmatrix}1&-1\\-1&2\end{pmatrix}\begin{pmatrix}1&-1\\-1&2\end{pmatrix}=\begin{pmatrix}2&-3\\-3&5\end{pmatrix}$.
Also $A^{2}=\begin{pmatrix}5&3\\3&2\end{pmatrix}$, $\det(A^{2})=10-9=1$, so $(A^{2})^{-1}=\begin{pmatrix}2&-3\\-3&5\end{pmatrix}$ ✓. They match, illustrating $(A^{2})^{-1}=(A^{-1})^{2}$.

### Problem 4 (MSQ, trap variety) — which statements are **always** true?
For conformable square matrices $A,B$:
**(a)** If $A^{2}=A$ and $A$ is invertible, then $A=I$.
**(b)** If $AB=O$ then $A=O$ or $B=O$.
**(c)** $(AB)^{T}=A^{T}B^{T}$.
**(d)** If $A,B$ are symmetric, then $AB$ is symmetric **iff** $AB=BA$.

**Solution.**
- **(a) TRUE.** Multiply $A^{2}=A$ on the left by $A^{-1}$: $A^{-1}A^{2}=A^{-1}A \Rightarrow A=I$. *(Idempotent + invertible forces the identity.)*
- **(b) FALSE.** Counterexample: $A=\begin{pmatrix}0&1\\0&0\end{pmatrix}$, $B=\begin{pmatrix}0&1\\0&0\end{pmatrix}$ give $AB=O$ with $A,B\ne O$. ($AB=O$ never lets you conclude a factor is zero.)
- **(c) FALSE.** The correct identity reverses the order: $(AB)^{T}=B^{T}A^{T}$.
- **(d) TRUE.** $(AB)^{T}=B^{T}A^{T}=BA$ (since $A,B$ symmetric). So $AB$ symmetric $\iff (AB)^{T}=AB \iff BA=AB$.

**Correct choices: (a) and (d).**

---

## 6. Common GATE question types on this topic

- Compute a **product or power** $A^k$, or simplify a product expression.
- **Trace** tricks: use $\operatorname{tr}(AB)=\operatorname{tr}(BA)$; relate trace to eigenvalue sums.
- **True/False / MSQ** on properties: commutativity, the reversal rules $(AB)^{T}=B^{T}A^{T}$ and $(AB)^{-1}=B^{-1}A^{-1}$, zero-product, cancellation.
- **Find an inverse** (2×2 directly; 3×3 by adjoint or Gauss–Jordan).
- **Symmetric/skew decomposition**; **count** independent entries of a symmetric/skew matrix.
- **Reasoning with special matrices** — idempotent, nilpotent, involutory, orthogonal (e.g., "if $A^2=A$ and invertible…").
- **Invertibility conditions** linking det, rank, eigenvalue $0$, and $Ax=0$.

---

## 7. Traps, pitfalls & common mistakes

- **Assuming commutativity.** $AB\ne BA$ in general. Don't reorder factors.
- **Forgetting the order reverses** in $(AB)^{T}=B^{T}A^{T}$ and $(AB)^{-1}=B^{-1}A^{-1}$. The single most common error.
- **Zero-product / cancellation fallacy.** $AB=O$ ⇏ ($A=O$ or $B=O$); $AB=AC$ ⇏ $B=C$ (only valid if $A$ is invertible).
- **Wrong binomial.** $(A+B)^2=A^2+AB+BA+B^2$, *not* $A^2+2AB+B^2$.
- **Scalar–inverse slip.** $(kA)^{-1}=\tfrac1k A^{-1}$, **not** $kA^{-1}$.
- **Inverse existence.** $A^{-1}$ needs $A$ **square and nonsingular** ($\det A\ne 0$). Non-square matrices have no two-sided inverse.
- **Skew-symmetric details.** Diagonal entries must be $0$; trace is $0$; an **odd-order** skew-symmetric matrix is **singular** ($\det=0$). *(Proved with determinant facts — next topic.)*
- **Determinant of a sum.** $\det(A+B)\ne \det A+\det B$ (and $\operatorname{tr}$ adds but $\det$ does not).
- **Dimension mismatch.** Always check inner dimensions before multiplying; $AB$ may exist while $BA$ does not.
- **Identity over-generalized.** Scalar/identity matrices commute with everything, but that's special — don't extend it to arbitrary matrices.

---

## 8. Connection forward

We can now *write down* an inverse and a list of conditions for it to exist — but checking "is $\det A\ne0$?", "is the rank full?", "is $0$ an eigenvalue?" all hinge on one missing tool: a **single number that detects singularity**. That number is the **Determinant** (next topic). The determinant will also hand us the **adjoint formula** for the inverse, the rule $\det(AB)=\det A\,\det B$, and the singularity test reused in rank, eigenvalues, and SVD.

---

## 9. One-line cheat-sheet summary

**Matrices are grids you add/scale entrywise and multiply by row-into-column (multiplication is associative & distributive but NOT commutative); transpose flips rows↔columns and reverses order in products ($(AB)^T=B^TA^T$); the inverse undoes a matrix, also reverses order ($(AB)^{-1}=B^{-1}A^{-1}$), and exists only when the matrix is square and nonsingular.**

---

### ▶ Next topic (in order): **Determinant**
*(the number that decides invertibility, scales volume, and supplies the adjoint route to the inverse).*

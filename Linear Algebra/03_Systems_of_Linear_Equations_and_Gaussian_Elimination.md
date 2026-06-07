# Systems of Linear Equations & Gaussian Elimination
*GATE DA · Linear Algebra · Notes Topic 3*

---

## 0. Topic header & syllabus map

- **Topic:** Linear systems $Ax=b$ — the row/column pictures, homogeneous vs non-homogeneous systems, elementary row operations, row-echelon and reduced-row-echelon forms, **Gaussian elimination** (forward elimination + back substitution) and Gauss–Jordan, partial pivoting, and the **rank-based existence/uniqueness theorem**.
- **Subject:** Linear Algebra.
- **Where it sits:** Third topic of *Matrices & Systems*. It **comes right after Determinant** — the determinant told us *whether* a square system has a unique solution; here we learn *how to actually solve any system* and how to classify "none / unique / infinitely many." It **leads into LU Decomposition**, which is nothing more than Gaussian elimination *recorded as a matrix factorization* so the elimination work can be reused.

---

## 1. Why this topic exists (motivation + connection)

A linear system is the most basic computational object in the subject: fit a line, balance a circuit, solve equilibrium equations, find weights in linear regression. We need a **reliable, general method** to solve $Ax=b$ — and to detect when it can't be solved.

- **What the previous topic couldn't do:** the determinant gives a yes/no verdict for **square, nonsingular** systems ("unique solution exists") and Cramer's rule even writes the answer — but Cramer is $O(n!)$ slow, and the determinant says *nothing actionable* the moment $\det A=0$ (no/infinite solutions?) or the moment the system is **rectangular** ($m\neq n$, where no determinant exists at all).
- **The new idea:** **Gaussian elimination** simplifies the system with operations that *never change its solution set*, reducing $A$ to a staircase (echelon) form from which the solution — or the verdict "inconsistent / infinitely many" — can be **read off directly**, for any shape of $A$. The number of "pivots" produced (the **rank**) is what decides everything.
- **What it leads into:** the forward-elimination steps can be stored in a lower-triangular matrix $L$ and the resulting staircase in an upper-triangular $U$, giving $A=LU$ — **LU decomposition** (next), which lets you solve $Ax=b$ for *many* right-hand sides without redoing the expensive elimination each time.

---

## 2. Prerequisites

- Matrices, matrix–vector product, and especially the **"$Ax$ = linear combination of the columns of $A$"** view (Topic 1).
- Determinant and the fact $\det A\neq0 \iff A$ invertible $\iff$ square system has a unique solution (Topic 2); also that **row-replacement leaves $\det$ unchanged** (Property P6).
- An **operational** notion of rank, introduced here as needed: *rank = number of pivots = number of nonzero rows in a row-echelon form*. The full theory (column space, rank–nullity, four subspaces) is **Topic 7** — we borrow only the working definition now.

---

## 3. Core idea — intuition first

A system $Ax=b$ has **two equally valid pictures**:

- **Row picture:** each equation is a flat (a line in 2-D, a plane in 3-D, a hyperplane in general). A **solution** is a point lying on *all* of them — their common intersection. Three planes can meet in a point (unique), in a line (infinitely many), or not at all (none).
- **Column picture:** writing $Ax$ as a combination of $A$'s columns, the question becomes *"which weights $x$ mix the columns of $A$ to produce $b$?"* A solution exists exactly when **$b$ lies in the span of the columns** of $A$.

So there are only **three possible outcomes**, ever:

| Outcome | Row picture | Column picture |
|---|---|---|
| **Unique** | flats meet at one point | $b$ is a unique combination of independent columns |
| **None (inconsistent)** | no common point | $b$ is *outside* the column span |
| **Infinitely many** | flats overlap on a line/plane | $b$ is in the span, but columns are dependent → many weights work |

**Gaussian elimination** is just *systematic simplification*: replace the system with an equivalent, easier one (same solutions) by sweeping out variables one at a time, until the structure is a staircase you can solve by inspection.

---

## 4. Formal treatment

### 4.1 The system, the augmented matrix, homogeneous vs not

A system of $m$ equations in $n$ unknowns is $Ax=b$ with $A$ an $m\times n$ **coefficient matrix**, $x\in\mathbb{R}^n$, $b\in\mathbb{R}^m$. We work on the **augmented matrix**

$$
[\,A\mid b\,]\in\mathbb{R}^{m\times(n+1)}.
$$

- **Homogeneous:** $b=0$, i.e. $Ax=0$. **Always consistent** — $x=0$ (the *trivial* solution) always works. Its solution set is a **subspace** (the null space, Topic 7).
- **Non-homogeneous:** $b\neq0$. May be consistent or not.

### 4.2 Elementary row operations (EROs) — and why they're safe

There are exactly three:

1. **Swap** two rows: $R_i\leftrightarrow R_j$.
2. **Scale** a row by a nonzero constant: $R_i\to cR_i$, $c\neq0$.
3. **Replacement:** add a multiple of one row to another: $R_i\to R_i+cR_j$.

> **Key fact:** EROs **do not change the solution set** of the system (each is reversible). They also **preserve the rank** of the matrix. *(Their effect on the determinant — sign flip on swap, factor $c$ on scaling, no change on replacement — is exactly Topic 2's P2/P5/P6.)*

### 4.3 Row-echelon form (REF) and reduced row-echelon form (RREF)

A matrix is in **row-echelon form** when:
- all all-zero rows are at the bottom, and
- the first nonzero entry of each row (the **pivot**) lies strictly to the right of the pivot in the row above (a "staircase"), with zeros below each pivot.

**Pivot columns** contain a pivot; the remaining columns are **free columns** (their variables become **free variables**).

A matrix is in **reduced row-echelon form (RREF)** if additionally every **pivot is $1$** and is the **only nonzero entry in its column** (zeros above *and* below). **The RREF of a matrix is unique**; an REF is not.

### 4.4 The algorithm: Gaussian elimination & Gauss–Jordan

**Gaussian elimination (forward elimination → back substitution):**
1. **Forward elimination:** using swaps and replacements, create zeros below each pivot, moving left to right, to reach REF (upper-triangular for a square full-rank $A$).
2. **Back substitution:** solve from the bottom equation upward, substituting known values.

**Gauss–Jordan elimination:** continue past REF, also clearing entries *above* each pivot and scaling pivots to $1$, reaching **RREF**; the solution is then read off with no back substitution. *(This is also the engine of the $[\,A\mid I\,]\to[\,I\mid A^{-1}\,]$ inverse method from Topic 1.)*

**Partial pivoting (do not skip — GATE asks *why*):** before eliminating in a column, **swap** the row with the **largest-magnitude entry** in that column up to the pivot position. Two reasons:
- **Necessity:** if the natural pivot is $0$, you *must* swap to continue (can't divide by zero).
- **Stability:** even a small (not zero) pivot magnifies round-off error; using the largest pivot keeps the computation numerically stable.

### 4.5 Rank, consistency & the existence–uniqueness theorem

Let $r=\operatorname{rank}(A)=$ number of pivots in an REF of $A$. Always $r\le\min(m,n)$, and $\operatorname{rank}(A)\le\operatorname{rank}([A\mid b])\le\operatorname{rank}(A)+1$.

> **Master theorem (Rouché–Capelli).** For $Ax=b$ with $n$ unknowns:
> - **No solution (inconsistent)** $\iff \operatorname{rank}(A) < \operatorname{rank}([A\mid b])$ $\iff$ an REF row reads $[\,0\ \cdots\ 0\mid c\,]$ with $c\neq0$ $\iff b\notin$ column space of $A$.
> - **Consistent** $\iff \operatorname{rank}(A)=\operatorname{rank}([A\mid b])=r$. Then:
>   - **Unique solution** $\iff r=n$ (no free variables);
>   - **Infinitely many solutions** $\iff r<n$, with exactly $(n-r)$ **free parameters**.

**Homogeneous case $Ax=0$** (always consistent, since $b=0$ is in the column space):
- **only the trivial solution** $\iff r=n$;
- **nontrivial solutions exist** $\iff r<n$, forming a solution space of dimension $(n-r)$.

**Two corollaries GATE loves:**
- A homogeneous system with **more unknowns than equations** ($n>m$) **always** has a nontrivial solution, because $r\le m<n$.
- For a **square** $A$ ($m=n$): $r=n\iff\det A\neq0$. So the determinant test is just the **square special case** of the rank test. If $\det A=0$, the system is *either* inconsistent *or* has infinitely many solutions — never unique.

### 4.6 Structure of the general solution

For a **consistent** non-homogeneous system, the complete solution set is

$$
\boxed{x = x_p + x_h}
$$

where $x_p$ is **any one particular solution** of $Ax=b$, and $x_h$ ranges over **all** solutions of the homogeneous system $Ax=0$ (the null space, of dimension $n-r$). Concretely: set each free variable to a parameter, solve the pivot variables in terms of them, and the parameter-free part is $x_p$ while the parameter-carrying part is $x_h$. *(This is the affine "shifted subspace" structure; the homogeneous part is studied as the null space in Topic 7.)*

### 4.7 Cost, and the link back to determinant / Cramer

- **Cost of Gaussian elimination:** $\approx \tfrac{1}{3}n^3$ multiply-add operations (forward elimination dominates; back substitution is $O(n^2)$).
- **Cramer's rule** (Topic 2): solves a square nonsingular system as $x_i=\det(A_i)/\det A$, but needs $n{+}1$ determinants $\Rightarrow\ O(n\cdot n!)$ work — unusable beyond $3\times3$.
- **Why this matters next:** the expensive part of elimination depends only on $A$, not on $b$. Recording it once gives **LU decomposition** (next), after which each new right-hand side costs only $O(n^2)$.

---

## 5. Worked problems (GATE-style)

### Problem 1 (NAT) — solve a unique 3×3 system by elimination
$$
\begin{aligned}
2x+y-z&=1\\ x+3y+2z&=13\\ x-y+z&=2
\end{aligned}
$$

**Solution.** First check determinant (Topic 2) — $\det A = 2(3\cdot1-2(-1))-1(1\cdot1-2\cdot1)+(-1)(1(-1)-3\cdot1)=10+1+4=15\neq0$, so a **unique** solution exists. Now eliminate. Augmented matrix, swap to put a $1$-pivot on top ($R_1\leftrightarrow R_2$):

$$
\left[\begin{array}{ccc|c}1&3&2&13\\2&1&-1&1\\1&-1&1&2\end{array}\right]
\xrightarrow[\;R_3\to R_3-R_1\;]{R_2\to R_2-2R_1}
\left[\begin{array}{ccc|c}1&3&2&13\\0&-5&-5&-25\\0&-4&-1&-11\end{array}\right]
$$

Scale $R_2\to -\tfrac15 R_2=(0,1,1\mid5)$, then $R_3\to R_3+4R_2=(0,0,3\mid9)$:

$$
\left[\begin{array}{ccc|c}1&3&2&13\\0&1&1&5\\0&0&3&9\end{array}\right]
$$

**Back-substitute:** $3z=9\Rightarrow z=3$; $y+z=5\Rightarrow y=2$; $x+3y+2z=13\Rightarrow x=13-6-6=1$.
$$\boxed{(x,y,z)=(1,2,3).}$$

### Problem 2 (MCQ, the canonical parameter problem) — classify by $(\lambda,\mu)$
For the system
$$
x+y+z=6,\qquad x+2y+3z=10,\qquad x+2y+\lambda z=\mu,
$$
find conditions on $\lambda,\mu$ for (i) a unique solution, (ii) no solution, (iii) infinitely many.

**Solution.** Eliminate with $R_2\to R_2-R_1$, $R_3\to R_3-R_1$, then $R_3\to R_3-R_2$:

$$
\left[\begin{array}{ccc|c}1&1&1&6\\0&1&2&4\\0&0&\lambda-3&\mu-10\end{array}\right]
$$

The last row is $[\,0\ 0\ (\lambda-3)\mid(\mu-10)\,]$. Read off:
- **(i) Unique** $\iff \lambda\neq3$ (then $z$ has a pivot), for *any* $\mu$.
- **(ii) No solution** $\iff \lambda=3$ **and** $\mu\neq10$ (row becomes $[0\,0\,0\mid\text{nonzero}]$ → inconsistent).
- **(iii) Infinitely many** $\iff \lambda=3$ **and** $\mu=10$ (row is all zero → one free variable, $n-r=3-2=1$).

### Problem 3 (NAT/descriptive) — infinitely many: write the full solution $x_p+x_h$
Solve
$$
x_1+x_2+2x_3=4,\qquad 2x_1+3x_2+3x_3=9.
$$

**Solution.** $R_2\to R_2-2R_1$ gives REF
$$
\left[\begin{array}{ccc|c}1&1&2&4\\0&1&-1&1\end{array}\right].
$$
Pivots in $x_1,x_2$; $x_3$ is **free**. $\operatorname{rank}A=\operatorname{rank}[A\mid b]=2<3$ ⇒ **infinitely many** (one parameter). Let $x_3=t$:
$$
x_2=1+t,\qquad x_1=4-x_2-2x_3=4-(1+t)-2t=3-3t.
$$
$$
\boxed{\begin{pmatrix}x_1\\x_2\\x_3\end{pmatrix}=\underbrace{\begin{pmatrix}3\\1\\0\end{pmatrix}}_{x_p}+\,t\underbrace{\begin{pmatrix}-3\\1\\1\end{pmatrix}}_{x_h},\quad t\in\mathbb{R}.}
$$
Check $x_h$ solves $Ax=0$: $-3+1+2=0$ and $-6+3+3=0$ ✓. The direction $(-3,1,1)$ spans the **null space** (dimension $n-r=1$) — exactly the object Topic 7 names.

### Problem 4 (MSQ, trap + Topic-2 callback) — homogeneous nontrivial solutions
For which $\lambda$ does
$$
\lambda x+y+z=0,\qquad x+\lambda y+z=0,\qquad x+y+\lambda z=0
$$
have a **nonzero** solution, and how large is the solution space in each case?

**Solution.** A homogeneous square system has a nontrivial solution **iff $\det A=0$**. This is the **same matrix** as Topic 2, Problem 4, where we found
$$
\det A=(\lambda+2)(\lambda-1)^2.
$$
So nontrivial solutions exist $\iff\boxed{\lambda=1\ \text{or}\ \lambda=-2}$. The *size* of the solution space depends on **rank**, not just on $\det=0$:

- **$\lambda=1$:** the matrix is all-ones, $\operatorname{rank}=1$, so $n-r=3-1=2$ → the solution space is **2-dimensional** (the single equation $x+y+z=0$).
- **$\lambda=-2$:** $\operatorname{rank}=2$, so $n-r=1$ → the solution space is **1-dimensional**, spanned by $(1,1,1)$ (check: $-2+1+1=0$ ✓).

**Takeaway:** $\det=0$ only guarantees *some* nontrivial solution; the *number of free parameters* is $n-\operatorname{rank}$.

---

## 6. Common GATE question types on this topic

- **Solve** a $3\times3$ (occasionally $4\times4$) system by elimination + back substitution.
- **Classify** a parameterized system: find values of $\lambda,\mu,k$ giving unique / no / infinitely many solutions (the Problem-2 pattern — extremely frequent).
- **Homogeneous nontrivial-solution** condition: set $\det A=0$ (square) or $\operatorname{rank}<n$.
- **Count free parameters / dimension of the solution space** $=n-\operatorname{rank}$.
- **Consistency check** via $\operatorname{rank}(A)$ vs $\operatorname{rank}([A\mid b])$.
- **Conceptual:** which EROs preserve solutions/rank/determinant; why partial pivoting; identify REF vs RREF.
- **Express the general solution** in the form $x_p+x_h$.

---

## 7. Traps, pitfalls & common mistakes

- **$\det A=0$ does *not* mean "no solution."** It means *not unique* — either none **or** infinitely many. You must check $\operatorname{rank}([A\mid b])$ to decide which.
- **Free variables count $=n-\operatorname{rank}$**, where $n$ is the number of **unknowns** (columns of $A$), *not* the number of equations.
- **Rank of $A$ vs rank of $[A\mid b]$.** Compute ranks on the **augmented** matrix for consistency; using only $A$ misses inconsistency.
- **Pivot of zero:** you cannot eliminate with a zero pivot — you must **swap** (partial pivoting). Forgetting this stalls the algorithm or hides a solution.
- **Sign/arithmetic slips** in replacement steps; mislabeling which row got modified.
- **RREF is unique, an REF is not** — don't claim "the" echelon form for plain REF.
- **Reading a $[\,0\cdots0\mid 0\,]$ row as a contradiction** — it's harmless (a dependent equation); only $[\,0\cdots0\mid c\,]$, $c\neq0$, signals inconsistency.
- **Homogeneous systems are never inconsistent** — $x=0$ always works; the only question is trivial-only vs nontrivial.
- **More unknowns than equations + homogeneous ⇒ always nontrivial** ($r\le m<n$); a frequent shortcut students forget.
- **Cramer's rule misuse:** it applies *only* when $A$ is square and $\det A\neq0$.

---

## 8. Connection forward

Look back at Problem 1: forward elimination used multipliers ($R_2\to R_2-2R_1$, etc.) to turn $A$ into an upper-triangular $U$. If we **record those multipliers** in a lower-triangular matrix $L$ (with $1$s on the diagonal), we get the factorization $A=LU$ — and any required row swaps get packaged into a permutation matrix, giving $PA=LU$. That is **LU Decomposition** (next topic). Its payoff is exactly the cost observation of Section 4.7: the $O(n^3)$ elimination is done **once**, and afterward each new right-hand side $b$ is solved by two cheap $O(n^2)$ triangular sweeps ($Ly=b$ then $Ux=y$).

---

## 9. One-line cheat-sheet summary

**Elimination simplifies $Ax=b$ without changing its solutions; the number of pivots is $\operatorname{rank}(A)$, and a system is inconsistent iff $\operatorname{rank}(A)<\operatorname{rank}([A\mid b])$, unique iff both equal $n$, and has $\infty$-many with $n-\operatorname{rank}$ free parameters otherwise — homogeneous systems are always consistent and have nontrivial solutions iff $\operatorname{rank}<n$ (i.e. $\det A=0$ when square).**

---

### ▶ Next topic (in order): **LU Decomposition**
*(Gaussian elimination written as $A=LU$ / $PA=LU$, so the elimination work is stored once and reused for many right-hand sides.)*

# Topic 12 — Singular Value Decomposition (SVD)
*Subject: Linear Algebra · GATE Data Science & AI (DA)*

---

## 0. Topic header & syllabus map

- **Topic:** Singular Value Decomposition (SVD)
- **Subject:** Linear Algebra
- **Where it sits:** The **final topic** of the *"Eigenvalues, special matrices, forms & SVD"* cluster — and the **last topic of the whole Linear Algebra subject**. It is the **capstone** that unifies everything before it: it **generalizes the eigendecomposition (Topic 8)** to *any* matrix, it is **built from the PSD quadratic forms $A^TA$ and $AA^T$ (Topic 11)**, it **generalizes least squares and projections (Topic 10)** through the pseudoinverse, and it **exposes the four fundamental subspaces (Topic 7)** with orthonormal bases. After this, linear algebra is complete and we step into Machine Learning.

---

## 1. Why this topic exists (motivation + connection)

The eigendecomposition is powerful but **limited**: it only applies to **square** matrices, and even then only to **diagonalizable** ones (defective matrices from Topic 8 have no eigenbasis), and it can drag in complex numbers. But real data matrices are almost always **rectangular** — $m$ samples $\times$ $n$ features — and we still need to find their dominant directions, compress them, invert them approximately, and read off their rank and subspaces. **SVD is the universal tool:** it factors *any* matrix, always, into orthonormal bases connected by real, nonnegative stretch factors.

**What the previous topics couldn't do / what this adds.**
- **Topic 8** (eigendecomposition) **fails** on rectangular matrices and on defective square ones. SVD always exists.
- **Topic 10** (least squares) needed $A^TA$ invertible (full column rank); it **broke** for rank-deficient or non-square $A$. SVD's **pseudoinverse** solves least squares even then.
- **Topic 11** showed $A^TA$ is symmetric **PSD** with nonnegative eigenvalues but didn't yet exploit it. SVD is exactly that exploitation: $\sigma_i=\sqrt{\lambda_i(A^TA)}$ are real and nonnegative *because* $A^TA$ is PSD.

**Forward.** This **completes the linear-algebra subject**. SVD is the computational engine of **PCA**, dimensionality reduction, denoising, image/data compression, latent-factor (recommender) models, and the pseudoinverse — i.e. the linear-algebra spine of the Machine Learning topics ahead.

---

## 2. Prerequisites

- **Eigenvalues & eigenvectors (Topic 8):** SVD is assembled from eigendecompositions of $A^TA$ and $AA^T$.
- **Symmetric matrices, spectral theorem, orthogonal matrices (Topic 9):** $A=Q\Lambda Q^T$; orthogonal $U,V$.
- **Quadratic forms / PSD (Topic 11):** $A^TA$, $AA^T$ are symmetric **PSD** ⇒ real, **nonnegative** eigenvalues ⇒ real singular values.
- **Projections & least squares (Topic 10):** the pseudoinverse generalizes the normal-equations solution; $AA^+$, $A^+A$ are projections.
- **Rank, nullity, four fundamental subspaces (Topic 7).**
- **Matrix norms** (introduced here): spectral $\lVert\cdot\rVert_2$ and Frobenius $\lVert\cdot\rVert_F$.

---

## 3. Core idea — intuition first

**Geometric picture (rotate–stretch–rotate).** Every linear map, however messy, does just three simple things in sequence:
$$A=U\,\Sigma\,V^{T}:\qquad \underbrace{V^T}_{\text{rotate input}}\ \to\ \underbrace{\Sigma}_{\text{stretch along axes by }\sigma_i}\ \to\ \underbrace{U}_{\text{rotate output}}.$$
Feed in the unit circle/sphere of all unit vectors; $A$ maps it to an **ellipse/ellipsoid**. The **singular values $\sigma_i$ are the semi-axis lengths**, and the **left singular vectors $u_i$ are the axis directions**. The **right singular vectors $v_i$ are the input directions** that get mapped onto those axes.

**Versus eigendecomposition.** Eigen-analysis uses **one** basis (eigenvectors, not necessarily orthogonal), can be complex, can involve shears (defective), and needs a square matrix. SVD uses **two orthonormal bases** — input $v_i$ and output $u_i$ — linked by real nonnegative stretches $\sigma_i$. Clean, real, and universal.

**PCA flavour.** The singular vectors are the *natural directions* of the data; the singular values say how much *spread/energy* lives in each direction. Keep the big ones, discard the small ones → compression with minimal loss.

---

## 4. Formal treatment

### 4A. Statement and components

Every real $m\times n$ matrix $A$ factors as
$$\boxed{A=U\,\Sigma\,V^{T}}$$
where
- $U$ is $m\times m$ **orthogonal** ($U^TU=I_m$); its columns $u_1,\dots,u_m$ are the **left singular vectors**;
- $V$ is $n\times n$ **orthogonal** ($V^TV=I_n$); its columns $v_1,\dots,v_n$ are the **right singular vectors**;
- $\Sigma$ is $m\times n$ "diagonal," with **singular values** $\sigma_1\ge\sigma_2\ge\dots\ge\sigma_r>0=\dots=0$ on its diagonal, sorted **descending and nonnegative**, where $r=\text{rank}(A)$.

This decomposition **exists for any matrix** — square or rectangular, any rank.

### 4B. The bridge from quadratic forms — $A^TA$ and $AA^T$

Substituting $A=U\Sigma V^T$:
$$A^TA=V(\Sigma^T\Sigma)V^T\qquad\text{and}\qquad AA^T=U(\Sigma\Sigma^T)U^T.$$
These are the **eigendecompositions** of two symmetric PSD matrices, so:
- the eigenvalues of $A^TA$ (size $n\times n$) and of $AA^T$ (size $m\times m$) are the $\sigma_i^2$;
- $\boxed{\sigma_i=\sqrt{\lambda_i(A^TA)}}$ (real and $\ge0$ because the forms are PSD);
- the columns of $V$ are the orthonormal **eigenvectors of $A^TA$** (right); the columns of $U$ are the orthonormal **eigenvectors of $AA^T$** (left);
- $A^TA$ and $AA^T$ have the **same nonzero eigenvalues** (they differ only in how many zero eigenvalues — $n-r$ vs $m-r$).

**Fundamental coupling relation:**
$$A v_i=\sigma_i\,u_i,\qquad A^T u_i=\sigma_i\,v_i\qquad(i=1,\dots,r).$$

### 4C. How to compute an SVD (standard procedure)

1. Form $A^TA$ (or $AA^T$ — pick the **smaller** of the two: $n\times n$ if $n<m$).
2. Find its eigenvalues $\Rightarrow$ singular values $\sigma_i=\sqrt{\lambda_i}$, sorted descending.
3. Find orthonormal eigenvectors of $A^TA\Rightarrow$ these are the $v_i$ (columns of $V$). *(If you used $AA^T$, you get the $u_i$ instead.)*
4. For each nonzero $\sigma_i$, set $u_i=\dfrac{1}{\sigma_i}A v_i$ (these come out orthonormal). Complete to an orthonormal basis of $\mathbb R^m$ for the remaining columns (the left null space).
5. Assemble $A=U\Sigma V^T$.

### 4D. Three forms of SVD

- **Full SVD:** $U$ is $m\times m$, $\Sigma$ is $m\times n$, $V$ is $n\times n$.
- **Reduced / thin / compact SVD:** keep only the $r$ nonzero singular values — $A=U_r\Sigma_r V_r^T$ with $U_r$ ($m\times r$), $\Sigma_r$ ($r\times r$, positive diagonal), $V_r$ ($n\times r$).
- **Outer-product (dyadic) form** — the one used for approximation:
$$A=\sum_{i=1}^{r}\sigma_i\,u_iv_i^{T}\quad(\text{sum of }r\text{ rank-1 matrices, weighted by }\sigma_i).$$

### 4E. Key properties and quantities (with conditions)

| Quantity / fact | Value / condition |
|---|---|
| $\text{rank}(A)$ | $=$ number of **nonzero** singular values $=r$ |
| Spectral norm $\lVert A\rVert_2$ | $=\sigma_1$ (largest singular value) |
| Frobenius norm $\lVert A\rVert_F$ | $=\sqrt{\sigma_1^2+\dots+\sigma_r^2}=\sqrt{\text{tr}(A^TA)}$ |
| $\lvert\det A\rvert$ (square $A$) | $=\sigma_1\sigma_2\cdots\sigma_n$ (the *absolute* value) |
| Condition number $\kappa_2(A)$ | $=\sigma_1/\sigma_r$ (largest over smallest **nonzero**) |
| Symmetric $A$ | $\sigma_i=\lvert\lambda_i\rvert$ (absolute eigenvalues) |
| Symmetric **PSD** $A$ | $\sigma_i=\lambda_i$ (singular values $=$ eigenvalues) |
| Orthogonal $Q$ | all $\sigma_i=1$ |
| Uniqueness | the $\sigma_i$ are **unique**; $U,V$ are **not** (sign flips; rotations within repeated-$\sigma$ subspaces) |

### 4F. The four fundamental subspaces from SVD (ties to Topic 7)

With $r=\text{rank}(A)$:
- **Column space** $\text{Col}(A)=\text{span}\{u_1,\dots,u_r\}$ — $\dim r$.
- **Left null space** $\text{Null}(A^T)=\text{span}\{u_{r+1},\dots,u_m\}$ — $\dim m-r$.
- **Row space** $\text{Col}(A^T)=\text{span}\{v_1,\dots,v_r\}$ — $\dim r$.
- **Null space** $\text{Null}(A)=\text{span}\{v_{r+1},\dots,v_n\}$ — $\dim n-r$.

SVD hands you **orthonormal bases for all four at once**, and $r+(n-r)=n$ recovers rank–nullity.

### 4G. Low-rank approximation — the Eckart–Young–Mirsky theorem (key ML result)

The **best rank-$k$ approximation** of $A$ (in *both* the spectral and Frobenius norm) keeps the top $k$ terms:
$$A_k=\sum_{i=1}^{k}\sigma_i u_iv_i^{T},\qquad \lVert A-A_k\rVert_2=\sigma_{k+1},\qquad \lVert A-A_k\rVert_F=\sqrt{\sigma_{k+1}^2+\dots+\sigma_r^2}.$$
This is the mathematical foundation of dimensionality reduction, image/data compression, denoising, latent semantic analysis, and PCA.

### 4H. The pseudoinverse (Moore–Penrose) — least squares, generalized (ties to Topic 10)

$$A^{+}=V\,\Sigma^{+}U^{T},\qquad \Sigma^{+}=\text{(reciprocate each nonzero }\sigma_i,\text{ then transpose to }n\times m).$$
- If $A$ has **full column rank**: $A^{+}=(A^TA)^{-1}A^T$ — exactly the least-squares operator from Topic 10.
- If $A$ has **full row rank**: $A^{+}=A^T(AA^T)^{-1}$.
- In **general** (any shape, any rank), $\hat x=A^{+}b$ is the **minimum-norm least-squares solution** of $Ax=b$ — it works even when $(A^TA)^{-1}$ does not exist.
- $AA^{+}=$ orthogonal projection onto $\text{Col}(A)$; $A^{+}A=$ orthogonal projection onto $\text{Row}(A)$.

### 4I. PCA connection (brief, ML-facing)

Center the data matrix $X$ (subtract the column means). The **principal directions are the right singular vectors of the centered $X$** (equivalently, the eigenvectors of the covariance $\frac{1}{n-1}X^TX$), and $\frac{\sigma_i^2}{n-1}$ are the **variances** along those directions. So **SVD of the centered data is PCA**.

### 4J. SVD vs eigendecomposition (quick comparison)

| | Eigendecomposition $A=PDP^{-1}$ | SVD $A=U\Sigma V^T$ |
|---|---|---|
| Applies to | square **and diagonalizable** | **any** $m\times n$ matrix |
| Bases used | one (eigenvectors; need not be orthogonal) | **two orthonormal** ($U$, $V$) |
| Diagonal entries | eigenvalues (real/complex, any sign) | singular values (**real, $\ge0$**) |
| For symmetric PSD | the two coincide | the two coincide |

---

## 5. Worked problems (GATE-style)

### Problem 1 (compute a full SVD) — the canonical $2\times3$ example
Find the SVD of $A=\begin{bmatrix}3&2&2\\2&3&-2\end{bmatrix}$.

**Solution.** Here $m=2<n=3$, so use the smaller $AA^T$ (a $2\times2$):
$$AA^T=\begin{bmatrix}17&8\\8&17\end{bmatrix},\quad\text{eigenvalues }17\pm8=25,\,9\ \Rightarrow\ \sigma_1=5,\ \sigma_2=3.$$
Eigenvectors of $AA^T$ (these are the $u_i$): for $25$, $u_1=\tfrac1{\sqrt2}[1,1]^T$; for $9$, $u_2=\tfrac1{\sqrt2}[1,-1]^T$. So
$$U=\frac1{\sqrt2}\begin{bmatrix}1&1\\1&-1\end{bmatrix},\qquad \Sigma=\begin{bmatrix}5&0&0\\0&3&0\end{bmatrix}.$$
Right singular vectors via $v_i=\tfrac1{\sigma_i}A^Tu_i$:
$$v_1=\tfrac1{5}A^Tu_1=\tfrac1{\sqrt2}[1,1,0]^T,\qquad v_2=\tfrac1{3}A^Tu_2=\tfrac1{3\sqrt2}[1,-1,4]^T.$$
The third right vector spans $\text{Null}(A)$ (solve $Ax=0$): $v_3=\tfrac13[-2,2,1]^T$. Thus $V=[\,v_1\ v_2\ v_3\,]$ and $A=U\Sigma V^T$.
*Read-offs:* $\text{rank}(A)=2$ (two nonzero $\sigma$); $\lVert A\rVert_2=5$; $\lVert A\rVert_F=\sqrt{25+9}=\sqrt{34}$; $\text{Null}(A)=\text{span}\{[-2,2,1]^T\}$. *(Checks: $v_1\!\cdot\!v_3=v_2\!\cdot\!v_3=0$, and each $v_i$ is a unit vector. ✔)*

### Problem 2 (NAT) — singular values from eigenvalues + norms
A symmetric matrix $A$ has eigenvalues $3,\,-4,\,0$. Find its singular values, rank, spectral norm, and Frobenius norm.

**Solution.** For symmetric $A$, $\sigma_i=\lvert\lambda_i\rvert$: $\{|3|,|-4|,|0|\}=\{3,4,0\}$, sorted $\boxed{4,3,0}$. Rank $=2$ (two nonzero). $\lVert A\rVert_2=\sigma_1=4$. $\lVert A\rVert_F=\sqrt{4^2+3^2}=\sqrt{25}=5$.
*(Note: $\sigma=|\lambda|$, **not** $\lambda$ — the $-4$ contributes a singular value $+4$.)*

### Problem 3 (NAT) — best low-rank approximation (Eckart–Young)
A matrix has singular values $\sigma=6,4,3,1$. For the best rank-2 approximation $A_2$, find $\lVert A-A_2\rVert_2$ and $\lVert A-A_2\rVert_F$.

**Solution.** Keep $\sigma_1=6,\sigma_2=4$; drop $\sigma_3=3,\sigma_4=1$.
$$\lVert A-A_2\rVert_2=\sigma_3=3,\qquad \lVert A-A_2\rVert_F=\sqrt{\sigma_3^2+\sigma_4^2}=\sqrt{9+1}=\sqrt{10}.$$
*(Frobenius error is the root-sum-**square** of dropped singular values, not their sum.)*

### Problem 4 (NAT) — pseudoinverse and minimum-norm least squares
For $A=\begin{bmatrix}3&0\\0&0\end{bmatrix}$, find $A^{+}$ and the minimum-norm least-squares solution of $Ax=b$ with $b=\begin{bmatrix}6\\5\end{bmatrix}$.

**Solution.** $A$ is already diagonal with $U=V=I$, $\sigma_1=3$, $\sigma_2=0$. Reciprocate the nonzero singular value:
$$A^{+}=\begin{bmatrix}\tfrac13&0\\0&0\end{bmatrix},\qquad \hat x=A^{+}b=\begin{bmatrix}\tfrac13&0\\0&0\end{bmatrix}\begin{bmatrix}6\\5\end{bmatrix}=\begin{bmatrix}2\\0\end{bmatrix}.$$
*Why this is correct:* $A\hat x=[6,0]^T$ is the projection of $b$ onto $\text{Col}(A)$ (the $x$-axis), with residual $[0,5]^T\perp\text{Col}(A)$. *Every* $x=[2,t]^T$ achieves the same minimal residual, and $\hat x=[2,0]^T$ is the one of **smallest norm** ($t=0$). The pseudoinverse picks exactly that one — something $(A^TA)^{-1}A^T$ can't do here, since $A^TA=\begin{bmatrix}9&0\\0&0\end{bmatrix}$ is singular.

### Problem 5 (MSQ) — concepts & traps
Which statements are **always true**?
(i) Every real matrix has an SVD.
(ii) The singular values of $A$ are the eigenvalues of $A^TA$.
(iii) For a symmetric positive definite matrix, the singular values equal the eigenvalues.
(iv) $\text{rank}(A)=$ the number of nonzero singular values.
(v) $U$ and $V$ in $A=U\Sigma V^T$ are uniquely determined.
(vi) $\lVert A\rVert_2$ equals the largest singular value.
(vii) The nonzero eigenvalues of $A^TA$ and $AA^T$ are identical.

**Solution.**
- **(i) TRUE** — SVD exists for every matrix.
- **(ii) FALSE** — singular values are the **square roots** of the eigenvalues of $A^TA$.
- **(iii) TRUE** — PSD ⇒ $\sigma_i=\lambda_i$ (eigenvalues already $\ge0$).
- **(iv) TRUE.**
- **(v) FALSE** — only $\Sigma$ (the sorted singular values) is unique; $U,V$ have sign/rotation freedom.
- **(vi) TRUE.**
- **(vii) TRUE** — both equal the $\sigma_i^2$; they differ only in the count of zero eigenvalues.

**Correct: (i), (iii), (iv), (vi), (vii).**

---

## 6. Common GATE question types on this topic

- **Compute an SVD** (or just the singular values) of a small matrix via $A^TA$ or $AA^T$.
- **Singular values from eigenvalues** for a symmetric matrix ($\sigma=|\lambda|$).
- **Rank from singular values**; **spectral norm** $\sigma_1$; **Frobenius norm** $\sqrt{\sum\sigma_i^2}$; $\lvert\det\rvert=\prod\sigma_i$.
- **Low-rank approximation error** (Eckart–Young) in spectral / Frobenius norm.
- **Pseudoinverse** and **minimum-norm least-squares** solutions (including rank-deficient $A$).
- **Four fundamental subspaces** read off from columns of $U$ and $V$.
- **Conceptual:** existence, non-uniqueness of $U,V$, relation to eigendecomposition, condition number.
- **PCA:** principal directions $=$ right singular vectors of centered data.

---

## 7. Traps, pitfalls & common mistakes

- **Singular values are $\sqrt{\lambda(A^TA)}$, not $\lambda(A^TA)$** — forgetting the square root is the single most common error.
- **Singular values are always real and $\ge0$**, even when $A$ has negative or complex eigenvalues. In general $\sigma_i\neq\lambda_i$.
- **For symmetric $A$, $\sigma_i=\lvert\lambda_i\rvert$ (absolute value)** — they equal the eigenvalues *only* when $A$ is PSD.
- **$U$ and $V$ are not unique** (sign flips, and rotations within subspaces of repeated $\sigma$). Only the set of singular values is unique. Don't assert a "the" SVD.
- **$\Sigma$ has the same shape as $A$ ($m\times n$)** — it need not be square; $\Sigma^+$ is $n\times m$.
- **Use the smaller of $A^TA$ ($n\times n$) and $AA^T$ ($m\times m$)** to compute — they share nonzero eigenvalues but differ in the number of zeros.
- **$\lvert\det A\rvert=\prod\sigma_i$ gives only the magnitude** — the sign of $\det$ comes from the orientations of $U,V$.
- **Eckart–Young:** the **Frobenius** error is the root-sum-**square** of dropped singular values; the **spectral** error is just $\sigma_{k+1}$ (the largest dropped one). Don't sum the dropped $\sigma$ directly.
- **$u_i=\frac1{\sigma_i}Av_i$ only for nonzero $\sigma_i$** — you cannot divide by zero; the left vectors for $\sigma=0$ come from completing the basis (the left null space).
- **Condition number uses the smallest *nonzero* singular value $\sigma_r$**, not $\sigma_n$ (which may be $0$ — that would make the matrix singular, $\kappa=\infty$).
- **The pseudoinverse always solves least squares; $(A^TA)^{-1}A^T$ does so only for full column rank.**

---

## 8. Connection forward — and the subject wrap-up

**This completes the Linear Algebra subject.** SVD is the keystone that ties the cluster together:
- it **subsumes the eigendecomposition** (for symmetric PSD matrices the two coincide; for symmetric matrices $\sigma=|\lambda|$);
- it **generalizes projections and least squares** (Topic 10) into the **pseudoinverse**, valid for any shape and rank;
- it **rests on PSD quadratic forms** (Topic 11), since $\sigma_i=\sqrt{\lambda_i(A^TA)}$ are real and nonnegative precisely because $A^TA$ is PSD;
- it **lays bare the four fundamental subspaces** (Topic 7) with orthonormal bases.

**Into Machine Learning.** The ML topics ahead run on exactly this machinery: **covariance matrices** are symmetric PSD quadratic forms; **PCA** is the SVD of centered data; **linear regression** is projection / least squares / the pseudoinverse; and **gradient-based optimization** turns on Hessian definiteness. The whole linear-algebra arc — matrices → systems → vector spaces → rank → eigenvalues → special matrices → projections → quadratic forms → SVD — is the foundation the rest of the syllabus is built on.

---

## 9. One-line cheat-sheet summary

$A=U\Sigma V^T$ for **any** $m\times n$ matrix ($U,V$ orthogonal; $\Sigma$ diagonal $\sigma_1\ge\dots\ge\sigma_r>0$). $\sigma_i=\sqrt{\lambda_i(A^TA)}$; $v_i=$ eigenvectors of $A^TA$, $u_i=Av_i/\sigma_i$. $\text{rank}=\#$nonzero $\sigma$; $\lVert A\rVert_2=\sigma_1$; $\lVert A\rVert_F=\sqrt{\sum\sigma_i^2}$; $\lvert\det\rvert=\prod\sigma_i$. Best rank-$k$ $=$ top-$k$ terms of $\sum\sigma_iu_iv_i^T$ (errors $\sigma_{k+1}$ / $\sqrt{\sum_{i>k}\sigma_i^2}$). Pseudoinverse $A^+=V\Sigma^+U^T$ gives the min-norm least-squares solution. **Symmetric ⇒ $\sigma=|\lambda|$; PSD ⇒ $\sigma=\lambda$.**

---

### ✅ Linear Algebra complete
That closes the **Eigenvalues, Special Matrices, Forms & SVD** cluster and the entire **Linear Algebra** subject (Topics 1–12). When you're ready to move on, tell me the next subject and topic (e.g., *"Subject: Machine Learning. Topic: …"*) and I'll continue in the same format.

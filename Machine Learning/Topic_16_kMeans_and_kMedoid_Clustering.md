# Topic 16 — k-Means & k-Medoid Clustering

**Subject:** Machine Learning · **Section:** Unsupervised Learning

> **Scope:** This is a *full computational* topic, not an overview. GATE tests k-means by making you **run the assign–update loop by hand**, compute **SSE**, reason about **convergence / local optima / initialization**, and contrast **centroid vs medoid**. All of that is here, with fully worked iterations.

---

## 0. Topic header & syllabus map

- **Topic:** k-Means and k-Medoid (k-Medoids / PAM) — the two core **partitional** clustering algorithms.
- **Where it sits:** Comes immediately **after** Clustering Algorithms (Topic 15), which gave the framing + distance measures but **no actual algorithm**. It is the **first concrete clustering procedure**. It **leads into** Hierarchical Clustering (Topic 17), which drops the "pick $k$ upfront, flat partition" idea in favour of a nested tree.
- **One line:** *Pick $k$ representatives, assign every point to its nearest one, move the representatives, repeat until nothing changes.*

---

## 1. Why this topic exists (motivation + connection)

The overview (Topic 15) told you **what** a cluster is (high cohesion, low separation) and **how to measure** similarity (Euclidean, Manhattan, cosine, …). What it did **not** give you is a **procedure that actually outputs the groups**. k-Means is that procedure — the simplest, fastest, most-used partitional algorithm.

**The bridge from the overview:** Topic 15 ended with the cohesion objective
$\mathrm{SSE}=\sum_j\sum_{\mathbf{x}\in C_j}\|\mathbf{x}-\boldsymbol{\mu}_j\|^2$.
k-Means is precisely a **greedy iterative minimizer of that SSE**.

**Why k-Medoid then exists (k-Means → k-Medoid bridge):** k-Means has two structural weaknesses baked into its choices:
1. It uses the **mean** as a cluster centre. The mean is **dragged by outliers** — one extreme point can pull the centroid far from the bulk.
2. It is tied to **squared Euclidean** geometry, so it assumes roughly **spherical** clusters and needs **numeric** coordinates to take a mean.

**k-Medoid fixes both:** it forces the cluster centre to be an **actual data point (a medoid)**, minimizing a **sum of (un-squared) dissimilarities**. That makes it **robust to outliers** and lets it run on **any distance/dissimilarity matrix** (even non-Euclidean or categorical), at the cost of being **slower**.

So the story is: *overview defines the goal → k-means greedily minimizes SSE → k-medoid hardens k-means against outliers and non-Euclidean data.*

---

## 2. Prerequisites

- **Distance measures & the four metric axioms** (Topic 15) — k-means uses (squared) Euclidean; k-medoid can use any.
- **Centroid / mean** of a point set: $\boldsymbol{\mu}=\frac1m\sum_i\mathbf{x}_i$.
- **SSE / within-cluster sum of squares** (Topic 15 §4.4).
- **Argmin / nearest-point reasoning** (as in k-NN, Topic 6).
- Light calculus (one short gradient, to see *why* the centre is the mean).

---

## 3. Core idea — intuition first

**Analogy (k-means).** You want to place $k$ ice-cream carts in a park so everyone walks as little as possible. You don't know the best spots, so:
1. Drop the carts anywhere ($k$ initial centres).
2. Every visitor goes to the **nearest** cart (assignment step).
3. Each cart **slides to the average position** of the visitors who came to it (update step).
4. Visitors re-pick their nearest cart; carts move again. Repeat. Soon nobody switches carts and the carts stop moving — **converged**.

That alternating "**assign points → move centres**" loop is the whole algorithm. Each half-step can only **reduce** total walking distance (SSE), so it always settles.

**Intuition for k-medoid.** Same loop, but a cart **must sit exactly on top of a visitor** (a real data point), not float at an average location. To pick the best cart spot you ask: *"if this person were the cart, what's the total distance everyone walks?"* and choose the person that minimizes it. Because a real point can't fly off to infinity the way an average can, a lone weirdo far away (outlier) **cannot hijack** the centre — that's the robustness.

**The key mental contrast to lock in now:**

| | **Centroid (k-means)** | **Medoid (k-medoid)** |
|---|---|---|
| Centre is | the **mean** (often *not* a data point) | an **actual data point** |
| Minimizes | sum of **squared** distances | sum of **(un-squared)** distances |
| Outliers | **sensitive** (mean gets pulled) | **robust** |
| Distance allowed | needs Euclidean-style/numeric | **any** dissimilarity |
| Speed | **fast** | **slow** |

---

## 4. Formal treatment

### 4.1 k-Means: objective and algorithm

Given points $\{\mathbf{x}_1,\dots,\mathbf{x}_n\}\subset\mathbb{R}^d$ and a fixed integer $k$, k-means seeks a partition $C_1,\dots,C_k$ and centres $\boldsymbol{\mu}_1,\dots,\boldsymbol{\mu}_k$ minimizing

$$
\boxed{\;J=\mathrm{SSE}=\sum_{j=1}^{k}\sum_{\mathbf{x}\in C_j}\bigl\|\mathbf{x}-\boldsymbol{\mu}_j\bigr\|^2\;}
\qquad
\begin{aligned}&\text{numeric features; uses (squared)}\\&\text{Euclidean distance; }k\text{ fixed in advance.}\end{aligned}
$$

**Lloyd's algorithm (standard k-means):**

```
1. Initialize k centres μ₁…μ_k   (random points, or k-means++)
2. REPEAT until assignments stop changing (or max iters):
   (A) Assignment: assign each point xᵢ to the cluster of its
       NEAREST centre:   C(xᵢ) = argmin_j ‖xᵢ − μ_j‖²
   (B) Update: recompute each centre as the MEAN of its members:
       μ_j = (1/|C_j|) Σ_{x∈C_j} x
3. RETURN the partition and centres.
```

> Because $\arg\min_j \|\mathbf{x}-\boldsymbol{\mu}_j\|^2 = \arg\min_j \|\mathbf{x}-\boldsymbol{\mu}_j\|$, you may compare **squared** distances in the assignment step (no square roots needed) — handy and exam-safe.

### 4.2 Why the centre is the *mean* (short derivation GATE may expect)

Fix one cluster with members $\mathbf{x}_1,\dots,\mathbf{x}_m$. Minimize $f(\boldsymbol{\mu})=\sum_{i=1}^m\|\mathbf{x}_i-\boldsymbol{\mu}\|^2$:

$$
\nabla_{\boldsymbol{\mu}} f = \sum_{i=1}^m -2(\mathbf{x}_i-\boldsymbol{\mu}) = 0
\;\Longrightarrow\; \sum_i \mathbf{x}_i = m\,\boldsymbol{\mu}
\;\Longrightarrow\; \boxed{\boldsymbol{\mu}=\frac1m\sum_{i=1}^m \mathbf{x}_i.}
$$

So **for the squared-Euclidean objective, the optimal centre is exactly the mean** — that is *why* the update step averages. (If you instead minimized the sum of *un-squared* distances, the optimum would be the geometric median / medoid — the k-medoid idea.)

### 4.3 Convergence, optimality, complexity (high-yield facts)

- **Monotone decrease:** each step (A) and (B) **cannot increase** SSE. Assignment lowers (or holds) SSE by moving each point to a nearer centre; update lowers (or holds) it because the mean is the SSE-minimizing centre (§4.2).
- **Guaranteed to converge** to a **local minimum** in a **finite** number of steps (there are finitely many possible partitions and SSE never increases). It does **NOT** find the **global** optimum.
- **Initialization-dependent:** different starting centres can give different final clusters and SSE. Standard fix: **run several random restarts, keep the lowest SSE**, or use **k-means++** (§4.4).
- **Complexity:** $O(n\,k\,d)$ per iteration; $O(n\,k\,d\,t)$ for $t$ iterations. Very scalable.
- **Implicit assumptions:** clusters are roughly **spherical, similar size, similar density**; uses Euclidean distance; **sensitive to feature scale** (standardize first) and to **outliers**.
- **Empty-cluster issue:** if a centre attracts no points, that cluster is empty; re-initialize it (e.g. to the point farthest from its centre) to keep $k$ clusters.

### 4.4 Choosing the initialization and $k$

- **k-means++ initialization:** pick the first centre at random, then pick each subsequent centre from the remaining points with probability proportional to its **squared distance to the nearest already-chosen centre**. This spreads centres out, giving better and more reliable solutions than plain random seeding. *(Recognition level — GATE may ask "what does k-means++ improve?" → initialization, not the loop.)*
- **Choosing $k$ (it is an input, not an output):**
  - **Elbow method:** plot SSE vs $k$; SSE always decreases as $k$ grows, so pick the **"elbow"** where the marginal drop flattens.
  - **Silhouette method** (Topic 15 §4.6): pick the $k$ maximizing the average silhouette coefficient.

### 4.5 k-Medoid (PAM — Partitioning Around Medoids)

A **medoid** of a cluster is the **member point** with the **smallest total dissimilarity** to all other members:
$$
\text{medoid}(C)=\arg\min_{\mathbf{p}\in C}\ \sum_{\mathbf{x}\in C} d(\mathbf{x},\mathbf{p}).
$$

The objective is the **total cost** = sum of each point's dissimilarity to its assigned medoid:
$$
\boxed{\;\text{cost}=\sum_{j=1}^{k}\sum_{\mathbf{x}\in C_j} d\bigl(\mathbf{x},\ \mathbf{m}_j\bigr)\;}
\qquad d(\cdot,\cdot)\text{ may be ANY dissimilarity; un-squared.}
$$

**PAM algorithm:**

```
1. Select k initial medoids (actual data points).
2. Assignment: assign each point to its nearest medoid; compute total cost.
3. Swap step: for every (medoid m, non-medoid o) pair, tentatively swap
   (make o a medoid, drop m) and compute the new total cost.
4. Perform the single swap that DECREASES total cost the most.
5. REPEAT 2–4 until no swap reduces the total cost.
```

- **Robust to outliers / noise** (medoid is a real point; uses un-squared distances).
- **Works with any distance** — including non-Euclidean and categorical data via a dissimilarity matrix.
- **More expensive:** the swap step examines $k(n-k)$ candidate swaps per iteration, each needing an $O(n)$ cost re-evaluation, so roughly $O\!\big(k(n-k)^2\big)$ per iteration — much slower than k-means. For large $n$, sampled variants **CLARA / CLARANS** are used *(awareness only)*.

### 4.6 k-Means vs k-Medoid (one-look summary)

| Aspect | **k-Means** | **k-Medoid (PAM)** |
|---|---|---|
| Cluster centre | mean (centroid), may be non-data point | an actual data point (medoid) |
| Objective | $\sum\|\mathbf{x}-\boldsymbol{\mu}\|^2$ (squared) | $\sum d(\mathbf{x},\mathbf{m})$ (un-squared) |
| Distance | (squared) Euclidean | **any** dissimilarity |
| Outlier robustness | low | high |
| Cost per iteration | $O(nkd)$ | $\approx O(k(n-k)^2)$ |
| Best when | large, numeric, roughly spherical data | smaller data, noise/outliers, non-Euclidean |
| Shared limitation | both need **$k$ fixed in advance**, give a **flat hard** partition, find a **local** optimum, are **init-dependent** | same |

---

## 5. Worked problems (GATE-style)

### Problem 1 — Full k-means run in 2-D (NAT / multi-part)

Points $P_1(1,1),\,P_2(2,1),\,P_3(4,3),\,P_4(5,4)$, with $k=2$ and initial centres $\boldsymbol{\mu}_1=(1,1),\ \boldsymbol{\mu}_2=(2,1)$. Run k-means to convergence (compare **squared** Euclidean distances). Give the final centres and final SSE.

**Iteration 1 — assign** (squared distances to $\boldsymbol{\mu}_1=(1,1)$, $\boldsymbol{\mu}_2=(2,1)$):

| point | $d^2$ to $\mu_1$ | $d^2$ to $\mu_2$ | → cluster |
|---|---|---|---|
| $P_1(1,1)$ | $0$ | $1$ | $C_1$ |
| $P_2(2,1)$ | $1$ | $0$ | $C_2$ |
| $P_3(4,3)$ | $9+4=13$ | $4+4=8$ | $C_2$ |
| $P_4(5,4)$ | $16+9=25$ | $9+9=18$ | $C_2$ |

$C_1=\{P_1\},\ C_2=\{P_2,P_3,P_4\}$. **Update:** $\boldsymbol{\mu}_1=(1,1)$, $\boldsymbol{\mu}_2=\big(\tfrac{2+4+5}{3},\tfrac{1+3+4}{3}\big)=\big(\tfrac{11}{3},\tfrac{8}{3}\big)\approx(3.67,2.67)$.

**Iteration 2 — assign** (to $\mu_1=(1,1)$, $\mu_2=(3.67,2.67)$):

| point | $d^2$ to $\mu_1$ | $d^2$ to $\mu_2$ | → cluster |
|---|---|---|---|
| $P_1(1,1)$ | $0$ | $\approx 9.89$ | $C_1$ |
| $P_2(2,1)$ | $1$ | $\approx 5.56$ | $C_1$ |
| $P_3(4,3)$ | $13$ | $\approx 0.22$ | $C_2$ |
| $P_4(5,4)$ | $25$ | $\approx 3.56$ | $C_2$ |

$C_1=\{P_1,P_2\},\ C_2=\{P_3,P_4\}$. **Update:** $\boldsymbol{\mu}_1=(1.5,1)$, $\boldsymbol{\mu}_2=(4.5,3.5)$.

**Iteration 3 — assign** (to $\mu_1=(1.5,1)$, $\mu_2=(4.5,3.5)$): $P_1,P_2\to C_1$; $P_3,P_4\to C_2$ — **assignments unchanged ⇒ converged.**

**Final centres:** $\boxed{\boldsymbol{\mu}_1=(1.5,\,1),\quad \boldsymbol{\mu}_2=(4.5,\,3.5)}$.
**Final SSE:**
$$
\underbrace{(0.25+0.25)}_{C_1:\,P_1,P_2}+\underbrace{(0.5+0.5)}_{C_2:\,P_3,P_4}=0.5+1.0=\boxed{1.5}.
$$

---

### Problem 2 — Why does the update step use the mean? (derivation MCQ)

In k-means, the centre that minimizes the within-cluster sum of **squared** distances is:

(A) the median of the cluster &nbsp; (B) the mean of the cluster &nbsp; (C) any member point &nbsp; (D) the mode

**Solution.** Setting $\nabla_{\boldsymbol{\mu}}\sum_i\|\mathbf{x}_i-\boldsymbol{\mu}\|^2=-2\sum_i(\mathbf{x}_i-\boldsymbol{\mu})=0$ gives $\boldsymbol{\mu}=\frac1m\sum_i\mathbf{x}_i$, the **mean**. **Answer: (B).** *(If the objective were un-squared distances, the answer would be the median/medoid — the k-medoid case.)*

---

### Problem 3 — Centroid vs medoid: outlier robustness (NAT)

For the 1-D points $\{1,2,3,4,100\}$, find (i) the k-means centroid and (ii) the k-medoid medoid (Manhattan distance) of the single cluster. Which represents the bulk better?

**Solution.**
- **Centroid (mean):** $\dfrac{1+2+3+4+100}{5}=\dfrac{110}{5}=\mathbf{22}$ — dragged far from $\{1,2,3,4\}$ by the outlier $100$.
- **Medoid:** total $|$distance$|$ for each candidate:

| candidate | total distance |
|---|---|
| 1 | $0+1+2+3+99=105$ |
| 2 | $1+0+1+2+98=102$ |
| 3 | $2+1+0+1+97=\mathbf{101}$ |
| 4 | $3+2+1+0+96=102$ |
| 100 | $99+98+97+96+0=390$ |

Minimum at **medoid $=3$**, sitting inside the bulk. **The medoid (3) is robust; the centroid (22) is not.** This is exactly why k-medoid resists outliers.

---

### Problem 4 — k-medoid cost and the swap step (NAT / reasoning)

1-D points $\{1,2,3,8,9,10\}$, $k=2$, Manhattan distance. Start with medoids $\{1,9\}$. Show one improving swap and the resulting cost.

**Cost of $\{1,9\}$** (assign each to nearer medoid):

| point | to 1 | to 9 | assigned | cost |
|---|---|---|---|---|
| 1 | 0 | 8 | 1 | 0 |
| 2 | 1 | 7 | 1 | 1 |
| 3 | 2 | 6 | 1 | 2 |
| 8 | 7 | 1 | 9 | 1 |
| 9 | 8 | 0 | 9 | 0 |
| 10 | 9 | 1 | 9 | 1 |

Total cost $=0+1+2+1+0+1=\mathbf{5}$.

**Try swapping medoid $1\to 2$**, new medoids $\{2,9\}$:

| point | to 2 | to 9 | assigned | cost |
|---|---|---|---|---|
| 1 | 1 | 8 | 2 | 1 |
| 2 | 0 | 7 | 2 | 0 |
| 3 | 1 | 6 | 2 | 1 |
| 8 | 6 | 1 | 9 | 1 |
| 9 | 7 | 0 | 9 | 0 |
| 10 | 8 | 1 | 9 | 1 |

Total cost $=1+0+1+1+0+1=\mathbf{4}$. Since $4<5$, the swap **reduces cost → accept it**. No further swap lowers the cost, so PAM converges at medoids $\{2,9\}$, cost $4$, clusters $\{1,2,3\}$ and $\{8,9,10\}$.

---

### Problem 5 — Conceptual trap (MSQ): which statements about k-means are TRUE?

(A) k-means always converges to the global minimum of SSE.
(B) k-means always terminates in finitely many iterations.
(C) The final clustering can depend on the initial centres.
(D) SSE can increase from one iteration to the next.

**Solution.**
- (A) **False** — only a **local** optimum is guaranteed.
- (B) **True** — SSE is non-increasing and partitions are finite, so it halts.
- (C) **True** — different seeds → possibly different local optima (motivates restarts / k-means++).
- (D) **False** — both steps are non-increasing, so SSE never goes up.

**Answer: (B) and (C).** *(The classic GATE trap is (A): "always converges" is true, but "to the global optimum" is the false part — read the full sentence.)*

---

## 6. Common GATE question types on this topic

- **Run k-means by hand:** given points + initial centres, produce assignments / new centroids after $t$ iterations, or the **final** centroids — NAT or MCQ.
- **Compute SSE / WCSS** for a given clustering, or compare SSE of two clusterings.
- **k-medoid cost / swap:** compute total cost for given medoids, or decide whether a proposed swap is accepted.
- **Centroid vs medoid** identification, and **outlier-robustness** reasoning.
- **True/false on properties:** convergence, local vs global optimum, initialization sensitivity, monotone SSE, complexity.
- **"What does k-means++ improve?"** (initialization) / **"How to choose $k$?"** (elbow, silhouette).
- **Why k-means fails** on non-spherical / unequal-density / unscaled data.

---

## 7. Traps, pitfalls & common mistakes

- **"Converges to the global optimum."** No — only a **local** optimum; result depends on the seed. (Most common conceptual trap.)
- **Forgetting SSE uses *squared* distances** while k-medoid cost uses **un-squared** distances. Don't mix the two objectives.
- **Recomputing centroids with old assignments (or vice-versa).** Strict order each round: **assign with current centres → then update centres**. Skipping or reordering changes the answer.
- **Tie in the assignment step** (a point equidistant from two centres): break ties by a stated rule (e.g. lowest index); don't put it in both.
- **Not standardizing features.** k-means distances are scale-sensitive — a large-unit feature dominates and distorts clusters.
- **Treating $k$ as something the algorithm discovers.** $k$ is an **input/hyperparameter**; choose it (elbow/silhouette).
- **Using the mean for k-medoid.** The medoid must be an **actual data point** minimizing total dissimilarity — not the average.
- **Empty clusters** in k-means: a centre with no assigned points must be re-seeded, or you silently lose a cluster.
- **Assuming k-means handles any shape.** It implicitly assumes spherical, comparable-size clusters; elongated, nested, or very different-density clusters break it (density-based methods handle those).
- **Off-by-one / arithmetic in the mean:** divide by the correct cluster size; recompute *both* coordinates in 2-D.

---

## 8. Connection forward

Both algorithms here share two limitations that **Hierarchical Clustering (Topic 17)** removes:
- They need **$k$ chosen in advance** — hierarchical does **not**; it builds a full **dendrogram** and you cut it at any level afterward.
- They produce a single **flat** partition — hierarchical produces a **nested** family of clusterings (from $n$ singletons up to one big cluster).

Hierarchical clustering still **reuses the distance measures from Topic 15**, but adds rules for the distance *between clusters* — **single-, complete-, and average-linkage** — which is the new machinery you'll learn next. Conceptually: *k-means/medoid = "commit to $k$ and optimize"; hierarchical = "don't commit — build the whole merge tree and decide later."*

---

## 9. One-line cheat-sheet summary

> **k-Means:** fix $k$, then loop {assign each point to nearest **centroid** (mean) by squared-Euclidean → recompute centroids}; minimizes $\mathrm{SSE}=\sum\|\mathbf{x}-\boldsymbol\mu\|^2$, converges to a **local** optimum (init-dependent, $O(nkd)$ per iter, outlier-sensitive). **k-Medoid (PAM):** centre is a **real data point** minimizing **un-squared** total dissimilarity (any metric), robust to outliers but slow ($\approx O(k(n-k)^2)$/iter) via the **swap** step.

---

### ➡️ Next topic (in order): **Topic 17 — Hierarchical Clustering** (agglomerative bottom-up vs divisive top-down, single- / complete- / average-linkage, building and cutting a dendrogram, with worked merge tables). Ask for it when ready.

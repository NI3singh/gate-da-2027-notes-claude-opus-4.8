# Topic 17 — Hierarchical Clustering

**Subject:** Machine Learning · **Section:** Unsupervised Learning

> **Scope:** Full computational topic. GATE tests this by handing you a **proximity (distance) matrix** and asking you to run **agglomerative** clustering with a given **linkage**, report the **merge order and merge heights**, build/read the **dendrogram**, and **cut** it for $k$ clusters. All of that is worked below from a single matrix, across single-, complete-, and average-linkage.
>
> **Terminology note (honesty about the syllabus):** the GATE DA syllabus lists "single-linkage, multiple-linkage." In standard usage **single-linkage = MIN** (one nearest pair), while "**multiple**-linkage" is the umbrella for the linkages that aggregate over **many** point-pairs — chiefly **complete-linkage (MAX)** and **average-linkage**. All three are covered here.

---

## 0. Topic header & syllabus map

- **Topic:** Hierarchical clustering — **agglomerative (bottom-up)** and **divisive (top-down)**, with **single-, complete-, and average-linkage**, presented via the **dendrogram**.
- **Where it sits:** Comes **after** k-Means / k-Medoid (Topic 16). Those were **partitional** (commit to $k$, output one flat partition). Hierarchical clustering is the alternative paradigm — **don't fix $k$; build the whole merge tree**. It is the last *clustering* topic; it **leads into** Dimensionality Reduction (Topic 18) and PCA (Topic 19).
- **One line:** *Repeatedly merge the two closest clusters (or recursively split one) to grow a whole tree of clusterings, then cut it wherever you like.*

---

## 1. Why this topic exists (motivation + connection)

k-Means and k-Medoid (Topic 16) share three uncomfortable commitments:
1. **You must pick $k$ in advance.** Wrong $k$ → wrong clustering, and you only find out after.
2. They output a **single flat** partition — no notion of "clusters within clusters."
3. They depend on **random initialization** (k-means) and only find a **local** optimum, so results vary run to run.

**The bridge (k-means → hierarchical).** Hierarchical clustering removes all three:
- It produces a **whole hierarchy** of clusterings at once (from $n$ singletons up to one big cluster), so you **choose $k$ afterward** by cutting the tree at the desired level.
- The hierarchy itself is **interpretable** (taxonomies: species → genus → family; document topics → subtopics).
- Agglomerative hierarchical clustering is **deterministic** — no random seed, same input → same tree.

What it **reuses from Topic 15** is the point-to-point distance; what it **adds** is a rule for the distance **between two clusters** — the **linkage** criterion — which is the genuinely new machinery here.

The price (foreshadowing the forward bridge): it needs the full $O(n^2)$ distance matrix and is **greedy/irrevocable** — a merge, once made, is never undone.

---

## 2. Prerequisites

- **Distance measures & metric axioms** (Topic 15) — point-to-point distances fill the proximity matrix.
- **Proximity / distance matrix** — symmetric $n\times n$ table of pairwise distances.
- **min / max / mean of a set of numbers** — that's literally what the three linkages compute.
- **SSE / within-cluster variance** (Topic 16) — only for Ward's method (recognition level).
- Comfort reading a **tree / dendrogram**.

---

## 3. Core idea — intuition first

**Agglomerative (bottom-up) analogy.** Think of merging companies. Start with every person as their own one-person company ($n$ singletons). Find the **two closest companies** and **merge** them. Recompute how far the new, bigger company is from all the others, find the next-closest pair, merge again. Keep going until everyone is in one giant company. The **record of which merged with which, and at what "distance"**, is the **dendrogram** — a tree whose branch heights are the merge distances.

**Divisive (top-down) analogy.** The reverse: start with **one** company containing everyone and repeatedly **split** the most internally-dissimilar company into two, until everyone is solo.

**Why linkage matters.** Once clusters have more than one point, "distance between two clusters" is ambiguous. Three natural answers:
- **Single-linkage (MIN):** the **closest** pair of members — "are the two groups touching *anywhere*?" → tends to build long, straggly, **chained** clusters.
- **Complete-linkage (MAX):** the **farthest** pair — "are the two groups close *even at their worst*?" → tends to build **tight, compact** clusters of similar size.
- **Average-linkage:** the **average** over all cross-pairs — a **compromise** between the two.

A picture to hold: place two banana-shaped blobs near each other. **Single-linkage** happily merges them (one near pair is enough — it follows the shape), **complete-linkage** resists merging until *all* points are mutually close (it prefers round blobs). That single difference drives most exam reasoning.

---

## 4. Formal treatment

### 4.1 Agglomerative algorithm (the one GATE computes)

```
Input: n points and their pairwise distance (proximity) matrix.
1. Start with n singleton clusters (each point alone).
2. REPEAT (n − 1 times):
   (a) Find the two clusters with the SMALLEST inter-cluster distance
       (per the chosen linkage); record this distance as the merge HEIGHT.
   (b) Merge them into one cluster.
   (c) Update the distance matrix: recompute the distance from the new
       cluster to every other cluster using the linkage rule.
3. STOP when one cluster remains. The merge record IS the dendrogram.
```

> **Key point:** in *every* round you merge the **closest** pair — the linkage only changes **how the cluster-to-cluster distance is computed** in step (c). Picking which pair to merge always means "smallest current distance."

### 4.2 Linkage criteria (definitions + behaviour)

For clusters $A$ and $B$:

$$
\boxed{
\begin{aligned}
\textbf{Single (MIN):}\quad & D(A,B)=\min_{a\in A,\,b\in B} d(a,b)\\[4pt]
\textbf{Complete (MAX):}\quad & D(A,B)=\max_{a\in A,\,b\in B} d(a,b)\\[4pt]
\textbf{Average (group avg, UPGMA):}\quad & D(A,B)=\frac{1}{|A|\,|B|}\sum_{a\in A}\sum_{b\in B} d(a,b)
\end{aligned}}
$$

| Linkage | Cluster shape it favours | Known weakness |
|---|---|---|
| **Single (MIN)** | arbitrary, elongated, non-elliptical | **chaining effect** (one near pair links big clusters); very sensitive to noise/outliers |
| **Complete (MAX)** | compact, roughly equal-diameter | **breaks large clusters**; a far outlier inflates distances |
| **Average** | moderately compact | compromise; less sensitive to noise than the other two |

Two more (recognition level, occasionally named):
- **Centroid linkage:** $D(A,B)=\|\boldsymbol{\mu}_A-\boldsymbol{\mu}_B\|$. Can cause **inversions** (a merge at a *lower* height than an earlier one → non-monotone dendrogram).
- **Ward's method:** merge the pair that **minimizes the increase in total within-cluster SSE**. Produces compact, similar-size clusters; very popular in practice.

### 4.3 The dendrogram, and how to cut it

A **dendrogram** is a tree: **leaves = data points**, and each **internal node's height = the distance at which its two children merged**. For single/complete/average linkage the heights are **monotone non-decreasing** (no inversions).

**Cutting for $k$ clusters:** draw a horizontal line at some height; the number of vertical branches it crosses = the number of clusters, and the leaves under each branch form a cluster. To get exactly $k$ clusters, cut **just above the $(n-k)$-th merge** (equivalently, below the $(k-1)$ highest merges). This is how hierarchical clustering yields a flat clustering *without committing to $k$ beforehand*.

### 4.4 Divisive (top-down) clustering

The mirror image of agglomerative. Conceptually:

```
1. Start with ALL points in one cluster.
2. REPEAT: pick a cluster and SPLIT it into two (e.g., DIANA: pull out the
   point with the largest average distance to the rest to seed a "splinter
   group," then move points to whichever side they are closer to).
3. Recurse on the cluster with the largest diameter until all are singletons
   (or a stopping level is reached).
```

- **Why it's rarely computed by hand:** an exact split of a cluster of size $m$ has $2^{m-1}-1$ possibilities, so **exhaustive divisive clustering is exponential**. Practical versions (DIANA, or recursive 2-means bisection) use heuristics.
- Conceptually divisive can make **better global decisions early** (it sees the whole data before the first split) but is far costlier; agglomerative dominates in practice and in exams.

### 4.5 Lance–Williams update (the unifying recurrence — recognition level)

After merging clusters $i,j$ into $(ij)$, the distance to any other cluster $k$ is

$$
D\big((ij),k\big)=\alpha_i D(i,k)+\alpha_j D(j,k)+\beta\, D(i,j)+\gamma\,\bigl|D(i,k)-D(j,k)\bigr|,
$$

with linkage-specific coefficients:

| Linkage | $\alpha_i,\ \alpha_j$ | $\beta$ | $\gamma$ |
|---|---|---|---|
| Single (MIN) | $\tfrac12,\ \tfrac12$ | $0$ | $-\tfrac12$ |
| Complete (MAX) | $\tfrac12,\ \tfrac12$ | $0$ | $+\tfrac12$ |
| Average (UPGMA) | $\tfrac{|i|}{|i|+|j|},\ \tfrac{|j|}{|i|+|j|}$ | $0$ | $0$ |

(Using $a+b-|a-b|=2\min(a,b)$ and $a+b+|a-b|=2\max(a,b)$, the single/complete rows reduce exactly to MIN/MAX — a neat sanity check.) GATE rarely needs the formula itself; it's the framework behind the by-hand updates.

### 4.6 Complexity & properties (high-yield)

- **Time:** naive agglomerative is $O(n^3)$ ($n-1$ merges, each scanning the $O(n^2)$ matrix); with priority queues, $O(n^2\log n)$. Single-linkage specifically can hit $O(n^2)$. **Space:** $O(n^2)$ for the proximity matrix.
- **No $k$ needed upfront**, **deterministic** (agglomerative), produces an interpretable **hierarchy**.
- **Greedy & irrevocable:** a merge is never undone — there is **no global objective being optimized**, so an early bad merge persists.
- **Scales poorly** to very large $n$ (the $O(n^2)$ memory alone is prohibitive) — contrast with k-means' $O(nkd)$.
- **Noise/outlier sensitivity:** single- and complete-linkage especially.

---

## 5. Worked problems (GATE-style)

We use this single distance matrix for all parts (points $A,B,C,D,E$):

$$
\begin{array}{c|ccccc}
 & A & B & C & D & E\\\hline
A & 0 & 2 & 6 & 10 & 9\\
B & 2 & 0 & 5 & 9 & 8\\
C & 6 & 5 & 0 & 4 & 5\\
D & 10 & 9 & 4 & 0 & 3\\
E & 9 & 8 & 5 & 3 & 0
\end{array}
$$

### Problem 1 — Single-linkage (MIN) agglomerative clustering (NAT/multi-part)

**Round 1.** Smallest distance overall is $d(A,B)=2$. **Merge $\{A,B\}$, height $=2$.**
Update with MIN: $D(\{AB\},C)=\min(6,5)=5$, $D(\{AB\},D)=\min(10,9)=9$, $D(\{AB\},E)=\min(9,8)=8$.

$$
\begin{array}{c|cccc}
 & \{AB\} & C & D & E\\\hline
\{AB\} & 0 & 5 & 9 & 8\\
C & 5 & 0 & 4 & 5\\
D & 9 & 4 & 0 & 3\\
E & 8 & 5 & 3 & 0
\end{array}
$$

**Round 2.** Smallest is $d(D,E)=3$. **Merge $\{D,E\}$, height $=3$.**
MIN update: $D(\{AB\},\{DE\})=\min(9,8)=8$, $D(C,\{DE\})=\min(4,5)=4$.

$$
\begin{array}{c|ccc}
 & \{AB\} & C & \{DE\}\\\hline
\{AB\} & 0 & 5 & 8\\
C & 5 & 0 & 4\\
\{DE\} & 8 & 4 & 0
\end{array}
$$

**Round 3.** Smallest is $D(C,\{DE\})=4$. **Merge $C$ into $\{D,E\}\Rightarrow\{C,D,E\}$, height $=4$.**
MIN update: $D(\{AB\},\{CDE\})=\min(5,8)=5$.

**Round 4.** **Merge $\{A,B\}$ with $\{C,D,E\}$, height $=5$.**

**Single-linkage merge summary**

| step | merged | height |
|---|---|---|
| 1 | $A,B$ | 2 |
| 2 | $D,E$ | 3 |
| 3 | $C$ into $\{D,E\}$ | 4 |
| 4 | $\{A,B\}$ + $\{C,D,E\}$ | 5 |

**Dendrogram (single-linkage):**

```
 height
  5.0 ┤ ┌──────────────────┐
      │ │                  │
  4.0 ┤ │            ┌─────┴────┐
      │ │            │          │
  3.0 ┤ │            │      ┌───┴──┐
      │ │            │      │      │
  2.0 ┤ ┌┴─┐         │      │      │
      │ │  │         │      │      │
      └─┴──┴─────────┴──────┴──────┴──
        A  B         C      D      E
```

---

### Problem 2 — Complete-linkage (MAX) on the *same* matrix (NAT)

**Round 1.** Merge $\{A,B\}$ at height $2$ (closest pair is unchanged).
MAX update: $D(\{AB\},C)=\max(6,5)=6$, $D(\{AB\},D)=\max(10,9)=10$, $D(\{AB\},E)=\max(9,8)=9$.

**Round 2.** Smallest current distance is $d(D,E)=3$. Merge $\{D,E\}$ at height $3$.
MAX update: $D(\{AB\},\{DE\})=\max(10,9)=10$, $D(C,\{DE\})=\max(4,5)=5$.

**Round 3.** Smallest is $D(C,\{DE\})=5$. Merge $C$ into $\{D,E\}\Rightarrow\{C,D,E\}$ at height $\mathbf{5}$.
MAX update: $D(\{AB\},\{CDE\})=\max(6,10)=10$.

**Round 4.** Merge $\{A,B\}$ with $\{C,D,E\}$ at height $\mathbf{10}$.

**Complete-linkage merge summary**

| step | merged | height |
|---|---|---|
| 1 | $A,B$ | 2 |
| 2 | $D,E$ | 3 |
| 3 | $C$ into $\{D,E\}$ | **5** |
| 4 | $\{A,B\}$ + $\{C,D,E\}$ | **10** |

**Compare with single-linkage:** the **merge order is identical here**, but the **heights are larger** under complete-linkage (the C-join rises from 4→5, the final merge from 5→10). In general, linkage can change **both** the heights **and** the order; complete-linkage always reports heights $\ge$ single-linkage for the same merge (MAX $\ge$ MIN).

---

### Problem 3 — Average-linkage on the same matrix (NAT)

Same first two merges ($\{A,B\}$ at 2, $\{D,E\}$ at 3). The instructive update is at the third and fourth merges, using the **mean of all cross-pairs**:

- After $\{A,B\}$ formed: $D(\{AB\},C)=\tfrac{6+5}{2}=5.5$, etc.
- After $\{D,E\}$ formed: $D(\{AB\},\{DE\})=\tfrac{10+9+9+8}{4}=\tfrac{36}{4}=9$, and $D(C,\{DE\})=\tfrac{4+5}{2}=4.5$.
- **Round 3:** smallest is $D(C,\{DE\})=4.5$ → merge $C$ into $\{D,E\}$ at height $\mathbf{4.5}$.
- **Round 4:** $D(\{AB\},\{CDE\})=\dfrac{6+10+9+5+9+8}{6}=\dfrac{47}{6}\approx \mathbf{7.83}$ → final merge.

**All three side-by-side (merge heights):**

| merge | single (MIN) | average | complete (MAX) |
|---|---|---|---|
| $A,B$ | 2 | 2 | 2 |
| $D,E$ | 3 | 3 | 3 |
| $C$ + $\{D,E\}$ | 4 | 4.5 | 5 |
| $\{A,B\}$ + $\{C,D,E\}$ | 5 | 7.83 | 10 |

Notice **single $\le$ average $\le$ complete** at every aggregating merge — a useful self-check.

---

### Problem 4 — Cutting the dendrogram for a target $k$ (MCQ)

Using the **single-linkage** result (heights 2, 3, 4, 5), which clusters do you get if you **cut between heights 3 and 4**?

**Solution.** A cut at height $h$ keeps only merges with height $< h$. Below $3.5$: merges at 2 ($\{A,B\}$) and 3 ($\{D,E\}$) have happened; the merges at 4 and 5 have **not**. So the clusters are
$$
\boxed{\{A,B\},\ \{C\},\ \{D,E\}}\quad(k=3).
$$
(Cut between 4 and 5 → $\{A,B\},\{C,D,E\}$, i.e. $k=2$; cut below 2 → all four/five singletons.)

---

### Problem 5 — Conceptual trap (MSQ): which are TRUE?

(A) Single-linkage tends to produce long "chained" clusters.
(B) In agglomerative clustering, a merge can later be undone if it looks suboptimal.
(C) Complete-linkage merge heights are never smaller than single-linkage heights for the same pair of clusters.
(D) Hierarchical clustering requires you to specify $k$ before running.

**Solution.**
- (A) **True** — the chaining effect, because one near pair suffices to link clusters.
- (B) **False** — agglomerative merges are **greedy and irrevocable**.
- (C) **True** — $\text{MAX}\ge\text{MIN}$ over the same set of cross-pairs.
- (D) **False** — you cut the dendrogram afterward; $k$ is *not* needed upfront (this is the whole advantage over k-means).

**Answer: (A) and (C).**

---

## 6. Common GATE question types on this topic

- **Run agglomerative clustering** from a given proximity matrix with single- or complete-linkage; report the **merge at step $t$**, the **height** of a particular merge, or the **full dendrogram**.
- **Cut the dendrogram**: "how many clusters at height $h$?" or "which points are together for $k=\_$?"
- **Compare linkages**: which linkage gives chaining / compact clusters / a given height.
- **Average-linkage** distance computation between two clusters.
- **Agglomerative vs divisive**, and why divisive is exponential.
- **Properties / true-false**: determinism, irrevocable greedy merges, no preset $k$, $O(n^2)$ memory, outlier sensitivity, monotone vs inversion (centroid linkage).
- **Compute one Lance–Williams update** (rare) or identify coefficients.

---

## 7. Traps, pitfalls & common mistakes

- **Forgetting to update the matrix after each merge.** You must recompute the new cluster's distances *before* picking the next merge. Using stale distances is the #1 error.
- **Applying the linkage rule to choose the pair, instead of to the update.** You always merge the **currently smallest** distance; the linkage (min/max/avg) only governs the **recomputation** step.
- **Mixing min and max** within one run, or reading "complete-linkage" but taking the minimum. Lock the rule before you start.
- **Average-linkage denominator.** Use $|A|\cdot|B|$ (all cross-pairs), **not** the count within one cluster, and **not** the average of previously-merged cluster distances (that's a different, weighted scheme, WPGMA).
- **Off-by-one when cutting for $k$.** To get $k$ clusters, undo the **top $k-1$** merges (cut below them). Cutting at the wrong gap gives $k\pm1$.
- **Assuming all dendrograms are monotone.** Single/complete/average are; **centroid linkage can invert** (a later merge at a lower height).
- **Confusing agglomerative with divisive direction.** Bottom-up = start singletons, merge; top-down = start with all, split.
- **Treating heights as cluster *counts* or distances between *final* clusters.** A node's height is the merge distance at the moment those two sub-clusters joined — nothing more.
- **Expecting it to scale.** $O(n^2)$ memory and $\ge O(n^2\log n)$ time — don't propose hierarchical clustering for millions of points (that's k-means territory).

---

## 8. Connection forward

Every clustering method you've now seen — k-means, k-medoid, hierarchical — lives or dies by **distances in the feature space**. Two problems with that set up the next part of the syllabus:

1. **The curse of dimensionality.** In high-dimensional data, pairwise distances **concentrate** (nearest and farthest points become almost equidistant), so "closest cluster" and "farthest pair" lose meaning — single-, complete-, and average-linkage all degrade, and k-means' spheres blur. **Dimensionality Reduction (Topic 18)** tackles exactly this by projecting data into a few informative dimensions where distances are meaningful again.
2. **Cost & visualization.** Hierarchical clustering's $O(n^2)$ proximity matrix and the inability to *see* clusters beyond 3-D both push toward reducing dimensions first. **PCA (Topic 19)** is the standard tool: it finds the directions of maximum variance and represents each point with far fewer coordinates — often used as a **preprocessing step before clustering**.

So the arc closes: *measure similarity (15) → partition flatly (16) → build the full hierarchy (17) → make the space itself low-dimensional and well-behaved so all of the above work (18–19).*

---

## 9. One-line cheat-sheet summary

> **Agglomerative (bottom-up):** start with $n$ singletons, repeatedly **merge the two closest clusters** (distance via linkage), recording merge **heights** to form a **dendrogram**; cut it for any $k$ — no preset $k$, deterministic, but greedy/irrevocable and $O(n^2)$ memory. **Linkages:** single = **MIN** (chaining, elongated clusters), complete = **MAX** (compact, breaks big clusters), average = mean of cross-pairs (compromise); heights obey **single ≤ average ≤ complete**. **Divisive (top-down):** start with one cluster and split — conceptually nice but exponential, so heuristic.

---

### ➡️ Next topic (in order): **Topic 18 — Dimensionality Reduction (overview)** — why high-dimensional data hurts distance-based methods (curse of dimensionality), feature selection vs feature extraction, and the framing that leads into PCA (Topic 19). Ask for it when ready.

# Topic 15 — Clustering Algorithms (Overview)

**Subject:** Machine Learning · **Section:** Unsupervised Learning (this is the *first* topic of the section)

> **Note on scope:** This is a *framing / overview* topic. Its job is to set up the entire Unsupervised Learning block — what clustering is, how it differs from everything you have done so far, and the one piece of machinery (distance/similarity measures) that every later clustering algorithm reuses. The specific algorithms (k-means, k-medoid, hierarchical, PCA) get full computational treatment in their own topics. So here the *taxonomy* and *evaluation* are kept at framing level, while **distance measures** — the part GATE actually computes in an "overview" question — get full treatment.

---

## 0. Topic header & syllabus map

- **Topic:** Clustering Algorithms (overview of the whole family).
- **Where it sits:** It opens the **Unsupervised Learning** half of the ML syllabus. It comes immediately **after** the entire **Supervised Learning** block (regression → classification → decision trees & evaluation → neural networks, ending at Feed-Forward Neural Network). It **leads into** k-Means / k-Medoid (Topic 16), then Hierarchical Clustering (Topic 17), and the section finishes with Dimensionality Reduction (Topic 18) and PCA (Topic 19).
- **One line:** *Clustering is the first time you learn from data that has no answer key.*

---

## 1. Why this topic exists (motivation + connection)

Every method from Topics 1–14 — simple/multiple/ridge/logistic regression, k-NN, naïve Bayes, LDA, SVM, decision trees, the MLP and feed-forward net — needed **labeled** training data: pairs $(\mathbf{x}_i, y_i)$ where a human already supplied the target $y$. That is **supervised learning**: "here are examples *with answers*; learn the input→output map."

But labels are **expensive, scarce, or simply nonexistent** in huge amounts of real data: millions of customer records with no "segment" tag, gene-expression profiles with no known groupings, documents with no topic labels. The previous section **cannot run at all** without $y$. 

**Clustering** removes that dependency. It answers a different question: *given only the inputs $\{\mathbf{x}_1,\dots,\mathbf{x}_n\}$ and no labels, what natural groups exist in the data?* It is the foundation of the whole unsupervised block.

**The bridge (supervised → unsupervised):**

> Supervised learning: *"Given labeled examples, predict the label of a new point."*  
> Clustering: *"Given only unlabeled points, discover the groups yourself, and (implicitly) assign each point a group ID."*

A nice way to see the continuity: **k-NN** (Topic 6) already used a *distance* to find "the points most similar to me" for prediction. Clustering keeps the same idea of "similar = close in feature space," but instead of borrowing a known label from neighbours, it **invents the groups** because no labels exist.

---

## 2. Prerequisites

Revise these before reading on — clustering is built directly on top of them:

- **Feature vectors & feature space** — a data point is a vector $\mathbf{x}\in\mathbb{R}^d$ (from regression / k-NN).
- **Distance in $\mathbb{R}^d$** — Euclidean distance from k-NN; here it is generalized to a whole family of measures.
- **Vector norms & dot product** — $\|\mathbf{x}\|=\sqrt{\mathbf{x}^\top\mathbf{x}}$, and $\mathbf{x}^\top\mathbf{y}$ (needed for cosine similarity).
- **Mean / centroid** — the average of a set of points $\bar{\mathbf{x}}=\frac1m\sum \mathbf{x}_i$ (k-means uses this).
- **Variance / spread** — intra-cluster "tightness" is a sum-of-squared-distances idea.
- **Covariance matrix** (light) — only for Mahalanobis distance; ties back to LDA (Topic 8).

---

## 3. Core idea — intuition first

**Analogy.** Imagine someone dumps 10,000 unlabeled books in a warehouse. Nobody tells you the genres. Your job: pile them so that **books in the same pile are similar** (same topic, style) and **books in different piles are different**. You never learn the genre *names* — you only produce coherent piles. That is exactly clustering: you output **group memberships, not labels with meaning**.

**The two competing goals of every clustering algorithm:**

1. **High intra-cluster similarity (cohesion):** points inside a cluster should be close to each other.
2. **Low inter-cluster similarity (separation):** points in different clusters should be far apart.

A "good" clustering scores well on **both** at once: tight clusters that are well separated.

**Why "distance" is the entire heart of the topic.** To say two points are "similar," you must *measure* similarity. Change the measure and you change which points look similar, which changes the clusters. So **choosing a distance/similarity function is the first and most consequential decision** — and it is exactly what GATE makes you compute in an overview question. Everything downstream (k-means, linkage, etc.) is "apply this distance, then group."

A quick mental picture of the **two big families** you'll meet next:

- **Partitional (flat):** chop the data into a fixed number $k$ of non-overlapping groups in one shot — *k-means, k-medoid* (Topic 16).
- **Hierarchical (nested):** build a whole *tree* of clusters, from "everything in one cluster" down to "each point its own cluster" — *agglomerative / divisive linkage* (Topic 17). No fixed $k$ chosen in advance; you cut the tree wherever you like.

---

## 4. Formal treatment

### 4.1 What clustering *is* (definition)

Given $n$ data points $X=\{\mathbf{x}_1,\dots,\mathbf{x}_n\}$, $\mathbf{x}_i\in\mathbb{R}^d$, **clustering** produces an assignment of each point to a group such that intra-group similarity is high and inter-group similarity is low.

- **Hard (crisp) clustering:** each point belongs to **exactly one** cluster. A partition $C_1,\dots,C_k$ with $C_i\cap C_j=\varnothing$ and $\bigcup_i C_i = X$. *(k-means, k-medoid, standard hierarchical.)*
- **Soft (fuzzy) clustering:** each point gets a **degree of membership** in every cluster, e.g. memberships $u_{ij}\in[0,1]$ with $\sum_j u_{ij}=1$. *(Fuzzy c-means, Gaussian mixtures — only awareness needed.)*

> **Key contrast with classification:** in classification the number of classes and their *meaning* are fixed and known. In clustering the groups are *discovered*; even the number of clusters $k$ is usually **not** given — it is a **hyperparameter** you must choose.

### 4.2 Distance and similarity measures — the computational core

A **distance (dissimilarity)** is large when points are different; a **similarity** is large when points are alike. The most common measures for numeric vectors $\mathbf{x},\mathbf{y}\in\mathbb{R}^d$:

$$
\boxed{
\begin{aligned}
\textbf{Euclidean (}L_2\textbf{):}\quad & d(\mathbf{x},\mathbf{y}) = \sqrt{\textstyle\sum_{i=1}^{d}(x_i-y_i)^2}\\[4pt]
\textbf{Manhattan / city-block (}L_1\textbf{):}\quad & d(\mathbf{x},\mathbf{y}) = \textstyle\sum_{i=1}^{d}|x_i-y_i|\\[4pt]
\textbf{Minkowski (}L_p\textbf{):}\quad & d(\mathbf{x},\mathbf{y}) = \Big(\textstyle\sum_{i=1}^{d}|x_i-y_i|^{p}\Big)^{1/p}\\[4pt]
\textbf{Chebyshev (}L_\infty\textbf{):}\quad & d(\mathbf{x},\mathbf{y}) = \max_i |x_i-y_i|
\end{aligned}}
$$

**Conditions / how they relate:**
- Minkowski is the **general** form: $p=1\Rightarrow$ Manhattan, $p=2\Rightarrow$ Euclidean, $p\to\infty\Rightarrow$ Chebyshev.
- All defined on **numeric** features. Distances are **not scale-invariant** — a feature measured in large units dominates the sum (see §7), so features are usually **normalized/standardized** first.

$$
\boxed{
\begin{aligned}
\textbf{Cosine similarity:}\quad & \cos(\mathbf{x},\mathbf{y}) = \frac{\mathbf{x}^\top\mathbf{y}}{\|\mathbf{x}\|\,\|\mathbf{y}\|}\in[-1,1]\\[6pt]
\textbf{Cosine distance:}\quad & d_{\cos}(\mathbf{x},\mathbf{y}) = 1-\cos(\mathbf{x},\mathbf{y})
\end{aligned}}
$$

*Condition / meaning:* measures the **angle** between vectors, **ignoring magnitude**. Two vectors pointing the same way have similarity $1$ no matter how long they are. Used for text/TF-IDF and high-dimensional sparse data. For non-negative vectors the range is $[0,1]$.

$$
\boxed{\textbf{Hamming distance:}\quad d_H(\mathbf{x},\mathbf{y}) = \#\{\,i : x_i \neq y_i\,\}}
$$

*Condition:* for **categorical / binary strings of equal length** — counts the positions that differ.

$$
\boxed{\textbf{Mahalanobis distance:}\quad d_M(\mathbf{x},\mathbf{y}) = \sqrt{(\mathbf{x}-\mathbf{y})^\top \mathbf{S}^{-1}(\mathbf{x}-\mathbf{y})}}
$$

*Conditions:* $\mathbf{S}$ is the (invertible) covariance matrix; it **accounts for correlation and differing scales** between features. When $\mathbf{S}=\mathbf{I}$ it **reduces to Euclidean**. (Conceptually important and it links back to the pooled-covariance idea in **LDA**; rarely a heavy GATE computation, so keep it at recognition level.)

### 4.3 When is a distance a *metric*? (a favourite GATE check)

A function $d(\cdot,\cdot)$ is a **metric** iff it satisfies **all four** axioms:

$$
\boxed{
\begin{aligned}
&\text{(1) Non-negativity:}\quad d(\mathbf{x},\mathbf{y})\ge 0\\
&\text{(2) Identity of indiscernibles:}\quad d(\mathbf{x},\mathbf{y})=0 \iff \mathbf{x}=\mathbf{y}\\
&\text{(3) Symmetry:}\quad d(\mathbf{x},\mathbf{y})=d(\mathbf{y},\mathbf{x})\\
&\text{(4) Triangle inequality:}\quad d(\mathbf{x},\mathbf{z})\le d(\mathbf{x},\mathbf{y})+d(\mathbf{y},\mathbf{z})
\end{aligned}}
$$

- Euclidean, Manhattan, Minkowski ($p\ge1$), Chebyshev, Hamming are **all metrics**.
- **Squared Euclidean distance is NOT a metric** — it violates the triangle inequality (proven in Worked Problem 3). It is still used (k-means minimizes it) but it is *not* a metric.
- **Cosine similarity is NOT a distance/metric** — it is a *similarity*, it can be negative, and it does not obey the metric axioms. (Cosine *distance* $1-\cos$ is a dissimilarity but still not a true metric.)

### 4.4 What "good clusters" means numerically (intro only — full version in k-means)

The most common cohesion score is the **within-cluster sum of squares / SSE** (sum of squared distances of points to their cluster centre $\boldsymbol{\mu}_j$):

$$
\boxed{\;\mathrm{SSE} = \sum_{j=1}^{k}\;\sum_{\mathbf{x}\in C_j}\|\mathbf{x}-\boldsymbol{\mu}_j\|^2\;}
\qquad \text{(smaller = tighter clusters)}
$$

This is exactly the objective **k-means** minimizes — full treatment in Topic 16. Mentioned here only so the family makes sense as a single story.

### 4.5 Taxonomy of clustering methods (framing level — names you must recognize)

| Family | Idea | Members (later topics) | Needs $k$ upfront? |
|---|---|---|---|
| **Partitional (flat)** | One-shot split into $k$ groups around representatives | **k-means** (centroid = mean), **k-medoid** (representative = an actual point) | Yes |
| **Hierarchical** | Build a tree (dendrogram) of nested clusters | **Agglomerative** (bottom-up: start with singletons, merge), **Divisive** (top-down: start with all, split); merges use **linkage** (single / complete / average) | No |
| **Density-based** | Clusters = dense regions separated by sparse ones; finds arbitrary shapes & outliers | DBSCAN (*awareness only*) | No |
| **Model-based** | Assume data comes from a mixture of distributions | Gaussian Mixture Models (*awareness only*) | Yes |

> For GATE DA, the syllabus explicitly names **partitional (k-means/k-medoid)** and **hierarchical (single-/complete-linkage, top-down/bottom-up)** — those are where the computation lives. Density- and model-based are recognition-level only.

### 4.6 How clustering quality is judged (framing level)

- **Internal measures** (no labels needed): **SSE / WCSS** (cohesion), and the **silhouette coefficient** of a point $i$
$$
s(i)=\frac{b(i)-a(i)}{\max\{a(i),\,b(i)\}}\in[-1,1],
$$
where $a(i)$ = mean distance of $i$ to other points in **its own** cluster (cohesion) and $b(i)$ = mean distance of $i$ to points in the **nearest other** cluster (separation). Higher $s(i)$ is better; near $0$ means borderline; negative means likely mis-assigned. (Davies–Bouldin index is another, recognition-level.)
- **External measures** (used only when *true* labels happen to exist, for validation): **purity, Rand index, adjusted Rand index, normalized mutual information**.
- **Choosing $k$:** the **elbow method** (plot SSE vs $k$, pick the "elbow" where the drop flattens) and the **silhouette** are the standard heuristics.

*Derivations skipped here on purpose:* the convergence argument for k-means, full linkage update (Lance–Williams) formulas, and PCA's eigen-decomposition all belong to their own topics. GATE never asks them *inside* an overview question.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Distance computation (NAT)

For $\mathbf{x}=(2,3)$ and $\mathbf{y}=(5,7)$, compute the Euclidean, Manhattan, and Chebyshev distances.

**Solution.** Coordinate differences: $|5-2|=3$, $|7-3|=4$.
- Euclidean: $\sqrt{3^2+4^2}=\sqrt{9+16}=\sqrt{25}=\mathbf{5}$.
- Manhattan: $3+4=\mathbf{7}$.
- Chebyshev: $\max(3,4)=\mathbf{4}$.

*Sanity check on ordering:* $L_\infty \le L_2 \le L_1$, i.e. $4\le5\le7$ ✓ (Chebyshev is always smallest, Manhattan largest, for the same pair).

---

### Problem 2 — Cosine vs Euclidean: the magnitude trap (MCQ)

Points $\mathbf{x}=(1,1)$ and $\mathbf{y}=(3,3)$. Which is true?

(A) They are dissimilar by both Euclidean distance and cosine similarity.  
(B) Euclidean distance is large but cosine similarity is maximal ($=1$).  
(C) Cosine similarity is $0$ because the vectors differ.  
(D) Euclidean distance is $0$.

**Solution.**
- Euclidean: $\sqrt{(3-1)^2+(3-1)^2}=\sqrt{8}\approx 2.83$ — **not** small.
- Cosine: $\dfrac{\mathbf{x}^\top\mathbf{y}}{\|\mathbf{x}\|\|\mathbf{y}\|}=\dfrac{1\cdot3+1\cdot3}{\sqrt{2}\cdot\sqrt{18}}=\dfrac{6}{\sqrt{2}\cdot3\sqrt2}=\dfrac{6}{6}=1.$

The two vectors point in **exactly the same direction**, so cosine similarity is $1$ (maximally similar) even though they sit far apart in Euclidean terms. **Answer: (B).**

*Lesson:* cosine ignores magnitude; Euclidean does not. Picking the wrong measure changes the clustering. This is precisely why measure choice is the heart of the topic.

---

### Problem 3 — "Is it a metric?" / triangle-inequality trap (MSQ-style reasoning)

Show that **squared Euclidean distance** $d(x,y)=(x-y)^2$ on $\mathbb{R}$ is **not** a metric, even though it is non-negative, symmetric, and zero only when $x=y$.

**Solution.** Test the triangle inequality with three collinear points $x=0,\ y=1,\ z=2$:
$$
d(x,z)=(2-0)^2=4,\qquad d(x,y)+d(y,z)=(1-0)^2+(2-1)^2=1+1=2.
$$
Triangle inequality demands $d(x,z)\le d(x,y)+d(y,z)$, i.e. $4\le 2$ — **false**. So axiom (4) fails and squared Euclidean is **not a metric**. ∎

*Why it still matters:* k-means *minimizes* squared distances (the squaring makes the math/centroid update clean), so an algorithm can legitimately use a non-metric objective. "Used in clustering" $\ne$ "is a metric."

---

### Problem 4 — Hamming distance on categorical data (NAT)

Two binary feature vectors $\mathbf{x}=10110$ and $\mathbf{y}=11010$. Find the Hamming distance.

**Solution.** Compare position by position:

| pos | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| $x$ | 1 | 0 | 1 | 1 | 0 |
| $y$ | 1 | 1 | 0 | 1 | 0 |
| differ? | – | ✓ | ✓ | – | – |

Two positions differ ⇒ $d_H=\mathbf{2}$. (For numeric vectors you'd use Euclidean/Manhattan; for categorical strings, Hamming is the natural choice.)

---

## 6. Common GATE question types on this topic

- **Compute a distance** between two given points (Euclidean / Manhattan / Minkowski-$p$ / Chebyshev / cosine / Hamming) — pure NAT.
- **"Which of these is a valid metric?"** — apply the four axioms; the trap answer is usually squared-Euclidean or cosine.
- **Match the measure to the data type / scenario** — e.g. text → cosine, categorical strings → Hamming, correlated features → Mahalanobis.
- **Identify the clustering family** from a description ("builds a dendrogram" → hierarchical; "needs $k$ fixed, uses means" → k-means/partitional).
- **Supervised vs unsupervised classification** of a given task ("no labels, find groups" → clustering).
- **Reasoning about a measure's behaviour** — e.g. why scaling a feature changes Euclidean clusters, or why cosine ignores magnitude.

---

## 7. Traps, pitfalls & common mistakes

- **Forgetting the square root** in Euclidean distance — that gives you *squared* Euclidean (a different value, and not a metric). Read whether the question wants $d$ or $d^2$.
- **Treating cosine *similarity* as a *distance*.** Big cosine = *similar* (small distance). If a question wants a dissimilarity, use $1-\cos$. Mixing them flips every comparison.
- **Assuming every "distance" is a metric.** Squared Euclidean violates the triangle inequality; cosine similarity violates the axioms entirely.
- **Ignoring feature scaling.** Distances are **not** scale-invariant: a feature in "rupees" ($10^4$ range) crushes a feature in "years" ($10^1$ range). Always check whether the data is standardized; unstandardized features silently dominate Euclidean/Manhattan sums.
- **Confusing the two big families.** k-means/k-medoid need $k$ **before** running; hierarchical does **not** — you cut the dendrogram afterward.
- **Centroid vs medoid slip (preview).** k-means centre = the **mean** (may be a non-data point); k-medoid centre = an **actual data point**. Don't swap them.
- **Curse of dimensionality.** In very high $d$, all pairwise distances tend to look *similar* (distances "concentrate"), so "nearest" vs "farthest" becomes meaningless and clustering degrades — this is the motivation for **dimensionality reduction / PCA** later.
- **Number of clusters $k$ is not given by the algorithm.** Choosing it (elbow, silhouette) is a separate, judgment-laden step; questions sometimes test that you know $k$ is an input/hyperparameter, not an output.
- **Minkowski edge cases.** $p\to\infty$ gives Chebyshev (the *max*), not infinity; $p<1$ is **not** a valid metric (fails triangle inequality), so "Minkowski for any $p$" is false.

---

## 8. Connection forward

This overview hands the next topics exactly what they need:

- **k-Means / k-Medoid (Topic 16)** take the **partitional** idea + a **distance** (Euclidean for k-means) + the **SSE** objective from §4.4, and add an iterative assign-then-update loop. **k-medoid** swaps the mean centre for an actual data point to be **robust to outliers** — fixing a known weakness of k-means.
- **Hierarchical Clustering (Topic 17)** takes the same distances and adds **linkage** rules (single / complete / average) to decide how to merge clusters into a tree.
- **Dimensionality Reduction & PCA (Topics 18–19)** directly address the **curse of dimensionality** raised in §7: reduce $d$ first so that distances — and therefore clustering — become meaningful again.

So the story is one continuous arc: *no labels → measure similarity (this topic) → group greedily and flatly (k-means/medoid) → group hierarchically (linkage) → fix high-dimensional distance failure (PCA).*

---

## 9. One-line cheat-sheet summary

> **Clustering = unsupervised grouping of unlabeled points to maximize within-group similarity and between-group separation; everything hinges on the chosen distance/similarity measure (Euclidean $L_2$, Manhattan $L_1$, Minkowski $L_p$, Chebyshev $L_\infty$, cosine, Hamming, Mahalanobis) and on knowing the four metric axioms — squared-Euclidean and cosine are NOT metrics; families are partitional (k-means/medoid, need $k$) and hierarchical (linkage, build a tree).**

---

### ➡️ Next topic (in order): **Topic 16 — k-Means / k-Medoid** (the first concrete partitional algorithms; full assign–update loop, SSE objective, centroid-vs-medoid, and worked iterations). Ask for it when ready.

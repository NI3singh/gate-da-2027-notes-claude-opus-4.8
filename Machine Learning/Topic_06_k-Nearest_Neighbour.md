# Topic 6 — k-Nearest Neighbour (k-NN)

*GATE DA · Machine Learning · Classification Methods (Chat 2, Topic 6)*

---

## 0. Topic header & syllabus map

- **Topic:** k-Nearest Neighbour (k-NN)
- **Subject:** Machine Learning
- **Where it sits:** This is the **first topic of the Classification Methods cluster**. It comes right after **Logistic Regression** (which closed the regression cluster) and leads into **Naive Bayes Classifier**. Logistic regression was our first *parametric* classifier (it learns fixed weights and draws a straight decision boundary). k-NN is the opposite philosophy — a *non-parametric, "lazy"* classifier that draws no boundary at all and instead decides one point at a time by looking at its neighbours.

---

## 1. Why this topic exists (motivation + connection)

**The real problem it solves:** Given a labelled dataset, classify a brand-new point. The simplest honest idea a human uses is *"this new thing looks most like those known things, so it's probably the same kind."* k-NN formalises exactly that: **look at the k closest labelled examples and take a majority vote.**

**What the previous topic (Logistic Regression) couldn't handle:** Logistic regression assumes the **log-odds are linear in the features**, which forces a **single straight (hyperplane) decision boundary**. If the true boundary between classes is curved, broken into pockets, or otherwise non-linear, logistic regression fits poorly unless you hand-engineer extra features. It also commits to **one global model** for the whole input space.

**How k-NN fixes that:** k-NN is **non-parametric** — it makes *no assumption about the shape of the boundary*. It can carve out arbitrarily complex, even disconnected, regions because every prediction is *local*. There is **no training step** (it just stores the data), so all the work happens at prediction time — hence "lazy" / "instance-based" learning. The price: it keeps the entire dataset, is slow at query time, and (unlike logistic regression) gives no compact equation or weight vector to interpret.

---

## 2. Prerequisites

- **Distance metrics** in a feature space — Euclidean and Manhattan distance especially.
- **Classification basics** (Topic 1: regression vs classification) — the idea of a discrete label.
- **Feature vectors / vector space** — representing each example as a point.
- **Majority voting / mode** of a small set of labels.
- **Standardization & normalization** (z-score, min–max) — essential, because k-NN is scale-sensitive.
- **Bias–variance trade-off** (Topic 11, forward reference) — to reason about how `k` controls model complexity. You can read this topic without it, but the link is flagged below.

---

## 3. Core idea — intuition first

> *"You are the average of your neighbours."*

You just moved to a new street and want to guess your new neighbours' favourite sport. You don't have a theory of the whole city; you simply ask the **k people who live closest** and go with the most common answer. If you ask only **1 neighbour** (k = 1), your guess swings wildly with whoever happens to live next door. If you ask **the whole city** (k = n), you always guess the city-wide favourite and ignore your street entirely. Somewhere in between is sensible.

**Tiny example.** You have fruits described by (sweetness, weight). "Apples" cluster around moderate sweetness/medium weight; "lemons" around low sweetness/light weight. A new fruit arrives. You measure its sweetness and weight, find the 3 known fruits closest to it, and if 2 of those 3 are apples, you call it an apple. That's the entire algorithm.

Two knobs control everything:
1. **How you measure "close"** → the *distance metric*.
2. **How many neighbours you trust** → the value of *k*.

---

## 4. Formal treatment

### 4.1 Definition (classification)

Given a training set $\{(\mathbf{x}_i, y_i)\}_{i=1}^{n}$ with $\mathbf{x}_i \in \mathbb{R}^d$ and labels $y_i \in \{1,\dots,c\}$, and a query point $\mathbf{x}_q$:

1. Compute the distance $d(\mathbf{x}_q, \mathbf{x}_i)$ to **every** training point.
2. Pick the **k smallest** distances — these are the *k nearest neighbours* $N_k(\mathbf{x}_q)$.
3. **Predict the majority class** among those k neighbours.

$$\hat{y}(\mathbf{x}_q) \;=\; \arg\max_{\,\ell \in \{1,\dots,c\}} \sum_{i \in N_k(\mathbf{x}_q)} \mathbb{1}\{y_i = \ell\}$$

> **Conditions / assumptions:**
> - Features are **numeric and comparably scaled** (else the largest-scale feature dominates — see §7).
> - A meaningful distance metric exists for the data.
> - For binary classification, **k is usually chosen odd** to avoid ties.

### 4.2 Distance metrics

| Metric | Formula | When used |
|---|---|---|
| **Euclidean (L2)** | $\sqrt{\sum_{j}(x_j - z_j)^2}$ | Default; continuous features |
| **Manhattan (L1)** | $\sum_{j}\lvert x_j - z_j\rvert$ | Grid-like / high-dim, robust to outliers |
| **Minkowski (Lp)** | $\left(\sum_{j}\lvert x_j - z_j\rvert^{p}\right)^{1/p}$ | General form: $p=1\Rightarrow$ Manhattan, $p=2\Rightarrow$ Euclidean |
| **Chebyshev (L∞)** | $\max_{j}\lvert x_j - z_j\rvert$ | $p\to\infty$ limit of Minkowski |
| **Hamming** | number of positions where $x_j \neq z_j$ | Categorical / binary features |

> **Trap-flag:** For ranking the neighbours by **Euclidean** distance you can compare **squared** distances ($\sum(x_j-z_j)^2$) and skip the square root — it doesn't change the ordering and saves arithmetic in the exam.

### 4.3 k-NN regression

For a numeric target, replace "majority vote" with the **average** of the k neighbours' values:

$$\hat{y}(\mathbf{x}_q) \;=\; \frac{1}{k}\sum_{i \in N_k(\mathbf{x}_q)} y_i$$

### 4.4 Distance-weighted k-NN

Give closer neighbours more say, typically with weight $w_i = 1/d(\mathbf{x}_q,\mathbf{x}_i)$ (or $1/d^2$):

$$\hat{y} = \frac{\sum_{i\in N_k} w_i \, y_i}{\sum_{i\in N_k} w_i}\quad\text{(regression)}, \qquad \hat{y} = \arg\max_{\ell}\sum_{i \in N_k} w_i\,\mathbb{1}\{y_i=\ell\}\quad\text{(classification)}$$

> Weighting reduces the harm of a bad choice of k and softens ties.

### 4.5 The role of k (this is the heart of exam reasoning)

- **k = 1:** decision regions are the **Voronoi cells** of the training points. **Lowest bias, highest variance** — fits (even memorises) the data, very sensitive to noise/outliers. *Overfits.*
- **k = n:** every query gets the **global majority class**, ignoring $\mathbf{x}_q$ entirely. **Highest bias, lowest variance.** *Underfits* (constant prediction).
- **Increasing k → smoother boundary → ↑ bias, ↓ variance.** Decreasing k does the reverse.

> **Memory hook (the most-tested fact):** *Large k = smooth = underfit = high bias. Small k = jagged = overfit = high variance.* k is the **complexity / smoothness knob**, and it moves opposite to most "parameters." (Forward link to Topic 11, Bias–Variance.)

### 4.6 Computational cost (brute force)

- **Training time:** $O(1)$ — just store the data (lazy learner).
- **Prediction time:** $O(n d)$ per query — compute distance to all n points, each of d dimensions.
- **Space:** $O(n d)$ — the **whole training set must be kept**.

(k-d trees / ball trees speed up low-dimensional search but degrade toward $O(nd)$ in high dimensions — usually beyond GATE detail, mentioned in one line.)

### 4.7 Curse of dimensionality

As $d$ grows, points become roughly **equidistant** — the ratio of nearest to farthest distance approaches 1 — so "nearest" stops being meaningful and k-NN deteriorates. This is why dimensionality reduction (Topic 18–19, PCA) matters before k-NN in high dimensions.

### 4.8 Asymptotic 1-NN error bound (Cover–Hart, classic ML result)

As $n\to\infty$, the error rate $R$ of the **1-NN rule** is bounded by the **Bayes (optimal) error rate** $R^\*$:

$$R^\* \;\le\; R \;\le\; R^\*\!\left(2 - \tfrac{c}{c-1}\,R^\*\right) \;\le\; 2R^\*$$

where $c$ is the number of classes.

> **Reading it:** 1-NN, even with infinite data, is **never worse than twice** the best achievable error. *"Half the information is in the nearest neighbour's label."* For $c=2$: $R \le 2R^\*(1-R^\*)$. (Derivation not required for GATE; the **inequality and the "≤ 2× Bayes error" headline** are the testable parts.)

---

## 5. Worked problems (GATE-style)

### Problem 1 — Classification, and how the answer changes with k (MCQ)

Training points in 2-D with classes (C0 / C1):

| Point | (x, y) | Class |
|---|---|---|
| A | (1, 1) | C0 |
| B | (2, 3) | C0 |
| C | (2, 1) | C0 |
| D | (5, 6) | C1 |
| E | (4, 4) | C1 |
| F | (5, 3) | C1 |

Classify the query **Q = (3, 3)** using **Euclidean** distance for **k = 1, 3, 5**.

**Solution.** Compare *squared* distances (ordering only):

$$d^2(Q,A)=2^2+2^2=8,\quad d^2(Q,B)=1^2+0^2=1,\quad d^2(Q,C)=1^2+2^2=5$$
$$d^2(Q,D)=2^2+3^2=13,\quad d^2(Q,E)=1^2+1^2=2,\quad d^2(Q,F)=2^2+0^2=4$$

Sorted ascending: **B(1) < E(2) < F(4) < C(5) < A(8) < D(13)**.

- **k = 1:** nearest is **B** → **C0**.
- **k = 3:** {B(C0), E(C1), F(C1)} → 2 vs 1 → **C1**.
- **k = 5:** {B(C0), E(C1), F(C1), C(C0), A(C0)} → C0 has B,C,A = 3; C1 has E,F = 2 → **C0**.

**Answers:** k=1 → **C0**, k=3 → **C1**, k=5 → **C0**.

> **Lesson:** the predicted class genuinely depends on k, and increasing k can even *flip it back*. GATE loves "for which k does the prediction change?"

---

### Problem 2 — Feature scaling (the classic trap) (NAT / MCQ)

Two features on very different scales:

| Point | (X₁, X₂) | Class |
|---|---|---|
| P1 | (1, 5000) | A |
| P2 | (9, 5400) | B |

Classify **Q = (2, 5300)** with **k = 1, Euclidean**, (a) on raw values, then (b) after **min–max normalization** (using P1, P2 as the observed min/max). Does the answer change?

**(a) Raw distances (squared):**
$$d^2(Q,P1)=(2-1)^2+(5300-5000)^2 = 1 + 90000 = 90001$$
$$d^2(Q,P2)=(2-9)^2+(5300-5400)^2 = 49 + 10000 = 10049$$
Nearest = **P2 → class B**. Note: in the *meaningful* feature X₁, Q (=2) is almost on top of P1 (=1) and far from P2 (=9) — but the huge-scale feature X₂ completely dominated the distance.

**(b) Min–max normalize** $x' = \dfrac{x-\min}{\max-\min}$, with X₁∈[1,9], X₂∈[5000,5400]:

- P1 → (0, 0); P2 → (1, 1)
- Q: $X_1' = \frac{2-1}{8}=0.125,\;\; X_2'=\frac{5300-5000}{400}=0.75$ → (0.125, 0.75)

Normalized distances (squared):
$$d^2(Q,P1)=0.125^2+0.75^2 = 0.0156 + 0.5625 = 0.5781$$
$$d^2(Q,P2)=(0.125-1)^2+(0.75-1)^2 = 0.7656 + 0.0625 = 0.8281$$
Nearest = **P1 → class A**.

**Answer:** The prediction **flips from B to A** after scaling. Raw k-NN was decided almost entirely by salary-scale feature X₂; normalization let X₁ speak.

> **Lesson:** *Always scale features before k-NN.* A single large-magnitude feature can silently hijack every distance.

---

### Problem 3 — k-NN regression, plain vs distance-weighted (NAT)

1-D data:

| x | 1 | 2 | 3 | 6 | 7 |
|---|---|---|---|---|---|
| y | 2 | 4 | 5 | 10 | 11 |

Predict y at **x = 2.5** with **k = 3**, (a) by uniform average, (b) by distance weighting $w=1/d$.

**Neighbours.** $|x-2.5|$: 1→1.5, 2→0.5, 3→0.5, 6→3.5, 7→4.5. The 3 nearest are **x=2 (d=0.5), x=3 (d=0.5), x=1 (d=1.5)** with y = 4, 5, 2.

**(a) Uniform:** $\hat{y} = \dfrac{4+5+2}{3} = \dfrac{11}{3} \approx \mathbf{3.67}$.

**(b) Weighted:** weights $w$ = $1/0.5=2$, $1/0.5=2$, $1/1.5=0.667$.
$$\hat{y}=\frac{2(4)+2(5)+0.667(2)}{2+2+0.667}=\frac{8+10+1.333}{4.667}=\frac{19.333}{4.667}\approx \mathbf{4.14}$$

**Answer:** uniform **≈ 3.67**, weighted **≈ 4.14**. Weighting pulls the estimate toward the two nearest (closer) points.

> **Lesson:** in regression, k-NN *averages* (it doesn't vote); weighting changes the estimate by trusting nearer points more.

---

## 6. Common GATE question types on this topic

- **Classify a query point** given a small table, a `k`, and a metric (the bread-and-butter question).
- **"For which k does the predicted class change / become a tie?"**
- **Effect of k on bias/variance, over/underfitting** (small k ↔ high variance; large k ↔ high bias).
- **Compute / compare distances** under Euclidean vs Manhattan vs Minkowski-p.
- **Recognise the need for feature scaling**, or compute results before vs after normalization.
- **Time/space complexity** of training vs prediction (lazy learner facts).
- **1-NN asymptotic error vs Bayes error** ($R \le 2R^\*$).
- **Conceptual T/F:** "k-NN is parametric" (False), "k-NN has no training cost" (True), "k-NN decision boundary is always linear" (False).
- **1-NN ⇒ Voronoi diagram** of training points.

---

## 7. Traps, pitfalls & common mistakes

- **Not scaling features.** The biggest-magnitude feature dominates the distance and the rest are ignored (Problem 2). Standardize/normalize first.
- **Even k in binary classification** → vote ties. Prefer **odd k**, and have a tie-break rule (e.g., nearest neighbour wins, or smallest-index class).
- **Getting the k–complexity direction backwards.** *Large* k is the *simpler, higher-bias* model — opposite to the intuition that "more = more powerful."
- **Forgetting tie distances.** Two points may share the k-th smallest distance; the "k nearest" set isn't unique then — state your tie rule.
- **Including the query itself as a neighbour** when the query is a training point (its distance is 0). Watch the problem's wording (training-error vs test setting).
- **Wrong metric for categorical data** — use **Hamming**, not Euclidean, on nominal attributes.
- **Curse of dimensionality** — assuming k-NN "just works" in very high d, where all distances collapse together.
- **Minkowski confusion** — $p=1$ Manhattan, $p=2$ Euclidean; mixing them up changes which neighbour is nearest.
- **Mis-sorting** — "nearest" = **smallest** distance. Don't sort descending.
- **Treating k-NN as having a model to interpret** — there are no learned weights; it stores data and votes.
- **Unweighted at k = n ignores the query** (always the global majority); distance-weighted k-NN still depends on the query — don't conflate the two.

---

## 8. Connection forward

k-NN estimates the class purely from the **local crowd** of labelled points — implicitly approximating $P(\text{class}\mid \mathbf{x})$ by what's nearby, with **no model and all work at query time**. The next classifier, **Naive Bayes**, attacks the *same* goal $P(\text{class}\mid\mathbf{x})$ from the opposite direction: it builds an **explicit probabilistic model** upfront using **Bayes' theorem** plus a strong **feature-independence assumption**, then predicts instantly. So we move from *non-parametric, lazy, distance-based* (k-NN) to *parametric, eager, probability-based* (Naive Bayes) — two contrasting answers to "how do I assign a class?"

---

## 9. One-line cheat-sheet summary

**k-NN:** classify by majority vote of the k closest labelled neighbours (regression → average); **non-parametric & lazy** (no training, $O(nd)$ per query, stores all data); **small k = low bias/high variance, large k = high bias/low variance**; **always scale features**; 1-NN error $\le 2\times$ Bayes error.

---

### ➡️ Next topic
**Topic 7 — Naive Bayes Classifier** (still in the Classification Methods cluster). Ask for it when ready.

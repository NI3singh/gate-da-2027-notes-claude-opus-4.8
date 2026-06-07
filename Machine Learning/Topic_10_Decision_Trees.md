# Topic 10 — Decision Trees

> **Subject:** Machine Learning  **|**  **Section:** Trees & Model Evaluation  **|**  **Position:** Topic 10 of the ML sequence

---

## 0. Topic header & syllabus map

- **Topic:** Decision Trees (classification *and* regression trees).
- **Subject:** Machine Learning → Supervised Learning.
- **Where it sits:** This is the **first topic of the *Trees & Model Evaluation* block**. It comes **immediately after Support Vector Machine (Topic 9)** and feeds directly into **Bias–Variance Trade-off (Topic 11)**.
- **One-line placement:** A decision tree is the cleanest model in the whole syllabus for *seeing* bias and variance with your own eyes — a stump is high-bias, a fully grown tree is high-variance — which is exactly why it is placed right before the bias–variance topic.

---

## 1. Why this topic exists (motivation + connection)

Every classifier we have built so far draws **one global decision rule** over the whole feature space: logistic regression and LDA draw a single (linear) boundary, kNN votes among neighbours under a single distance metric, and the **SVM (previous topic)** finds one maximum-margin hyperplane (possibly in a kernel space). These are powerful but they share three practical headaches: they are **hard to read off as human rules**, they usually **need feature scaling / a kernel choice**, and they do not naturally mix **numeric and categorical features**.

A decision tree fixes all three at once by abandoning the "one global rule" idea. Instead it **recursively chops the feature space into axis-parallel boxes** with simple yes/no questions ("Is `age ≤ 30`?", "Is `outlook = sunny`?"), and predicts a constant inside each box. The result is a flowchart a human can literally follow — **interpretable, scale-free, and able to capture non-linear interactions** that a single linear boundary (the thing SVM gives you without a kernel) cannot.

It also generalises what earlier topics couldn't: where **linear/logistic regression and SVM are essentially linear** in the raw features, a tree carves curved, piecewise-constant regions; and where every previous method was a *classifier or a regressor*, **one tree does both** (classification trees and regression trees). The price we pay — and the reason the **next topic exists** — is that a tree grown to purity **memorises the training data (very high variance)**, which is the bias–variance tension made concrete.

---

## 2. Prerequisites

- **Probability basics:** proportions, conditional probability $P(Y \mid A)$.
- **Logarithms (base 2):** $\log_2 x$, and the identity $\log_2(a/b)=\log_2 a-\log_2 b$.
- **Entropy / information** at an intuitive level (re-used here as the impurity measure).
- **Variance & sum of squared errors (SSE):** carried over from the regression chat — used by *regression* trees.
- **Classification vs regression** (Topic 1) and the general idea of overfitting (you saw it in ridge/regularisation, Topic 4).

---

## 3. Core idea — intuition first

Imagine you are playing **20 Questions** to guess someone's label. A good question is one whose answer **tells you the most** about the label. If asking "Did you study?" splits the class into "almost all Pass" vs "almost all Fail", that question was great. If after the answer each group is still a 50–50 mix, the question told you nothing.

A decision tree is exactly this game, played greedily:

1. Look at the data sitting at the current node.
2. For **every possible question** (split), measure how *pure* the resulting child groups become.
3. Ask the **single question that gives the biggest jump in purity**.
4. Send each row down the matching branch and **repeat on each child** until the groups are pure (or you decide to stop).
5. At each final box (**leaf**), predict the **majority class** (classification) or the **mean value** (regression) of the rows that landed there.

The only technical content is **how to measure "purity"** (Entropy, Gini, or — for regression — variance) and **how to score a split** (Information Gain, Gini Gain, SSE reduction). Everything else is just "repeat greedily." The tree is **greedy and top-down**: it never looks back to undo an early split, so the tree it builds is *good*, not provably *smallest/optimal*.

**Tiny example.** 10 students, 5 Pass / 5 Fail. If splitting on "Practice problems? Yes/No" gives Yes = {4 Pass, 1 Fail} and No = {1 Pass, 4 Fail}, that split clearly made each side *much* purer than the 5–5 parent. A tree would prefer this split over one that leaves the children near 50–50.

---

## 4. Formal treatment

### 4.1 Anatomy of a tree

- **Root node** — holds all training data, site of the first split.
- **Internal (decision) node** — applies a test on one feature; has one branch per outcome.
- **Branch / edge** — an outcome of a test.
- **Leaf (terminal) node** — makes the prediction (majority class / mean).
- **Depth** — longest root-to-leaf path (in edges). **Size** — number of leaves $|\tilde T|$.

A test is typically `feature = value` (categorical) or `feature ≤ threshold` (numeric). Hence **every split is a hyperplane perpendicular to one axis** → tree boundaries are **axis-parallel** (key property, §4.9).

---

### 4.2 Impurity measures (classification)

Let a node contain classes with proportions $p_1,\dots,p_k$ (so $\sum_i p_i = 1$). "Impurity" should be **0 when the node is pure** and **maximal when classes are evenly mixed**.

$$
\boxed{\;\text{Entropy:}\quad H = -\sum_{i=1}^{k} p_i \log_2 p_i\;}
\qquad\text{(units: bits; convention } 0\log_2 0 = 0)
$$

$$
\boxed{\;\text{Gini index:}\quad G = 1 - \sum_{i=1}^{k} p_i^{\,2} = \sum_{i=1}^k p_i(1-p_i)\;}
$$

$$
\boxed{\;\text{Misclassification error:}\quad E = 1 - \max_i p_i\;}
$$

| Measure | Binary form, $p=P(\text{class }1)$ | Pure node ($p=0$ or $1$) | Maximum (binary) | Maximum over $k$ classes |
|---|---|---|---|---|
| Entropy | $-p\log_2 p-(1-p)\log_2(1-p)$ | $0$ | $\mathbf{1}$ at $p=0.5$ | $\log_2 k$ |
| Gini | $2p(1-p)$ | $0$ | $\mathbf{0.5}$ at $p=0.5$ | $1-\tfrac1k$ |
| Misclassification | $\min(p,\,1-p)$ | $0$ | $\mathbf{0.5}$ at $p=0.5$ | $1-\tfrac1k$ |

**Conditions / facts GATE tests:**
- Entropy is **not** capped at 1 in general — only for **binary** classes. For $k$ classes its max is $\log_2 k$ (e.g. 4 equal classes → $H_{\max}=2$).
- All three equal **0 iff the node is pure**.
- Entropy and Gini are **strictly concave**; misclassification error is **piecewise linear**. That concavity is *why* entropy/Gini are used to **grow** trees (they reward making a node "more pure" even when the majority class doesn't change), whereas misclassification error is mostly used during **pruning**.

---

### 4.3 Splitting criteria (how to score a candidate split)

A split on attribute $A$ partitions node $S$ into children $S_v$ (one per value/branch $v$). The child impurity is combined by **weighting each child by its share of the rows** $\frac{|S_v|}{|S|}$.

**(a) Information Gain — used by ID3**
$$
\boxed{\;IG(S,A) = H(S) - \sum_{v}\frac{|S_v|}{|S|}\,H(S_v)\;}
$$
The subtracted term is exactly the **conditional entropy** $H(S\mid A)$, so $IG = H(S)-H(S\mid A) = I(S;A)$ — the **mutual information** between the attribute and the label. **Pick the attribute with the largest $IG$.**

> **$IG \ge 0$ always.** Conditioning never increases expected entropy ($H(S\mid A)\le H(S)$, by concavity / Jensen). So a *negative* information gain is always an arithmetic mistake.

**(b) Gain Ratio — used by C4.5** (fixes $IG$'s bias toward many-valued attributes, §7)
$$
\boxed{\;GainRatio(S,A)=\dfrac{IG(S,A)}{SplitInfo(S,A)}\;},\qquad
SplitInfo(S,A) = -\sum_{v}\frac{|S_v|}{|S|}\log_2\frac{|S_v|}{|S|}
$$
$SplitInfo$ is the entropy of the *partition sizes*; it is large when $A$ has many roughly-equal branches, **penalising** such splits.

**(c) Gini Gain — used by CART**
$$
\boxed{\;\Delta G = G(S) - \sum_{v}\frac{|S_v|}{|S|}\,G(S_v)\;}\qquad\text{pick the largest } \Delta G.
$$

**(d) Variance / SSE reduction — used by regression trees (CART)**
For a continuous target, "impurity" of a node is its **sum of squared errors about the node mean** $\bar y$:
$$
SSE(S)=\sum_{i\in S}(y_i-\bar y)^2 \;=\; |S|\cdot \mathrm{Var}(y\text{ in }S).
$$
$$
\boxed{\;\text{SSE reduction} = SSE(S) - \Big(SSE(S_{\text{left}})+SSE(S_{\text{right}})\Big)\;}\qquad\text{pick the largest reduction.}
$$

---

### 4.4 The generic learning algorithm (greedy, top-down recursive partitioning)

```
BuildTree(node S):
    if stopping condition(S):           # pure, or max-depth, or too few samples
        make S a leaf
        label = majority class (classif.) | mean (regression)
        return
    for each candidate split (attribute, and threshold if numeric):
        score it (IG / GainRatio / Gini gain / SSE reduction)
    choose the best-scoring split
    partition S into children by that split
    for each child:  BuildTree(child)
```

This is **greedy** (locally best split, never reconsidered) and **top-down**. Consequence: trees are **not guaranteed to be the smallest or globally optimal** tree (finding the optimal tree is NP-hard).

---

### 4.5 ID3 vs C4.5 vs CART (frequently asked "which uses what")

| Algorithm | Split criterion | Split type | Feature types | Regression? |
|---|---|---|---|---|
| **ID3** | Information Gain | Multi-way (one branch per category) | Categorical only | No |
| **C4.5** | **Gain Ratio** | Multi-way | Categorical **+ continuous** | No |
| **CART** | **Gini** (classif.), **SSE/variance** (regression) | **Binary only** | Both | **Yes** |

---

### 4.6 Handling continuous attributes

Sort the rows by the numeric feature. The only candidate thresholds are the **midpoints between consecutive distinct values**; evaluate the split criterion at each candidate `feature ≤ t` and keep the best $t$. (A feature can be **re-used** at multiple depths with different thresholds — unlike a categorical feature in ID3, which is "used up" once split on.)

---

### 4.7 Stopping, overfitting, and pruning

**Why a tree overfits:** if you keep splitting, you can drive **training error to 0** (every leaf is a pure singleton). Such a tree memorises noise → **excellent on train, poor on test** → **high variance**.

**Pre-pruning (early stopping):** stop growing when a limit is hit —
- maximum depth,
- minimum samples to split / minimum samples per leaf,
- minimum impurity decrease (split only if gain exceeds a threshold).
Cheap, but can stop too early ("horizon effect").

**Post-pruning:** grow the **full** tree, then cut back subtrees that don't help generalisation —
- **Reduced-error pruning:** replace a subtree by a leaf if validation error does not increase.
- **Cost-complexity (weakest-link) pruning — CART:** minimise
$$
\boxed{\;R_\alpha(T) = R(T) + \alpha\,|\tilde T|\;}
$$
where $R(T)$ is the tree's error (or total leaf impurity/SSE), $|\tilde T|$ is the **number of leaves**, and $\alpha \ge 0$ is the complexity penalty. **Larger $\alpha$ → smaller tree** ($\alpha=0$ gives the full tree; $\alpha\to\infty$ collapses to the root). The best $\alpha$ is chosen by cross-validation — directly previewing **Topic 12**.

---

### 4.8 Prediction at a leaf
- **Classification:** majority class of training rows in that leaf (class probability = class proportion in the leaf).
- **Regression:** **mean** of the $y$-values of training rows in that leaf (a piecewise-constant function).

---

### 4.9 Key structural properties (high-yield for MSQ)
- **Axis-parallel boundaries:** each split is $\perp$ one axis. A single tree cannot draw one *diagonal* boundary; it must staircase many splits to approximate it. (XOR-type / diagonal data needs a deep tree.)
- **Non-parametric:** no assumed functional form or data distribution (contrast LDA/Naive Bayes/logistic regression, which assume a form).
- **Scale-invariant:** monotone rescaling of a feature doesn't change `≤ t` orderings → **no feature scaling needed** (unlike SVM/kNN).
- **Handles mixed data**, captures **feature interactions** automatically, and is **interpretable**.
- **Low bias, high variance** when grown deep; unstable (small data change → very different tree).

---

## 5. Worked problems (GATE-style)

These three problems share one dataset so the criteria can be compared directly.

> **Dataset $S$** — predict `Pass` (P) / `Fail` (F) from `Studied` (A) and `DidPractice` (B). 10 rows, **5 P, 5 F**.

| # | Studied (A) | Practice (B) | Class |
|---|---|---|---|
| 1 | Yes | Yes | P |
| 2 | Yes | Yes | P |
| 3 | Yes | Yes | P |
| 4 | Yes | Yes | P |
| 5 | Yes | No | F |
| 6 | Yes | No | F |
| 7 | No | Yes | F |
| 8 | No | No | P |
| 9 | No | No | F |
| 10 | No | No | F |

Useful constants: $\log_2 3 = 1.585,\ \log_2 5 = 2.322,\ \log_2 10 = 3.322.$

---

### Problem 1 (NAT / MCQ) — Information Gain: which attribute is the root?

**Find $IG(S,A)$ and $IG(S,B)$ and state which attribute ID3 splits on first.**

**Step 1 — parent entropy.** $S$ has 5 P, 5 F:
$$H(S) = -\tfrac{5}{10}\log_2\tfrac{5}{10}-\tfrac{5}{10}\log_2\tfrac{5}{10} = -0.5(-1)-0.5(-1)=\mathbf{1}.$$

**Step 2 — split on A (Studied).**
- $A=\text{Yes}$ (rows 1–6): 6 rows, 4 P, 2 F.
$$H=-\tfrac{4}{6}\log_2\tfrac{4}{6}-\tfrac{2}{6}\log_2\tfrac{2}{6}= -\tfrac23(-0.585)-\tfrac13(-1.585)=0.390+0.528=0.918.$$
- $A=\text{No}$ (rows 7–10): 4 rows, 1 P, 3 F.
$$H=-\tfrac14\log_2\tfrac14-\tfrac34\log_2\tfrac34 = -\tfrac14(-2)-\tfrac34(-0.415)=0.5+0.311=0.811.$$
- Weighted child entropy $=\tfrac{6}{10}(0.918)+\tfrac{4}{10}(0.811)=0.551+0.324=0.875.$
$$IG(S,A)=1-0.875=\mathbf{0.125}.$$

**Step 3 — split on B (Practice).**
- $B=\text{Yes}$ (rows 1,2,3,4,7): 5 rows, 4 P, 1 F → $H=-\tfrac45\log_2\tfrac45-\tfrac15\log_2\tfrac15=0.258+0.464=0.722.$
- $B=\text{No}$ (rows 5,6,8,9,10): 5 rows, 1 P, 4 F → $H=0.722$ (symmetric).
- Weighted child entropy $=0.722.$
$$IG(S,B)=1-0.722=\mathbf{0.278}.$$

**Answer.** $IG(S,B)=0.278 > IG(S,A)=0.125$, so **ID3 splits on `DidPractice` (B) at the root.**

---

### Problem 2 (NAT) — Gini Gain on the same split

**Compute the Gini gain of A and of B; confirm the chosen attribute.**

Parent: $G(S)=1-(0.5)^2-(0.5)^2=\mathbf{0.5}.$

**Attribute A:**
- $A=\text{Yes}$ (4 P,2 F): $G=1-(\tfrac46)^2-(\tfrac26)^2 = 1-0.444-0.111=0.444.$
- $A=\text{No}$ (1 P,3 F): $G=1-(\tfrac14)^2-(\tfrac34)^2 = 1-0.0625-0.5625=0.375.$
- Weighted $=0.6(0.444)+0.4(0.375)=0.2667+0.15=0.4167.$
$$\Delta G(A)=0.5-0.4167=\mathbf{0.083}.$$

**Attribute B:**
- $B=\text{Yes}$ (4 P,1 F): $G=1-(0.8)^2-(0.2)^2=1-0.64-0.04=0.32.$
- $B=\text{No}$ (1 P,4 F): $G=0.32.$
- Weighted $=0.32.$
$$\Delta G(B)=0.5-0.32=\mathbf{0.18}.$$

**Answer.** $\Delta G(B)=0.18>\Delta G(A)=0.083$ → again split on **B**. **Teaching point:** entropy and Gini agreed here, but the *numbers differ* (Gini gain ≠ information gain) and on other datasets the two criteria can rank attributes differently — never quote a Gini value where an entropy value is asked.

---

### Problem 3 (NAT) — Regression tree: best split (the "wrong-criterion" trap)

A regression tree is grown on one numeric feature $x$:

| $x$ | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|
| $y$ | 2 | 4 | 6 | 30 | 32 | 34 |

**Among threshold splits, which `x ≤ t` maximises SSE reduction, and what does each leaf predict?**

> **Trap:** there is no entropy/Gini here — the target is continuous, so we minimise **SSE**, not impurity.

**Parent SSE.** $\bar y = 108/6 = 18$.
$$SSE(S)=16^2+14^2+12^2+12^2+14^2+16^2=256+196+144+144+196+256=\mathbf{1192}.$$

**Candidate $x\le 2.5$:** Left $\{2,4\}$, $\bar y=3$, $SSE=1+1=2$; Right $\{6,30,32,34\}$, $\bar y=25.5$, $SSE=19.5^2+4.5^2+6.5^2+8.5^2=380.25+20.25+42.25+72.25=515$.
Total $=517$ → reduction $=1192-517=675$.

**Candidate $x\le 3.5$:** Left $\{2,4,6\}$, $\bar y=4$, $SSE=4+0+4=8$; Right $\{30,32,34\}$, $\bar y=32$, $SSE=4+0+4=8$.
Total $=16$ → reduction $=1192-16=\mathbf{1176}$.

**Candidate $x\le 4.5$:** Left $\{2,4,6,30\}$, $\bar y=10.5$, $SSE=515$; Right $\{32,34\}$, $\bar y=33$, $SSE=2$. Total $=517$ → reduction $=675$.

**Answer.** The best split is **$x \le 3.5$** (largest reduction, $1176$). Leaf predictions: **$\hat y = 4$** for $x\le 3.5$ and **$\hat y = 32$** for $x>3.5$.

---

## 6. Common GATE question types on this topic

- **Compute entropy / Gini** of a given node or full dataset (watch base-2 and multi-class max).
- **Compute Information Gain or Gini gain for a split → pick the root / next split** (the single most common type).
- **Compute Gain Ratio** and contrast with Information Gain (high-cardinality bias).
- **Regression tree:** best threshold by **SSE / variance reduction**; leaf value = **mean**.
- **"Which algorithm uses which criterion / split type?"** (ID3 ↔ IG ↔ multiway; CART ↔ Gini/SSE ↔ binary).
- **Structural MCQ/MSQ:** axis-parallel boundaries, scale-invariance, non-parametric, greedy ≠ optimal, can-overfit-to-0-training-error, low-bias/high-variance.
- **Pruning concepts:** pre- vs post-pruning; effect of $\alpha$ in $R_\alpha(T)=R(T)+\alpha|\tilde T|$ on tree size.
- **Count leaves / depth**, or "minimum depth needed to separate this data."

---

## 7. Traps, pitfalls & common mistakes

- **Log base.** Entropy uses **$\log_2$** (bits) unless told otherwise. Using $\ln$ gives nats — every entropy is off by a factor $\ln 2 \approx 0.693$. *Information gain ratios* are base-independent, but *raw entropy values* are not.
- **Forgetting to weight children.** The split term is $\sum_v \frac{|S_v|}{|S|}H(S_v)$ — weight by the **fraction of rows**, do **not** just average the two child entropies.
- **Entropy is not capped at 1.** Only for binary classes. For $k$ classes the max is $\log_2 k$. A 3-class node can have entropy up to $\log_2 3 = 1.585$.
- **Negative information gain = arithmetic error.** $IG \ge 0$ always (conditioning can't raise expected entropy). Same for Gini gain and SSE reduction.
- **Mixing the formulas.** Entropy $=-\sum p\log_2 p$; Gini $=1-\sum p^2$. They are different functions — a "Gini gain" answer to an "information gain" question is wrong even if the chosen attribute matches.
- **$0\log_2 0 = 0$.** A pure node contributes $0$, not `NaN`. Pure node ⇒ entropy 0 ⇒ split stops there.
- **High-cardinality bias of Information Gain (classic).** Add a unique-key column (e.g. `RollNo`, 10 distinct values). Each value isolates **one** row → every child is pure → weighted child entropy $=0$ → $IG = H(S)-0 = 1$, the **maximum possible**. ID3 would "prefer" `RollNo` — a useless, perfectly-overfitting split. **Gain Ratio** divides by $SplitInfo=\log_2 10=3.322$, heavily discounting such many-valued attributes; **CART**'s binary splits also mitigate this. Lesson: *the highest-IG attribute is not always the best predictor.*
- **Wrong criterion for regression.** Regression trees use **SSE/variance reduction**, never entropy/Gini; the leaf outputs the **mean**, not a class.
- **Greedy ≠ optimal/smallest.** ID3/CART pick the locally best split; the tree is not guaranteed minimal or globally optimal.
- **Diagonal boundary in one split — impossible.** Boundaries are axis-parallel; a 45° boundary needs many staircase splits.
- **Misclassification error vs entropy/Gini.** Misclassification error can report **zero gain** for a split that genuinely purifies a node (if the majority class is unchanged on both sides), which is exactly why entropy/Gini are preferred for **growing**.
- **Overfitting trap in MSQ.** "An unpruned decision tree always has higher *test* accuracy than a pruned one" → **False**: it can hit 0 *training* error yet generalise worse.

---

## 8. Connection forward

A decision tree is the **textbook illustration of the bias–variance trade-off (Topic 11)**: a shallow tree / stump is **high-bias, low-variance** (too crude), while a fully grown tree is **low-bias, high-variance** (memorises noise, unstable to tiny data changes). The knobs you just learned — **depth limits, min-samples, and cost-complexity pruning ($\alpha$)** — are literally bias–variance dials. And choosing the *right* depth or $\alpha$ is done by **cross-validation (Topic 12)**. (Variance-reducing ensembles like bagging/random forests and boosting extend trees further, but they are **outside this listed GATE DA syllabus**, so only noted here in passing.)

---

## 9. One-line cheat-sheet summary

**Greedily split the data with axis-parallel yes/no questions chosen to maximise purity gain — Information Gain $=H(S)-\sum\frac{|S_v|}{|S|}H(S_v)$ (ID3), Gini gain (CART-classification), or SSE reduction (CART-regression); leaves predict majority class or mean; grown fully it has low bias but high variance, so prune (depth / min-samples / $\alpha$).**

---

> **Next topic (logically):** **Topic 11 — Bias–Variance Trade-off**, which formalises the low-bias/high-variance behaviour we just saw a decision tree exhibit, and explains *why* pruning and depth control trade one for the other.

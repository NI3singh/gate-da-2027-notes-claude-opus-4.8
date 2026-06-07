# Topic 7 — Naive Bayes Classifier

*GATE DA · Machine Learning · Classification Methods (Chat 2, Topic 7)*

---

## 0. Topic header & syllabus map

- **Topic:** Naive Bayes Classifier
- **Subject:** Machine Learning
- **Where it sits:** Second topic of the **Classification Methods** cluster. It comes right after **k-Nearest Neighbour** and leads into **Linear Discriminant Analysis (LDA)**. k-NN was *non-parametric and lazy* (no model, all work at query time); Naive Bayes is its opposite — a **parametric, eager, probabilistic (generative)** classifier that learns a compact model upfront from **Bayes' theorem** and predicts in a flash.

---

## 1. Why this topic exists (motivation + connection)

**The real problem it solves:** Compute the probability that a point belongs to each class, then pick the most probable class. Concretely: *"Given the features I observe, which class is most likely?"* — i.e. estimate $P(\text{class}\mid \text{features})$.

**What the previous topic (k-NN) couldn't handle:** k-NN must **store the entire training set**, costs $O(nd)$ **per query**, needs careful **feature scaling**, gives **no principled class probabilities**, and **collapses in high dimensions** (the curse of dimensionality makes all distances nearly equal). For something like text with thousands of word-features, k-NN is hopeless.

**How Naive Bayes fixes that:** It assumes a **probability model** and learns a **tiny set of parameters once** (priors + per-feature conditionals). Prediction is then $O(cd)$, storage is just the parameters, and it outputs **calibrated-ish class probabilities**. Crucially, the **naive conditional-independence assumption** lets it dodge the curse of dimensionality: instead of estimating one hard $d$-dimensional joint distribution, it estimates $d$ easy **1-dimensional** conditionals. That is exactly why Naive Bayes shines on high-dimensional data (spam filtering, document classification) where k-NN drowns.

*(It is also the generative twin of Logistic Regression from Topic 5 — see §8.)*

---

## 2. Prerequisites

- **Conditional probability & Bayes' theorem**: $P(A\mid B)=\dfrac{P(B\mid A)P(A)}{P(B)}$.
- **Independence** and **conditional independence** of events.
- **Prior, likelihood, posterior, evidence** terminology.
- **Probability mass vs density** — for discrete (categorical) vs continuous (Gaussian) features.
- **The Gaussian (normal) PDF** and its parameters $\mu,\sigma^2$.
- **Maximum-likelihood estimation (MLE)** of probabilities from frequencies (just "count and divide").
- **argmax / MAP decision** idea.

---

## 3. Core idea — intuition first

You get an email and must decide **spam or not**. You don't compare it to every past email (that's k-NN). Instead you ask: *"In general, how often is mail spam? And among spam, how often do I see the word 'lottery'? Among non-spam?"* You combine these learned frequencies with Bayes' rule.

The **"naive"** part is a deliberate simplification: pretend the words appear **independently of each other given the class**. That's obviously not literally true ("free" and "money" co-occur), yet it makes the math trivial — you just **multiply** each word's per-class probability — and it works surprisingly well **for picking the winning class**, even if the probability numbers themselves are off. We only need the *argmax* to be right, not the exact probabilities.

> One-line intuition: **Bayes' theorem + "features are independent given the class" → multiply simple per-feature probabilities, then pick the biggest.**

---

## 4. Formal treatment

### 4.1 Bayes' theorem for classification

For class $y$ and feature vector $\mathbf{x}=(x_1,\dots,x_d)$:

$$\underbrace{P(y\mid \mathbf{x})}_{\text{posterior}} \;=\; \frac{\overbrace{P(\mathbf{x}\mid y)}^{\text{likelihood}}\;\overbrace{P(y)}^{\text{prior}}}{\underbrace{P(\mathbf{x})}_{\text{evidence}}}$$

### 4.2 The naive (conditional independence) assumption

$$\boxed{\,P(\mathbf{x}\mid y) = \prod_{j=1}^{d} P(x_j \mid y)\,}\qquad\text{(features are conditionally independent given the class)}$$

> **Condition:** This assumption is the *only* thing that makes NB "naive." It is usually **false** in reality (features are often correlated), but NB is robust enough that classification is frequently still accurate.

### 4.3 The classifier (MAP decision rule)

The evidence $P(\mathbf{x})$ is the **same for every class**, so it does not affect the argmax and is dropped:

$$\boxed{\;\hat{y} \;=\; \arg\max_{y}\; P(y)\prod_{j=1}^{d} P(x_j \mid y)\;}$$

- With a **uniform prior** this reduces to the **maximum-likelihood (ML)** rule $\arg\max_y \prod_j P(x_j\mid y)$.
- With class priors it is the **maximum a posteriori (MAP)** rule above.

**Log form (to avoid numeric underflow from multiplying many small numbers):**

$$\hat{y} = \arg\max_{y}\;\Big[\log P(y) + \sum_{j=1}^{d}\log P(x_j\mid y)\Big]$$

### 4.4 Estimating the parameters (MLE = count and divide)

- **Priors:** $\displaystyle P(y=\ell)=\frac{N_\ell}{N}$ (fraction of training points in class $\ell$).
- **Categorical likelihood:** $\displaystyle P(x_j=v\mid y=\ell)=\frac{N_{\ell,\,x_j=v}}{N_\ell}$ (within class $\ell$, fraction with feature value $v$).

### 4.5 Laplace (additive) smoothing — the zero-frequency fix

If a feature value never co-occurs with a class in training, its estimated probability is **0**, which **annihilates the entire product** (one zero kills the posterior). Fix with add-$\alpha$ smoothing (usually $\alpha=1$):

$$\boxed{\;P(x_j=v\mid y=\ell)=\frac{N_{\ell,\,x_j=v}+\alpha}{N_\ell+\alpha\,m_j}\;}$$

where $m_j$ = number of possible values of feature $j$.

> **Trap-flag:** the denominator gets $+\alpha m_j$ (**not** $+\alpha$). For a binary feature, $m_j=2$, so denominator is $N_\ell+2$ when $\alpha=1$.

### 4.6 Variants by feature type

| Variant | Feature type | $P(x_j\mid y)$ model |
|---|---|---|
| **Categorical / Multinomial NB** | discrete (categories, word counts) | frequency-based (with smoothing) |
| **Bernoulli NB** | binary (present/absent) | $p^{x_j}(1-p)^{1-x_j}$ |
| **Gaussian NB** | continuous | Gaussian density (below) |

### 4.7 Gaussian Naive Bayes (continuous features)

Assume each feature is normally distributed within each class:

$$P(x_j\mid y=\ell)=\frac{1}{\sqrt{2\pi\,\sigma_{j\ell}^2}}\,\exp\!\left(-\frac{(x_j-\mu_{j\ell})^2}{2\sigma_{j\ell}^2}\right)$$

Parameters estimated per class per feature: $\mu_{j\ell}$ = class-$\ell$ sample mean of feature $j$; $\sigma_{j\ell}^2$ = class-$\ell$ (MLE/biased) sample variance. For multiple continuous features, multiply these per-feature Gaussians (the naive product).

> **Boundary shape:** Gaussian NB generally produces a **quadratic** decision boundary; if all class variances are forced equal, the boundary becomes **linear** (this links to LDA in §8). And note: with **unequal variances, "nearest mean" is NOT the rule** — a wider-variance class can win even at equal distance.

### 4.8 Why NB scales (parameter count)

For $d$ binary features and $c$ classes:

- **Naive Bayes:** $c\cdot d$ likelihood parameters $+\,(c-1)$ priors → **linear in $d$**.
- **Full joint $P(\mathbf{x}\mid y)$:** $c\,(2^{d}-1)$ parameters → **exponential in $d$**.

Example ($d=10$, $c=2$): NB needs $20+1=21$ parameters; the full joint needs $2(2^{10}-1)=2046$. This is the whole reason NB survives high-dimensional data. *(Skipping the boundary-derivation; GATE tests the count and the assumption, not the derivation.)*

---

## 5. Worked problems (GATE-style)

### Problem 1 — Categorical Naive Bayes (the staple) (MCQ)

Predict **Buy ∈ {Yes, No}** from **Income ∈ {High, Low}** and **Student ∈ {Yes, No}**:

| # | Income | Student | Buy | | # | Income | Student | Buy |
|---|---|---|---|---|---|---|---|---|
| 1 | High | No | No | | 6 | Low | No | Yes |
| 2 | High | No | No | | 7 | High | Yes | Yes |
| 3 | Low | No | Yes | | 8 | High | No | Yes |
| 4 | Low | Yes | Yes | | 9 | Low | Yes | Yes |
| 5 | Low | Yes | No | | 10| High | No | No |

Classify the query **(Income = High, Student = Yes)**.

**Step 1 — Priors.** Buy=Yes in rows {3,4,6,7,8,9}=6; No in {1,2,5,10}=4.
$$P(\text{Yes})=\tfrac{6}{10}=0.6,\qquad P(\text{No})=\tfrac{4}{10}=0.4$$

**Step 2 — Likelihoods.**
*Within Yes (6 rows):* Income=High in {7,8}=2 → $P(\text{High}\mid\text{Yes})=\tfrac{2}{6}=\tfrac13$. Student=Yes in {4,7,9}=3 → $P(\text{Yes}_{stu}\mid\text{Yes})=\tfrac{3}{6}=\tfrac12$.
*Within No (4 rows):* Income=High in {1,2,10}=3 → $P(\text{High}\mid\text{No})=\tfrac{3}{4}$. Student=Yes in {5}=1 → $P(\text{Yes}_{stu}\mid\text{No})=\tfrac{1}{4}$.

**Step 3 — Posterior scores (drop the common denominator).**
$$P(\text{Yes}\mid x)\propto 0.6\cdot\tfrac13\cdot\tfrac12 = 0.1$$
$$P(\text{No}\mid x)\propto 0.4\cdot\tfrac34\cdot\tfrac14 = 0.075$$

**Step 4 — Decide & normalize.** $0.1>0.075$ → **Buy = Yes**. Probability $=\dfrac{0.1}{0.1+0.075}\approx \mathbf{0.571}$ (and $P(\text{No}\mid x)\approx0.429$).

> **Lesson:** Income=High alone favoured No ($\tfrac34$ vs $\tfrac13$), but the prior plus Student=Yes outweighed it. NB **combines all evidence multiplicatively** — never decide on one feature.

---

### Problem 2 — Zero-frequency problem & Laplace smoothing (NAT / MCQ — the trap)

A spam filter (binary features present/absent). Training counts:

| | total | "lottery" present | "free" present |
|---|---|---|---|
| **Spam** | 6 | 4 | 5 |
| **Ham** | 4 | **0** | 1 |

Classify the email **(lottery = present, free = present)**. Priors $P(\text{Spam})=0.6,\ P(\text{Ham})=0.4$.

**(a) Without smoothing:**
$$P(\text{Spam}\mid x)\propto 0.6\cdot\tfrac46\cdot\tfrac56 = 0.6\cdot 0.5556 = 0.3333$$
$$P(\text{Ham}\mid x)\propto 0.4\cdot\underbrace{\tfrac04}_{=0}\cdot\tfrac14 = \mathbf{0}$$
Decision: **Spam** — but Ham's posterior is **exactly 0**. The single never-seen pair (lottery|Ham) wiped out *all* other evidence. That's the **zero-frequency problem**: overconfident and brittle.

**(b) With Laplace smoothing** ($\alpha=1$, each feature has $m=2$ values, so denominator $=N_\ell+2$):
$$P(\text{lottery=pres}\mid\text{Spam})=\tfrac{4+1}{6+2}=\tfrac58,\quad P(\text{free=pres}\mid\text{Spam})=\tfrac{5+1}{6+2}=\tfrac34$$
$$P(\text{lottery=pres}\mid\text{Ham})=\tfrac{0+1}{4+2}=\tfrac16,\quad P(\text{free=pres}\mid\text{Ham})=\tfrac{1+1}{4+2}=\tfrac13$$

$$P(\text{Spam}\mid x)\propto 0.6\cdot\tfrac58\cdot\tfrac34 = 0.28125,\qquad P(\text{Ham}\mid x)\propto 0.4\cdot\tfrac16\cdot\tfrac13 = 0.02222$$

Decision: **Spam** (≈ **0.927** after normalizing). Now Ham has a small **non-zero** probability instead of an absurd 0.

> **Lesson:** A single zero likelihood annihilates the whole product → **always smooth**. And remember the denominator gets $+\alpha m$, not $+\alpha$.

---

### Problem 3 — Gaussian Naive Bayes (continuous feature) (NAT — with a variance trap)

One continuous feature. Class parameters and equal priors:

$$C_1:\ \mu_1=5,\ \sigma_1^2=1 \qquad C_2:\ \mu_2=8,\ \sigma_2^2=4 \qquad P(C_1)=P(C_2)=0.5$$

Classify the query **$x=6.5$**.

**Likelihoods** (note $|x-\mu_1| = |x-\mu_2| = 1.5$ — equidistant from both means):
$$P(x\mid C_1)=\frac{1}{\sqrt{2\pi(1)}}\exp\!\Big(-\frac{1.5^2}{2(1)}\Big)=0.3989\cdot e^{-1.125}=0.3989\cdot0.3247=0.1295$$
$$P(x\mid C_2)=\frac{1}{\sqrt{2\pi(4)}}\exp\!\Big(-\frac{1.5^2}{2(4)}\Big)=0.1995\cdot e^{-0.28125}=0.1995\cdot0.7548=0.1506$$

**Posteriors** (equal priors, so $\propto$ likelihoods): $C_1\propto0.1295,\ C_2\propto0.1506$.
$$P(C_1\mid x)=\frac{0.1295}{0.1295+0.1506}\approx0.462,\qquad P(C_2\mid x)\approx0.538$$

**Answer: classify as $C_2$ (≈ 53.8%).**

> **Lesson (trap):** $x$ is *equally far* from both means, yet it goes to $C_2$ — the class with **larger variance** has a flatter, wider bell, so its density is higher at distance 1.5. With **unequal variances, the boundary is not the midpoint** and "nearest mean" is wrong. (Equal variances → midpoint boundary, the LDA case in §8.)

---

## 6. Common GATE question types on this topic

- **Compute posteriors and classify** from a small categorical table (Problem 1 type) — most common.
- **Spot/fix the zero-frequency problem** with **Laplace smoothing** (Problem 2 type).
- **Gaussian NB density computation** and classification (Problem 3 type).
- **Conceptual:** what does *"naive"* mean? Why can we **drop $P(\mathbf{x})$**? Is NB **generative or discriminative**? (Generative.)
- **Effect of the independence assumption** being violated (probabilities biased, but classification often fine).
- **Parameter count** of NB vs the full joint (linear vs exponential in $d$).
- **MAP vs ML** — when does the prior change the decision?
- **Log-likelihood form** / why logs are used (underflow).

---

## 7. Traps, pitfalls & common mistakes

- **Zero-frequency annihilation** — one unseen value gives probability 0 and kills the product. **Always smooth.**
- **Wrong Laplace denominator** — it is $N_\ell+\alpha m_j$ (add the number of feature values), not $N_\ell+\alpha$.
- **Multiplying many tiny probabilities → underflow.** Use the **log-sum** form in larger problems.
- **Inverting the conditional** — the likelihood is $P(\text{feature}\mid\text{class})$, **not** $P(\text{class}\mid\text{feature})$. Mixing these is the #1 conceptual error.
- **Forgetting the prior** (using ML when the classes are imbalanced and the prior matters → wrong MAP answer).
- **Forgetting to normalize** when the question asks for an actual *probability* (the dropped $P(\mathbf{x})$ must be restored as the sum of the scores).
- **"Nearest mean" intuition for Gaussian NB** — only valid when variances are equal; with unequal variances a farther-but-wider class can win (Problem 3).
- **Using Gaussian NB on clearly non-Gaussian features** — the model assumption breaks.
- **Believing the probabilities are well-calibrated** — because of the false independence assumption, NB's probability *values* are often too extreme (near 0/1); the *ranking/argmax* is what's reliable.
- **Treating $P(\mathbf{x})$ as something to compute for classification** — it's the same for all classes and irrelevant to the argmax.

---

## 8. Connection forward

Naive Bayes (Gaussian version) models each class as a Gaussian and applies Bayes' rule — it is a **generative** classifier, just like the next topic, **Linear Discriminant Analysis (LDA)**. The difference is the covariance assumption:

- **Gaussian NB:** features independent given class → **diagonal** covariance (and possibly different variances per class) → generally a **quadratic** boundary.
- **LDA:** **drops independence** — it models each class as a *multivariate Gaussian with a full covariance* that captures feature **correlations** — but **ties that covariance to be shared across all classes**, which yields a **linear** boundary.

So the bridge is: **LDA = Gaussian NB that allows correlated features (full $\Sigma$) but assumes every class shares one common $\Sigma$.** (And **QDA** allows each class its own full covariance → quadratic — completing the NB → LDA → QDA family.)

*Also note the discriminative twin:* Naive Bayes is the **generative** counterpart of **Logistic Regression** (Topic 5). In fact, Gaussian NB with shared variance produces a posterior of the exact **logistic (sigmoid) form** — the same two methods seen from two directions (model $P(\mathbf{x},y)$ then invert, vs model $P(y\mid\mathbf{x})$ directly).

---

## 9. One-line cheat-sheet summary

**Naive Bayes:** $\hat{y}=\arg\max_y P(y)\prod_j P(x_j\mid y)$ — generative, parametric, eager; **assume features independent given the class** (drop $P(\mathbf{x})$, use logs for underflow); estimate priors & conditionals by counting; **Laplace-smooth** (denominator $+\alpha m$) to kill zeros; Gaussian NB for continuous features (boundary quadratic, linear if equal variances; *not* "nearest mean" when variances differ); scales to high-$d$ because it needs only $O(cd)$ parameters.

---

### ➡️ Next topic
**Topic 8 — Linear Discriminant Analysis (LDA)** (still in the Classification Methods cluster). Ask for it when ready.

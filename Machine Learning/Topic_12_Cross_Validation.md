# Topic 12 — Cross-Validation Methods (LOO & k-Fold)

> **Subject:** Machine Learning  **|**  **Section:** Trees & Model Evaluation  **|**  **Position:** Topic 12 of the ML sequence (final topic of this section)

---

## 0. Topic header & syllabus map

- **Topic:** Cross-Validation — **Leave-One-Out (LOOCV)** and **k-Fold** — for estimating generalisation error and selecting models/hyperparameters.
- **Subject:** Machine Learning → model evaluation (a procedure that wraps *around* every supervised model).
- **Where it sits:** The **last topic of the *Trees & Model Evaluation* block**. It comes **immediately after Bias–Variance Trade-off (Topic 11)** — which told us we must minimise test error but can't see it directly — and **closes the evaluation toolkit** before the syllabus resumes building models with **Multi-Layer Perceptron (Topic 13, next chat)**.
- **One-line placement:** Bias–variance told us *what* to optimise (test error); cross-validation is *how* we estimate it from data alone, so we can locate the bottom of the U-curve and tune knobs like $\lambda$, tree depth, and $k$.

---

## 1. Why this topic exists (motivation + connection)

The previous topic ended on a cliff-hanger: expected test error $=\text{Bias}^2+\text{Variance}+\sigma^2$, the best model sits at the bottom of a **U-shaped curve**, but in practice we **cannot compute bias or variance** — we don't know the true $f$ or the noise $\sigma^2$, and we hold only **one** training set, not the many that those expectations require. We need a way to **estimate test error using only the data we have**, and then to **pick the complexity** (depth, $\lambda$, $k$, degree) that minimises it.

The naive fixes fail. **Training error** is optimistically biased — a flexible model can memorise the training set (recall 1-NN has *zero* training error, an unpruned tree can hit zero too), so training error rewards exactly the overfitting we are trying to avoid. A single **train/validation split (holdout)** is honest but wasteful: it trains on only part of the data (pessimistic) and its estimate swings wildly depending on *which* rows happened to fall in the validation set (high variance).

**Cross-validation** fixes both: it **reuses every observation for both training and validation** by rotating which part is held out, then averages. This squeezes a low-variance, nearly-unbiased estimate of test error out of a small dataset. That estimate is precisely what lets us **act on the bias–variance trade-off** — choose the model with the smallest estimated test error — which is why this topic is the natural sequel to Topic 11 and the gateway to evaluating the more flexible (high-capacity, high-variance) models, starting with **neural networks**, that come next.

---

## 2. Prerequisites

- **Bias–variance trade-off (Topic 11):** the target quantity (expected test error) and the U-curve.
- **MSE / 0–1 loss** as error measures (regression vs classification).
- **The hyperparameters to be tuned:** $\lambda$ in ridge (Topic 4), $k$ in kNN (Topic 6), depth/pruning in trees (Topic 10).
- **For the LOOCV shortcut (§4.5):** the linear-regression **hat matrix** $H=X(X^\top X)^{-1}X^\top$ and the idea of **leverage** $h_{ii}$.
- **Mean and variance of an average** (used to explain why LOOCV is high-variance).

---

## 3. Core idea — intuition first

You have a small dataset and one question: *how well will my model do on data it has never seen?* You can't peek at future data, so you **simulate** unseen data by **hiding part of your own data, training on the rest, and testing on the hidden part.** Do this once and the answer depends on a lucky/unlucky split. So do it **many times, rotating the hidden part**, and **average**.

- **k-Fold:** chop the data into $k$ equal blocks ("folds"). In round $i$, hide fold $i$, train on the other $k-1$, score on fold $i$. After $k$ rounds **every point has been a test point exactly once**; average the $k$ scores.
- **Leave-One-Out (LOOCV):** push this to the extreme — each "fold" is a **single point**. Train on $n-1$ points, test on the 1 left out, repeat $n$ times.

**Why it beats the alternatives.** Compared with training error, CV tests on data the model didn't train on, so it can't be fooled by memorisation. Compared with one holdout split, CV uses *all* the data for training across the rounds (so the estimate isn't pessimistic) and averages away the luck of a single split (so it isn't high-variance).

**The one design choice — how big to make $k$ — is itself a bias–variance decision** (§4.6): small folds (large $k$, up to LOOCV) train on almost-all the data → low bias but unstable; moderate $k$ (5 or 10) is the sweet spot.

---

## 4. Formal treatment

### 4.1 What we are estimating

For a fixed point, **test (generalisation) error** is the expected loss on a fresh observation; averaged over inputs it is the model's true error rate / MSE. **Training error** systematically **underestimates** it (the model was fit *to* the training points), and the gap grows with model flexibility. CV is a resampling estimate of test error that avoids this optimism.

### 4.2 The validation-set (holdout) approach — and why it isn't enough

Split once into train / validation (say 80/20). Train on the train part, report the loss on the validation part. Two weaknesses:

1. **High variance:** the estimate depends heavily on which rows landed in validation — a different split gives a different number.
2. **Pessimistic (upward) bias:** the model is trained on only a fraction of the data, and models trained on less data perform worse, so holdout *over*-estimates the error of the model you'd train on the full data.

### 4.3 k-Fold cross-validation

**Procedure.** Partition the $n$ points into $k$ disjoint folds $F_1,\dots,F_k$ (roughly equal, randomly assigned). For $i=1,\dots,k$: train on all data **except** $F_i$, evaluate the loss $\text{Err}_i$ on $F_i$. Then average.

$$
\boxed{\;CV_{(k)} = \frac{1}{k}\sum_{i=1}^{k}\text{Err}_i\quad\text{(equal folds)}\;}
\qquad\quad
CV_{(k)} = \sum_{i=1}^{k}\frac{n_i}{n}\,\text{Err}_i\quad\text{(general / unequal folds)}
$$

where $\text{Err}_i$ is the average loss on fold $i$ (MSE for regression, misclassification rate for classification) and $n_i=|F_i|$.

- **Each point is validated exactly once and used for training $k-1$ times.**
- **Cost: $k$ model fits** per hyperparameter setting.
- **Typical choice: $k=5$ or $k=10$.**
- **Stratified k-fold (classification):** assign folds so each preserves the overall **class proportions** — essential for imbalanced data, so no fold misses a class.

### 4.4 Leave-One-Out cross-validation (LOOCV)

**LOOCV is the special case $k=n$.** Each fold is one observation: train on the other $n-1$ points, predict the held-out point $\hat y_{-i}$, repeat for all $i$.

$$
\boxed{\;CV_{(n)} = \frac{1}{n}\sum_{i=1}^{n}\text{Err}_i,\qquad
\text{Err}_i=\begin{cases}(y_i-\hat y_{-i})^2 & \text{regression (squared loss)}\\[2pt]\mathbb{1}\!\left[\hat y_{-i}\neq y_i\right] & \text{classification (0–1 loss)}\end{cases}\;}
$$

- **Cost: $n$ model fits** (expensive for large $n$) — *except* for the linear-model shortcut below.
- **Deterministic:** there is no random fold assignment, so LOOCV gives the **same answer every run** (unlike $k$-fold, which depends on the random split).

### 4.5 The LOOCV shortcut for linear / ridge regression (leverage formula)

For **ordinary least-squares linear regression** (and ridge), LOOCV needs **only one fit on the full data** — no $n$ refits:

$$
\boxed{\;CV_{(n)} = \frac{1}{n}\sum_{i=1}^{n}\left(\frac{y_i-\hat y_i}{1-h_{ii}}\right)^2\;}
$$

where $\hat y_i$ is the fitted value from the model trained on **all** data, and $h_{ii}$ is the $i$-th diagonal of the **hat matrix** $H=X(X^\top X)^{-1}X^\top$ (the **leverage** of point $i$). Key facts:

- $\hat{\mathbf y}=H\mathbf y$, residuals $\mathbf e=(I-H)\mathbf y$, and $0\le h_{ii}\le 1$.
- $\displaystyle\sum_i h_{ii}=\text{trace}(H)=p$ (number of fitted parameters — a handy check).
- The factor $\frac{1}{1-h_{ii}}$ **inflates** the residual of high-leverage points (those far from $\bar x$), which is exactly how much *worse* the model does on a point it was *not* trained on.
- **Conditions:** linear least-squares (for ridge, use $H_\lambda=X(X^\top X+\lambda I)^{-1}X^\top$) and **squared-error loss**. It does **not** apply to trees, kNN, SVM, logistic regression, etc. — those require real refitting.

### 4.6 LOOCV vs k-Fold — the bias–variance trade-off *of the estimator* (ties back to Topic 11)

The choice of $k$ trades bias against variance **in the error estimate itself**:

| Property | **LOOCV** ($k=n$) | **k-Fold** ($k=5$ or $10$) |
|---|---|---|
| Training-set size per fit | $n-1$ (≈ full) | $\frac{k-1}{k}n$ (e.g. 80–90%) |
| **Bias** of the estimate | **low** (≈ unbiased for the full-data model) | **slightly higher / pessimistic** (trains on less) |
| **Variance** of the estimate | **high** | **lower** |
| Compute | $n$ fits (or 1 via shortcut) | $k$ fits |
| Randomness | none (deterministic) | depends on the random split |

**Why LOOCV has *higher* variance** (standard/textbook result, and a favourite conceptual question): LOOCV averages $n$ models each trained on **almost identical** data, so their errors are **highly positively correlated**, and *the mean of highly correlated quantities does not shrink in variance.* For the average of $m$ terms each with variance $s^2$ and pairwise correlation $\rho$,
$$
\mathrm{Var}(\text{average})=\rho s^2+\frac{(1-\rho)s^2}{m}\;\xrightarrow{\ \rho\to 1\ }\;s^2,
$$
so high correlation ($\rho$ near 1, as in LOOCV) keeps the variance large no matter how many folds. $k$-fold's folds overlap less → lower $\rho$ → lower variance. **Empirical sweet spot: $k=5$ or $10$**, balancing modest bias against modest variance and cost.

### 4.7 Using CV in practice

- **Model assessment:** estimate the test error of a *given* model.
- **Model / hyperparameter selection:** compute $CV$ for each candidate (each $\lambda$, $k$, depth, degree) and pick the one with the **smallest CV error** — this is literally finding the bottom of the bias–variance U-curve from data.
- **One-standard-error (1-SE) rule:** among models within one standard error of the minimum CV error, choose the **simplest** (extra protection against overfitting the selection).
- **Refit on all data:** once CV has chosen the hyperparameters, **retrain the final model on the entire dataset**. CV is for *estimating/choosing*, not the deliverable model.
- **Nested CV** (one line, mostly beyond GATE): if you tune *and* want an unbiased performance estimate, wrap selection in an inner CV and assessment in an outer CV — because the CV error of the *selected* model is itself slightly optimistic.

### 4.8 What breaks CV — leakage (read with §7)

Anything that lets information from the held-out fold sneak into training **biases CV optimistically**. The rule: **every data-dependent preprocessing step must be fit *inside* the CV loop, on the training folds only**, then applied to the validation fold — this includes **feature scaling/normalisation, feature selection, PCA, missing-value imputation, and resampling/oversampling**. Doing them once on the *whole* dataset before splitting leaks the validation fold (the notorious result: select "best" features on all data, then CV, and you can get great accuracy even on **pure noise**).

---

## 5. Worked problems (GATE-style)

### Problem 1 (NAT) — k-Fold for hyperparameter selection

A ridge-regression model is tuned by **5-fold CV** over $\lambda\in\{0.1,\,1,\,10\}$ (equal folds). The five fold-MSEs for each $\lambda$ are:

| $\lambda$ | Fold-MSEs | $CV_{(5)}$ |
|---|---|---|
| 0.1 | 5.2, 4.8, 6.0, 5.0, 4.0 | ? |
| 1 | 3.0, 3.5, 2.5, 3.0, 3.0 | ? |
| 10 | 4.0, 5.0, 4.5, 5.5, 6.0 | ? |

**Compute each $CV_{(5)}$ and choose $\lambda$.**

$$CV(0.1)=\tfrac{5.2+4.8+6.0+5.0+4.0}{5}=\tfrac{25}{5}=5.0,$$
$$CV(1)=\tfrac{3.0+3.5+2.5+3.0+3.0}{5}=\tfrac{15}{5}=\mathbf{3.0},$$
$$CV(10)=\tfrac{4.0+5.0+4.5+5.5+6.0}{5}=\tfrac{25}{5}=5.0.$$

**Choose $\lambda=1$** (smallest CV error). Note the **U-shape**: $\lambda=0.1$ is the low-regularisation (low-bias/high-variance) end and $\lambda=10$ the high-bias end; CV located the trade-off minimum in between — exactly the Topic-11 picture, now found from data.

---

### Problem 2 (NAT / MCQ) — LOOCV by hand for 1-NN classification

Five points on a line with class labels:
$$(1,A),\ (2,A),\ (4,B),\ (5,B),\ (8,A).$$
Using **1-nearest-neighbour** and **LOOCV (0–1 loss)**, find the LOOCV error. *(For reference: what is the 1-NN training error?)*

Leave each point out and classify it by its nearest **remaining** neighbour:

| Held-out | Nearest remaining | Predict | Actual | Correct? |
|---|---|---|---|---|
| $x=1$ (A) | $x=2$ (dist 1) | A | A | ✓ |
| $x=2$ (A) | $x=1$ (dist 1) vs $x=4$ (dist 2) → $x=1$ | A | A | ✓ |
| $x=4$ (B) | $x=5$ (dist 1) vs $x=2$ (dist 2) → $x=5$ | B | B | ✓ |
| $x=5$ (B) | $x=4$ (dist 1) vs $x=8$ (dist 3) → $x=4$ | B | B | ✓ |
| $x=8$ (A) | $x=5$ (dist 3) vs $x=4$ (dist 4) → $x=5$ | **B** | A | ✗ |

One error out of five:
$$\boxed{CV_{(n)}=\tfrac{1}{5}=0.2\ (20\%)}.$$

**Reference / teaching point:** 1-NN **training error is 0** (each point is its own nearest neighbour), yet LOOCV error is 0.2. This is the optimism of training error made concrete — the outlier at $x=8$ (an A stranded among B's) is only exposed once you hold it out.

---

### Problem 3 (NAT) — LOOCV shortcut via leverage (the "forgot the inflation" trap)

Fit **simple linear regression** ($y=a+bx$) to three points:
$$(-1,\,1),\ (0,\,2),\ (1,\,6).$$
Compute LOOCV (squared loss) using the leverage shortcut, and verify by brute force.

**Step 1 — full-data fit.** $\bar x=0,\ \bar y=3$. $S_{xx}=\sum(x_i-\bar x)^2=1+0+1=2$. $S_{xy}=(-1)(-2)+(0)(-1)+(1)(3)=2+0+3=5$. So $b=S_{xy}/S_{xx}=2.5$, $a=\bar y-b\bar x=3$. Line: $\hat y=3+2.5x$.

Residuals $e_i=y_i-\hat y_i$:
$$e_1=1-(3-2.5)=0.5,\quad e_2=2-3=-1,\quad e_3=6-(3+2.5)=0.5\quad(\textstyle\sum e_i=0\ \checkmark).$$

**Step 2 — leverages** $h_{ii}=\frac1n+\frac{(x_i-\bar x)^2}{S_{xx}}$ (with $n=3$):
$$h_{11}=\tfrac13+\tfrac{1}{2}=\tfrac56,\quad h_{22}=\tfrac13+0=\tfrac13,\quad h_{33}=\tfrac13+\tfrac12=\tfrac56.$$
Check: $\sum h_{ii}=\tfrac56+\tfrac13+\tfrac56=2=p$ (slope + intercept) $\checkmark$.

**Step 3 — apply the shortcut.**
$$
\frac{e_1}{1-h_{11}}=\frac{0.5}{1/6}=3,\quad
\frac{e_2}{1-h_{22}}=\frac{-1}{2/3}=-1.5,\quad
\frac{e_3}{1-h_{33}}=\frac{0.5}{1/6}=3.
$$
$$
CV_{(n)}=\tfrac13\big(3^2+(-1.5)^2+3^2\big)=\tfrac13(9+2.25+9)=\tfrac13(20.25)=\mathbf{6.75}.
$$

**Verification by brute force** (refit leaving each out):
- Drop $(-1,1)$: fit $(0,2),(1,6)\Rightarrow y=2+4x$; predict at $x=-1$: $-2$; residual $1-(-2)=3$ → $9$.
- Drop $(0,2)$: fit $(-1,1),(1,6)\Rightarrow y=3.5+2.5x$; predict at $x=0$: $3.5$; residual $-1.5$ → $2.25$.
- Drop $(1,6)$: fit $(-1,1),(0,2)\Rightarrow y=2+x$; predict at $x=1$: $3$; residual $3$ → $9$.

$CV=\tfrac13(9+2.25+9)=6.75$ — **identical**, with **one** full fit instead of three.

**Trap caught:** a student who uses the raw residuals $0.5,-1,0.5$ (forgetting the $\tfrac{1}{1-h_{ii}}$ inflation) gets $\tfrac13(0.25+1+0.25)=0.5$ — wildly too small. The high-leverage endpoints ($x=\pm1$) blow up from $0.5$ to $3$ precisely because the model relies on them heavily and does much worse once they are removed.

---

## 6. Common GATE question types on this topic

- **Compute $CV_{(k)}$** as the average of given fold errors (Problem 1); pick the hyperparameter with the lowest CV error.
- **Compute LOOCV by hand** for a small dataset + simple model — mean predictor, 1-NN, or simple linear regression (Problem 2).
- **Apply the LOOCV leverage shortcut** for linear/ridge regression (Problem 3).
- **Counting:** "How many models are trained in $k$-fold? in LOOCV?" → $\mathbf{k}$ and $\mathbf{n}$.
- **Definitional:** "LOOCV is $k$-fold with $k=?$" → $\mathbf{k=n}$.
- **Compare LOOCV vs $k$-fold** on bias, variance, and cost (§4.6).
- **Spot the leakage:** identify why a described CV procedure is optimistically biased (preprocessing outside the loop).
- **Conceptual:** why CV beats a single holdout / why not just use training error.

---

## 7. Traps, pitfalls & common mistakes

- **Data leakage from preprocessing — the big one.** Scaling, feature **selection**, PCA, imputation, and oversampling must be fit on the **training folds only** *inside* each CV round. Fitting them on the whole dataset first leaks the validation fold → falsely low error (you can get high "accuracy" on pure noise this way).
- **LOOCV is NOT low-variance.** Common misconception. LOOCV is **low-bias but high-variance** (its $n$ models are nearly identical → highly correlated errors). Moderate $k$ (5–10) gives a **lower-variance** estimate.
- **Training error ≠ CV error.** 1-NN and unpruned trees have ~0 training error but nonzero CV error; never report training error as a generalisation estimate.
- **$k$ = number of folds, not number of points** — except LOOCV, where $k=n$. Don't conflate them.
- **Unequal folds need size-weighting.** Use $\sum_i\frac{n_i}{n}\text{Err}_i$ (equivalently, pool all errors and divide by $n$); plainly averaging fold MSEs is only valid when folds are equal.
- **Forgetting leverage inflation** in the LOOCV shortcut: you must divide each residual by $(1-h_{ii})$; raw residuals badly **under**estimate LOOCV.
- **The shortcut is linear-models-only.** It works for least-squares (and ridge with $H_\lambda$); it does **not** apply to trees, kNN, SVM, logistic regression — those require actual refitting.
- **Dependent data breaks i.i.d. folds.** For **time series**, random folds leak the future into the past → use forward-chaining / time-ordered splits. For **grouped data** (e.g. several rows per patient), keep each group entirely in one fold (group-aware CV), else leakage.
- **Class imbalance:** use **stratified** folds, or a fold may contain none of a rare class.
- **The CV error of the *selected* model is optimistic.** If you both tune *and* report performance, you've used validation folds for selection → use a held-out test set or nested CV for an honest estimate.
- **Don't touch the test set during tuning.** It is evaluated once, at the very end.
- **$k$-fold is random, LOOCV is deterministic.** Different $k$-fold splits give different CV numbers; repeat and average if you need a stable estimate.
- **More folds ≠ always better.** LOOCV maximises folds but raises variance and cost (n fits); $k=5/10$ is usually the better practical choice.

---

## 8. Connection forward

With cross-validation, the **model-evaluation toolkit is complete**: we can now estimate test error from data and pick the complexity that minimises the bias–variance sum derived in Topic 11. From here the syllabus **goes back to building models** — beginning a new chat with the **Multi-Layer Perceptron (Topic 13)** and feed-forward neural networks. Those models are **universal approximators** with very high capacity, so they live on the **low-bias / high-variance** end of the spectrum and are unusually prone to overfitting. That makes everything from this section indispensable for them: their many hyperparameters (number of layers and units, learning rate, regularisation strength, early-stopping point) are selected and validated using exactly the **k-fold / holdout cross-validation** procedures developed here. In short, we finish the evaluation chapter so that the powerful models coming next can be **trained, tuned, and trusted**.

---

## 9. One-line cheat-sheet summary

**Cross-validation estimates test error by rotating a held-out part: $k$-fold averages the error over $k$ folds ($CV_{(k)}=\frac1k\sum\text{Err}_i$, $k$ fits, $k=5/10$ typical), LOOCV is the $k=n$ extreme (one point out at a time, $n$ fits, low-bias but high-variance, with the linear-regression shortcut $CV=\frac1n\sum(\frac{y_i-\hat y_i}{1-h_{ii}})^2$); use the smallest-CV-error hyperparameter, keep all preprocessing inside the loop, then refit on all data.**

---

> **Next topic (new chat — Neural Networks):** **Topic 13 — Multi-Layer Perceptron**, where we resume building models with a flexible, high-capacity learner whose hyperparameters we will tune with the cross-validation methods just covered.

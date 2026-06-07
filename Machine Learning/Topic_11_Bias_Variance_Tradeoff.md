# Topic 11 — Bias–Variance Trade-Off

> **Subject:** Machine Learning  **|**  **Section:** Trees & Model Evaluation  **|**  **Position:** Topic 11 of the ML sequence

---

## 0. Topic header & syllabus map

- **Topic:** Bias–Variance Trade-off (the decomposition of expected prediction error into bias², variance, and irreducible noise).
- **Subject:** Machine Learning → Supervised Learning (a *model-evaluation* idea that applies to every supervised model).
- **Where it sits:** Second topic of the *Trees & Model Evaluation* block. It comes **immediately after Decision Trees (Topic 10)** — which gave us a model whose bias/variance we could dial by hand — and leads into **Cross-Validation (Topic 12)**, the tool we use to *measure* generalisation error and locate the trade-off's sweet spot.
- **One-line placement:** This is the lens through which we explain *why* any model under- or over-fits; trees motivated it, cross-validation will let us act on it.

---

## 1. Why this topic exists (motivation + connection)

By now we have built many models — linear/ridge/logistic regression, kNN, Naive Bayes, LDA, SVM, decision trees — and we keep meeting the same two opposite failure modes. A model can be **too rigid**, missing the real pattern no matter how much data you give it (e.g. a straight line fit to a curve, or a 1-node tree stump). Or it can be **too flexible**, chasing the noise in the particular training set so it changes wildly if you reshuffle the data (e.g. a fully grown, unpruned tree, or 1-NN). The **previous topic made this vivid**: a deep tree had near-zero training error but was unstable, while a stump was stable but crude.

The bias–variance trade-off turns this qualitative observation into an **exact accounting identity**: for squared-error loss, expected test error splits cleanly into **bias²** (error from the model being systematically wrong / too simple), **variance** (error from sensitivity to the particular training sample / too complex), and **irreducible error** $\sigma^2$ (noise no model can remove). This is what the earlier topics couldn't give us — a *principled* way to say which failure mode we have and what to do: the same identity explains **why ridge regularisation (Topic 4) helps** (it accepts a little bias to cut variance) and **why pruning a tree helps**.

It exists, finally, because it tells us the goal is **not** to minimise *training* error (that just rewards complexity and overfitting) but to balance bias against variance — and since we can't see bias or variance directly from one dataset, it **motivates the next topic, Cross-Validation**, which estimates the test error so we can find the balance point.

---

## 2. Prerequisites

- **Expectation and variance:** $E[X]$, $\mathrm{Var}(X)=E[(X-E[X])^2]=E[X^2]-(E[X])^2$.
- **Squared-error loss / MSE** (from the regression chat).
- **Key mental model:** a fitted model $\hat f$ is a *random object* — it depends on the random training set $D$. "Variance of the model" means variance **across different training sets**.
- **Regularisation (ridge, Topic 4)** and **decision-tree depth/pruning (Topic 10)** and **kNN's $k$ (Topic 6)** — used throughout as the knobs that move bias and variance.

---

## 3. Core idea — intuition first

**The dartboard analogy.** Think of throwing darts at a bullseye. The bullseye is the true value $f(x_0)$; each dart is the prediction $\hat f(x_0)$ you'd get from one training set. Train on many different datasets → many darts.

- **Bias** = how far the *average* dart lands from the bullseye (a systematic offset).
- **Variance** = how *scattered* the darts are around their own average (instability).

| | **Low variance** | **High variance** |
|---|---|---|
| **Low bias** | Tight cluster on the bullseye — *ideal* | Scattered around the bullseye: right *on average*, but any single fit is off → **overfitting** |
| **High bias** | Tight cluster, but off-target: consistently wrong → **underfitting** | Scattered *and* off-target → worst case |

**Why it's a trade-off.** Making a model more flexible (deeper tree, higher-degree polynomial, smaller $k$) lets the average dart move onto the bullseye (**bias ↓**) but makes the throws jumpy (**variance ↑**). Making it more rigid does the reverse. You usually cannot kill both at once, so you tune complexity to the point where **bias² + variance** is smallest.

**Train vs test, in one breath.** A very flexible model can drive **training** error to ~0 by memorising, yet do badly on new data — that gap is variance. A very rigid model has high error on **both** train and test — that's bias. The thing we actually care about, **expected test error**, is U-shaped in complexity, while training error just keeps falling. (See §4.5.)

---

## 4. Formal treatment

### 4.1 Setup — the data-generating model

Assume observations come from an unknown true function plus noise:
$$
\boxed{\,y = f(x) + \epsilon,\qquad E[\epsilon]=0,\quad \mathrm{Var}(\epsilon)=\sigma^2,\quad \epsilon \text{ independent of } x \text{ and of the training set } D\,}
$$
From a random training set $D$ we learn an estimate $\hat f_D$. Fix a **test point** $x_0$; write $f = f(x_0)$ (a fixed number), $\hat f = \hat f_D(x_0)$ (random, via $D$), and its mean over training sets $\bar f = E_D[\hat f]$. The new test label is $y = f+\epsilon$ with its **own fresh noise** $\epsilon$.

### 4.2 Definitions

$$
\boxed{\;\text{Bias}\big(\hat f(x_0)\big) = E_D[\hat f(x_0)] - f(x_0) = \bar f - f\;}
$$
$$
\boxed{\;\text{Var}\big(\hat f(x_0)\big) = E_D\big[(\hat f(x_0) - \bar f)^2\big]\;}
$$
$$
\boxed{\;\text{Irreducible error} = \sigma^2 = \mathrm{Var}(\epsilon)\;}
$$

Bias can be positive or negative; **only its square enters the error**. Variance and $\sigma^2$ are always $\ge 0$.

### 4.3 The decomposition (the headline result)

For **squared-error loss**, the expected test error at $x_0$ (averaged over training sets *and* test noise) is:
$$
\boxed{\;\mathrm{Err}(x_0)=E_{D,\epsilon}\big[(y-\hat f)^2\big]=\underbrace{\big(\bar f - f\big)^2}_{\text{Bias}^2}+\underbrace{E_D\big[(\hat f-\bar f)^2\big]}_{\text{Variance}}+\underbrace{\sigma^2}_{\text{Irreducible}}\;}
$$

**Derivation (GATE has asked for the vanishing cross-terms — know this).**

*Step 1 — split off the noise.* Substitute $y=f+\epsilon$:
$$
E[(f+\epsilon-\hat f)^2]=E[(f-\hat f)^2]+2\,E[\epsilon(f-\hat f)]+E[\epsilon^2].
$$

*Step 2 — the noise cross-term is 0.* Test noise $\epsilon$ is independent of $D$ and $E[\epsilon]=0$:
$$
E[\epsilon(f-\hat f)]=E[\epsilon]\,E[f-\hat f]=0.
$$

*Step 3 — the noise square.* $E[\epsilon^2]=\mathrm{Var}(\epsilon)+(E[\epsilon])^2=\sigma^2$. So far: $\mathrm{Err}(x_0)=E[(f-\hat f)^2]+\sigma^2$.

*Step 4 — split the rest with $\bar f$.* Write $f-\hat f=(f-\bar f)+(\bar f-\hat f)$:
$$
E[(f-\hat f)^2]=(f-\bar f)^2+E[(\bar f-\hat f)^2]+2(f-\bar f)\,E[\bar f-\hat f].
$$
Here $(f-\bar f)$ is a constant and $E[\bar f-\hat f]=\bar f-E[\hat f]=0$, so the cross-term vanishes, leaving $\text{Bias}^2+\text{Var}$. Combining with Step 3 gives the boxed result. $\blacksquare$

### 4.4 Reading the decomposition — *which term moves when?*

This is the key to "if we change X, which term changes?" questions:

- **Variance** depends *only* on how sensitive the **learning procedure** is to the training sample. It does **not** depend on $f$ or on $\sigma^2$.
- **Bias²** depends on the **model family's flexibility** relative to the true $f$ (the systematic gap $\bar f - f$). It does **not** depend on $\sigma^2$.
- **Irreducible $\sigma^2$** depends *only* on the noise. It is **independent of the model** — you cannot reduce it by changing the algorithm. It is a hard **floor**: $\mathrm{Err}(x_0)\ge\sigma^2$ for *every* model.
- Overall expected error usually averages the pointwise $\mathrm{Err}(x_0)$ over the input distribution. Bias/variance are defined **pointwise**, so a model can be high-bias in one region and high-variance in another.

### 4.5 Model complexity and the U-curve

| Complexity | Bias² | Variance | Training error | Expected **test** error | Regime |
|---|---|---|---|---|---|
| Too low (rigid) | **high** | low | high | high | **Underfitting** |
| Just right | moderate | moderate | moderate | **minimum** | Good fit |
| Too high (flexible) | low | **high** | ~0 | high | **Overfitting** |

- **Training error decreases monotonically** as complexity rises (a flexible model can memorise).
- **Expected test error is U-shaped**: it falls as bias drops, then rises as variance takes over. The best model sits at the **bottom of the U**, *not* at minimum training error.

### 4.6 The levers (how to push bias vs variance) — high-yield table

| Action | Bias | Variance | Examples in this syllabus |
|---|---|---|---|
| ↑ model **complexity / flexibility** | **↓** | **↑** | deeper tree, higher polynomial degree, **smaller $k$** in kNN, more features |
| ↑ **regularisation / smoothing** | **↑** | **↓** | larger $\lambda$ in **ridge**, **pruning** / shallower tree, **larger $k$** in kNN |
| ↑ **training-set size $N$** | ≈ unchanged | **↓** | more data steadies any model |
| **Average many models** (bagging) | ≈ unchanged | **↓** | (ensembles — see note below) |

- **kNN limits:** $k=1$ → lowest bias / highest variance; $k=N$ → predicts the global mean/majority → **highest bias / lowest variance**.
- **Ridge limits:** $\lambda=0$ is OLS (lower bias, higher variance); $\lambda\to\infty$ shrinks all coefficients to 0 (highest bias, lowest variance). Ridge deliberately *adds bias to cut variance* — exactly the trade-off, and why it can beat OLS in test MSE.
- **Trees:** unpruned/deep = low bias, high variance; pruned/shallow = high bias, low variance.
- *(Bagging / random forests / boosting extend this — bagging averages to cut variance, boosting reduces bias — but these are **outside the listed GATE DA syllabus**, so noted only in passing.)*

### 4.7 Conditions & caveats (honesty about the math)

- The clean **additive** identity $\text{Err}=\text{Bias}^2+\text{Var}+\sigma^2$ holds for **squared-error loss**. Under **0–1 (classification) loss** the notions of bias and variance still exist but **do not add up additively**, and higher variance does not always increase error. The classification analogue of $\sigma^2$ is the **Bayes error**. Do **not** apply the squared-error formula blindly to classification accuracy.
- The decomposition is an expectation over **the distribution of training sets** (and test noise) — it is a theoretical/conceptual quantity, not something you read off a single dataset (which is why we need cross-validation).

### 4.8 Estimating bias and variance from a finite ensemble

In practice (or in a GATE problem) you may be given the predictions $\hat f_1,\dots,\hat f_m$ at $x_0$ from $m$ models trained on $m$ different datasets, plus the true $f$ and noise $\sigma^2$:
$$
\bar f=\frac1m\sum_{j=1}^m \hat f_j,\quad
\widehat{\text{Bias}}=\bar f-f,\quad
\widehat{\text{Var}}=\frac1m\sum_{j=1}^m(\hat f_j-\bar f)^2,\quad
\widehat{\text{Err}}=\widehat{\text{Bias}}^2+\widehat{\text{Var}}+\sigma^2.
$$
**Convention trap (read §7):** the variance term in the decomposition is the *population* variance, so divide by $m$. If a question instead asks for an *unbiased sample estimate* of the variance, divide by $m-1$. State which you use.

---

## 5. Worked problems (GATE-style)

### Problem 1 (NAT) — Compute bias², variance and total error from an ensemble

At a test point $x_0$ the true value is $f(x_0)=9$ and the noise variance is $\sigma^2=4$. The same algorithm trained on five different datasets predicts
$$\hat f = 8,\ 9,\ 11,\ 12,\ 10.$$
**Find (a) bias², (b) variance, (c) expected total test error.**

**Step 1 — mean prediction.** $\bar f=\dfrac{8+9+11+12+10}{5}=\dfrac{50}{5}=10.$

**Step 2 — bias².** $\text{Bias}=\bar f-f=10-9=1\Rightarrow \text{Bias}^2=\mathbf{1}.$

**Step 3 — variance** (population form, $\div m$). Deviations from $\bar f$: $-2,-1,1,2,0$; squares $4,1,1,4,0$; sum $=10$.
$$\text{Var}=\frac{10}{5}=\mathbf{2}.$$

**Step 4 — total.** $\text{Err}=\text{Bias}^2+\text{Var}+\sigma^2=1+2+4=\mathbf{7}.$

> **If** the question had asked for the *unbiased sample variance* instead, you would use $\tfrac{10}{5-1}=2.5$, giving total $=1+2.5+4=7.5$. The plain decomposition uses $\div m=2$.

---

### Problem 2 (NAT / MCQ) — Algebra of the decomposition + the regularisation trade-off

A model's expected test MSE at a point is $0.50$. The irreducible noise variance is $\sigma^2=0.10$, and the model's squared bias is $0.25$.

**(a)** Find the variance.
**(b)** You increase regularisation; this raises bias² to $0.30$ and lowers variance to $0.05$. Does total error improve?

**(a)** $\text{Err}=\text{Bias}^2+\text{Var}+\sigma^2\Rightarrow 0.50=0.25+\text{Var}+0.10\Rightarrow \text{Var}=\mathbf{0.15}.$

**(b)** New total $=0.30+0.05+0.10=\mathbf{0.45}<0.50$. **Yes — it improves.** Variance fell by $0.10$ while bias² rose only $0.05$, a net reduction of $0.05$. This is the trade-off working in your favour: regularisation pays a small bias cost to buy a larger variance saving.

---

### Problem 3 (MSQ-style, with the "what reduces what" trap)

A regression problem is fit with polynomials of increasing degree. At the **optimal** degree, $\text{Bias}^2=0.2$, $\text{Var}=0.3$, and $\sigma^2=0.1$.

**(a)** Current expected test error?
**(b)** You **double the training-set size**, which roughly **halves the variance** and leaves bias essentially unchanged. New expected test error?
**(c)** What is the lowest test error this *same model family* could ever reach (infinite data), and the lowest *any* model could reach?

**(a)** $\text{Err}=0.2+0.3+0.1=\mathbf{0.6}.$

**(b)** Variance $\to 0.15$, bias unchanged: $\text{Err}=0.2+0.15+0.1=\mathbf{0.45}.$ More data **fought variance, not bias** — exactly as expected.

**(c)** With infinite data the variance $\to 0$, so this model family floors at $\text{Bias}^2+\sigma^2=0.2+0.1=\mathbf{0.3}$. **No** model can beat the irreducible floor $\sigma^2=\mathbf{0.1}$. (To get below $0.3$ you must reduce **bias** — use a more flexible model — not gather more data.)

**Trap caught:** "more data drives test error to 0" is **false** (floored at $\sigma^2$, and at $\text{Bias}^2+\sigma^2$ for a fixed-flexibility model); "more data reduces bias" is **false** for a fixed model — it reduces variance.

---

## 6. Common GATE question types on this topic

- **Compute bias², variance, and total error** from a set of predictions across training sets (Problem 1).
- **Solve for the missing term** in $\text{Err}=\text{Bias}^2+\text{Var}+\sigma^2$ given the other three (Problem 2).
- **Direction questions:** "As tree depth / polynomial degree increases (or $k$ in kNN / $\lambda$ in ridge changes), how do bias and variance move?" (use §4.6).
- **Identify the regime:** given train vs test error, label underfitting (high bias) vs overfitting (high variance).
- **Derivation gaps:** "Which cross-term vanishes and *why*?" → the noise term, because $\epsilon\!\perp\! D$ and $E[\epsilon]=0$ (§4.3).
- **Floor / limits:** lowest achievable error; effect of infinite data; what $\sigma^2$ represents.
- **MSQ true/false** on properties (irreducible error is model-independent; training error underestimates test error; bias² not bias enters the sum; etc.).

---

## 7. Traps, pitfalls & common mistakes

- **Underfitting is HIGH BIAS, not high variance.** A frequent slip. Overfitting = high variance (low train, high test); underfitting = high bias (high train **and** high test).
- **Bias vs Bias².** The decomposition uses **Bias²**. Bias $=\bar f-f$ may be negative; squaring is mandatory — never add a negative bias into the total.
- **$1/m$ vs $1/(m-1)$.** The variance *term* of the decomposition is the population variance ($\div m$). Only use $\div(m-1)$ if explicitly asked for an unbiased sample estimate. Pick deliberately and say so.
- **Irreducible error is not a model error.** You cannot reduce $\sigma^2$ by changing the model, adding features within the same noise, or getting more data. It is the hard lower bound $\text{Err}\ge\sigma^2$.
- **More data ≠ less bias.** More data mainly cuts **variance**. Bias is set by the model family's inability to represent $f$; to cut bias, use a more flexible model. (Caveat: *nonparametric* methods whose complexity grows with $N$ can see effective bias fall — but for a *fixed* model, data ↓ variance only.)
- **Minimum training error ≠ best model.** Training error keeps dropping with complexity; chasing it lands you deep in the overfitting (high-variance) zone. Optimise *test/validation* error.
- **Squared-loss only.** Do **not** apply $\text{Bias}^2+\text{Var}+\sigma^2$ to classification *accuracy*; the additive split is for squared error. For 0–1 loss the analogue of $\sigma^2$ is the **Bayes error**, and variance can even help.
- **"Variance" is across training sets**, not the spread of a single model's outputs over different inputs. High variance ≠ large predicted values.
- **Bias/variance are pointwise.** The tidy U-curves are *averages* over inputs; locally the picture can differ.
- **Regularisation doesn't always lower total error** — it helps only when the variance reduction it buys exceeds the bias it adds (compare Problem 2: it could equally have been a wash or worse if the bias rose more than the variance fell).

---

## 8. Connection forward

We now have the exact target: minimise **bias² + variance** (the irreducible $\sigma^2$ is fixed), which means picking the complexity at the **bottom of the U-curve**. The catch is that in real life we **cannot compute bias or variance** — we don't know the true $f$ or $\sigma^2$, and we hold just **one** training set, not the many that the expectations require. So we need a data-only way to *estimate generalisation error* and then choose the depth / $\lambda$ / $k$ that minimises it. That is exactly the job of the **next topic, Cross-Validation (Topic 12)** — LOO and $k$-fold give an estimate of expected test error, letting us empirically locate the sweet spot of the bias–variance trade-off and tune hyperparameters.

---

## 9. One-line cheat-sheet summary

**Expected test MSE $=\text{Bias}^2+\text{Variance}+\sigma^2$; simple models = high-bias/low-variance (underfit), complex models = low-bias/high-variance (overfit); test error is U-shaped in complexity and floored by the irreducible noise $\sigma^2$; regularisation, pruning, larger $k$, and more data trade variance for (a little) bias.**

---

> **Next topic (logically):** **Topic 12 — Cross-Validation Methods (LOO, $k$-fold)**, the practical procedure for *estimating* the test error from data so we can pick the model complexity that minimises the bias–variance sum we just derived.

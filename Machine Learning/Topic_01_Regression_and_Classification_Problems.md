# Topic 1 — Regression and Classification Problems

**Subject:** Machine Learning
**Section:** Supervised Learning — *opening / framing topic*
**Position in syllabus:** This is the **entry point of the entire Supervised Learning block.** It teaches no specific algorithm; it sets up the vocabulary, the data setup, and the two task types — **regression** and **classification** — that every later method is an instance of (Simple LR, Multiple LR, Ridge, Logistic Regression, k-NN, Naive Bayes, LDA, SVM, Decision Trees, MLP/FFNN). *Comes first; leads into Simple Linear Regression.*

---

## 1. Why this topic exists (motivation + connection)

In a huge number of real problems we already possess examples where the **answer is known** — past house sales with their prices, emails already marked spam or not, patients with confirmed diagnoses — and we want a rule that produces the right answer for **new, unseen** cases. Learning such an input→output rule from labelled examples is **supervised learning**.

Before we fix a framework, "data" is just a pile of numbers and labels: you can't yet say *what* you're predicting or *how good* a prediction is. This topic supplies that framework — it names the inputs (**features**), the output (**target**), the rule (**model**), and the way we score a rule (**loss**). Crucially, it splits all supervised tasks into exactly two kinds based on **what the output looks like**: a continuous number (**regression**) or a category (**classification**). That single distinction decides which algorithm, which loss, and which error metric you may use for the rest of the section. Every later topic plugs into this skeleton.

---

## 2. Prerequisites

- **Vectors & matrices** — a dataset is stored as a feature matrix (rows = samples, columns = features) and a target vector.
- **Functions / mappings** — a model is just a function $f$ from inputs to outputs.
- **Basic probability** — random variables, joint & conditional distributions, expectation (needed to state the *optimal* predictor).
- **Elementary calculus** — minimisation of a loss (used heavily from Simple LR onward).

*(All drawn from the broader GATE DA Linear Algebra / Probability / Calculus sections.)*

---

## 3. Core idea — intuition first

Picture a student handed a stack of **solved** problems (question + correct answer). By studying enough solved examples, the student learns to answer a *fresh* question. Supervised learning is exactly this: the solved examples are the **training data**, each "question" is a feature vector $x$, each "answer" is the target $y$, and the learned skill is the model $f$.

Two flavours of "answer":

- If the answer is a **number on a sliding scale** — price ₹, temperature °C, height cm — the task is **regression**. You care *how far off* you are.
- If the answer is a **choice from a fixed menu** — spam/not-spam, cat/dog/horse, blood group A/B/AB/O — the task is **classification**. You care *whether you picked the right box*.

One line to carry forever: **the task type is decided by the OUTPUT, never by the input.** A regression problem can have categorical inputs; a classification problem can have purely numeric inputs.

---

## 4. Formal treatment

### Setup (common to both task types)

Training data:

$$D = \{(x_1,y_1),(x_2,y_2),\dots,(x_n,y_n)\}, \qquad x_i \in \mathbb{R}^p,\ \ y_i \in \mathcal{Y}$$

- $n$ = number of samples, $p$ = number of features.
- We assume each pair is drawn i.i.d. from an **unknown** joint distribution $P(X,Y)$.
- **Goal:** learn a model $f:\mathbb{R}^p \to \mathcal{Y}$ so that $f(x)\approx y$ on *new* data from the same $P$.

The two tasks differ **only in the output space $\mathcal{Y}$**:

| Task | Output space $\mathcal{Y}$ | Example target | Typical loss |
|---|---|---|---|
| **Regression** | continuous, $\mathcal{Y}\subseteq\mathbb{R}$ | price, temperature | squared error |
| **Classification** | discrete, $\mathcal{Y}=\{1,\dots,K\}$ | spam/ham, digit 0–9 | 0–1 (misclassification) |

### Loss functions (how we score a single prediction)

> **Regression — squared error:** $\;L(y,\hat y) = (y-\hat y)^2\;$ (also used: absolute error $|y-\hat y|$).
>
> **Classification — 0–1 loss:** $\;L(y,\hat y) = \mathbb{1}(y \neq \hat y)\;$ (1 if wrong, 0 if right).
> **Condition:** 0–1 loss treats every mistake equally and ignores *how close* or *how confident* the prediction was.

### Risk and empirical risk

True objective — **expected risk** (cannot be computed, since $P$ is unknown):

$$R(f) = \mathbb{E}_{(X,Y)\sim P}\big[\,L(Y, f(X))\,\big]$$

What we actually minimise — **empirical risk** on the training set (**ERM**):

$$\hat{R}(f) = \frac{1}{n}\sum_{i=1}^{n} L\big(y_i, f(x_i)\big)$$

**Caveat:** minimising $\hat R$ is only a *proxy* for minimising $R$. Driving $\hat R$ to zero can make $R$ worse — this gap is **overfitting**, formalised later in **Bias–Variance** and controlled by **cross-validation** and **ridge**.

### What is the *best possible* predictor? (the target every method approximates)

> **Regression (squared-error optimal):** the minimiser of $\mathbb{E}[(Y-f(X))^2]$ is the **regression function**
> $$f^*(x) = \mathbb{E}[\,Y \mid X = x\,]$$
> (For *absolute-error* loss the optimum is the conditional **median**.)

> **Classification (0–1 optimal):** the minimiser of misclassification probability is the **Bayes classifier**
> $$f^*(x) = \arg\max_{k}\ P(Y = k \mid X = x)$$
> Its error rate — the **Bayes error** — is the lowest any classifier can achieve.

*Why the regression result holds (short, GATE asks this):* condition on $X$; for each fixed $x$ minimise $\mathbb{E}[(Y-c)^2\mid X=x]$ over the constant $c$. Expanding, the minimiser is $c=\mathbb{E}[Y\mid X=x]$ (the mean minimises mean-squared deviation). Averaging over $X$ preserves optimality. ∎

### Two organising axes for the methods ahead (framing only)

- **Parametric vs non-parametric** — parametric models fit a *fixed* number of parameters (Linear/Logistic Regression, LDA); non-parametric models keep the data around and grow with it (k-NN).
- **Discriminative vs generative** — discriminative models learn the boundary / $P(Y\mid X)$ directly (Logistic Regression, SVM); generative models learn $P(X\mid Y)$ and $P(Y)$, then apply Bayes' rule (Naive Bayes, LDA).

Hold this 2×2 in mind — it explains *why* each upcoming algorithm exists.

---

## 5. Worked problems (GATE-style)

### Problem 1 (MSQ)
Which of the following are **classification** problems?
(A) Predicting tomorrow's maximum temperature in °C
(B) Deciding whether an incoming email is spam
(C) Predicting the number of vehicles crossing a toll in one hour
(D) Predicting a blood sample's group as A, B, AB, or O

**Solution.** Task type = output type.
- (A) temperature is a continuous number → **regression**.
- (B) two categories (spam/ham) → **classification** (binary). ✔
- (C) a count is still a numeric quantity on a scale; we predict "how many" → **regression** (count regression). *Integer ≠ classification.*
- (D) one of four labels → **classification** (multi-class). ✔

**Answer: (B) and (D).**

### Problem 2 (MCQ)
Under squared-error loss, the function $f$ that minimises $\mathbb{E}[(Y-f(X))^2]$ is:
(A) the conditional **median** of $Y$ given $X$
(B) $f(x)=\mathbb{E}[Y\mid X=x]$
(C) the conditional **mode** of $Y$ given $X$
(D) $\arg\max_k P(Y=k\mid X=x)$

**Solution.** Condition on $X$: $\mathbb{E}[(Y-f(X))^2]=\mathbb{E}_X\big[\mathbb{E}[(Y-f(X))^2\mid X]\big]$. For each fixed $x$, the constant $c=f(x)$ minimising $\mathbb{E}[(Y-c)^2\mid X=x]$ is the **mean**, $c=\mathbb{E}[Y\mid X=x]$. (Median minimises *absolute* error; mode / argmax-posterior belong to classification.)

**Answer: (B).**

### Problem 3 (MCQ — trap)
A bank's first model outputs a customer's **credit score**, an integer from 300 to 850. A second model outputs a decision: **approve** or **reject** a loan. Choose the correct statement.
(A) Both are classification (outputs are discrete).
(B) Both are regression (outputs are numeric).
(C) The first is regression, the second is classification.
(D) The first is classification, the second is regression.

**Solution.** The credit score is measured on a numeric scale where *nearness matters* (predicting 800 vs 805 is "almost right") → **regression**, even though values are integers. Approve/reject is a two-option label with no notion of distance → **classification**. The trap is treating "integer-valued" as "categorical".

**Answer: (C).**

---

## 6. Common GATE question types on this topic

- **Classify the task** — given several scenarios, mark which are regression vs classification (MCQ/MSQ). Decided purely by the output.
- **Match the loss/metric** — pick the right loss for a regression vs classification task (squared error vs 0–1 / accuracy).
- **Identify the optimal predictor** — "what minimises $\mathbb{E}[(Y-f(X))^2]$?" → conditional **mean**; absolute error → conditional **median**; 0–1 loss → **Bayes classifier**.
- **Supervised vs unsupervised vs output type** — separate supervised tasks (labels present) from clustering/PCA (no labels), and within supervised, identify the output type.
- **Spot the misnamed method** — recognise that *logistic regression* is a **classification** method.

---

## 7. Traps, pitfalls & common mistakes

- **"Discrete output ⇒ classification."** False. Counts / integer quantities (goals scored, page views) are **regression** — distance between values is meaningful.
- **Letting the *input* decide the task.** The task is defined by the **output**. Categorical features (city, gender) are perfectly fine inside a regression problem.
- **Logistic Regression is *not* regression.** Despite the name, its output is a class label (via a probability) → **classification**. Classic GATE bait.
- **Binary labels coded 0/1 look numeric.** Encoding classes as 0/1 doesn't make it regression — the values are just names.
- **Confusing the optimal predictors.** Mean ↔ squared error; median ↔ absolute error; mode / argmax-posterior ↔ 0–1 loss. Mixing these up is a frequent slip.
- **Ordinal targets (e.g., rating 1–5).** Genuinely sits between the two; GATE usually frames it as classification (or ordinal). Read the question — don't assume.
- **Thinking ERM = generalisation.** Low *training* error is not low *true* error; that distinction is the whole point of bias–variance and cross-validation later.

---

## 8. Connection forward

With the supervised skeleton in place (features → model → target, scored by a loss), we need the **simplest concrete model** to hang on it. That is **Simple Linear Regression**: one feature, one straight-line fit, trained by minimising squared error. It makes the abstract "learn $f$, minimise loss" idea fully explicit and introduces the **least-squares** machinery that Multiple LR and Ridge will generalise.

---

## 9. One-line cheat-sheet summary

**Supervised learning learns input→output from labelled data; the OUTPUT decides the task — continuous → regression (squared-error-optimal $=\mathbb{E}[Y\mid X]$), categorical → classification (0–1-optimal = Bayes classifier); and logistic regression is classification.**

---

*Next topic (in order):* **Simple Linear Regression.**

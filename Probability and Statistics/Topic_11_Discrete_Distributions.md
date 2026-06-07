# Topic 11 — Discrete Distributions (Uniform, Bernoulli, Binomial & Poisson)

> **File this under:** Probability & Statistics → *Probability Distributions* (Section start).

---

## 0. Topic header & syllabus map

- **Topic:** Discrete probability distributions — the four named ones GATE DA tests: **Discrete Uniform, Bernoulli, Binomial, Poisson.**
- **Subject:** Probability and Statistics (GATE Data Science & AI).
- **Where it sits:** This is the **first topic of the "Probability Distributions" section**. It comes right after the random-variables block (Topic 6 PMF, Topic 7 PDF/CDF, Topic 9 expectation & variance, Topic 10 summary statistics). It leads directly into **Topic 12 — Continuous Distributions**, which are the continuous mirror images of these.

---

## 1. Why this topic exists (motivation + connection)

So far you learned the *machinery*: a discrete random variable is a function on a sample space, a **PMF** assigns probabilities to its values (Topic 6), and **expectation/variance** are computed by summing $\sum x\,p(x)$ and $\sum (x-\mu)^2 p(x)$ (Topic 9). That machinery is general but **slow** — for a coin tossed 50 times you would have to write out $\binom{50}{k}$ for every $k$ and sum by hand.

The fix: most real randomness is **not arbitrary**. The same handful of patterns recur everywhere — "equally likely outcomes," "one success/failure trial," "count successes in $n$ trials," "count rare events in an interval." Statisticians catalogued these recurring patterns as **named distributions** with **ready-made closed-form** PMF, mean, and variance. You stop deriving from scratch and instead *recognize the pattern → quote the formula*.

- **What the previous topic couldn't do:** Topic 6/9 told you *how* to build a PMF and compute moments, but gave no shortcut. Summing $\sum k\binom{n}{k}p^kq^{n-k}$ is painful; the binomial result $E[X]=np$ replaces it with a single product. These named families *are* those shortcuts.
- **What the next topic builds on it:** Topic 12 (Continuous Distributions) is the continuous analogue — discrete uniform → continuous uniform, the count-of-rare-events idea (Poisson) → waiting-time (exponential), and binomial → **normal** through the Central Limit Theorem (Topic 14). Poisson and binomial are also the raw material for **hypothesis testing** (Topic 16).

**The four form one chain — read this once and the whole topic clicks:**

> **Discrete Uniform** = "all outcomes equally likely" (no notion of success). →
> **Bernoulli** = introduce *success/failure*, just **one** trial (the atom). →
> **Binomial** = repeat that Bernoulli trial **$n$ independent times and count successes** (Binomial = sum of $n$ Bernoullis; Bernoulli = Binomial with $n=1$). →
> **Poisson** = take the Binomial limit when trials become **infinite and each success rare** ($n\to\infty,\ p\to 0,\ np=\lambda$). It counts rare events over a continuum (time/space) where $n$ is not even fixed.

---

## 2. Prerequisites

- **Counting** (Topic 1): you need $\binom{n}{k}=\dfrac{n!}{k!(n-k)!}$ cold for the binomial PMF.
- **Probability axioms & independence** (Topics 2–3): "independent trials" and "$\sum p = 1$" underlie every formula.
- **Discrete RV & PMF** (Topic 6): a distribution *is* a PMF with a name.
- **Expectation & variance** (Topic 9): $E[X]$, $\mathrm{Var}(X)=E[X^2]-(E[X])^2$, and **linearity of expectation**; for independent variables, $\mathrm{Var}(X+Y)=\mathrm{Var}(X)+\mathrm{Var}(Y)$.
- **Series facts:** the exponential series $\displaystyle\sum_{k=0}^{\infty}\frac{\lambda^k}{k!}=e^{\lambda}$ (needed to verify the Poisson sums to 1 and to find its mean).

---

## 3. Core idea — intuition first

Think of choosing the right distribution as **answering four questions about the experiment**:

1. **Are all outcomes equally likely with no "success" concept?** (rolling a fair die, picking a random slot) → **Discrete Uniform**. Every value has the same probability $1/n$. Nothing fancy.

2. **Is it a single yes/no trial?** (one coin flip; one item defective or not) → **Bernoulli**. Success with prob $p$, failure with prob $q=1-p$. This is the building block; on its own it's almost too simple, but it's the *atom* everything else is made of.

3. **Do I repeat that yes/no trial a fixed number $n$ of times and count how many successes?** (50 coin flips → number of heads; 20 items → number defective) → **Binomial**. The PMF $\binom{n}{k}p^kq^{n-k}$ reads literally: $\binom{n}{k}$ ways to choose *which* $k$ trials succeed, $p^k$ for those successes, $q^{n-k}$ for the failures.

4. **Am I counting how many times something happens in a fixed window (an hour, a page, a square metre), where the events are rare and individually unpredictable but have a steady average rate $\lambda$?** (calls per hour, typos per page, accidents per month) → **Poisson**. Here there is no fixed "$n$" — you don't have a fixed number of trials, you have a continuum. Poisson is what binomial *becomes* when there are zillions of tiny opportunities each with a tiny success chance.

**Analogy for the binomial→Poisson limit:** Imagine a long highway. Split it into $n$ tiny segments; in each, an accident either happens (prob $p$, tiny) or not — that's binomial with huge $n$, tiny $p$. As you slice finer ($n\to\infty,\ p\to0$) but keep the *expected number* $np=\lambda$ fixed, the binomial count smooths into the **Poisson** with mean $\lambda$. That's *why* Poisson is the law of rare events.

**Signature you can use to identify them on sight:**
- Binomial: **variance $<$ mean** (because $\mathrm{Var}=npq=\text{mean}\times q$ and $q\le1$).
- Poisson: **variance $=$ mean** ($=\lambda$). This single fact is a frequent GATE giveaway.

---

## 4. Formal treatment

Notation throughout: $q := 1-p$, support = the set of values the RV can take.

### 4A. Discrete Uniform distribution

A random variable equally likely to take any one of $n$ specified values.

$$\boxed{\,P(X=x_i)=\frac{1}{n}\quad\text{for each of the } n \text{ values } x_1,\dots,x_n\,}$$

**Conditions/assumptions:** finitely many outcomes; *all equally likely*. (If they aren't equally likely, it is **not** uniform.)

For the standard case $X\in\{1,2,\dots,n\}$:

| Quantity | Formula | Condition |
|---|---|---|
| PMF | $P(X=k)=\tfrac1n,\ k=1,\dots,n$ | $n$ equally likely values |
| Mean | $\mu=\dfrac{n+1}{2}$ | support $\{1,\dots,n\}$ |
| Variance | $\sigma^2=\dfrac{n^2-1}{12}$ | support $\{1,\dots,n\}$ |

For a *shifted* support $\{a, a+1,\dots,b\}$ with $n=b-a+1$ equally-spaced integer values:
$$\mu=\frac{a+b}{2},\qquad \sigma^2=\frac{(b-a+1)^2-1}{12}=\frac{n^2-1}{12}.$$
(The variance depends only on *how many* values, not on where they start — shifting doesn't change spread.)

**Short derivation of the variance** (GATE has asked this for a fair die):
$$E[X]=\frac1n\sum_{k=1}^n k=\frac{n+1}{2},\qquad E[X^2]=\frac1n\sum_{k=1}^n k^2=\frac{(n+1)(2n+1)}{6}.$$
$$\mathrm{Var}=E[X^2]-(E[X])^2=\frac{(n+1)(2n+1)}{6}-\frac{(n+1)^2}{4}=\frac{(n+1)(n-1)}{12}=\frac{n^2-1}{12}.$$

**CDF:** a staircase; $F(x)=\dfrac{\lfloor x\rfloor}{n}$ on $\{1,\dots,n\}$ (jumps of $1/n$).

---

### 4B. Bernoulli distribution

One trial, two outcomes: success ($X=1$) or failure ($X=0$).

$$\boxed{\,P(X=1)=p,\quad P(X=0)=1-p=q\,}\qquad\Longleftrightarrow\qquad P(X=k)=p^k q^{1-k},\ k\in\{0,1\}$$

**Conditions:** exactly **one** trial; exactly **two** outcomes; $0\le p\le 1$.

| Quantity | Formula |
|---|---|
| Mean | $E[X]=p$ |
| Variance | $\mathrm{Var}(X)=pq=p(1-p)$ |
| MGF (optional) | $M_X(t)=q+pe^{t}$ |

**Derivation (trivial but shows the pattern):** $E[X]=1\cdot p+0\cdot q=p$; $E[X^2]=1^2\cdot p+0^2\cdot q=p$, so $\mathrm{Var}=p-p^2=pq$.

**Why it matters:** A Bernoulli RV is also called an **indicator variable** $\mathbf{1}_A$ — it is $1$ if event $A$ happens, else $0$, with $p=P(A)$. Writing complicated counts as **sums of indicators** is the single most powerful trick for finding expectations (it's how we get the binomial mean below).

---

### 4C. Binomial distribution $\;X\sim \mathrm{Bin}(n,p)$

Number of successes in $n$ independent Bernoulli($p$) trials.

$$\boxed{\,P(X=k)=\binom{n}{k}p^{k}q^{\,n-k},\qquad k=0,1,2,\dots,n\,}$$

**Conditions (memorize — GATE tests violations):**
1. **Fixed** number of trials $n$ (decided in advance).
2. Each trial **independent** of the others.
3. **Same** success probability $p$ on every trial.
4. Each trial has exactly **two** outcomes (success/failure).

> ⚠️ If you sample **without replacement** from a *small* population, the trials are **not independent** and $p$ changes each draw → the correct model is **hypergeometric**, not binomial. (Hypergeometric is outside the DA syllabus, but recognizing "without replacement ⇒ not binomial" is a fair-game trap.) With replacement, or from a very large population, binomial is fine.

| Quantity | Formula | Condition |
|---|---|---|
| Mean | $E[X]=np$ | the 4 conditions above |
| Variance | $\mathrm{Var}(X)=npq=np(1-p)$ | trials independent |
| Mode (most likely $k$) | $\lfloor (n+1)p\rfloor$; if $(n+1)p\in\mathbb{Z}$, **two** modes $(n+1)p$ and $(n+1)p-1$ | — |
| MGF (optional) | $M_X(t)=(q+pe^{t})^{n}$ | — |

**Derivation of mean & variance — the indicator trick (this is the GATE-expected derivation):**
Write $X=X_1+X_2+\cdots+X_n$, where $X_i\sim$ Bernoulli($p$) is the indicator that trial $i$ succeeds.
$$E[X]=\sum_{i=1}^n E[X_i]=\sum_{i=1}^n p = np\qquad(\text{linearity — no independence needed}).$$
Because the trials are **independent**, variances add:
$$\mathrm{Var}(X)=\sum_{i=1}^n \mathrm{Var}(X_i)=\sum_{i=1}^n pq = npq.$$

**Useful facts:**
- **Complement:** if $X\sim\mathrm{Bin}(n,p)$ then $n-X\sim\mathrm{Bin}(n,q)$ (count failures instead).
- **"At least one"** is almost always done by complement: $P(X\ge1)=1-P(X=0)=1-q^{\,n}$.
- **CDF** has no closed form: $F(k)=\sum_{i=0}^{k}\binom{n}{i}p^iq^{\,n-i}$ — you sum terms.
- **Variance $\le$ mean always** ($npq\le np$ since $q\le1$); a binomial can never be *over-dispersed*.

---

### 4D. Poisson distribution $\;X\sim \mathrm{Poi}(\lambda)$

Number of events in a fixed interval, given a constant average rate $\lambda$ of (rare, independent) events.

$$\boxed{\,P(X=k)=\frac{e^{-\lambda}\,\lambda^{k}}{k!},\qquad k=0,1,2,3,\dots\,}\qquad(\lambda>0)$$

**Conditions/assumptions (the "Poisson process" assumptions):**
1. Events occur **independently**.
2. **Constant average rate** $\lambda$ over the chosen interval (homogeneous).
3. Events do **not occur simultaneously** (two can't happen at the exact same instant).
4. The count is over a **fixed** interval/region; $\lambda$ = expected count *for that interval*.

> ⚠️ $\lambda$ scales with the size of the interval. If the rate is $r$ per unit and the interval has length $t$, then $\lambda=rt$. **Forgetting to rescale $\lambda$ for a different interval length is the #1 Poisson mistake** (see worked problem 4).

| Quantity | Formula | Condition |
|---|---|---|
| Mean | $E[X]=\lambda$ | — |
| Variance | $\mathrm{Var}(X)=\lambda$ | **mean = variance** (signature!) |
| Mode | $\lfloor \lambda\rfloor$; if $\lambda\in\mathbb{Z}$, **two** modes $\lambda$ and $\lambda-1$ | — |
| MGF (optional) | $M_X(t)=e^{\lambda(e^{t}-1)}$ | — |

**Check it sums to 1:** $\displaystyle\sum_{k=0}^{\infty}\frac{e^{-\lambda}\lambda^k}{k!}=e^{-\lambda}\sum_{k=0}^{\infty}\frac{\lambda^k}{k!}=e^{-\lambda}e^{\lambda}=1.$ ✓

**Derivation of mean (series manipulation — GATE-expected):**
$$E[X]=\sum_{k=0}^{\infty}k\,\frac{e^{-\lambda}\lambda^k}{k!}=e^{-\lambda}\lambda\sum_{k=1}^{\infty}\frac{\lambda^{k-1}}{(k-1)!}=e^{-\lambda}\lambda\,e^{\lambda}=\lambda.$$
For the variance use $E[X(X-1)]=\displaystyle\sum_{k=2}^{\infty}k(k-1)\frac{e^{-\lambda}\lambda^k}{k!}=e^{-\lambda}\lambda^2\,e^{\lambda}=\lambda^2$, so
$$E[X^2]=E[X(X-1)]+E[X]=\lambda^2+\lambda\ \Rightarrow\ \mathrm{Var}=\lambda^2+\lambda-\lambda^2=\lambda.$$

**Poisson as the limit of Binomial (why & when to use the approximation):**
If $n$ is large, $p$ is small, and $np=\lambda$ is moderate, then
$$\binom{n}{k}p^k(1-p)^{n-k}\ \xrightarrow[\;n\to\infty,\ p\to0,\ np=\lambda\;]{}\ \frac{e^{-\lambda}\lambda^k}{k!}.$$
**Rule of thumb for GATE:** use the Poisson approximation to the binomial when $n\ge20$ and $p\le0.05$ (roughly $n\ge100$, $np\le10$ is "very safe"). It turns a nasty $\binom{1000}{3}$ computation into a one-line Poisson evaluation (worked problem 5).

**Handy ratio (for finding the most probable value / chained probabilities):**
$$\frac{P(X=k)}{P(X=k-1)}=\frac{\lambda}{k}.$$

---

### 4E. Reproductive (additivity) properties — *frequently tested*

| Statement | Holds when |
|---|---|
| $\mathrm{Bin}(n_1,p)+\mathrm{Bin}(n_2,p)=\mathrm{Bin}(n_1+n_2,\,p)$ | **same** $p$, **independent** |
| $\mathrm{Poi}(\lambda_1)+\mathrm{Poi}(\lambda_2)=\mathrm{Poi}(\lambda_1+\lambda_2)$ | **independent** |

> ⚠️ Binomials with **different** $p$ do **not** add to a binomial. Two independent Poissons *always* add to a Poisson (no extra condition besides independence). These two facts are a classic MSQ.

---

### 4F. One-glance comparison table (revision gold)

| | Discrete Uniform $\{1..n\}$ | Bernoulli($p$) | Binomial($n,p$) | Poisson($\lambda$) |
|---|---|---|---|---|
| **Models** | equally likely outcome | one yes/no trial | #successes in $n$ trials | #rare events in an interval |
| **Support** | $1,\dots,n$ | $0,1$ | $0,1,\dots,n$ | $0,1,2,\dots$ (infinite) |
| **PMF** | $\frac1n$ | $p^kq^{1-k}$ | $\binom{n}{k}p^kq^{n-k}$ | $\frac{e^{-\lambda}\lambda^k}{k!}$ |
| **Mean** | $\frac{n+1}{2}$ | $p$ | $np$ | $\lambda$ |
| **Variance** | $\frac{n^2-1}{12}$ | $pq$ | $npq$ | $\lambda$ |
| **Mean vs Var** | — | Var $\le$ Mean | **Var $<$ Mean** | **Var $=$ Mean** |
| **Special case of** | — | Bin($1,p$) | sum of $n$ Bernoullis | limit of Bin, $n\to\infty,\,p\to0$ |

---

## 5. Worked problems (GATE-style)

### Problem 1 — Discrete Uniform (NAT)
A fair tetrahedral die shows faces numbered $1,2,3,4$. It is rolled once. Find the **variance** of the number shown (round to 2 decimals).

**Solution.** Equally likely values $\{1,2,3,4\}\Rightarrow n=4$.
$$\sigma^2=\frac{n^2-1}{12}=\frac{16-1}{12}=\frac{15}{12}=1.25.$$
(Sanity: mean $=\frac{n+1}{2}=2.5$.) **Answer: $1.25$.**

---

### Problem 2 — Binomial, *reverse* problem (NAT trap)
The number of defectives in batches follows a binomial distribution with **mean $3$** and **variance $2.1$**. Find $n$, and then $P(X=0)$.

**Solution.** From $np=3$ and $npq=2.1$:
$$q=\frac{npq}{np}=\frac{2.1}{3}=0.7\ \Rightarrow\ p=0.3,\qquad n=\frac{3}{p}=\frac{3}{0.3}=10.$$
$$P(X=0)=q^{\,n}=(0.7)^{10}\approx 0.0282.$$
**Answer: $n=10$, $P(X=0)\approx0.0282$.**
*Trap:* students compute $q=\text{Var}/\text{Mean}$ but then forget $p=1-q$, or solve $n$ before $p$. Always extract $q$ first.

---

### Problem 3 — Binomial, standard (MCQ)
A test has $10$ MCQs, each with $4$ options and exactly one correct. A student guesses every answer. The probability of getting **exactly $3$** correct is closest to:
(A) 0.12 (B) 0.25 (C) 0.30 (D) 0.38

**Solution.** Guessing ⇒ $p=\tfrac14=0.25$, $n=10$, $k=3$.
$$P(X=3)=\binom{10}{3}(0.25)^3(0.75)^7=120\times0.015625\times0.13348\approx 0.2503.$$
**Answer: (B) $\approx0.25$.**

---

### Problem 4 — Poisson, interval scaling (NAT trap)
A call centre receives on average **$6$ calls per hour**. Assuming a Poisson process, find the probability of exactly **$2$ calls in a $20$-minute** period (2 decimals).

**Solution.** Rescale $\lambda$ to the interval: $20$ min $=\tfrac13$ hour, so
$$\lambda=6\times\tfrac13=2.$$
$$P(X=2)=\frac{e^{-2}2^2}{2!}=2e^{-2}=2(0.13534)=0.2707\approx 0.27.$$
**Answer: $\approx0.27$.**
*Trap:* plugging $\lambda=6$ (the hourly rate) instead of $\lambda=2$ (the 20-minute rate). The rate must match the interval.

---

### Problem 5 — Poisson approximation to Binomial (NAT, bridge)
In a production line $0.1\%$ of items are defective. In a random sample of **$2000$** items, find the approximate probability of exactly **$3$** defectives.

**Solution.** $n=2000$ large, $p=0.001$ small ⇒ use Poisson with
$$\lambda=np=2000\times0.001=2.$$
$$P(X=3)\approx\frac{e^{-2}2^3}{3!}=\frac{8e^{-2}}{6}=1.3333\times0.13534\approx 0.1804.$$
**Answer: $\approx0.18$.** (Computing the exact $\binom{2000}{3}(0.001)^3(0.999)^{1997}$ gives essentially the same value — that's the point of the approximation.)

---

### Problem 6 — Poisson, find $\lambda$ from a condition (NAT trap)
For a Poisson random variable, $P(X=1)=P(X=2)$. Find $\lambda$ and hence $P(X=0)$.

**Solution.** Set the two PMFs equal:
$$e^{-\lambda}\lambda=\frac{e^{-\lambda}\lambda^2}{2}\ \Rightarrow\ 1=\frac{\lambda}{2}\ \Rightarrow\ \lambda=2.$$
$$P(X=0)=e^{-\lambda}=e^{-2}\approx 0.1353.$$
**Answer: $\lambda=2$, $P(X=0)\approx0.1353$.** (Tip: the ratio $\tfrac{P(X=2)}{P(X=1)}=\tfrac{\lambda}{2}=1$ gives $\lambda$ instantly.)

---

### Problem 7 — Conceptual relationships (MSQ)
Which of the following statements are **correct**?
(A) A Bernoulli RV is a binomial RV with $n=1$.
(B) For a Poisson RV, the mean equals the variance.
(C) The sum of two independent binomials $\mathrm{Bin}(n_1,p_1)$ and $\mathrm{Bin}(n_2,p_2)$ with $p_1\ne p_2$ is binomial.
(D) For a binomial RV, the variance can exceed the mean.

**Solution.**
- (A) **True** — $\mathrm{Bin}(1,p)$ has PMF $p^kq^{1-k}$, exactly Bernoulli.
- (B) **True** — both equal $\lambda$ (the Poisson signature).
- (C) **False** — additivity needs the **same** $p$; different $p$ ⇒ not binomial.
- (D) **False** — $\mathrm{Var}=npq=np\cdot q\le np=$ mean since $q\le1$; binomial variance never exceeds its mean.

**Answer: (A) and (B).**

---

## 6. Common GATE question types on this topic

1. **Plug-and-compute a PMF value** — "exactly $k$," "at most $k$," "at least one" (use complement $1-q^n$ or $1-e^{-\lambda}$).
2. **Mean/variance directly** — quote $np$, $npq$, $\lambda$, $\frac{n+1}{2}$, $\frac{n^2-1}{12}$.
3. **Reverse problems** — given mean & variance of a binomial, recover $n,p$ (Problem 2); given a PMF condition, recover $\lambda$ (Problem 6).
4. **Interval rescaling for Poisson** — change the time/space window, rescale $\lambda=rt$ (Problem 4).
5. **Poisson-approximation-to-binomial** — spot large $n$, small $p$, switch to $\lambda=np$ (Problem 5).
6. **Identify the right distribution / spot the violated assumption** — "with replacement?", "constant rate?", "independent?" (binomial vs hypergeometric; valid Poisson process or not).
7. **Reproductive/additivity** and **mean–vs–variance signature** MSQs (Problem 7).
8. **Mode / most-probable value** via $\lfloor(n+1)p\rfloor$ or $\lfloor\lambda\rfloor$, or the ratio $\frac{P(X=k)}{P(X=k-1)}$.

---

## 7. Traps, pitfalls & common mistakes

- **Poisson interval not rescaled.** Rate is *per unit*; for an interval of length $t$, use $\lambda=rt$. (Most common Poisson error.)
- **Reverse-binomial order.** Compute $q=\text{Var}/\text{Mean}$ **first**, then $p=1-q$, then $n=\text{Mean}/p$. Don't grab $n$ prematurely.
- **"At least one" by brute force.** Don't sum many terms — use $1-P(X=0)$.
- **Without-replacement = not binomial.** Dependent trials with changing $p$ ⇒ hypergeometric. Binomial needs independence + constant $p$ (with replacement, or a huge population).
- **Confusing $n$ and $k$, or $p$ and $q$.** In $\binom{n}{k}p^kq^{n-k}$: $p$ pairs with successes $k$, $q$ with failures $n-k$. A swapped exponent is an instant wrong answer.
- **Adding binomials with different $p$** (or forgetting independence) — additivity fails.
- **Treating discrete uniform variance as depending on the starting value.** It depends only on the *count* of values: $\frac{n^2-1}{12}$.
- **Forgetting Poisson support is infinite** — $X$ can be any non-negative integer; there is no upper cap like the binomial's $n$.
- **Mean vs variance mix-up.** Binomial: Var $=npq<$ mean. Poisson: Var $=$ mean $=\lambda$. If a problem says "mean = variance," it's hinting **Poisson**.
- **Rounding too early** in NAT answers — keep $e^{-\lambda}$ symbolic until the last step ($e^{-2}=0.13534$, $e^{-1}=0.36788$, $e^{-3}=0.049787$ are worth memorizing).
- **Mode with integer parameter has *two* modes** — $(n+1)p$ & $(n+1)p-1$ for binomial; $\lambda$ & $\lambda-1$ for Poisson.

---

## 8. Connection forward

These discrete models are the launchpad for the rest of the syllabus:

- **Topic 12 (Continuous Distributions):** continuous uniform mirrors the discrete uniform; the **Poisson process** (counts) and the **exponential distribution** (waiting time between those events) are two sides of the same coin; and the **normal** distribution appears as the large-$n$ shape of the binomial.
- **Topic 14 (Central Limit Theorem):** a binomial *is* a sum of $n$ i.i.d. Bernoullis, so for large $n$ it becomes approximately **Normal$(np,\,npq)$** (De Moivre–Laplace); a Poisson with large $\lambda$ likewise $\approx$ Normal$(\lambda,\lambda)$. (The "continuity correction" you'll meet there exists precisely because we're approximating a *discrete* count by a continuous curve.)
- **Topic 16 (Hypothesis testing):** counts and proportions modeled here feed directly into the **$z$-test for proportions** and **chi-squared** count tests.

---

## 9. One-line cheat-sheet summary

**Uniform $\{1..n\}$:** $P=\tfrac1n$, mean $\tfrac{n+1}{2}$, var $\tfrac{n^2-1}{12}$. **Bernoulli:** mean $p$, var $pq$. **Binomial $\binom{n}{k}p^kq^{n-k}$:** mean $np$, var $npq$ (**var $<$ mean**); sum of $n$ Bernoullis. **Poisson $\tfrac{e^{-\lambda}\lambda^k}{k!}$:** mean $=$ var $=\lambda$ (rescale $\lambda=rt$); it's the rare-event limit of the binomial ($n\to\infty,p\to0,np=\lambda$).

---

### ➡️ Next topic (in order)
**Topic 12 — Continuous Distributions (Uniform, Exponential, Normal & Standard Normal).** Ask for it when ready; we'll connect the Poisson process to the exponential waiting time and the binomial to the normal curve.

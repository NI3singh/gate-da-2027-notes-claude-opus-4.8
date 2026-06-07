# Topic 8 — Approximate Inference: Sampling

*Subject: Artificial Intelligence · GATE DA · Section: Reasoning Under Uncertainty (final topic)*

---

## 0. Topic header & syllabus map

- **Topic:** Approximate inference in Bayesian networks by **sampling** — the family of Monte Carlo methods: **prior (direct/ancestral) sampling, rejection sampling, likelihood weighting,** and **MCMC / Gibbs sampling.**
- **Subject:** Artificial Intelligence.
- **Where it sits:** The **last topic of "Reasoning Under Uncertainty."** It comes **right after Variable Elimination (Topic 7)**, which answered queries *exactly* but at a cost exponential in treewidth. This topic answers the **same** queries *approximately* but with cost we control — the tool you reach for when exact inference is infeasible. With this, the uncertainty toolkit is complete: **represent** (Topic 6, Bayesian networks) → **infer exactly** (Topic 7, VE) → **infer approximately** (Topic 8, sampling).

---

## 1. Why this topic exists (motivation + connection)

**The problem it solves.** We still want posteriors $P(Q\mid e)$, but on **large, densely connected networks** Variable Elimination's largest factor explodes (cost $\Theta(n\,2^w)$, exponential in treewidth $w$). For such networks exact inference is simply not affordable.

**What the previous topic couldn't do.** VE is the gold standard *when you can run it*. But there is no escaping its exponential dependence on treewidth — on a high-treewidth network even the best elimination order produces unmanageable factors. We need a method whose **per-step cost does not blow up with connectivity.**

**The key idea.** Instead of computing the answer, **simulate the network**: draw many random "possible worlds" consistent with the model (and the evidence), then **estimate** the probability of the query by how often it occurs among the samples. Each sample costs only **linear** time (one pass over the variables), and accuracy improves as you draw more samples — the classic Monte Carlo trade: exactness for **scalability and tunable cost.**

**What it connects back to.** Generating a sample uses the **topological order + CPTs** from Topic 6; the most-used MCMC method (Gibbs) resamples each variable from its **Markov blanket** — the exact object defined in Topic 6. And every method here estimates the **same** quantity VE computes — sampling is judged correct precisely because it converges to VE's exact answer.

**Forward (closing the section).** This is the last topic of the section, so there is no "next topic" within Reasoning Under Uncertainty — but §8 notes how these ideas reappear across the rest of AI/ML (e.g., Monte Carlo estimation, simulation-based reasoning).

---

## 2. Prerequisites

- **Bayesian networks (Topic 6):** factorization $\prod_i P(X_i\mid\mathrm{Pa}(X_i))$, **topological order**, CPTs, and the **Markov blanket** $\mathrm{MB}(X)=\mathrm{Pa}(X)\cup\mathrm{Children}(X)\cup\{\text{co-parents}\}$.
- **The inference query** $P(Q\mid e)$ and the normaliser $\alpha$ (Topic 7).
- **Sampling from a discrete distribution / CPT** using a uniform random number and a cumulative rule.
- **Law of large numbers** intuition: the sample average of an event's indicator converges to its probability.

---

## 3. Core idea — intuition first

Suppose you want $P(\text{Rain}=t)$ but find the joint awkward to sum. **Just simulate the world many times.** Walk the network top-down: flip a (biased) coin for *Cloudy* using its prior, then — knowing cloudiness — flip for *Sprinkler* and *Rain* from their CPTs, then for *WetGrass*. That gives **one** complete sample. Do it 10,000 times and count: the fraction of worlds with Rain $=t$ **estimates** $P(\text{Rain}=t)$. As the number of samples grows, the estimate homes in on the true value.

Two complications, each solved by a smarter sampler:

1. **There's evidence** (e.g., "WetGrass is true"). Most simulated worlds won't match it. We need to either (a) throw away the non-matching ones (**rejection sampling** — simple but wasteful), or (b) **force** the evidence to hold and **weight** each world by how plausible that forcing was (**likelihood weighting** — no waste), or (c) wander through worlds that already satisfy the evidence (**Gibbs / MCMC**).

2. **Rare evidence** makes rejection throw away almost everything, and makes likelihood-weighting weights tiny. Gibbs sidesteps this by *never* sampling evidence — it only ever proposes states consistent with it.

That's the whole topic: four ways to generate samples, and how to turn the samples into a probability estimate.

---

## 4. Formal treatment

Throughout, $N$ = number of samples. We use the **Sprinkler network** (R&N), all Boolean, for every example so the estimates can be checked against the exact answer.

```
Structure:  C → S,  C → R,  S → W,  R → W   (Cloudy, Sprinkler, Rain, WetGrass)
Topological order:  C, then S and R, then W

P(C=t)=0.5
P(S=t|C=t)=0.1   P(S=t|C=f)=0.5
P(R=t|C=t)=0.8   P(R=t|C=f)=0.2
P(W=t|S,R):  (t,t)=0.99  (t,f)=0.90  (f,t)=0.90  (f,f)=0.00
```

**Sampling a single Boolean variable.** Given $P(X{=}t)=p$ and a uniform random number $r\in[0,1)$, the standard convention is:
```
output  X = true   if  r < p,   else  X = false.
```
*(State this convention in exams — the comparison direction is a trap. For >2 values, use the cumulative distribution.)*

---

### 4.1 Prior sampling (direct / ancestral sampling) — no evidence

> **Procedure.** Sample the variables **in topological order**; sample each $X_i$ from $P(X_i\mid \mathrm{Pa}(X_i))$ using the parents' already-sampled values. The result is one full assignment drawn from the **joint** $P(X_1,\dots,X_n)$.

**Estimating a probability:** generate $N$ samples; for any event,
```
P̂(event) = (number of samples satisfying the event) / N
```
**Property:** $\hat P$ is an **unbiased, consistent** estimator of $P(\text{event})$ — it converges to the true probability as $N\to\infty$. Used only for **prior** quantities $P(X)$ (no evidence).

---

### 4.2 Rejection sampling — evidence by discarding

> **Procedure.** To estimate $P(Q\mid e)$: do prior sampling of the **whole** network (including evidence variables); **discard** every sample whose evidence variables don't match $e$; among the kept (accepted) samples, estimate
> ```
> P̂(Q | e) = (accepted samples with Q) / (total accepted samples)
> ```

**Property:** consistent — converges to $P(Q\mid e)$.

> **Acceptance rate $=P(e)$.** The expected fraction of samples kept equals the prior probability of the evidence. So when $P(e)$ is small, almost all samples are wasted, and the estimate is based on very few — **rejection sampling is inefficient for unlikely evidence,** and uselessly so as the number of evidence variables grows.

---

### 4.3 Likelihood weighting (LW) — evidence by fixing + weighting

This is **importance sampling**: it never wastes a sample. It samples non-evidence variables but **clamps** evidence variables to their observed values, then corrects with a weight.

> **Procedure.** Initialise weight $w=1$. Process variables in **topological order**:
> - If $X_i$ is a **non-evidence** variable: **sample** it from $P(X_i\mid \mathrm{Pa}(X_i))$.
> - If $X_i$ is an **evidence** variable (fixed to $e_i$): **do not sample**; instead multiply
>   ```
>   w ← w · P(X_i = e_i | Parents(X_i))
>   ```
> Each sample is the (clamped) assignment together with its weight $w$.

> **Weight of a sample:**
> ```
> w = Π_{E_i ∈ evidence}  P(e_i | Parents(E_i))      ← product over EVIDENCE variables only
> ```

**Estimating the posterior (weighted, not counted):**
```
P̂(Q | e) = ( Σ over samples with Q of their weights ) / ( Σ over ALL samples of their weights )
```

**Why it works (one line).** Clamping evidence makes samples come from a *different* distribution; the weight $w$ is exactly the correction so that weighted samples are distributed as $P(\text{non-evidence},\,e)$. Hence the weighted fraction with $Q$ estimates $P(Q,e)/P(e)=P(Q\mid e)$. **LW is consistent.**

> **Weakness:** when evidence is unlikely (or evidence nodes lie *downstream*), most weights are tiny and a few samples dominate → **high variance**, slow convergence. Still consistent, just inefficient.

---

### 4.4 Markov Chain Monte Carlo (MCMC): Gibbs sampling — wander among evidence-consistent worlds

> **Procedure.**
> 1. Fix evidence variables to $e$ for the entire run.
> 2. Initialise the non-evidence variables to any values (a starting state).
> 3. Repeat many times: pick a non-evidence variable $X_i$ and **resample** it from its conditional **given all the other current values**, which (crucially) depends only on its **Markov blanket**:
>    ```
>    P(xi | mb(Xi))  ∝  P(xi | Parents(Xi)) · Π_{Y ∈ Children(Xi)} P(y | Parents(Y))
>    ```
>    (Normalise over the values of $X_i$.) Each visited state is one sample.
> 4. After a **burn-in** period, estimate
>    ```
>    P̂(Q | e) = (fraction of sampled states satisfying Q)
>    ```

**Why the Markov blanket?** From Topic 6, a node is conditionally independent of everything else given its Markov blanket, so resampling needs only those neighbours — that locality is what keeps each step cheap.

**Properties.**
- The sequence of states is a **Markov chain whose stationary distribution is the true posterior $P(X\mid e)$**; the estimate is **consistent** (asymptotically unbiased).
- **Burn-in:** discard early states (chain hasn't reached the stationary distribution).
- Consecutive samples are **correlated** (not independent), so the effective sample size is smaller than $N$.
- **Caveat:** deterministic CPTs (probabilities $0$ or $1$) can make the chain **reducible** (gets stuck), breaking convergence.

---

### 4.5 What "approximate" means here — consistency & error rate

> An estimator is **consistent** if it converges to the true value as $N\to\infty$. **All four methods are consistent.** Prior sampling, rejection, and LW also give *unbiased* estimates of the relevant counts/weights; Gibbs is unbiased *in the limit* once mixed.

> **Monte Carlo error $\sim O(1/\sqrt{N})$** — to halve the error you need $4\times$ the samples. Crucially this rate is **independent of the number of variables**, which is exactly why sampling scales to networks where VE's $2^w$ does not.

---

### 4.6 Quick comparison

| Method | Samples evidence vars? | Handles evidence by | Wasteful when… | Samples i.i.d.? |
|---|---|---|---|---|
| **Prior sampling** | (no evidence) | — (priors only) | — | yes |
| **Rejection** | **yes**, then discards | discarding mismatches | $P(e)$ small (accept rate $=P(e)$) | yes (the kept ones) |
| **Likelihood weighting** | **no**, clamps them | weighting by $\prod P(e_i\mid\mathrm{Pa})$ | $P(e)$ small / downstream evidence (high variance) | yes (weighted) |
| **Gibbs (MCMC)** | **no**, fixes them | wandering via Markov blanket | poor mixing / deterministic CPTs | **no** (correlated) |

---

## 5. Worked problems (GATE-style)

All on the Sprinkler network above.

### Problem 1 (NAT/MCQ) — prior sampling from random numbers

**Q.** Using the convention "output *true* iff $r<P(\text{true})$" and the random numbers $r=[0.40,\ 0.60,\ 0.15,\ 0.30]$ consumed in topological order $C,S,R,W$, what sample is produced?

**Solution.**
```
C:  P(C=t)=0.50,            r=0.40 < 0.50  → C = true
S:  P(S=t|C=t)=0.10,        r=0.60 < 0.10? no  → S = false
R:  P(R=t|C=t)=0.80,        r=0.15 < 0.80? yes → R = true
W:  P(W=t|S=f,R=t)=0.90,    r=0.30 < 0.90? yes → W = true
```
**Sample = (C=t, S=f, R=t, W=t).** *(Trap: each variable's CPT row is selected by its already-sampled parents — here $S$ and $R$ use $C=t$, and $W$ uses $S=f,R=t$.)*

---

### Problem 2 (NAT) — rejection sampling: acceptance rate and estimate

**Q.** We estimate $P(R=t\mid S=t)$ by rejection sampling with $N=1000$ prior samples. (a) About how many samples are accepted? (b) If $90$ of the accepted samples have $R=t$, what is the estimate, and how does it compare to the exact value?

**Solution.**
**(a) Acceptance rate $=P(S=t)$:**
```
P(S=t) = P(S=t|C=t)P(C=t) + P(S=t|C=f)P(C=f) = 0.1·0.5 + 0.5·0.5 = 0.30
```
So $\approx 0.30\times1000 = \mathbf{300}$ accepted (and $\sim700$ **wasted** — rejection's weakness).

**(b) Estimate** $=90/300=\mathbf{0.30}$.
**Exact check:** $P(R{=}t,S{=}t)=\sum_C P(C)P(S{=}t|C)P(R{=}t|C)=0.5\cdot0.1\cdot0.8+0.5\cdot0.5\cdot0.2=0.09$, so $P(R{=}t\mid S{=}t)=0.09/0.30=0.30$. The estimate matches — and would fluctuate around $0.30$ for finite $N$.

*If the evidence were rarer, say $P(e)=0.01$, then $\sim990$ of $1000$ samples would be discarded — the motivation for likelihood weighting.*

---

### Problem 3 (NAT) — likelihood weighting estimate (the classic)

**Q.** Estimate $P(R=t\mid S=t,\,W=t)$ by likelihood weighting. Evidence $=\{S{=}t, W{=}t\}$; the sampled (non-evidence) variables are $C$ then $R$. Suppose LW produced $10$ samples, grouped by their sampled $(C,R)$ values:

| sampled $(C,R)$ | count | weight per sample $=P(S{=}t\mid C)\cdot P(W{=}t\mid S{=}t,R)$ |
|---|---|---|
| $(C{=}t,R{=}t)$ | 4 | $0.1\cdot0.99 = 0.099$ |
| $(C{=}t,R{=}f)$ | 1 | $0.1\cdot0.90 = 0.090$ |
| $(C{=}f,R{=}t)$ | 1 | $0.5\cdot0.99 = 0.495$ |
| $(C{=}f,R{=}f)$ | 4 | $0.5\cdot0.90 = 0.450$ |

**Solution.** First note the weight uses **only the evidence variables** $S$ and $W$: $w=P(S{=}t\mid C)\cdot P(W{=}t\mid S{=}t,R)$ (we never sample $S$ or $W$).

```
Total weight        = 4·0.099 + 1·0.090 + 1·0.495 + 4·0.450
                    = 0.396 + 0.090 + 0.495 + 1.800 = 2.781
Weight with R = t   = 4·0.099 + 1·0.495 = 0.396 + 0.495 = 0.891

P̂(R=t | S=t, W=t) = 0.891 / 2.781 ≈ 0.320
```
**Estimate ≈ 0.32.**

**Exact check (for confidence):**
```
P(R=t,S=t,W=t) = Σ_C P(C)P(S=t|C)P(R=t|C)·P(W=t|S=t,R=t)
               = (0.5·0.1·0.8 + 0.5·0.5·0.2)·0.99 = 0.09·0.99 = 0.0891
P(R=f,S=t,W=t) = (0.5·0.1·0.2 + 0.5·0.5·0.8)·0.90 = 0.21·0.90 = 0.1890
P(R=t|S=t,W=t) = 0.0891 / (0.0891+0.1890) = 0.0891/0.2781 ≈ 0.320  ✓
```
The weighted estimate equals the exact posterior here (because we used representative counts). *(Trap: divide by the **sum of weights**, not by the number of samples; and never weight by the non-evidence CPTs of $C$ or $R$.)*

---

### Problem 4 (NAT) — Gibbs sampling step from the Markov blanket

**Q.** We estimate $P(C\mid S=t,\,W=t)$ by Gibbs sampling. The current state has $S=t$ (evidence), $W=t$ (evidence), and $R=f$. Derive the distribution from which Gibbs resamples $C$, and give $P(C=t\mid\text{rest})$.

**Solution.** $\mathrm{MB}(C)=\mathrm{Pa}(C)\cup\mathrm{Children}(C)\cup\text{co-parents}=\{\}\cup\{S,R\}\cup\{\}=\{S,R\}$. So $C$'s resampling depends only on $S$ and $R$ (note $W\notin \mathrm{MB}(C)$ — locality). Using the Gibbs formula (children of $C$ are $S$ and $R$):
```
P(C=c | mb) ∝ P(C=c) · P(S=t | C=c) · P(R=f | C=c)

C=t:  0.5 · 0.1 · (1−0.8) = 0.5·0.1·0.2 = 0.010
C=f:  0.5 · 0.5 · (1−0.2) = 0.5·0.5·0.8 = 0.200
```
Normalise:
```
P(C=t | S=t, R=f) = 0.010 / (0.010 + 0.200) = 0.010/0.210 ≈ 0.0476
P(C=f | S=t, R=f) ≈ 0.952
```
We'd draw $C$ from this, then resample $R$ from its blanket $\mathrm{MB}(R)=\{C,S,W\}$, and so on; after burn-in, $\hat P(C{=}t\mid e)$ is the fraction of states with $C{=}t$.

*(Trap: include the **children's** CPTs, not just $C$'s prior; and remember to normalise.)*

---

## 6. Common GATE question types on this topic

1. **Generate a sample** from given random numbers + CPTs by ancestral sampling (state the comparison convention). *(Problem 1.)*
2. **Rejection sampling:** acceptance rate $=P(e)$; estimate from accepted counts; why it's wasteful. *(Problem 2.)*
3. **Likelihood weighting:** compute a sample's weight $=\prod_{E_i}P(e_i\mid\mathrm{Pa})$ and the weighted estimate. *(Problem 3.)*
4. **Gibbs sampling:** write the Markov-blanket resampling distribution and compute it for one variable. *(Problem 4.)*
5. **Method comparison MCQ:** which samples evidence variables, which weights, which is consistent, why rejection/LW degrade for unlikely evidence.
6. **Consistency / convergence:** all methods consistent; Monte Carlo error $O(1/\sqrt N)$, dimension-independent.
7. **When to use sampling vs VE:** sampling chosen when treewidth (and so VE cost) is too large.

---

## 7. Traps, pitfalls & common mistakes

- **Sampling convention.** Fix and state "true iff $r<p$" (or your chosen rule, and the value ordering for multi-valued variables). The comparison direction silently flips answers.
- **Wrong CPT row.** Sample each variable from the row selected by its **already-sampled parents** — not the marginal. *(Problem 1.)*
- **Likelihood weighting — what to weight.** The weight is the product over **evidence variables only**, $\prod_{E_i}P(e_i\mid\mathrm{Pa}(E_i))$. Do **not** include non-evidence CPTs; do **not** sample evidence variables.
- **LW estimate uses weights, not counts.** Denominator is the **sum of all weights**, numerator the sum of weights of samples with $Q$. Counting samples instead of summing weights is wrong.
- **Rejection vs LW confusion.** Rejection *does* sample evidence variables then discards mismatches; LW *clamps* them and weights. Don't mix the two.
- **Acceptance rate.** For rejection sampling it equals $P(e)$ — small for rare/conjunctive evidence, making the estimate noisy.
- **Gibbs resampling set.** Resample from the **Markov blanket** (parents + children + co-parents), and the conditional includes the **children's CPTs**, not just the node's own CPT. Then **normalise**.
- **Burn-in & correlation (Gibbs).** Early states are biased (discard them); consecutive samples are correlated, so effective sample size $<N$.
- **Deterministic CPTs break Gibbs.** Probabilities of exactly $0$ or $1$ can trap the chain (reducible), violating convergence.
- **Approximate ≠ wrong, but ≠ exact for finite $N$.** Estimates fluctuate; consistency is an $N\to\infty$ guarantee. Don't claim a finite-sample estimate equals the true value (our worked answers match only because we used representative counts).
- **Variance vs bias.** LW with downstream/unlikely evidence is consistent but **high variance** — needs many samples; this is a favourite "which statement is true" distractor.

---

## 8. Connection forward (and section close)

This topic **completes Reasoning Under Uncertainty.** You now have the full arc: **represent** a joint compactly and read its independencies (Topic 6, Bayesian networks); **infer exactly** with Variable Elimination when treewidth is small (Topic 7); and **infer approximately** by sampling when it isn't (Topic 8). The choice between the last two is an exactness-vs-scalability decision: VE gives the precise posterior at cost $\Theta(n\,2^w)$; sampling gives a consistent estimate at $O(1/\sqrt N)$ error, independent of the network's size. The same Monte Carlo idea — *estimate by simulating* — recurs throughout AI and machine learning whenever a quantity is easy to sample but hard to compute, so the habits built here (ancestral sampling, importance weighting, MCMC) transfer well beyond Bayesian networks.

---

## 9. One-line cheat-sheet summary

**Approximate inference estimates $P(Q\mid e)$ by simulation: ancestral sampling draws from the prior (no evidence); rejection sampling keeps only evidence-matching samples (accept rate $=P(e)$, wasteful); likelihood weighting clamps evidence and weights each sample by $\prod_{E_i}P(e_i\mid\mathrm{Pa}(E_i))$, then divides summed weights; Gibbs/MCMC resamples each non-evidence variable from its Markov-blanket conditional $\propto P(x_i\mid\mathrm{Pa})\prod_{\text{children}}P(y\mid\mathrm{Pa})$ — all consistent, with Monte Carlo error $O(1/\sqrt N)$ independent of network size.**

---

*This is the final topic of Reasoning Under Uncertainty — and the last topic in the Artificial Intelligence plan.*

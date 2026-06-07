# Topic 5 — Bayes' Theorem

**Subject:** Probability and Statistics (GATE DA)
**Section:** Probability Foundations (Topic 5 of 16 — **final topic of this section**)

---

## 0. Topic header & syllabus map

- **Topic:** **Bayes' theorem** — the rule for *inverting* a conditional probability (turning $P(A\mid B)$ into $P(B\mid A)$) — together with its vocabulary (**prior, likelihood, evidence, posterior**), the partition form, the **odds / likelihood-ratio** form, and **sequential updating**.
- **Where it sits:** This is the **capstone of Probability Foundations**. It is built *entirely* from the two rules of the previous topic — the **multiplication rule** and the **law of total probability**. After this, the syllabus shifts gears: the next topic (in a new chat) is **Discrete Random Variables & PMF**, where we stop talking about qualitative events and start attaching *numbers* to outcomes.

---

## 1. Why this topic exists (motivation + connection)

- **The real problem it solves:** We very often know a conditional in the "wrong" direction. A test tells us $P(\text{positive}\mid\text{disease})$, but a patient wants $P(\text{disease}\mid\text{positive})$. A factory's defect rate gives $P(\text{defective}\mid\text{Machine A})$, but quality control needs $P(\text{Machine A}\mid\text{defective})$. Bayes' theorem is the exact rule for flipping a conditional — for reasoning *from observed evidence back to its likely cause*.
- **What the previous topic couldn't handle:** Topic 4 gave the **forward** direction only. Using the law of total probability, we could compute $P(A)$ from the conditionals $P(A\mid B_i)$ and priors $P(B_i)$ — i.e. "cause → effect." But it offered no way to run that backwards to $P(B_j\mid A)$ — "effect → cause." It also explicitly warned that $P(A\mid B)\ne P(B\mid A)$ without telling us the *relationship* between them. Bayes is precisely that missing relationship.
- **What the next topic builds on it:** The Foundations section ends here. The next step — **random variables and the PMF** — repackages outcomes as numbers so we can compute distributions, expectations, and the named distributions. Bayesian reasoning then recurs throughout: as **Bayes for distributions** in inference, and as the backbone of **naive-Bayes classification** in machine learning. Conceptually, Bayes is the bridge from "describing chance" to "learning from data."

---

## 2. Prerequisites

- **Conditional probability** $P(A\mid B)=\dfrac{P(A\cap B)}{P(B)}$ and the **multiplication rule** $P(A\cap B)=P(A\mid B)P(B)$ (Topic 4).
- **Law of total probability** over a partition (Topic 4) — this is the denominator of Bayes.
- **Partitions** (mutually exclusive + exhaustive events) from Topic 2.
- **Independence restatement** (Topic 3) — to recognise when evidence is uninformative.

---

## 3. Core idea — intuition first

Bayes' theorem is **belief updating**. You start with a belief about how likely each hypothesis is (the **prior**). Then you observe some evidence. Each hypothesis predicted that evidence with some probability (the **likelihood**). Bayes says: *re-weight your prior beliefs by how well each hypothesis predicted what you actually saw, then renormalise.* The result is your updated belief (the **posterior**).

$$\text{posterior} \;\propto\; \text{likelihood} \times \text{prior}.$$

*Analogy.* Two coins in a box: one fair, one two-headed. Before flipping, each is equally likely to be in your hand (prior $\tfrac12,\tfrac12$). You flip and see **Heads**. The two-headed coin "predicted" Heads with certainty (likelihood $1$); the fair coin predicted it with probability $\tfrac12$. Heads is *twice as consistent* with the two-headed coin, so your belief shifts to $\tfrac23$ two-headed, $\tfrac13$ fair. You didn't ignore the prior — you *updated* it.

The single most important consequence is that the **prior matters**. A wildly accurate test for a very rare disease can still leave you probably-healthy after a positive result, because the prior (the disease is rare) pulls hard against the evidence. Forgetting this is the **base-rate fallacy**, and it's the #1 thing GATE tests here.

---

## 4. Formal treatment

### 4.1 Bayes' theorem — statement and derivation

**Two-hypothesis form.** For events with $P(A)>0$, $0<P(B)<1$:
$$\boxed{\,P(B\mid A)=\frac{P(A\mid B)\,P(B)}{P(A)}=\frac{P(A\mid B)\,P(B)}{P(A\mid B)P(B)+P(A\mid B^c)P(B^c)}\,}$$

**Derivation (one line each):**
$$P(B\mid A)=\frac{P(A\cap B)}{P(A)}\ \text{(definition)}\ =\ \frac{P(A\mid B)P(B)}{P(A)}\ \text{(multiplication rule)}\ =\ \frac{P(A\mid B)P(B)}{\sum P(A\mid B_i)P(B_i)}\ \text{(LTP).}$$

**General partition form.** If $\{B_1,\dots,B_n\}$ partitions $S$ (pairwise disjoint, exhaustive, each $P(B_i)>0$) and $P(A)>0$:
$$\boxed{\,P(B_j\mid A)=\frac{P(A\mid B_j)\,P(B_j)}{\displaystyle\sum_{i=1}^{n}P(A\mid B_i)\,P(B_i)}\,.}$$

*Conditions:* the $B_i$ must form a genuine partition and $A$ must have positive probability. The numerator is "the path through $B_j$"; the denominator is "all paths to $A$" (the law of total probability).

### 4.2 The four named quantities

| Quantity | Symbol | Meaning |
|---|---|---|
| **Prior** | $P(B_j)$ | belief in hypothesis $B_j$ **before** seeing evidence |
| **Likelihood** | $P(A\mid B_j)$ | probability the evidence $A$ occurs **if** $B_j$ holds |
| **Evidence** (marginal likelihood / normaliser) | $P(A)=\sum_i P(A\mid B_i)P(B_i)$ | total probability of the evidence |
| **Posterior** | $P(B_j\mid A)$ | updated belief in $B_j$ **after** seeing $A$ |

Key facts: **priors sum to 1** across the partition; **likelihoods need NOT sum to 1** across hypotheses (each is conditioned on a different event). The evidence $P(A)$ is just a constant that makes the posteriors sum to 1, which is why $\text{posterior}\propto\text{likelihood}\times\text{prior}$.

### 4.3 Odds (likelihood-ratio) form — a fast shortcut

For two competing hypotheses $H$ and $H^c$, divide the two posteriors:
$$\boxed{\,\underbrace{\frac{P(H\mid A)}{P(H^c\mid A)}}_{\text{posterior odds}}=\underbrace{\frac{P(A\mid H)}{P(A\mid H^c)}}_{\text{likelihood ratio (Bayes factor)}}\times\underbrace{\frac{P(H)}{P(H^c)}}_{\text{prior odds}}\,}$$
The evidence $P(A)$ cancels — no denominator needed. Convert odds $\to$ probability with $P=\dfrac{\text{odds}}{1+\text{odds}}$. This is the cleanest way to do two-hypothesis and **sequential** problems.

### 4.4 Sequential (recursive) updating

With independent pieces of evidence $A_1,A_2,\dots$ (conditionally independent given each hypothesis), **today's posterior is tomorrow's prior**: update on $A_1$, then feed that posterior in as the prior when updating on $A_2$. In odds form this is just multiplying by each likelihood ratio in turn:
$$\text{(final odds)}=\text{(prior odds)}\times\prod_k \frac{P(A_k\mid H)}{P(A_k\mid H^c)}.$$

### 4.5 Medical-test vocabulary (frequently tested, easy to mix up)

For a disease $D$ and a positive test $+$:
- **Sensitivity** (true-positive rate) $=P(+\mid D)$.
- **Specificity** (true-negative rate) $=P(-\mid D^c)$.
- **False-positive rate** $=P(+\mid D^c)=1-\text{specificity}$.
- **False-negative rate** $=P(-\mid D)=1-\text{sensitivity}$.
- **Prevalence / base rate** $=P(D)$ (the prior).
- **Positive predictive value** $=P(D\mid +)$ (the posterior you usually want).

*Derivations skipped:* there is nothing beyond §4.1 to derive — Bayes *is* the combination of the multiplication rule and LTP, both already established. GATE asks you to *apply* it, set up the partition correctly, and resist base-rate neglect.

---

## 5. Worked problems (GATE-style)

### Problem 1 — The medical-test base-rate problem (NAT, the canonical trap)
*A disease affects $1\%$ of a population. A test has sensitivity $99\%$ and specificity $95\%$. A randomly chosen person tests **positive**. Find $P(\text{disease}\mid \text{positive})$.*

Prior $P(D)=0.01$, $P(D^c)=0.99$. Likelihoods: $P(+\mid D)=0.99$, $P(+\mid D^c)=1-0.95=0.05$.

Evidence (LTP):
$$P(+)=0.99(0.01)+0.05(0.99)=0.0099+0.0495=0.0594.$$
Posterior:
$$P(D\mid +)=\frac{0.0099}{0.0594}\approx \boxed{0.167\ (16.7\%)}.$$

**Trap defused:** despite a "$99\%$ accurate" test, a positive result means only about a $1$-in-$6$ chance of disease — because the disease is rare, false positives ($0.05\times0.99\approx 5\%$ of people) **swamp** true positives ($0.99\times0.01\approx 1\%$). Ignoring the base rate $P(D)=0.01$ and answering "$\approx 99\%$" is the base-rate fallacy.

---

### Problem 2 — Inverting the factory problem (NAT, links to Topic 4)
*Factory I makes $60\%$ of items at a $2\%$ defect rate; Factory II makes $40\%$ at a $5\%$ defect rate. An item is found **defective**. Find $P(\text{Factory I}\mid\text{defective})$.*

From Topic 4 we already have $P(D)=0.02(0.6)+0.05(0.4)=0.032$. Now invert:
$$P(F_I\mid D)=\frac{P(D\mid F_I)P(F_I)}{P(D)}=\frac{0.02\times0.6}{0.032}=\frac{0.012}{0.032}=\boxed{0.375}.$$
And $P(F_{II}\mid D)=\dfrac{0.020}{0.032}=0.625$ (check: $0.375+0.625=1$ ✓).

*Note the shift:* Factory I makes the *majority* of items ($60\%$), yet given a defect it is the *less* likely source ($37.5\%$) — because its defect rate is lower. Bayes correctly trades off prior against likelihood.

---

### Problem 3 — Three-box partition (NAT)
*Three boxes contain red/blue balls — Box 1: $2R,8B$; Box 2: $5R,5B$; Box 3: $8R,2B$. A box is chosen at random ($\tfrac13$ each) and one ball drawn; it is **red**. Find the probability it came from Box 3.*

Priors $\tfrac13$ each. Likelihoods $P(R\mid B_1)=0.2,\ P(R\mid B_2)=0.5,\ P(R\mid B_3)=0.8$.

Evidence:
$$P(R)=\tfrac13(0.2+0.5+0.8)=\tfrac13(1.5)=0.5.$$
Posterior for Box 3:
$$P(B_3\mid R)=\frac{\tfrac13(0.8)}{0.5}=\frac{0.8}{1.5}=\boxed{\tfrac{8}{15}\approx 0.533}.$$
(Similarly $P(B_1\mid R)=\tfrac{2}{15}$, $P(B_2\mid R)=\tfrac{5}{15}$; they sum to $1$.)

---

### Problem 4 — Odds form & sequential updating (NAT)
*A box has one fair coin and one two-headed coin. You pick one at random and flip it. (a) Given it shows **Heads**, find $P(\text{two-headed})$. (b) You flip the **same** coin again and get **Heads** again — now find $P(\text{two-headed})$.*

Prior odds (two-headed : fair) $=1:1$. Likelihood ratio per Heads $=\dfrac{P(H\mid 2\text{H})}{P(H\mid \text{fair})}=\dfrac{1}{0.5}=2$.

**(a)** One Head: posterior odds $=2\times 1=2:1\Rightarrow P(2\text{H}\mid H)=\dfrac{2}{1+2}=\boxed{\tfrac23}$.

**(b)** Two Heads: posterior odds $=2\times2\times1=4:1\Rightarrow P(2\text{H}\mid HH)=\dfrac{4}{1+4}=\boxed{\tfrac45}$.

*Check (direct Bayes, part b):* $\dfrac{1\cdot\tfrac12}{1\cdot\tfrac12+(\tfrac12)^2\cdot\tfrac12}=\dfrac{0.5}{0.5+0.125}=\dfrac{0.5}{0.625}=0.8$ ✓. The odds form turns "update on each flip" into "multiply by $2$ each time."

---

## 6. Common GATE question types on this topic

- **Diagnostic / test problems:** given prevalence, sensitivity, specificity → find positive (or negative) predictive value $P(D\mid +)$.
- **Source-identification (reverse mixture):** given an observed defect/outcome, which machine/factory/box produced it — the inversion of a law-of-total-probability setup.
- **Base-rate problems** engineered so the intuitive answer is wrong (rare condition, accurate test).
- **Multi-hypothesis partitions** (3+ boxes/urns/sources) using the general Bayes form.
- **Odds-form / likelihood-ratio** computations and **sequential updating** across repeated independent observations.
- **Terminology identification:** label the prior, likelihood, evidence, posterior in a worded problem, or pick the correct Bayes expression (MCQ).

---

## 7. Traps, pitfalls & common mistakes

- **Base-rate neglect (the headline trap).** A very accurate test on a rare event still yields a modest posterior. Always include the prior $P(B_j)$ — never report the likelihood $P(A\mid B)$ as if it were the posterior $P(B\mid A)$.
- **Confusing $P(A\mid B)$ with $P(B\mid A)$.** This is the exact error Bayes fixes; they are equal only by coincidence.
- **Wrong / incomplete denominator.** $P(A)$ must sum over the **entire** partition (every hypothesis, including $B^c$). Dropping a term — or using $P(A\mid B)$ alone as the denominator — is a classic mistake.
- **Sensitivity/specificity mix-ups.** False-positive rate $=1-\text{specificity}=P(+\mid D^c)$; false-negative rate $=1-\text{sensitivity}=P(-\mid D)$. Plugging the wrong one in flips the answer.
- **Priors vs likelihoods.** Priors $P(B_i)$ sum to $1$; likelihoods $P(A\mid B_i)$ generally do **not** (different conditioning events). Don't "normalise" the likelihoods.
- **Forgetting to update the prior in sequential problems.** After the first observation, the **posterior becomes the new prior**; reusing the original prior for the second observation is wrong (unless you process all evidence jointly).
- **Equal-prior assumption.** Use the priors the problem states; assume $\tfrac1n$ each *only* when the selection is genuinely uniform.
- **Conditioning on a zero-probability evidence** ($P(A)=0$) leaves the posterior undefined.
- **Odds ↔ probability conversion errors:** $P=\dfrac{\text{odds}}{1+\text{odds}}$ and $\text{odds}=\dfrac{P}{1-P}$ — keep the direction straight.

---

## 8. Connection forward

Bayes' theorem completes the **Probability Foundations** arc: we can now define sample spaces and events, assign probabilities via the axioms, recognise independence and mutual exclusivity, move among joint/marginal/conditional probabilities, and — finally — **invert** conditionals to reason from evidence to cause. That last skill is the seed of all statistical *inference*.

The next chat opens a new chapter: **Discrete Random Variables & PMF**. So far probabilities have attached to *qualitative* events ("defective," "Heads," "from Factory I"). Random variables attach *numbers* to outcomes, and the **probability mass function (PMF)** records how probability is spread over those numbers. That single shift unlocks everything downstream — expectation and variance, the named distributions (Bernoulli, binomial, Poisson, normal, …), and the inference machinery (CLT, confidence intervals, hypothesis tests) — and Bayesian updating will reappear there in its distributional form and again in machine-learning classifiers.

---

## 9. One-line cheat-sheet summary

**Bayes flips a conditional: $P(B_j\mid A)=\dfrac{P(A\mid B_j)P(B_j)}{\sum_i P(A\mid B_i)P(B_i)}$, i.e. posterior $\propto$ likelihood $\times$ prior; equivalently posterior odds $=$ likelihood ratio $\times$ prior odds; always include the base rate (priors sum to 1, likelihoods need not), and never confuse $P(A\mid B)$ with $P(B\mid A)$.**

---

### ✅ End of Section 1 — **Probability Foundations** (Topics 1–5) complete.
### ➡️ Next topic (new chat): **Topic 6 — Discrete Random Variables & PMF** (start of *Random Variables & Expectation*).
*Open the new chat and ask for it when you're ready — I'll keep the same structure.*

# Topic 4 — Marginal, Conditional and Joint Probability

**Subject:** Probability and Statistics (GATE DA)
**Section:** Probability Foundations (Topic 4 of 16)

---

## 0. Topic header & syllabus map

- **Topic:** The three faces of probability for two (or more) events — **joint** $P(A\cap B)$, **marginal** $P(A)$, and **conditional** $P(A\mid B)$ — plus the tools that connect them: the **multiplication (chain) rule**, the **law of total probability**, the contingency-table view, and the restatement of independence via conditioning.
- **Where it sits:** Comes **right after Independent vs Mutually Exclusive Events**. That topic handled the special case where one event gives *no* information about another ($P(A\mid B)=P(A)$). This topic builds the machinery for the *general* case, where one event **does** change the odds of another. It leads **directly into Bayes' Theorem**, which is nothing but conditional probability "run backwards," powered by the two rules introduced here.

---

## 1. Why this topic exists (motivation + connection)

- **The real problem it solves:** Information updates probabilities. Once you learn that some event $B$ has happened, the chance of $A$ usually changes — a positive medical test changes the odds of disease; drawing a red card changes the odds the next card is red. We need a precise way to compute "$P(A)$ **given** that $B$ occurred," to combine such pieces into joint probabilities, and to recover a single-event probability from a richer joint description.
- **What the previous topic couldn't handle:** Independence covered only the degenerate case $P(A\mid B)=P(A)$ — *no* information transfer — and even there it just *recognised* independence, it didn't define conditioning. It gave us $P(A\cap B)=P(A)P(B)$ **only** when events were independent; for dependent events we still had no formula for the intersection. Conditional probability supplies exactly that general formula, with independence falling out as a special case.
- **What the next topic builds on it:** Bayes' theorem rearranges $P(A\mid B)=\dfrac{P(A\cap B)}{P(B)}$ together with the **multiplication rule** and the **law of total probability** (both defined here) to flip a conditional: from $P(\text{evidence}\mid\text{cause})$ to $P(\text{cause}\mid\text{evidence})$. So Topic 5 is a direct, mechanical consequence of Topic 4.

---

## 2. Prerequisites

- **Axioms & event algebra (Topic 2):** $\cup,\cap,{}^c$, partitions, the addition rule, $P(A^c)=1-P(A)$.
- **Independence & mutual exclusivity (Topic 3):** to see conditional probability as the generalisation and to use the independence restatement.
- **Counting (Topic 1):** for the underlying probabilities in concrete and table problems.

---

## 3. Core idea — intuition first

**Joint** = "both." $P(A\cap B)$ is the chance that $A$ and $B$ happen together. In a two-way table it's a *single cell*.

**Marginal** = "ignore the rest." $P(A)$ is the chance of $A$ regardless of what $B$ does. In a table it's a *row or column total* (a "margin," literally the number written in the margin) — you get it by summing the joint probabilities over all possibilities of the other variable.

**Conditional** = "zoom in." $P(A\mid B)$ asks: *given that we already know $B$ happened, how likely is $A$?* The trick is that learning $B$ shrinks your world from the whole sample space $S$ down to just $B$. So you no longer divide favourable outcomes by all of $S$; you divide by $B$:
$$P(A\mid B)=\frac{\text{outcomes in both }A\text{ and }B}{\text{outcomes in }B}=\frac{P(A\cap B)}{P(B)}.$$

*Analogy.* Roll a fair die. Unconditionally, $P(\text{number is }6)=\tfrac16$. Now someone tells you "the number is even." Your world is now just $\{2,4,6\}$, and $6$ is one of three equally likely survivors, so $P(6\mid\text{even})=\tfrac13$. Conditioning **renormalises** probabilities to the new, smaller sample space.

The three are tied together by one identity read two ways:
$$P(A\cap B)=P(B)\,P(A\mid B)=P(A)\,P(B\mid A).$$
"To get *both*, take the chance of the first times the chance of the second *given the first*." That's the multiplication rule, and it's the most useful sentence in this topic.

---

## 4. Formal treatment

### 4.1 Joint probability

For events $A,B$, the **joint probability** is simply $P(A\cap B)$ (often written $P(A,B)$). For a finite, equally-likely space it is $|A\cap B|/|S|$. In a **contingency (two-way) table**, the joint probabilities are the interior cells.

### 4.2 Marginal probability and marginalisation

The **marginal** of $A$ is $P(A)$. If $\{B_1,\dots,B_n\}$ partitions $S$, then $A$ splits into disjoint pieces $A\cap B_i$, so
$$\boxed{\,P(A)=\sum_{i=1}^{n}P(A\cap B_i)\,}\qquad\text{(marginalisation: ``sum the joints'').}$$
In a table this is exactly a row/column total.

### 4.3 Conditional probability

$$\boxed{\,P(A\mid B)=\frac{P(A\cap B)}{P(B)},\qquad \text{defined only when } P(B)>0.\,}$$

**Conditional probability is itself a valid probability** on the restricted space $B$ — it satisfies all the axioms:
- $P(A\mid B)\ge 0$;
- $P(S\mid B)=1$ (indeed $P(B\mid B)=1$);
- countable additivity over disjoint $A$'s;
- and consequently the familiar rules carry over **with the same conditioning event throughout**:
$$P(A^c\mid B)=1-P(A\mid B),\qquad P(A\cup C\mid B)=P(A\mid B)+P(C\mid B)-P(A\cap C\mid B).$$

> **Warning (a favourite trap):** complementing works on the *left*: $P(A\mid B)+P(A^c\mid B)=1$. It does **not** work on the conditioning side: $P(A\mid B)+P(A\mid B^c)$ is **not** generally $1$.

### 4.4 Multiplication (chain) rule

$$\boxed{\,P(A\cap B)=P(B)\,P(A\mid B)=P(A)\,P(B\mid A)\,.}$$
For $n$ events, the **chain rule**:
$$P(A_1\cap A_2\cap\cdots\cap A_n)=P(A_1)\,P(A_2\mid A_1)\,P(A_3\mid A_1\cap A_2)\cdots P(A_n\mid A_1\cap\cdots\cap A_{n-1}).$$
*Condition:* each conditioning event must have positive probability. This is the standard way to handle **sequential / without-replacement** experiments.

### 4.5 Law of Total Probability (LTP)

If $\{B_1,\dots,B_n\}$ is a **partition** of $S$ (pairwise mutually exclusive, exhaustive, each $P(B_i)>0$), then for any event $A$:
$$\boxed{\,P(A)=\sum_{i=1}^{n}P(A\mid B_i)\,P(B_i)\,.}$$
The two-part version (using $B$ and $B^c$) is the one you'll use most:
$$P(A)=P(A\mid B)P(B)+P(A\mid B^c)P(B^c).$$
*Why it matters:* it computes a marginal by **averaging conditionals over a partition** — and it is the denominator of Bayes' theorem.

### 4.6 Independence, restated

Combining the definition with Topic 3: when $P(B)>0$,
$$A\perp B\iff P(A\cap B)=P(A)P(B)\iff P(A\mid B)=P(A)\iff P(B\mid A)=P(B).$$
So **independence is exactly "conditioning doesn't change anything."** Dependence is everything else, and conditional probability measures *how much* it changes.

### 4.7 Asymmetry: $P(A\mid B)\ne P(B\mid A)$

In general these differ — they have different denominators ($P(B)$ vs $P(A)$). The exact relationship between them is **Bayes' theorem** (next topic). Treating them as equal is the classic "base-rate / prosecutor's fallacy."

*Derivations skipped:* none of substance. The definition, multiplication rule, and LTP each follow in one line from the axioms and are themselves the examinable content.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Conditioning renormalises the sample space (NAT, warm-up)
*A fair die is rolled. Find $P(\text{number}>3\mid\text{number is even})$.*

Conditioning event "even" $=\{2,4,6\}$ (3 outcomes). Within it, "$>3$" gives $\{4,6\}$ (2 outcomes).
$$P(>3\mid\text{even})=\frac{P(\{4,6\})}{P(\{2,4,6\})}=\frac{2/6}{3/6}=\boxed{\tfrac23}.$$
*(Contrast the unconditional $P(>3)=\tfrac36=\tfrac12$: knowing "even" changed the answer, so the events are dependent.)*

---

### Problem 2 — Contingency table: joint, marginal, conditional, independence (NAT / MSQ)
*Among $200$ people classified by gender and car ownership:*

| | Car = Yes | Car = No | **Total** |
|---|---|---|---|
| **Male** | 60 | 40 | **100** |
| **Female** | 50 | 50 | **100** |
| **Total** | **110** | **90** | **200** |

Divide every count by $200$ to get probabilities.
- **Joint:** $P(\text{Male}\cap\text{Car})=\dfrac{60}{200}=0.30$.
- **Marginals:** $P(\text{Male})=\dfrac{100}{200}=0.50$, $P(\text{Car})=\dfrac{110}{200}=0.55$.
- **Conditionals:** $P(\text{Car}\mid\text{Male})=\dfrac{60}{100}=0.60$, and $P(\text{Male}\mid\text{Car})=\dfrac{60}{110}\approx 0.545$.
- **Independent?** $P(\text{Male})P(\text{Car})=0.50\times0.55=0.275\ne 0.30=P(\text{Male}\cap\text{Car})$ → **dependent.**

*Note how $P(\text{Car}\mid\text{Male})=0.60\ne P(\text{Male}\mid\text{Car})\approx 0.545$ — the two conditionals are not interchangeable.*

---

### Problem 3 — Multiplication rule, drawing without replacement (NAT)
*An urn has $5$ red and $3$ blue balls. Two are drawn one after another **without replacement**. Find (a) $P(\text{both red})$, (b) $P(\text{first red, second blue})$, (c) $P(\text{one of each colour})$.*

Use $P(A\cap B)=P(A)P(B\mid A)$; the second draw's probabilities change because a ball was removed.

(a) $P(R_1\cap R_2)=\dfrac{5}{8}\cdot\dfrac{4}{7}=\dfrac{20}{56}=\boxed{\dfrac{5}{14}}$.

(b) $P(R_1\cap B_2)=\dfrac{5}{8}\cdot\dfrac{3}{7}=\dfrac{15}{56}$.

(c) One of each $=P(R_1B_2)+P(B_1R_2)=\dfrac{15}{56}+\dfrac{3}{8}\cdot\dfrac{5}{7}=\dfrac{15}{56}+\dfrac{15}{56}=\dfrac{30}{56}=\boxed{\dfrac{15}{28}}$.

*(With replacement the draws would be **independent** and the second-draw probabilities would not change — that's the Topic 3 case.)*

---

### Problem 4 — Law of total probability (NAT, sets up Bayes)
*Factory I makes $60\%$ of items with a $2\%$ defect rate; Factory II makes the remaining $40\%$ with a $5\%$ defect rate. A random item is picked. Find $P(\text{defective})$.*

Partition by source: $P(F_I)=0.6,\ P(F_{II})=0.4$, with $P(D\mid F_I)=0.02,\ P(D\mid F_{II})=0.05$.
$$P(D)=P(D\mid F_I)P(F_I)+P(D\mid F_{II})P(F_{II})=0.02(0.6)+0.05(0.4)=0.012+0.020=\boxed{0.032}.$$
*(The reverse question — "given defective, was it from Factory I?" — is **Bayes' theorem**, the very next topic.)*

---

### Problem 5 — The two-children conditioning trap (NAT / MSQ)
*A family has two children (each equally likely boy/girl, independent). (a) Given **at least one is a boy**, find $P(\text{both boys})$. (b) Given **the elder is a boy**, find $P(\text{both boys})$.*

Equally likely sample space (by birth order): $\{BB,\ BG,\ GB,\ GG\}$, each $\tfrac14$.

**(a)** Condition on "at least one boy" $=\{BB,BG,GB\}$ (probability $\tfrac34$). "Both boys" $=\{BB\}$.
$$P(\text{both}\mid\text{at least one boy})=\frac{1/4}{3/4}=\boxed{\tfrac13}.$$

**(b)** Condition on "elder is a boy" $=\{BB,BG\}$ (probability $\tfrac12$). "Both boys" $=\{BB\}$.
$$P(\text{both}\mid\text{elder boy})=\frac{1/4}{1/2}=\boxed{\tfrac12}.$$

**Trap defused:** the answers differ ($\tfrac13$ vs $\tfrac12$) purely because the *conditioning event* differs — "at least one boy" admits both $BG$ and $GB$, while "elder is a boy" rules out $GB$. The intuitive "it's just $\tfrac12$" is wrong in case (a). The conditioning information **must** be modelled exactly.

---

## 6. Common GATE question types on this topic

- **Compute a conditional probability** from a described experiment (dice, cards, digits) by restricting to the conditioning event.
- **Two-way contingency tables:** extract joint / marginal / conditional probabilities and **test independence** ($P(A\cap B)\overset{?}{=}P(A)P(B)$).
- **Sequential / without-replacement** experiments via the multiplication and chain rules; tree diagrams.
- **Law of total probability** for mixtures and multi-source problems (the LTP step that precedes Bayes).
- **Conditioning traps** ("at least one is a boy," sibling/coin variants) where the exact conditioning event determines the answer.
- **Check independence** using either $P(A\cap B)=P(A)P(B)$ or $P(A\mid B)=P(A)$.
- **Chain rule for three+ events**, e.g. probability of a specific sequence in a multi-stage process.

---

## 7. Traps, pitfalls & common mistakes

- **$P(A\mid B)$ vs $P(B\mid A)$.** Different denominators, generally unequal. Swapping them is the base-rate fallacy (fixed properly by Bayes, next topic).
- **$P(A\mid B)$ vs $P(A\cap B)$.** The conditional *divides by $P(B)$*; the joint does not. Don't report the joint when asked for the conditional.
- **Wrong denominator.** $P(A\mid B)=\dfrac{P(A\cap B)}{P(B)}$ — divide by the **conditioning** event $P(B)$, never by $P(A)$.
- **Forgetting to renormalise.** After conditioning, the sample space is $B$, not $S$; probabilities must sum to $1$ *within $B$*.
- **$P(A\mid B)+P(A\mid B^c)\ne 1$.** Only $P(A\mid B)+P(A^c\mid B)=1$ (complement on the left, same condition).
- **Conditioning on a probability-0 event** leaves $P(A\mid B)$ **undefined** — watch for $P(B)=0$.
- **LTP needs a genuine partition** — the $B_i$ must be mutually exclusive *and* exhaustive with positive probabilities; missing a case or overlapping cases breaks it.
- **Without replacement ⇒ condition.** Probabilities change after each draw; you must use $P(\text{2nd}\mid\text{1st})$, not the original fraction (that's the with-replacement/independent case).
- **The "at least one" conditioning subtlety.** "At least one boy" and "the elder is a boy" are different events and give different answers — model the wording precisely.
- **Assuming independence to factor a joint** when it wasn't given — use the multiplication rule with the *true* conditional, not $P(A)P(B)$.

---

## 8. Connection forward

We can now move freely between joint, marginal, and conditional probabilities, and we have the two rules that combine them: the **multiplication rule** $P(A\cap B)=P(A)P(B\mid A)$ and the **law of total probability** $P(A)=\sum_i P(A\mid B_i)P(B_i)$. The **next topic, Bayes' Theorem**, snaps these together to *invert* a conditional — turning $P(\text{evidence}\mid\text{cause})$ into $P(\text{cause}\mid\text{evidence})$ — which is the foundation of diagnostic reasoning, spam filtering, and naive-Bayes classification. Looking further, these same three notions (joint, marginal, conditional) reappear for **random variables** as joint/marginal/conditional **distributions** (Topics 6–9), where "sum the joints" becomes summing a joint PMF or integrating a joint PDF.

---

## 9. One-line cheat-sheet summary

**Joint $P(A\cap B)$ = "both"; marginal $P(A)=\sum_i P(A\cap B_i)$ = "sum the joints"; conditional $P(A\mid B)=\dfrac{P(A\cap B)}{P(B)}$ = "zoom into $B$ and renormalise"; multiplication $P(A\cap B)=P(A)P(B\mid A)$; total probability $P(A)=\sum_i P(A\mid B_i)P(B_i)$; independence ⟺ $P(A\mid B)=P(A)$; and $P(A\mid B)\ne P(B\mid A)$ in general.**

---

### ➡️ Next topic (in order): **Topic 5 — Bayes' Theorem.**
*Ask for it when you're ready, and I'll keep the same structure.*

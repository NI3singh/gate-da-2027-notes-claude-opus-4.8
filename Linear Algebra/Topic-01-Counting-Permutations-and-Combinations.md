# Topic 1 — Counting (Permutations and Combinations)

**Subject:** Probability and Statistics (GATE DA)
**Section:** Probability Foundations (Topic 1 of 16)

---

## 0. Topic header & syllabus map

- **Topic:** Counting — permutations and combinations (plus the counting principles, multiset arrangements, circular permutations, stars-and-bars, inclusion–exclusion, and pigeonhole).
- **Subject:** Probability and Statistics.
- **Where it sits:** This is the *very first* topic and the toolkit on which all of classical probability is built. It comes before everything else and leads directly into **Sample Space, Events & Probability Axioms**, where probability is defined as a ratio of two counts.

---

## 1. Why this topic exists (motivation + connection)

In its simplest, most-tested form, probability is

$$P(\text{event}) = \frac{\text{number of favorable outcomes}}{\text{number of total outcomes}}.$$

Both the numerator and denominator are **counts**. The catch: the sets you must count are usually far too large to list by hand. A standard 52-card deck yields $\binom{52}{5} = 2{,}598{,}960$ possible 5-card hands — you cannot enumerate those on exam paper. Counting gives you systematic *formulas* (multiplication rule, permutations, combinations) to find the size of a set without writing the set out.

- **What came before:** Nothing — this is the starting point. But the implicit problem it fixes is the failure of naive enumeration: as soon as a sample space has more than a handful of outcomes, listing them is impossible, so we need closed-form counting rules.
- **What comes next:** Sample space and the probability axioms. There, the "total outcomes" is the size of the sample space $|S|$ and "favorable outcomes" is $|E|$ — both produced by exactly the techniques in this topic. The **AND/OR** logic of independent vs. mutually exclusive events (next-next topic) is literally the multiplication and addition rules you learn here.

---

## 2. Prerequisites

- Factorial notation: $n! = n(n-1)(n-2)\cdots 2\cdot 1$, with the convention $0! = 1$.
- Basic algebra and arithmetic with fractions.
- Elementary set operations (union $\cup$, intersection $\cap$, complement) — needed for inclusion–exclusion.
- That's all. Counting is self-contained.

---

## 3. Core idea — intuition first

Every counting problem reduces to answering **two questions** about how you pick/arrange objects:

1. **Does order matter?**
   - *Yes* → **permutation** (arranging people in a row, forming a number, ranking).
   - *No* → **combination** (choosing a committee, picking a subset, dealing a hand).
2. **Is repetition allowed?**
   - Can the same object be reused (rolling a die twice can give two 6's) or not (dealing distinct cards)?

Once you classify a problem on these two axes, the right formula is fixed.

**The one principle behind everything — the multiplication rule.** Suppose you build an outcome in stages. If stage 1 has $m$ choices and stage 2 has $n$ choices *regardless of what you chose in stage 1*, then there are $m \times n$ combined outcomes.

> *Analogy.* You own 3 shirts and 2 pairs of trousers. Outfits $= 3 \times 2 = 6$. If you then line up 4 distinct trophies on a shelf, the first slot has 4 choices, the next 3, then 2, then 1: $4! = 24$ arrangements — that's a permutation. If instead you only want to *pick* 2 trophies to take home (order irrelevant), you'd divide out the orderings: a combination.

Permutations and combinations are not two unrelated ideas — **a combination is a permutation with the orderings removed:** $\binom{n}{r} = \dfrac{{}^{n}P_r}{r!}$. Hold onto that bridge; it explains the $r!$ in the denominator.

---

## 4. Formal treatment

### 4.1 The two fundamental counting principles

**Multiplication rule (AND — sequential, independent choices).**
If a procedure has $k$ stages, with $n_1$ choices at stage 1, $n_2$ at stage 2, …, $n_k$ at stage $k$, and the number of choices at each stage does not depend on the specific earlier choices, then the total number of outcomes is

$$n_1 \times n_2 \times \cdots \times n_k.$$

- **Condition:** the *count* at each stage must be fixed (it may follow from earlier stages, e.g. "one fewer left", but it cannot vary case-by-case). If it genuinely varies, split into cases and add.

**Addition rule (OR — mutually exclusive alternatives).**
If a task can be accomplished by method A ($m$ ways) **or** method B ($n$ ways), and the two methods share no common outcome, then there are $m + n$ ways.

- **Condition:** the cases must be **mutually exclusive** (disjoint). If they overlap, you must subtract the overlap (→ inclusion–exclusion, §4.7).

### 4.2 Permutations (order matters, no repetition)

Number of ways to arrange $r$ objects chosen from $n$ **distinct** objects:

$$\boxed{{}^{n}P_r = \frac{n!}{(n-r)!}}\qquad 0 \le r \le n.$$

- **Conditions:** objects are distinct; each used at most once; order matters.
- **Derivation (GATE expects this reasoning):** fill $r$ ordered slots. Slot 1: $n$ choices; slot 2: $n-1$; …; slot $r$: $n-r+1$. By the multiplication rule the product is $n(n-1)\cdots(n-r+1) = \dfrac{n!}{(n-r)!}$.
- **Special case $r=n$:** ${}^{n}P_n = n!$ — the number of ways to arrange all $n$ distinct objects.

### 4.3 Permutations with repetition allowed

If each of $r$ ordered positions can independently be any of $n$ objects (repetition allowed):

$$\boxed{n^{\,r}}$$

- **Example:** number of 3-letter strings over a 26-letter alphabet $= 26^3$; outcomes of rolling a die 4 times $= 6^4$.

### 4.4 Permutations of a multiset (objects not all distinct)

Arranging $n$ objects in a row where $n_1$ are alike of type 1, $n_2$ alike of type 2, …, $n_k$ alike of type $k$ (with $n_1 + n_2 + \cdots + n_k = n$):

$$\boxed{\frac{n!}{n_1!\, n_2! \cdots n_k!}}$$

- **Why divide:** start from $n!$ arrangements as if all distinct, then *quotient out* the $n_i!$ orderings among each block of identical items that produce no new distinguishable arrangement.
- **Classic example:** arrangements of the word `MISSISSIPPI` ($M{:}1, I{:}4, S{:}4, P{:}2$, total $11$): $\dfrac{11!}{1!\,4!\,4!\,2!} = 34650$.

### 4.5 Circular permutations

Arranging $n$ **distinct** objects around a circle, where only relative order matters (rotations are considered the same):

$$\boxed{(n-1)!}$$

- **Reason:** fixing one object kills the $n$ rotational duplicates ($n!/n = (n-1)!$).
- **Reflection symmetry (e.g., beads on a necklace, where clockwise = counterclockwise):**

$$\frac{(n-1)!}{2}\quad(n \ge 3).$$

### 4.6 Combinations (order does NOT matter, no repetition)

Number of ways to **choose** (select a subset of) $r$ objects from $n$ distinct objects:

$$\boxed{{}^{n}C_r = \binom{n}{r} = \frac{n!}{r!\,(n-r)!} = \frac{{}^{n}P_r}{r!}}\qquad 0 \le r \le n.$$

- **Conditions:** objects distinct; each chosen at most once; order irrelevant.
- **Bridge to permutations:** there are ${}^{n}P_r$ ordered selections; each *unordered* subset of size $r$ was counted $r!$ times (its $r!$ orderings), so divide by $r!$.

**Key identities (memorize — GATE uses them to simplify, not always to compute):**

| Identity | Statement |
|---|---|
| Symmetry | $\displaystyle\binom{n}{r} = \binom{n}{n-r}$ |
| Boundary | $\displaystyle\binom{n}{0} = \binom{n}{n} = 1,\quad \binom{n}{1} = n$ |
| Pascal's rule | $\displaystyle\binom{n}{r} = \binom{n-1}{r-1} + \binom{n-1}{r}$ |
| Total subsets | $\displaystyle\sum_{r=0}^{n}\binom{n}{r} = 2^{n}$ |
| Hockey-stick (less common) | $\displaystyle\sum_{i=r}^{n}\binom{i}{r} = \binom{n+1}{r+1}$ |

### 4.7 Combinations with repetition — "stars and bars"

Number of ways to choose $r$ objects from $n$ **types** with repetition allowed, order irrelevant (equivalently, distributing $r$ identical items into $n$ distinct bins):

$$\boxed{\binom{n+r-1}{r} = \binom{n+r-1}{n-1}}$$

This is the same as the number of **non-negative** integer solutions to

$$x_1 + x_2 + \cdots + x_n = r,\qquad x_i \ge 0,$$

which equals $\displaystyle\binom{n+r-1}{n-1}$.

- **Bijection (why it works):** represent a solution as $r$ "stars" split into $n$ groups by $n-1$ "bars". Each arrangement of $r$ stars and $n-1$ bars in a row is one solution; there are $\binom{n+r-1}{\,n-1\,}$ such arrangements.
- **Positive solutions** ($x_i \ge 1$): substitute $y_i = x_i - 1 \ge 0$, giving $y_1 + \cdots + y_n = r - n$, so the count is

$$\binom{r-1}{\,n-1\,}\qquad (r \ge n).$$

### 4.8 Binomial theorem (where the coefficients come from)

$$\boxed{(x+y)^{n} = \sum_{k=0}^{n}\binom{n}{k}\,x^{\,n-k}\,y^{k}}$$

- The coefficient of $x^{n-k}y^k$ is $\binom{n}{k}$ — hence the name *binomial coefficient*.
- Putting $x=y=1$ recovers $\sum_k \binom{n}{k} = 2^n$; putting $x=1, y=-1$ gives $\sum_k (-1)^k\binom{n}{k} = 0$.

### 4.9 Inclusion–Exclusion Principle (counting overlapping OR)

When the alternatives are **not** disjoint, the plain addition rule over-counts the overlaps; correct it by alternately subtracting and adding intersections.

$$|A \cup B| = |A| + |B| - |A \cap B|.$$

$$|A \cup B \cup C| = |A|+|B|+|C| - |A\cap B| - |A\cap C| - |B\cap C| + |A\cap B\cap C|.$$

General form for $n$ sets: add all single sets, subtract all pairwise intersections, add all triple intersections, etc., with sign $(-1)^{\text{(size of intersection)}+1}$.

**Derangements (a key application).** The number of permutations of $n$ objects with *no* element in its original position:

$$D_n = n!\sum_{k=0}^{n}\frac{(-1)^k}{k!}\ \approx\ \frac{n!}{e}.$$

Small values worth memorizing: $D_1=0,\ D_2=1,\ D_3=2,\ D_4=9,\ D_5=44$.

### 4.10 Pigeonhole Principle

If $n$ objects are placed into $m$ boxes and $n > m$, then at least one box contains **two or more** objects.

- **Generalized:** at least one box contains $\ge \left\lceil \dfrac{n}{m} \right\rceil$ objects.
- Used for existence-style claims ("show some two people share a property"), occasionally probed in GATE CS.

*Derivations deliberately skipped:* generating-function proofs of stars-and-bars, the full multinomial theorem expansion, and asymptotics of $\binom{n}{r}$ — GATE DA does not ask for these. We use the closed-form results only.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Multiset arrangement (NAT)

**Q.** How many distinct arrangements are there of the letters of the word **`STATISTICS`**?

**Solution.** Letter counts in `STATISTICS` (10 letters): $S{:}3,\ T{:}3,\ A{:}1,\ I{:}2,\ C{:}1$. Using the multiset permutation formula:

$$\frac{10!}{3!\,3!\,2!\,1!\,1!} = \frac{3{,}628{,}800}{6\cdot 6\cdot 2} = \frac{3{,}628{,}800}{72} = \mathbf{50400}.$$

**Answer: 50400.**

---

### Problem 2 — Stars and bars (NAT)

**Q.** Find the number of non-negative integer solutions of $x_1 + x_2 + x_3 + x_4 = 10$.

**Solution.** Non-negative solutions of $x_1+\cdots+x_n=r$ number $\binom{n+r-1}{n-1}$ with $n=4,\ r=10$:

$$\binom{4+10-1}{4-1} = \binom{13}{3} = \frac{13\cdot 12\cdot 11}{3!} = \frac{1716}{6} = \mathbf{286}.$$

**Answer: 286.**
*(Variant trap: if each $x_i \ge 1$ were required, you'd instead get $\binom{r-1}{n-1} = \binom{9}{3} = 84$.)*

---

### Problem 3 — Selection with an "at least" constraint (MCQ)

**Q.** From 7 men and 5 women, a committee of 5 must be formed with **at least 3 women**. In how many ways can it be done?

(A) 246 (B) 220 (C) 256 (D) 210

**Solution.** "At least 3 women" splits into disjoint cases (use the addition rule):

- 3W, 2M: $\binom{5}{3}\binom{7}{2} = 10 \cdot 21 = 210$
- 4W, 1M: $\binom{5}{4}\binom{7}{1} = 5 \cdot 7 = 35$
- 5W, 0M: $\binom{5}{5}\binom{7}{0} = 1 \cdot 1 = 1$

Total $= 210 + 35 + 1 = \mathbf{246}$. **Answer: (A).**

---

### Problem 4 — Inclusion–exclusion (NAT)

**Q.** How many integers from 1 to 100 (inclusive) are divisible by 2, 3, or 5?

**Solution.** Let $A,B,C$ be the sets divisible by 2, 3, 5.

$$|A| = \lfloor 100/2\rfloor = 50,\quad |B| = \lfloor100/3\rfloor = 33,\quad |C| = \lfloor100/5\rfloor = 20.$$
$$|A\cap B| = \lfloor100/6\rfloor = 16,\ |A\cap C| = \lfloor100/10\rfloor = 10,\ |B\cap C| = \lfloor100/15\rfloor = 6.$$
$$|A\cap B\cap C| = \lfloor100/30\rfloor = 3.$$

By inclusion–exclusion:

$$50+33+20 - 16-10-6 + 3 = \mathbf{74}.$$

**Answer: 74.**

---

### Problem 5 — Trap problem: derangement / "at least one" via complement (MCQ)

**Q.** Four letters are addressed to four different people and four envelopes are correctly addressed. The letters are inserted at random, one per envelope. In how many ways does **no** letter go into its correct envelope?

(A) 24 (B) 9 (C) 15 (D) 6

**Solution.** This is the number of derangements of 4 objects:

$$D_4 = 4!\left(\frac{1}{0!}-\frac{1}{1!}+\frac{1}{2!}-\frac{1}{3!}+\frac{1}{4!}\right) = 24\left(1 - 1 + \tfrac12 - \tfrac16 + \tfrac{1}{24}\right) = 24\cdot\tfrac{9}{24} = \mathbf{9}.$$

**Answer: (B).**

> **Where students lose marks:** they compute the total $4! = 24$ and forget to remove the cases where at least one letter is correct. The phrase "no letter in its correct place" signals a *derangement*, not a plain permutation. If the question instead asked "at least one letter correct," the answer would be $24 - 9 = 15$ (option C is the bait).

---

## 6. Common GATE question types on this topic

- **Classical probability via counting:** "probability that a random 5-card hand is a flush / a random arrangement satisfies a condition" — numerator and denominator are both counts.
- **Word-arrangement problems** with repeated letters (multiset permutations), sometimes with constraints (vowels together, no two alike adjacent).
- **Selection/committee problems** with "exactly $k$," "at least $k$," "at most $k$" constraints.
- **Integer-solution / distribution problems** solved by stars and bars (identical items into distinct bins).
- **"Divisible by … or …"** and "at least one of several properties" — inclusion–exclusion.
- **Circular arrangements** (people around a table, beads on a necklace).
- **Derangement / matching** problems framed as letters-in-envelopes, hats, gift-exchange.

---

## 7. Traps, pitfalls & common mistakes

- **Permutation vs combination mix-up.** Always ask "does order matter?" *first*. Ranking, seating, forming numbers → permutation; choosing a subset/committee/hand → combination.
- **Forgetting the repeated-object division.** In word problems with repeated letters you must divide by each $n_i!$; using $n!$ alone over-counts.
- **"At least one" handled the hard way.** Prefer the complement: $\text{(at least one)} = \text{total} - \text{(none)}$. Summing many "exactly $k$" cases is error-prone.
- **Adding overlapping cases.** The addition rule needs disjoint cases. If cases overlap, use inclusion–exclusion or you will double-count.
- **Circular slip-ups.** Forgetting it's $(n-1)!$, not $n!$; and forgetting the extra $/2$ when reflections coincide (necklaces, bracelets).
- **Stars-and-bars: non-negative vs positive.** Non-negative solutions $=\binom{n+r-1}{n-1}$; strictly positive $=\binom{r-1}{n-1}$. Confusing them is the single most common stars-and-bars error.
- **Dependent stages treated as independent.** The multiplication rule needs a *fixed count* at each stage. If choosing one object changes how many are valid for the next in a case-dependent way, split into cases.
- **Boundary cases.** $0! = 1$, $\binom{n}{0} = 1$, ${}^{n}P_0 = 1$. Forgetting these breaks "choose 0" / "empty selection" sub-cases.
- **Over-counting indistinguishable selections** by treating an unordered choice as ordered (the missing $\div r!$).

---

## 8. Connection forward

The very next topic — **Sample Space, Events & Probability Axioms** — defines the classical probability of an event $E$ as $P(E) = |E| / |S|$, a ratio of two counts produced entirely by the methods above. Right after that, **independent vs mutually exclusive events** formalize exactly the AND (multiplication) and OR (addition) rules you used here. In short: counting is not a one-off chapter — it is the arithmetic engine running underneath every probability you will compute for the rest of this subject.

---

## 9. One-line cheat-sheet summary

**Order matters → permutation $\left({}^{n}P_r=\tfrac{n!}{(n-r)!}\right)$; order doesn't → combination $\left(\binom{n}{r}=\tfrac{n!}{r!(n-r)!}\right)$; repetition-allowed selections → stars & bars $\binom{n+r-1}{r}$; repeated letters → divide by each $n_i!$; circle → $(n-1)!$; "at least one" → use the complement; overlapping OR → inclusion–exclusion.**

---

### ➡️ Next topic (in order): **Sample Space, Events & Probability Axioms** — where counting becomes probability via $P(E) = |E|/|S|$.

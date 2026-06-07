# Topic 5 — Tuple Relational Calculus (TRC)
**Subject:** Database Management and Warehousing
**Section:** Querying (Chat 2)

---

## 0. Topic header & syllabus map

- **Topic:** Tuple Relational Calculus (TRC)
- **Subject:** Database Management and Warehousing → *Querying* cluster.
- **Where it sits:** This is the **second querying topic**. It comes immediately after *Relational Algebra* and is its **declarative twin**: RA tells the system *how* to compute an answer (a sequence of operators), TRC tells it *what* the answer is (a logical description). By **Codd's theorem** the two are **equally expressive** (for *safe* expressions). It leads directly into *SQL* (Topic 6), whose `SELECT–FROM–WHERE` and `EXISTS`/`NOT EXISTS` constructs are essentially TRC written in keywords.

---

## 1. Why this topic exists (motivation + connection)

Relational algebra answered "how do we extract data?" — but it forces you to **spell out the procedure**: first do this join, then this selection, then this projection. Two different operator sequences can compute the **same** answer, and the user has to *choose* one. That is more than a user should have to care about: ideally you state *what you want* and let the database decide *how* to get it.

The previous topic (RA) could not do this — it is inherently **procedural**, so a query *is* a recipe, not a description. Tuple relational calculus fixes exactly this: a query becomes a **logical predicate** — "the set of all tuples `t` such that `P(t)` is true." You declare the *property* of the answer tuples; the engine figures out the execution. This is the **declarative** ideal.

This matters for two reasons. First, it is the **theoretical yardstick** for "relational completeness": a query language is judged powerful enough if it can express everything safe TRC can. Second, it is the **conceptual ancestor of SQL** — SQL is far closer in spirit to calculus (describe the result) than to algebra (list the operators), and SQL's correlated subqueries are a direct realization of TRC's quantifiers.

---

## 2. Prerequisites

- **Relational model:** relation, tuple, attribute, schema, domain (TRC quantifies over tuples of relations).
- **Relational algebra (Topic 4):** needed to appreciate the *procedural vs. declarative* contrast and to translate between RA and TRC.
- **First-order (predicate) logic:** connectives ∧, ∨, ¬, ⇒; quantifiers ∃ ("there exists") and ∀ ("for all"); free vs. bound variables; the quantifier-negation (De Morgan) laws. **This is the single most important prerequisite** — TRC *is* first-order logic over relations.
- **Comparison operators:** =, ≠, <, >, ≤, ≥.

---

## 3. Core idea — intuition first

In **relational algebra** you write a *recipe*: "join Student with Enrolled, select where dept = 'CSE', project the name." In **tuple relational calculus** you instead write a *description of the answer*:

> "Give me every tuple `t` such that `t` is a student **and** `t` is enrolled in some course **and** `t`'s department is CSE."

You never say *how* to compute it — you only state the **logical condition** the answer tuples must satisfy. The system is free to evaluate it any way it likes.

The general shape of every TRC query is:

$$\{\, t \mid P(t) \,\}$$

read as **"the set of all tuples `t` for which the predicate `P(t)` is true."** Here:
- `t` is a **tuple variable** — it ranges over tuples.
- `t[A]` means **"the value of attribute `A` in tuple `t`."**
- `P(t)` is a **formula** built from logic: membership tests like `Student(t)` ("`t` is in relation Student"), comparisons like `t[CGPA] > 8`, glued together with ∧, ∨, ¬, ⇒ and quantified with ∃, ∀.

Three intuitions to carry forward:
1. **∃ = "there is some matching tuple"** (the everyday workhorse, used for joins and "exists" conditions).
2. **∀ = "every tuple satisfies"** — the way TRC expresses **"for all"** queries (the analogue of relational-algebra **division**).
3. **Safety:** because a tuple variable could in principle range over *infinitely many* possible tuples, you must write predicates that **pin the answer down to finitely many real values** — otherwise the query is meaningless. (More in §4.7; this is the #1 GATE concept on this topic.)

---

## 4. Formal treatment

Let relations be referenced by name (e.g. `R`, `Student`). Notation `R(t)` means *"tuple `t` belongs to relation `R`"*; GATE also writes this as `t ∈ R`. Both are used below.

### 4.1 General form of a TRC query

$$\{\, t \mid P(t) \,\}$$

- Result = the set of all tuples `t` that make the formula `P(t)` **true**.
- The output relation's **schema** is determined by the attributes referenced for the (single) **free** variable `t`.

---

### 4.2 Tuple variables and attribute access

- A **tuple variable** ranges over the tuples of (some) relation.
- `t[A]` = value of attribute `A` of `t`. `t[A, B]` denotes the pair `(t[A], t[B])`.
- Constants are written literally (`'CSE'`, `8.0`, `2026`).

---

### 4.3 Atoms (the three atomic predicates)

A formula is built from **atoms**. There are exactly three kinds:

| # | Atom | Meaning |
|---|------|---------|
| 1 | `R(s)`  (a.k.a. `s ∈ R`) | tuple variable `s` is a tuple of relation `R` |
| 2 | `s[x] Θ u[y]` | compare attribute `x` of `s` with attribute `y` of `u` |
| 3 | `s[x] Θ c` | compare attribute `x` of `s` with a constant `c` |

where `Θ ∈ {=, ≠, <, >, ≤, ≥}` is a comparison operator.

---

### 4.4 Formulas (well-formed formula construction rules)

**Formation rules** (a formula is anything built by finitely many applications of these):

```
1. Every atom is a formula.
2. If P1 is a formula, then  ¬P1  and  (P1)  are formulas.
3. If P1, P2 are formulas, then  P1 ∧ P2 ,  P1 ∨ P2 ,  P1 ⇒ P2  are formulas.
4. If P1(s) is a formula with free tuple variable s, then
       ∃s ( P1(s) )      (“there exists a tuple s such that P1”)
       ∀s ( P1(s) )      (“for all tuples s, P1 holds”)
   are formulas.
```

**Relativized (range-restricted) quantifier shorthand** — GATE uses this constantly:

$$\exists\, s \in R\ (Q) \;\equiv\; \exists s\,(\,R(s) \wedge Q\,) \qquad\qquad \forall\, s \in R\ (Q) \;\equiv\; \forall s\,(\,R(s) \Rightarrow Q\,)$$

> ⚠️ **Note the pairing:** `∃` pairs with **`∧`**, but `∀` pairs with **`⇒`**. (Using `∧` with `∀`, or `⇒` with `∃`, is a classic error — see §7.)

---

### 4.5 Free vs. bound variables

- A tuple variable is **bound** if it is introduced by a quantifier (`∃s` or `∀s`); otherwise it is **free**.
- A correct query `{ t | P(t) }` must have **exactly one free variable, `t`** (the output tuple). Every other variable inside `P` must be **bound** by some quantifier.

*Example.* In
`{ t | Supplier(t) ∧ ∃c ( Catalog(c) ∧ c[sid] = t[sid] ) }`
`t` is **free** (the answer), `c` is **bound** by `∃`.

---

### 4.6 Quantifier semantics & the logic laws you must know

**Meaning.**
- `∃s (P(s))` is true iff **at least one** tuple `s` makes `P(s)` true.
- `∀s (P(s))` is true iff **every** tuple `s` makes `P(s)` true.

**Equivalence laws (used to rewrite/match expressions — heavily tested):**

$$\neg \forall s\, P(s) \;\equiv\; \exists s\, \neg P(s) \qquad\qquad \neg \exists s\, P(s) \;\equiv\; \forall s\, \neg P(s)$$
$$\forall s\, P(s) \;\equiv\; \neg\,\exists s\, \neg P(s) \qquad\qquad \exists s\, P(s) \;\equiv\; \neg\,\forall s\, \neg P(s)$$
$$A \Rightarrow B \;\equiv\; \neg A \vee B$$

These three lines are the engine behind every **"for all"** TRC query (and behind SQL's `NOT EXISTS … NOT EXISTS` pattern). In particular, the universal "supplier supplies **all** parts" can always be rewritten with **no `∀` at all**, using only `¬∃ … ¬∃`.

---

### 4.7 Safety of expressions ⭐ (the central GATE concept)

A tuple variable could, in principle, range over **all conceivable tuples** — an *infinite* universe. So a careless formula can describe an **infinite** (hence uncomputable) result. Such an expression is **unsafe**. We only allow **safe** expressions.

**Domain of a formula `P`, `dom(P)`:** the (finite) set of all values that *either* appear as **explicit constants** in `P`, *or* appear **inside some tuple of a relation named in `P`** (its active domain).

**Definition (safe expression).** `{ t | P(t) }` is **safe** if:
1. **Every value in every result tuple comes from `dom(P)`** (no value is "invented" from outside).
2. For each subformula `∃s (P₁(s))`: it is true **iff** there is a tuple `s`, all of whose values lie in `dom(P₁)`, making `P₁(s)` true.
3. For each subformula `∀s (P₁(s))`: it is true **iff** `P₁(s)` holds for **all** tuples `s` whose values lie in `dom(P₁)` (i.e. the `∀` is restricted to the finite domain).

**Practical test (what GATE actually checks):** a result is finite — *safe* — only if **every attribute of the free/output variable is forced to take a value from some relation**, i.e. the output tuple is "**guarded**" by a positive membership atom. **Unrestricted negation** is the usual culprit for unsafety.

| Expression | Safe? | Why |
|---|---|---|
| `{ t \| R(t) ∧ t[A] > 5 }` | ✅ safe | `t` is in `R`; values come from `R` |
| `{ t \| R(t) ∨ S(t) }` | ✅ safe | `t ∈ R` or `t ∈ S` (`= R ∪ S`) |
| `{ t \| R(t) ∧ ¬S(t) }` | ✅ safe | `t` guarded by `R(t)`; negation only filters (`= R − S`) |
| `{ t \| ¬R(t) }` | ❌ **unsafe** | infinitely many tuples are *not* in `R` |
| `{ t \| t[A] = t[A] }` | ❌ **unsafe** | tautology ⇒ the entire infinite universe |
| `{ t \| ∃s ( S(s) ∧ ¬R(t) ) }` | ❌ **unsafe** | `t` never positively bound to a relation |

**Why it matters (the bridge to RA):** see §4.8 — **safe TRC = relational algebra**; *unsafe* TRC has **no** RA equivalent.

---

### 4.8 Equivalence with Relational Algebra — Codd's theorem

**Codd's theorem.** Relational algebra, **safe** tuple relational calculus, and **safe** domain relational calculus are **equivalent in expressive power**. A query language able to express everything in this common set is called **relationally complete**.

Consequences GATE likes:
- Any query you can write in RA, you can write in safe TRC, and vice-versa.
- The restriction to **safe** is essential: RA can never produce an infinite relation, so it cannot match an *unsafe* calculus expression like `{t | ¬R(t)}`.

**Quick RA ↔ TRC dictionary:**

| Relational Algebra | Tuple Relational Calculus |
|---|---|
| `σ_{cond}(R)` | `{ t \| R(t) ∧ cond(t) }` |
| `π_{A}(R)` | `{ t \| ∃r ( R(r) ∧ t[A] = r[A] ) }` (t has just attribute A) |
| `R ∪ S` | `{ t \| R(t) ∨ S(t) }` |
| `R − S` | `{ t \| R(t) ∧ ¬S(t) }` |
| `R ⋈ S` (common attr B) | `{ t \| ∃r∃s ( R(r) ∧ S(s) ∧ r[B]=s[B] ∧ (t = merged r,s) ) }` |
| `R ÷ S` ("for all") | the `∀`-form / `¬∃…¬∃`-form (see Problem 2) |

---

### 4.9 Contrast: Domain Relational Calculus (DRC) — know the difference

DRC is the close cousin. Instead of a *tuple* variable, it uses one **domain variable per attribute**:

$$\{\, \langle x_1, x_2, \dots, x_n \rangle \mid P(x_1, \dots, x_n) \,\}$$

Example — names of CSE students: `{ ⟨n⟩ | ∃a∃d ( Student(n, a, d) ∧ d = 'CSE' ) }` (here `Student(n,a,d)` means a tuple with those component values exists).

- **Same expressive power** as safe TRC and RA (Codd's theorem covers both calculi).
- **Difference to remember:** TRC variables range over **whole tuples** (`t`, accessed via `t[A]`); DRC variables range over **individual attribute values** (`x₁,…,xₙ`). QBE (Query-By-Example) is based on DRC. The official syllabus item is **tuple** calculus, so the rest of these notes stay with TRC.

---

### 4.10 Expressiveness limits (one line, but worth knowing)

RA and (safe) relational calculus **cannot express recursive / transitive-closure queries** (e.g. "find *all* ancestors", "is node Y reachable from X?") and have **no built-in aggregation**. These are *fundamental* limits, not oversights — they motivate the extra machinery in SQL (aggregate functions, and the later `WITH RECURSIVE`).

---

## 5. Worked problems (GATE-style)

Common schema for Problems 1–2 (supplier–parts, mirrors RA **division**):
`Supplier(sid, sname)`, `Part(pid, pname)`, `Catalog(sid, pid)` (which supplier supplies which part).

### Problem 1 — English → TRC (existential / join) (MCQ)

*"Find the names of suppliers who supply at least one part with `pname = 'Bolt'`."*

**Solution.** "At least one" ⇒ use `∃`. Output tuple `t` carries only `sname`.

$$\{\, t \mid \exists s\,\exists c\,\exists p\ \big(\, \text{Supplier}(s) \wedge \text{Catalog}(c) \wedge \text{Part}(p)\ \wedge\ c[sid]=s[sid]\ \wedge\ c[pid]=p[pid]\ \wedge\ p[pname]=\text{'Bolt'}\ \wedge\ t[sname]=s[sname]\,\big)\,\}$$

*Reading it back:* there exist a supplier `s`, a catalog row `c`, and a part `p` that link up (same `sid`, same `pid`), the part is a Bolt, and the answer's `sname` equals `s`'s name. **Safe**, because `t[sname]` is forced to come from `Supplier`.

---

### Problem 2 — "For all" query: the division analogue, two equivalent forms (MCQ / MSQ)

*"Find the names of suppliers who supply **every** part."* (This is exactly RA's `R ÷ S`.)

**Form A — using `∀` and `⇒`:**

$$\{\, t \mid \exists s\,\big(\, \text{Supplier}(s)\ \wedge\ t[sname]=s[sname]\ \wedge\ \forall p\,(\, \text{Part}(p) \Rightarrow \exists c\,(\, \text{Catalog}(c) \wedge c[sid]=s[sid] \wedge c[pid]=p[pid] \,) \,) \,) \,\}$$

*Reading:* supplier `s` such that **for every** part `p`, **there is** a catalog row linking `s` to `p`.

**Form B — equivalent, using only `¬∃ … ¬∃` (no `∀`):** apply `∀p(Part(p) ⇒ Q) ≡ ¬∃p(Part(p) ∧ ¬Q)`:

$$\{\, t \mid \exists s\,\big(\, \text{Supplier}(s)\ \wedge\ t[sname]=s[sname]\ \wedge\ \neg\exists p\,(\, \text{Part}(p) \wedge \neg\exists c\,(\, \text{Catalog}(c) \wedge c[sid]=s[sid] \wedge c[pid]=p[pid] \,) \,) \,) \,\}$$

*Reading:* supplier `s` for whom **there is no part** `p` that `s` does **not** supply. This is the exact template SQL writes as `NOT EXISTS (… NOT EXISTS …)`. Both forms are **safe** and return the identical relation; recognising their equivalence is a frequent exam target.

---

### Problem 3 — Classify expressions as safe or unsafe (MSQ)

Relations `Loan(lno, amt)`, `Borrower(cust, lno)`. Which expressions are **safe**?

(a) `{ t | Loan(t) ∧ t[amt] > 10000 }`
(b) `{ t | ¬Loan(t) }`
(c) `{ t | ∃b ( Borrower(b) ∧ b[lno] = t[lno] ) }`
(d) `{ t | Loan(t) ∧ ¬∃b ( Borrower(b) ∧ b[lno] = t[lno] ) }`

**Solution.**
- **(a) Safe.** `t` guarded by `Loan(t)`; the comparison only filters. (= `σ_{amt>10000}(Loan)`.)
- **(b) Unsafe.** The complement of `Loan` is infinite — no positive guard on `t`.
- **(c) Safe** — *for the single attribute `lno`*: `t[lno]` is forced to equal some `b[lno]` from `Borrower`, so it draws only from `dom`. (It computes the `lno` values appearing in `Borrower`. If the result schema demanded attributes never bound to a relation, it would be unsafe — but here only `lno` is referenced, and it is bound.)
- **(d) Safe.** `t` is guarded by `Loan(t)`; the `¬∃` only *removes* matching tuples. (= loans with no borrower = an **anti-join**, RA: `Loan − π_{lno,amt}(Loan ⋈ Borrower)`-style.)

**Answer: (a), (c), (d).** The lone unsafe one is **(b)** — the unguarded negation.

---

### Problem 4 — "What does this expression compute?" (read TRC → RA / English)

Relations `R(A, B)` and `S(B, C)`. Identify the output of:

$$\{\, t \mid \exists r\,\exists s\,\big(\, R(r) \wedge S(s) \wedge r[B]=s[B] \wedge t[A]=r[A] \wedge t[C]=s[C] \,\big) \,\}$$

**Solution.** It pairs an `R`-tuple `r` with an `S`-tuple `s` that **agree on `B`** (a join on `B`), then keeps `r`'s `A` and `s`'s `C`. So the result has schema `(A, C)` and equals:

$$\pi_{A,\,C}\big(R \bowtie S\big)\quad(\text{natural join on } B).$$

**Variant with negation (anti-join).** What does this compute?
$$\{\, t \mid R(t) \wedge \neg\exists s\,(\, S(s) \wedge s[B]=t[B] \,) \,\}$$
*Answer:* the tuples of `R` whose `B`-value **does not appear** in `S` — an **anti-join** (`R` minus everything that matches `S` on `B`). Negation (`¬∃`) is precisely the calculus tool for "**does not match**".

---

### Problem 5 — Trap problem: the `∀` vs `∃`, and `⇒` vs `∧` mistake (MCQ)

Schema `Student(sid)`, `Course(cid)`, `Took(sid, cid)`. A student wrote the following to mean *"students who took **all** courses"*:

$$\{\, t \mid \text{Student}(t) \wedge \forall c\,\big(\, \text{Course}(c) \wedge \exists k\,(\,\text{Took}(k) \wedge k[sid]=t[sid] \wedge k[cid]=c[cid]\,) \,\big) \,\}$$

What does this expression *actually* compute, and what is the fix?

**Solution.** The bug is using **`∧` with `∀`**. The subformula `∀c ( Course(c) ∧ … )` is true only if **every tuple `c` in the entire universe** is *both* a course *and* taken — which is **never** true (there exist tuples that aren't courses). So as written the result is **empty** (`∅`) regardless of data.

**Fix:** a relativized `∀` must use **`⇒`**, not `∧`:

$$\{\, t \mid \text{Student}(t) \wedge \forall c\,\big(\, \text{Course}(c) \Rightarrow \exists k\,(\,\text{Took}(k) \wedge k[sid]=t[sid] \wedge k[cid]=c[cid]\,) \,\big) \,\}$$

Now it reads "for every `c`, **if** `c` is a course **then** the student took it" — the intended "all courses." (Symmetrically, a relativized `∃` must use `∧`, not `⇒`: writing `∃c(Course(c) ⇒ …)` is trivially satisfiable and also wrong.)

---

## 6. Common GATE question types on this topic

- **Translate an English query into a TRC expression** (and the reverse — read a TRC expression and pick the matching English).
- **"For all" queries** — express "supplies *all* parts / takes *all* courses" using `∀…⇒` or the equivalent `¬∃…¬∃` form, and recognise the two as equivalent.
- **Safe vs. unsafe** classification — the single most common pure-TRC question; spot the unguarded negation.
- **Match a TRC expression to its relational-algebra equivalent** (uses the §4.8 dictionary).
- **Quantifier / logic rewriting** — apply `¬∀ ≡ ∃¬`, `¬∃ ≡ ∀¬`, `A⇒B ≡ ¬A∨B`.
- **Free vs. bound variable** identification; well-formedness of a query.
- **Conceptual:** Codd's theorem / relational completeness; TRC vs. DRC distinction; why unsafe expressions have no RA equivalent; what RA/calculus *cannot* express (recursion, aggregation).

---

## 7. Traps, pitfalls & common mistakes

1. **`∃` pairs with `∧`; `∀` pairs with `⇒`.** Relativizing a quantifier the wrong way is the #1 error: `∀c(Course(c) ∧ …)` collapses to **empty**; `∃c(Course(c) ⇒ …)` is **trivially true**. (See Problem 5.)
2. **"All" needs `∀` (or `¬∃…¬∃`), not `∃`.** Writing `∃p(Part(p) ∧ …)` says "**some** part," never "**all** parts." Universal queries are exactly where students slip.
3. **Unguarded negation ⇒ unsafe.** `{t | ¬R(t)}`, or any output variable bound *only* by a negative/comparison condition, gives an infinite result. Every output attribute must trace back to **membership in some relation**.
4. **Negation alone is fine *only* when the variable is positively guarded.** `{t | R(t) ∧ ¬S(t)}` is safe; `{t | ¬S(t)}` is not. The guard `R(t)` is what saves it.
5. **Exactly one free variable.** The output variable `t` is free; everything else must be quantified. A stray free variable makes the query ill-formed (or changes its meaning).
6. **`t[A]` is a *value*, not a tuple.** Compare values to values (`t[A] = s[B]`) or to constants — don't compare a value to a whole tuple.
7. **TRC uses *set* semantics** (like RA): the result is a set, duplicates collapse. (SQL, next topic, is bag-based — a deliberate contrast to watch for.)
8. **Codd's theorem needs *safe*.** Don't claim "TRC = RA" without the safety qualifier; unsafe TRC strictly exceeds RA (it can be infinite, which RA can't be).
9. **TRC vs. DRC mix-up.** Tuple variables range over **tuples** (`t[A]`); domain variables range over **values** (`⟨x₁,…,xₙ⟩`). Don't write `t[A]` in a DRC query or bare domain variables in a TRC query.
10. **Don't expect recursion or aggregation.** "Find all ancestors", transitive closure, `COUNT`/`SUM` — **not expressible** in TRC/RA. If a question hinges on these, the point being tested is that they're *out of scope*.
11. **`⇒` is not symmetric.** `A ⇒ B` ≢ `B ⇒ A`. Mixing the implication direction (especially after rewriting `∀`) silently changes the query.

---

## 8. Connection forward

TRC reframed querying from *procedure* (RA) to *description* (logic), and Codd's theorem guarantees the two describe the **same** class of queries. **SQL (Topic 6)** is the practical embodiment of this declarative view: its `SELECT–FROM–WHERE` is "describe the result, not the steps," its `WHERE` predicate is the TRC formula, its `SELECT` list is the free variable's attributes, and its **correlated subqueries** `EXISTS` / `NOT EXISTS` are literally TRC's `∃` and the `¬∃…¬∃` "for all" pattern. The places where SQL goes **beyond** TRC — `GROUP BY` aggregation and `WITH RECURSIVE` — are precisely the §4.10 limits of pure calculus. So this topic is the logical lens through which SQL's design makes sense.

---

## 9. One-line cheat-sheet summary

**Tuple relational calculus = the *declarative* query language `{ t | P(t) }` over tuple variables in first-order logic (`∃` with `∧`, `∀` with `⇒`); a query is valid only if *safe* (every output value guarded into a finite domain, so no unrestricted `¬`), and by *Codd's theorem* safe TRC = relational algebra in power — with "for all" expressed as `∀…⇒` ≡ `¬∃…¬∃`.**

---

### ▶ Next topic (in order)
**Topic 6 — SQL:** the practical, declarative language that realizes tuple-calculus ideas (`SELECT–FROM–WHERE`, `EXISTS`/`NOT EXISTS` for quantifiers) and extends them with aggregation and grouping.

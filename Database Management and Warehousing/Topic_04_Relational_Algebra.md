# Topic 4 — Relational Algebra
**Subject:** Database Management and Warehousing
**Section:** Querying (Chat 2)

---

## 0. Topic header & syllabus map

- **Topic:** Relational Algebra (RA)
- **Subject:** Database Management and Warehousing → *Querying* cluster.
- **Where it sits:** This is the **first querying topic**. It comes right after the *relational model* (which gave us the table structure) and *integrity constraints* (which kept the data valid). It is the **formal, procedural foundation** for everything that follows. The very next topic, *Tuple Relational Calculus*, is its declarative twin; *SQL* (Topic 6) is the practical language that implements ideas from both.

---

## 1. Why this topic exists (motivation + connection)

The relational model handed us a clean way to **store** facts as relations (tables of tuples), and integrity constraints made sure those facts stayed consistent (keys unique, foreign keys valid, etc.). But storing data is useless if you cannot **ask questions of it**: "Which students scored above 80?", "Which employees work in departments located in Delhi?", "Which suppliers supply *every* part?"

The previous topics had **no mechanism to extract or combine information** — they describe what the data *is*, not how to *retrieve* it. Relational algebra fixes exactly this gap. It is a **closed set of operators** that take one or two relations as input and always produce another relation as output. Because the output is again a relation, operators can be **chained**, letting us build arbitrarily complex queries from a handful of primitives.

RA is **procedural**: a query is an explicit *sequence of operations* ("do this, then this"). This makes it the natural bridge from "tables sitting in storage" to "answers." It is also the language in which **query optimizers reason** (they rewrite RA expressions into equivalent cheaper ones), so it underpins both query *meaning* and query *performance*.

---

## 2. Prerequisites

- **Relational model:** relation, tuple, attribute, schema, degree (arity = #attributes), cardinality (#tuples), domain.
- **Set theory:** union, intersection, difference, Cartesian product, subset (⊆), set membership (∈) — RA is built on these.
- **Keys & integrity constraints:** primary key, candidate key, foreign key, referential integrity — needed to reason about how many tuples a join produces.
- **Propositional logic:** AND (∧), OR (∨), NOT (¬), comparison operators (=, ≠, <, >, ≤, ≥) — used inside selection conditions.

---

## 3. Core idea — intuition first

Think of relational algebra as **"algebra, but the values are tables instead of numbers."**

In ordinary algebra you have numbers and operators (+, −, ×) that combine numbers to produce numbers. In relational algebra you have **relations** and operators (σ, π, ∪, −, ×, …) that combine relations to produce **relations**. This single fact — *operations on relations yield relations* — is called the **closure property**, and it is what lets you nest operations: the output of one becomes the input of the next.

A tiny mental model with a `Student(Roll, Name, Dept, CGPA)` table:

- **Selection σ** = a *horizontal* filter: "keep the **rows** where CGPA > 8."
- **Projection π** = a *vertical* filter: "keep only the **columns** Name and Dept."
- **Cartesian product ×** = "pair *every* row of one table with *every* row of another."
- **Join ⋈** = "× but only keep the row-pairs that *agree* on the common columns" — i.e. *meaningfully* combine two tables.
- **Set ops (∪, ∩, −)** = treat two same-shaped tables as sets of rows and combine them.
- **Division ÷** = the "**for all**" operator: "find the students who took **every** course in this list."

Two cultural points that matter for GATE:
1. **RA uses pure set semantics — duplicate tuples are automatically removed.** (SQL, by contrast, keeps duplicates unless you say `DISTINCT`. This mismatch is a favourite trap — see §7.)
2. Every operator's result is a *new* relation; the inputs are never modified.

---

## 4. Formal treatment

Throughout, let `R` and `S` be relations. `|R|` = cardinality (number of tuples), `degree(R)` = number of attributes. `schema(R)` = set of attribute names.

### 4.1 The six fundamental (primitive) operators

These are the standard "basic" operators (Silberschatz set). Every other operator can be expressed using them.

---

**(1) Selection — σ (sigma)**

$$\sigma_{\text{condition}}(R)$$

- Returns the tuples of `R` that **satisfy** the boolean *condition* (also called the *predicate*). A horizontal subset.
- The condition uses attribute names, constants, comparison operators, and ∧, ∨, ¬.
- **Schema unchanged:** `degree(σ(R)) = degree(R)`.
- **Cardinality:** `0 ≤ |σ(R)| ≤ |R|`.
- *Set-semantics note:* selection never introduces duplicates (and the input is already a set), so no dedup needed.

**Key equivalences (used in optimization):**
- Cascade: `σ_{c1 ∧ c2}(R) = σ_{c1}(σ_{c2}(R))`
- Commutative: `σ_{c1}(σ_{c2}(R)) = σ_{c2}(σ_{c1}(R))`

---

**(2) Projection — π (pi)**

$$\pi_{A_1, A_2, \dots, A_k}(R)$$

- Returns only the listed **columns** `A_1…A_k`. A vertical subset.
- **Duplicates are removed** (set semantics) — this is why cardinality can *shrink*.
- **Schema:** `degree(π_L(R)) = k = |L|` (the size of the projection list).
- **Cardinality:** `0 ≤ |π_L(R)| ≤ |R|`; and if `|R| ≥ 1`, then `1 ≤ |π_L(R)| ≤ |R|`.
  (It equals `|R|` exactly when the projected columns are still unique across all rows, e.g. if `L` contains a key.)

**Equivalence:** `π_{L1}(π_{L2}(R)) = π_{L1}(R)` provided `L1 ⊆ L2`.

---

**(3) Union — ∪**

$$R \cup S = \{\, t : t \in R \ \text{or}\ t \in S \,\}$$

- **Requires union-compatibility:** `R` and `S` must have the **same degree** *and* corresponding attributes must be over **compatible domains**. (Attribute *names* need not match, but the column order/types must align.)
- **Duplicates removed.**
- **Cardinality:** `max(|R|, |S|) ≤ |R ∪ S| ≤ |R| + |S|`.
  - Lower bound: one is a subset of the other.
  - Upper bound: completely disjoint.

---

**(4) Set difference — − (minus)**

$$R - S = \{\, t : t \in R \ \text{and}\ t \notin S \,\}$$

- **Requires union-compatibility.**
- **NOT commutative:** `R − S ≠ S − R` in general.
- **Cardinality:** `max(0,\ |R| - |S|) \le |R - S| \le |R|`.
  - Upper bound `|R|`: `R` and `S` are disjoint (nothing removed).
  - Lower bound: as many of `R`'s tuples as possible appear in `S`.

---

**(5) Cartesian product — × (cross product)**

$$R \times S = \{\, (t, u) : t \in R,\ u \in S \,\}$$

- Pairs **every** tuple of `R` with **every** tuple of `S`.
- **Schema:** `degree(R × S) = degree(R) + degree(S)`. If both have an attribute with the same name, they are **qualified** as `R.A` and `S.A`.
- **Cardinality is exact:** `|R × S| = |R| \cdot |S|`. (Only operator with an *exact* product formula — great for counting questions.)
- On its own × is rarely the *final* answer (it produces meaningless pairings); it is the building block for joins via `σ(R × S)`.

---

**(6) Rename — ρ (rho)**

$$\rho_{x}(R) \quad\text{or}\quad \rho_{x(A_1, \dots, A_n)}(R)$$

- Gives the relation (and optionally its attributes) **new names**. `degree` and `|R|` unchanged.
- **Why it's fundamental:** it is the only way to **compare a relation with itself** (self-join / "find pairs" queries). To find pairs of employees in the same department you need two *differently named* copies of `Employee`, e.g. `ρ_{E1}(Employee)` and `ρ_{E2}(Employee)`, then join `E1` with `E2`.
- *Note:* Some textbooks count only **5** fundamental operators (dropping ρ, since it adds no new "computational" power); the 6-operator list is the standard GATE convention because rename is genuinely needed for self-comparisons.

> **Memory hook for the fundamentals:** **σ π ∪ − × ρ** → *"Select, Project, Union, Minus, Cross, Rename."*

---

### 4.2 Derived operators (defined from the fundamentals)

**(7) Intersection — ∩**

$$R \cap S = \{\, t : t \in R \ \text{and}\ t \in S \,\}$$

- **Requires union-compatibility.**
- **Derived:** `R ∩ S = R − (R − S) = S − (S − R)`.
- Also: when `schema(R) = schema(S)`, **natural join equals intersection**: `R ⋈ S = R ∩ S`.
- **Cardinality:** `0 ≤ |R ∩ S| ≤ min(|R|, |S|)`.

---

**(8) Theta join — ⋈_θ**

$$R \bowtie_{\theta} S = \sigma_{\theta}(R \times S)$$

- A Cartesian product **filtered** by an arbitrary condition `θ` (which mixes attributes of `R` and `S`, using =, ≠, <, >, ≤, ≥, ∧, ∨).
- **Keeps ALL attributes of both** relations (no merging). `degree = degree(R) + degree(S)`.
- **Equijoin:** the special case where `θ` is a conjunction of **equalities** only (e.g. `R.A = S.A ∧ R.B = S.B`). It still keeps **both** copies of the equated columns.

---

**(9) Natural join — ⋈**

$$R \bowtie S$$

- An **equijoin on *all* attributes that share the same name** in `R` and `S`, followed by **dropping the duplicate** copy of each common attribute (keep just one).
- **Schema:** `degree(R ⋈ S) = degree(R) + degree(S) − |\text{common attributes}|`.
- **Two extreme cases (memorize!):**
  - **No common attribute** ⇒ `R ⋈ S = R × S` (degenerates to Cartesian product, `|result| = |R|·|S|`).
  - **Identical schema** ⇒ `R ⋈ S = R ∩ S` (intersection).
- **Cardinality (general):** `0 ≤ |R ⋈ S| ≤ |R| \cdot |S|`.
  - **= 0** when no common-attribute values match.
  - **Reaches `|R|·|S|`** when *every* tuple of `R` matches *every* tuple of `S` on the common attributes (e.g. the join attribute holds the same single value everywhere).
- **The crucial special case (foreign-key join):** if the set of common attributes is a **key of `S`**, then each tuple of `R` matches **at most one** tuple of `S`, so `|R ⋈ S| ≤ |R|`. If, in addition, the common attribute is a **foreign key in `R` referencing that key of `S`** *with referential integrity* (and is non-null), then **every** `R` tuple matches **exactly one** `S` tuple, giving `|R ⋈ S| = |R|` **exactly**. This is one of the most-tested facts in the topic.

---

**(10) Division — ÷ (the "for all" operator)**

Let `schema(S) ⊆ schema(R)`. Write `X = schema(R) − schema(S)` and `Y = schema(S)`.

$$R \div S = \{\, t \text{ over } X : \text{for *every* } s \in S,\ (t, s) \in R \,\}$$

- In words: `R ÷ S` returns the `X`-values that are paired in `R` with **all** of the `Y`-values present in `S`.
- **Schema:** `degree(R ÷ S) = degree(R) − degree(S)` (the result has the attributes `X`).
- **Cardinality:** `0 ≤ |R ÷ S| ≤ |π_X(R)|`.
- **Canonical use:** "Find X that are related to **all** Y" — suppliers who supply **all** parts, students enrolled in **all** courses, etc. Division is the **only** RA operator that naturally expresses **universal quantification ("for all")**; everything else is naturally existential.

**Expressing division with fundamentals (GATE expects this derivation):**

$$R \div S \;=\; \pi_{X}(R)\;-\;\pi_{X}\big((\pi_{X}(R) \times S)\;-\;R\big)$$

*Why it works, step by step:*
1. `π_X(R)` → all **candidate** `X`-values.
2. `π_X(R) × S` → every (candidate, required-`Y`) pair that **ought to exist** in `R` *if* the candidate qualifies.
3. `(π_X(R) × S) − R` → the required pairs that are **missing** from `R`.
4. `π_X(…)` of that → the candidates that are **disqualified** (they miss at least one required pair).
5. Subtract the disqualified from the candidates → the survivors, who have **all** required pairs. ∎

---

### 4.3 The assignment operator — ←

$$\text{Temp} \leftarrow \text{expression}$$

- Stores an intermediate result under a name (`Temp`), so a long query can be written in readable steps. Pure convenience; adds no expressive power.

---

### 4.4 Extended relational algebra (know these; GATE touches them lightly)

These go beyond "pure" RA but are standard and occasionally tested.

**(a) Generalized projection** — allows **arithmetic / functions** in the projection list:
$$\pi_{\,Name,\ Salary \times 0.10}(R)$$
(e.g. compute a 10% figure as an output column.)

**(b) Aggregation — γ (or script G)** — *group by* + aggregate functions (`SUM, AVG, MIN, MAX, COUNT`):
$$_{Dept}\,\gamma_{\;AVG(Salary)}(R)$$
Group `R` by `Dept`, output the average salary per department. (The heavy testing of aggregation lives in **SQL**, Topic 6.)

**(c) Outer joins** — fix the "**dangling tuple**" problem of ordinary join. A tuple that finds **no match** is simply *lost* in a normal join; outer joins **preserve** it by padding the missing side with **NULL**s.

| Operator | Symbol | Keeps unmatched tuples from |
|---|---|---|
| Left outer join | `R ⟕ S` | **left** relation `R` (right side padded with NULLs) |
| Right outer join | `R ⟖ S` | **right** relation `S` (left side padded with NULLs) |
| Full outer join | `R ⟗ S` | **both** relations (both sides padded with NULLs) |

- The matched part of all three equals the ordinary natural join `R ⋈ S`.
- **Cardinality:**
  - `|R ⟕ S| ≥ |R ⋈ S|` (and `≥ |R|` when... see note); precisely, every left tuple appears at least once.
  - `|R ⟗ S| = |R ⋈ S| + (\text{unmatched left tuples}) + (\text{unmatched right tuples})`.

---

### 4.5 Algebraic equivalence rules (the optimizer's toolbox)

These let you rewrite a query into an equivalent one (same result, cheaper plan). GATE asks "which of these are equivalent?" — so know **which ones hold and which fail**.

**Hold (always true):**
1. **Selection cascade:** `σ_{c1 ∧ c2}(R) = σ_{c1}(σ_{c2}(R))`
2. **Selection commute:** `σ_{c1}(σ_{c2}(R)) = σ_{c2}(σ_{c1}(R))`
3. **Projection cascade:** `π_{L1}(π_{L2}(R)) = π_{L1}(R)` if `L1 ⊆ L2`
4. **Selection ↔ join:** `σ_θ(R × S) = R ⋈_θ S`
5. **Commutativity of × and ⋈:** `R × S = S × R`, `R ⋈ S = S ⋈ R`
6. **Associativity of × and ⋈:** `(R ⋈ S) ⋈ T = R ⋈ (S ⋈ T)`
7. **Selection pushdown into join:** `σ_c(R ⋈ S) = (σ_c(R)) ⋈ S`  *provided every attribute in `c` belongs to `R` only.*
8. **Selection distributes over set ops:**
   `σ_c(R ∪ S) = σ_c(R) ∪ σ_c(S)`,
   `σ_c(R ∩ S) = σ_c(R) ∩ σ_c(S)`,
   `σ_c(R − S) = σ_c(R) − σ_c(S)`.
9. **Projection distributes over union:** `π_L(R ∪ S) = π_L(R) ∪ π_L(S)`.

**Do NOT hold in general (classic traps):**
- **Projection does *not* distribute over intersection or difference:**
  `π_L(R ∩ S) ≠ π_L(R) ∩ π_L(S)`, and `π_L(R − S) ≠ π_L(R) − π_L(S)`. (Counterexample in §5, Problem 4.)
- **Union is *not* idempotent under arbitrary domain mismatches**, and set operators require union-compatibility — applying them to incompatible schemas is undefined, not "empty."

---

## 5. Worked problems (GATE-style)

### Problem 1 — Cardinality bounds of set operations (NAT / MSQ)

`R` has 30 tuples and `S` has 50 tuples; they are union-compatible. Give the **minimum and maximum** possible values of `|R ∪ S|`, `|R ∩ S|`, `|R − S|`, and `|S − R|`.

**Solution.** Use the bounds from §4.

- `|R ∪ S|`: max = disjoint = `30 + 50 = 80`; min = `max(30,50) = 50` (when `R ⊆ S`). → **[50, 80]**
- `|R ∩ S|`: max = `min(30,50) = 30` (when `R ⊆ S`); min = `0` (disjoint). → **[0, 30]**
- `|R − S|`: max = `30` (disjoint); min = `max(0, 30−50) = 0` (when `R ⊆ S`). → **[0, 30]**
- `|S − R|`: max = `50` (disjoint); min = `max(0, 50−30) = 20`. → **[20, 50]**

*Check the consistency:* `|R ∪ S| = |R| + |S| − |R ∩ S| = 30 + 50 − |R ∩ S|`. With `|R ∩ S| ∈ [0,30]`, this gives `|R ∪ S| ∈ [50, 80]` — matches. ✔

---

### Problem 2 — Natural join: maximum & minimum tuples (NAT)

Relations `R(A, B)` and `S(B, C)`. `R` has 200 tuples, `S` has 100 tuples. `B` is **not** declared a key in either relation. Find the **maximum** and **minimum** number of tuples in `R ⋈ S`.

**Solution.** Common attribute is `B`.

- **Maximum:** if **all** 200 `R`-tuples have the same `B`-value, and **all** 100 `S`-tuples also have that same `B`-value, every `R`-tuple matches every `S`-tuple. → `200 × 100 = ` **20000**.
- **Minimum:** if **no** `B`-value of `R` appears in `S`, nothing matches. → **0**.

So `0 ≤ |R ⋈ S| ≤ 20000`.

> **Trap variant:** Now suppose `B` is the **primary key of `S`** and a **foreign key in `R`** referencing `S` (referential integrity holds, no nulls). Then each `R`-tuple matches **exactly one** `S`-tuple ⇒ `|R ⋈ S| = |R| = ` **200** (exact, not a range). Recognising "foreign-key join ⇒ exactly `|R|`" is the whole point of such questions.

---

### Problem 3 — Division ("for all") computed by hand (NAT / MCQ)

```
Enrolled(Student, Course)            Required(Course)
---------------------------          -----------------
 (S1, C1)                             C1
 (S1, C2)                             C2
 (S2, C1)
 (S3, C1)
 (S3, C2)
```

Compute `Enrolled ÷ Required` (students enrolled in **all** required courses).

**Solution (direct meaning):** Required = {C1, C2}. We need students paired with **both** C1 and C2.
- S1 → {C1, C2} ✔ (has all)
- S2 → {C1} ✘ (missing C2)
- S3 → {C1, C2} ✔
- **Result = {S1, S3}.**

**Verification via the fundamental-operator formula** `π_X(R) − π_X((π_X(R)×S) − R)` with `X = {Student}`:
1. `π_Student(Enrolled) = {S1, S2, S3}`
2. `π_Student(Enrolled) × Required = {(S1,C1),(S1,C2),(S2,C1),(S2,C2),(S3,C1),(S3,C2)}`
3. minus `Enrolled` (remove the pairs that actually exist) → `{(S2,C2)}`  (this is the only required pair that's missing)
4. `π_Student({(S2,C2)}) = {S2}` (the disqualified student)
5. `{S1,S2,S3} − {S2} = {S1, S3}` ✔ — matches the direct answer.

---

### Problem 4 — Equivalence trap: does π distribute over − ? (MCQ / MSQ)

Is the identity `π_A(R − S) = π_A(R) − π_A(S)` valid for all relations `R, S` over schema `(A, B)`? Justify.

**Solution.** **No.** Counterexample:
- `R = {(1, 2)}`, `S = {(1, 3)}` (both over `(A, B)`, union-compatible).
- `R − S`: tuples `(1,2)` and `(1,3)` differ, so `R − S = {(1,2)}`; thus `π_A(R − S) = {1}`.
- But `π_A(R) = {1}` and `π_A(S) = {1}`, so `π_A(R) − π_A(S) = {} = ∅`.

LHS `= {1}`, RHS `= ∅` ⇒ **not equal**. Projection collapses distinct tuples that share the `A`-value, so it cannot be pushed through difference (or intersection). It *can* be pushed through **union**.

---

### Problem 5 — Translate English to RA + degree/cardinality reasoning (MCQ)

Schema: `Employee(EmpID, Name, DeptID)` (DeptID is a foreign key), `Department(DeptID, DeptName, City)` (DeptID is the primary key). Write an RA expression for **"the names of all employees who work in a department located in 'Delhi'"**, and state the degree of the final result.

**Solution.**

$$\pi_{Name}\Big(\sigma_{City = \text{'Delhi'}}\big(Employee \bowtie Department\big)\Big)$$

- `Employee ⋈ Department` joins on the common attribute `DeptID` (FK→PK), so each employee matches exactly one department; `degree = 3 + 3 − 1 = 5` (EmpID, Name, DeptID, DeptName, City).
- `σ_{City='Delhi'}` keeps only Delhi rows (degree still 5).
- `π_{Name}` outputs just the name ⇒ **degree of final result = 1**.

**Optimized (equivalent) form** — push selection down before the join (rule 7) so the join handles fewer rows:
$$\pi_{Name}\Big(Employee \bowtie \big(\sigma_{City=\text{'Delhi'}}(Department)\big)\Big)$$
Both expressions return the same relation; the second is what an optimizer prefers.

---

## 6. Common GATE question types on this topic

- **Counting tuples / degree** of the result of σ, π, ∪, ∩, −, ×, ⋈, ÷ — exact value for ×, **min–max range** for the others.
- **Max/min tuples of a natural join**, especially the **foreign-key ⇒ exactly `|R|`** special case.
- **Compute a division** ("students who took *all* courses", "suppliers supplying *all* parts") and/or **expand division** into fundamental operators.
- **Identify which algebraic equivalences hold** (selection pushdown, projection over union vs. over difference/intersection) — pick-the-correct-statements MSQ.
- **"What does this expression compute?"** — read a nested RA expression and describe its output in English (or pick the matching English statement).
- **Translate an English query into an RA expression** (and recognise the minimal/optimized form).
- **Minimal operator set** — which operators are fundamental vs. derived; express ∩, ⋈, ÷ using the basics.
- **Outer-join behaviour** — number of NULL-padded tuples in left/right/full outer join.
- **RA vs. SQL semantics** — duplicate handling (set vs. bag).

---

## 7. Traps, pitfalls & common mistakes

1. **Set vs. bag semantics.** Pure RA **removes duplicates automatically** (projection especially). SQL keeps them unless `DISTINCT` is used. If a question is phrased in RA, *dedupe*; if in SQL, *don't* (unless told). Mixing these up is the #1 error.
2. **Projection does NOT distribute over `−` or `∩`.** It *does* distribute over `∪`. (See Problem 4.)
3. **Cartesian product is the *only* exact count.** `|R × S| = |R|·|S|`. For ∪, ∩, −, ⋈, the answer is almost always a **range**, not a single number — don't give a point answer unless a key/FK pins it down.
4. **Natural join with no common attribute silently becomes `×`.** People assume "join means fewer rows"; with zero common attributes it *explodes* to `|R|·|S|`.
5. **Natural join matches on attribute *name*, not meaning.** If two relations both have an attribute called `ID` that mean different things, natural join will (wrongly) match on it. Rename first.
6. **Union-compatibility is mandatory** for ∪, ∩, −. Same **degree** *and* compatible **domains**, in matching column order. An exam may bait you with two relations of different arity.
7. **Set difference and division are NOT commutative.** `R − S ≠ S − R`; `R ÷ S ≠ S ÷ R`.
8. **Division schema:** the result has attributes `schema(R) − schema(S)`, *not* `schema(S)`. And it requires `schema(S) ⊆ schema(R)`.
9. **Theta/equi-join keep both copies of the join columns; natural join keeps one.** This changes the **degree** of the result — a frequent off-by-one in degree-counting questions.
10. **Equijoin ≠ natural join.** Equijoin retains duplicate columns; natural join removes them. (And theta join allows `<, >`, not just `=`.)
11. **"For all" ⇒ division, "there exists" ⇒ projection/join.** If a query says *every/all*, reach for ÷ (or a double-negation `NOT EXISTS … NOT EXISTS …` in SQL). Trying to do a universal query with plain selection/join is the classic dead-end.
12. **Selection pushdown has a *condition*:** `σ_c(R ⋈ S) = σ_c(R) ⋈ S` **only if** `c` mentions attributes of `R` *alone*. If `c` references both sides, you can't push it fully onto one side.
13. **NULLs and outer joins:** a tuple lost in a normal join because it had no match is *exactly* what outer joins preserve via NULL padding — remember to count those padded tuples.

---

## 8. Connection forward

Relational algebra is **procedural** — you specify *how* to compute the answer (the operator sequence). The **next topic, Tuple Relational Calculus (TRC)**, is its **declarative** mirror: you describe *what* you want (a logical predicate over tuple variables) and let the system figure out *how*. A landmark result (Codd's theorem) says **RA and (safe) TRC are equally expressive** — anything you can ask in one, you can ask in the other. After TRC, **SQL (Topic 6)** appears as the practical language that blends both worldviews; its `SELECT–FROM–WHERE` is essentially `π–×/⋈–σ`, and its `NOT EXISTS … NOT EXISTS` pattern is how the "for all" of **division** is written in real queries. So this topic is the algebraic engine that the rest of the Querying section is built on.

---

## 9. One-line cheat-sheet summary

**Relational algebra = a closed, *procedural* set of operators (σ π ∪ − × ρ fundamentals; ∩, ⋈, ÷ derived) that turn relations into relations under *set* semantics; `|R×S|=|R|·|S|` exactly, foreign-key natural join gives exactly `|R|` tuples, projection distributes over ∪ but not over − or ∩, and division (`R÷S`) is the unique "for all" operator.**

---

### ▶ Next topic (in order)
**Topic 5 — Tuple Relational Calculus (TRC):** the declarative counterpart to this topic; equally expressive (Codd's theorem), and the bridge to SQL's `WHERE`-clause logic.

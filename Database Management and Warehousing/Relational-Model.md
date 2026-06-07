# Topic 2 — Relational Model

*Subject: Database Management and Warehousing (GATE DA)*

---

## 0. Topic header & syllabus map

- **Subject:** Database Management and Warehousing.
- **Cluster:** Data Modelling & the Relational Model.
- **Where it sits:** **Topic 2**, immediately **after ER-Model** and immediately **before Integrity Constraints**. The ER model gave us a *picture*; this topic gives that picture a **precise mathematical home** (relations, tuples, keys). The next topic (Integrity Constraints) takes the constraints we *introduce* here and makes them **enforceable rules**.

> **One-sentence placement:** *The relational model is the formal, set-based structure into which the ER design is poured — the layer a real DBMS can actually store and query.*

---

## 1. Why this topic exists (motivation + connection)

- **The real problem it solves.** We need a way to represent data that is (i) **mathematically precise**, (ii) **uniform** (one single structure, not a zoo of shapes), and (iii) **operable** — something an engine can store on disk and answer queries against. The relational model gives exactly that: **everything is a relation (table)**, defined as a set of tuples.
- **What ER couldn't do (the bridge from Topic 1).** An ER diagram is wonderful for *humans*, but **no DBMS executes a diagram**. ER has **no formal operations** to retrieve or combine data, its notion of a "row" is informal, and it deliberately keeps entities and relationships as *different* shapes. The relational model **collapses all of that into one uniform object — the relation** — with a clean definition (a *subset of a Cartesian product of domains*). That single move unlocks (a) set-based query operations (relational algebra, SQL), (b) declarative constraints, and (c) a theory for removing redundancy (normalization). In Topic 1 we even *counted* the relations an ER design needs; **this topic says precisely what one of those relations *is*.**
- **What the next topic builds on it.** Here we *introduce* four constraints — domain, key, entity integrity, referential integrity. **Topic 3 (Integrity Constraints)** turns them into the rules the DBMS checks on every `INSERT/DELETE/UPDATE`, including foreign-key reaction policies (`CASCADE`, `SET NULL`, etc.). Keys defined here (candidate, prime/non-prime) are also the raw material for **normalization (Topic 7)**.

---

## 2. Prerequisites

- **Set theory:** sets, subsets, **Cartesian product** (a relation is literally a subset of one).
- **ER-Model (Topic 1):** entity sets, relationships, and especially the **ER → relational mapping** (the relations and foreign keys you produced there are the objects studied here).
- The key vocabulary from Topic 1 (superkey ⊇ candidate ⊇ primary) — reinforced and formalized below.

---

## 3. Core idea — intuition first

A **relation is just a table** — but defined rigorously so we can reason about it. Picture a `STUDENT` table:

| RollNo | Name | Dept |
|---|---|---|
| 21 | Asha | CSE |
| 34 | Bilal | ECE |
| 56 | Chetan | CSE |

- The **column headings** (`RollNo, Name, Dept`) + their allowed value-sets form the **schema** — the unchanging *structure*.
- Each **row** is a **tuple** — one student.
- The **set of current rows** is the **instance** (it changes as students join/leave).
- Two iron rules give the model its power: **(i) no two rows are identical**, and **(ii) row order carries no meaning**. The table *is a set of rows*, nothing more.

**Analogy.** Schema = the *form template* (the labelled blank boxes); instance = the *stack of filled-in forms* you have today. The template doesn't change when you add a form.

Everything else in this topic is making these intuitions exact and pulling out their consequences (keys, constraints, counting).

---

## 4. Formal treatment

### 4.1 The seven core terms (+ the formal definition)

- **Domain `dom(A)`:** a set of **atomic** (indivisible) values that attribute `A` may take, e.g., `dom(Age) = {0,…,150}`. Atomicity is the **First Normal Form (1NF)** assumption baked into the model — *no composite or multivalued cell values*. A domain may include **NULL**.
- **Attribute `A`:** a named use of a domain in a relation; `dom(Ai)` is its domain.
- **Relation schema `R(A₁,…,Aₙ)`:** relation name `R` + its attribute list. **Degree (arity)** `= n` (number of attributes).
- **Tuple `t`:** an ordered/named list of `n` values `⟨v₁,…,vₙ⟩` with each `vᵢ ∈ dom(Aᵢ)` or `vᵢ = NULL`. Notation `t[Aᵢ]` = the value of attribute `Aᵢ` in `t`.
- **Relation instance / state `r(R)`:** a **set** of tuples. Formally:

$$r(R)\ \subseteq\ \mathrm{dom}(A_1)\times \mathrm{dom}(A_2)\times\cdots\times \mathrm{dom}(A_n)$$

  i.e., a relation is a **subset of the Cartesian product** of its attribute domains. *(Condition: domains are atomic; this is the 1NF model.)*
- **Cardinality:** number of **tuples (rows)** in `r(R)`.
- **Degree:** number of **attributes (columns)** — a property of the *schema*, not the instance.

**Formal vs informal vocabulary** (memorize — GATE phrases questions in the formal terms):

| Formal | Informal |
|---|---|
| Relation | Table |
| Tuple | Row / record |
| Attribute | Column / field |
| Domain | Allowed values of a column |
| Degree (arity) | Number of columns |
| Cardinality | Number of rows |
| Relation schema | Table structure / heading |
| Relation instance/state | Current table contents |

### 4.2 Schema vs. instance (intension vs. extension)

- **Schema (intension):** the *structure* — name, attributes, domains. **Time-invariant** (changes only by an explicit redesign).
- **Instance / state (extension):** the *current set of tuples*. Changes with every update.
- **Database schema** = a set of relation schemas; **database state** = a set of relation instances.

> A valid (legal) state is one that satisfies all integrity constraints (next topic).

### 4.3 Properties of relations (a top GATE target)

Because a relation **is a set of tuples**, the following hold:

1. **No duplicate tuples.** All tuples are distinct. *(Direct consequence of "set." This is why every relation has at least one superkey — the set of all attributes.)*
2. **Tuple order is immaterial.** A relation has no inherent row ordering; any storage order is allowed.
3. **Values are atomic (1NF).** Each cell holds **one** indivisible value (or NULL) — no nested tables, no lists in a cell.
4. **Each attribute has a single value per tuple** drawn from its domain (or NULL).
5. **Attribute ordering — depends on the formalization:** under the **positional** view (tuple = ordered list) attribute order matters; under the **named/mapping** view (tuple = mapping `attribute → value`) it is **immaterial**. Most textbooks (and GATE) prefer the named view, so the standard answer is *"attribute order doesn't matter."* Know both, but default to **immaterial**.

### 4.4 Relational *model* vs. SQL *tables* — the duplicate distinction (classic trap)

- The pure **relational model** = **sets** ⇒ **duplicate rows are impossible**.
- **SQL tables** = **multisets / bags** by default ⇒ **duplicate rows are allowed** unless prevented by a `PRIMARY KEY`/`UNIQUE` constraint, and `SELECT` keeps duplicates unless you write `DISTINCT`.

> So a question like "Can two identical rows exist in a relation?" → **No** (model). "…in a base SQL table without keys?" → **Yes**. GATE exploits this gap.

### 4.5 Keys (the heart of this topic)

For a relation schema `R` with attribute set `U`:

- **Superkey (SK):** an attribute set `SK ⊆ U` such that **no two distinct tuples can ever agree on all of `SK`** in any legal instance. The full set `U` is always a superkey.
- **Candidate key (CK):** a **minimal** superkey — a superkey `K` with **no proper subset that is also a superkey**.
  - **"Minimal" ≠ "minimum."** Minimal means *locally irreducible* (can't drop any attribute). A relation may have candidate keys of **different sizes** simultaneously; only the smallest is the *minimum*. **Both are candidate keys.** *(Heavily tested trap.)*
- **Primary key (PK):** the one candidate key the designer selects to identify tuples; underlined.
- **Alternate keys:** candidate keys not chosen as the primary key.
- **Prime attribute:** an attribute that belongs to **at least one** candidate key.
- **Non-prime (non-key) attribute:** an attribute in **no** candidate key. *(These two definitions are load-bearing for normalization.)*
- **Foreign key (FK):** an attribute set in a *referencing* relation `R₁` whose values must **match an existing primary-key (or candidate-key) value** in a *referenced* relation `R₂`, **or be entirely NULL**. (`R₁` = child/referencing; `R₂` = parent/referenced.) Full enforcement = **referential integrity**, detailed next topic.

> Candidate keys are frequently **computed from functional dependencies** — that machinery lives in **Normalization (Topic 7)**. Here we work with keys that are *given*.

**Counting superkeys (GATE favourite).**

```
If the ONLY candidate key is K (|K| = k), over n total attributes:
    #superkeys = 2^(n − k)        ( K must be present; the other n−k attributes are free in/out )
    single-attribute key (k = 1): 2^(n − 1)

Multiple candidate keys K₁,…,K_m  → inclusion–exclusion
( a superkey = any subset that contains AT LEAST ONE candidate key ):
    #SK = Σ 2^(n − |Kᵢ|)  −  Σ 2^(n − |Kᵢ ∪ Kⱼ|)  +  Σ 2^(n − |Kᵢ ∪ Kⱼ ∪ Kₗ|)  −  ⋯
```

*Conditions:* the listed keys are **all** the candidate keys; each is a genuine key; a "superkey" is any attribute subset that **contains** a candidate key.

**Maximum tuples / relation states (formal-definition questions).**

```
Given attribute domains of sizes |dom(Aᵢ)| (atomic, NULLs ignored):
    max #distinct tuples          = Π |dom(Aᵢ)|
    max #distinct relation states = 2^( Π |dom(Aᵢ)| )    ( every subset of all possible tuples is a legal relation )
```

### 4.6 Constraints (introduced here → fully enforced in Topic 3)

The relational model carries built-in **integrity constraints**:

| Constraint | Statement | Violated by (example) |
|---|---|---|
| **Domain** | each value is atomic and ∈ its domain | inserting a non-domain / non-atomic value |
| **Key** | no two tuples agree on a candidate/primary key | `INSERT`/`UPDATE` creating a duplicate key |
| **Entity integrity** | the **primary key cannot be NULL** (only the PK; other unique keys *may* allow NULL) | `INSERT`/`UPDATE` putting NULL in a PK column |
| **Referential integrity** | an FK value must match some PK value in the referenced relation, or be wholly NULL | `INSERT` of a child with no parent; `DELETE`/`UPDATE` of a referenced parent |

**Which update operation can violate which constraint** (a recurring MCQ/MSQ):

- **INSERT** → can violate **domain, key, entity integrity, referential integrity** (all four).
- **DELETE** → can violate **referential integrity** only (removing a tuple still referenced by some FK).
- **UPDATE (modify)** → can violate any of the four, depending on the attribute changed (e.g., changing a PK value → key/entity/referential; changing a non-key value → domain only).

### 4.7 NULL values (brief)

`NULL` means **unknown**, **not applicable**, or **missing** — three meanings, one symbol. NULLs trigger **three-valued logic** (TRUE / FALSE / **UNKNOWN**) in comparisons and complicate joins, aggregates, and keys. *(Detailed behaviour is covered with SQL, Topic 6; entity integrity already forbids NULL in the primary key.)*

---

## 5. Worked problems (GATE-style)

### WP1 — Counting superkeys with a single candidate key — NAT

> A relation `R(A,B,C,D,E)` has **exactly one** candidate key. Find the number of superkeys when the candidate key is (a) `{A}`, (b) `{A,B}`.

**Solution.** `n = 5`. A superkey must contain the candidate key; the remaining attributes are free.
- (a) `k = 1`: `#SK = 2^(5−1) = 2⁴ =` **16**.
- (b) `k = 2`: `#SK = 2^(5−2) = 2³ =` **8**.

*(Check (a): every superkey contains `A`; the other 4 attributes are each in/out → 2⁴ = 16. ✓)*

### WP2 — Counting superkeys with two candidate keys (inclusion–exclusion) — NAT

> `R(A,B,C,D)` has candidate keys `{A}` and `{B}` (and no others). How many superkeys?

**Solution.** `n = 4`. A superkey contains `{A}` **or** `{B}`.
- Subsets containing `A`: `2^(4−1) = 8`.
- Subsets containing `B`: `2^(4−1) = 8`.
- Subsets containing both (i.e. `{A,B}`): `2^(4−2) = 4`.
- Inclusion–exclusion: `8 + 8 − 4 =` **12**.

**Answer: 12.** *(Trap: just adding `8 + 8 = 16` double-counts the 4 subsets that contain both `A` and `B`.)*

### WP3 — Minimal vs. minimum; prime vs. non-prime — MSQ

> `R(A,B,C,D)` has candidate keys `{A}` and `{B,C}`. Mark every TRUE statement.
> (i) `{B,C}` is a candidate key. (ii) Only `{A}` is a candidate key because it is smallest. (iii) Prime attributes are `{A,B,C}`. (iv) `D` is non-prime. (v) The primary key must be `{A}`.

**Solution.**
- (i) **TRUE** — `{B,C}` is a minimal superkey; size doesn't disqualify it.
- (ii) **FALSE** — "candidate" requires *minimal*, not *minimum*; both `{A}` and `{B,C}` qualify.
- (iii) **TRUE** — prime = in *some* candidate key = `{A} ∪ {B,C} = {A,B,C}`.
- (iv) **TRUE** — `D` is in no candidate key.
- (v) **FALSE** — the designer *may* pick `{A}` (smaller, convenient), but `{B,C}` is also eligible; nothing *forces* `{A}`.

**Answer: (i), (iii), (iv).**

### WP4 — Maximum tuples and maximum relation states — NAT

> `R(A,B)` with `|dom(A)| = 3`, `|dom(B)| = 2` (ignore NULLs). Find (a) the maximum number of distinct tuples and (b) the maximum number of distinct relation instances.

**Solution.**
- (a) `max tuples = |dom(A)| × |dom(B)| = 3 × 2 =` **6** (the full Cartesian product).
- (b) Each of the 6 possible tuples is independently present or absent ⇒ every **subset** of the 6 is a legal relation ⇒ `max states = 2⁶ =` **64** (this counts the empty relation too).

**Answers: 6 tuples; 64 instances.**

### WP5 — Which operation violates which constraint — MSQ

> Select all TRUE statements.
> (i) A `DELETE` can violate referential integrity. (ii) A `DELETE` can violate entity integrity. (iii) An `INSERT` can violate all four constraints. (iv) Changing a non-key attribute's value can violate referential integrity.

**Solution.**
- (i) **TRUE** — deleting a parent tuple that an FK still points to.
- (ii) **FALSE** — `DELETE` removes tuples; it cannot put NULL into a primary key, so it cannot violate entity integrity.
- (iii) **TRUE** — an `INSERT` can break domain, key, entity-integrity, and referential-integrity constraints.
- (iv) **FALSE** — referential integrity concerns FK/PK *value matching*; editing a non-key, non-FK attribute doesn't touch it. (It can only break the **domain** constraint.)

**Answer: (i), (iii).**

---

## 6. Common GATE question types on this topic

- **Count superkeys / candidate keys** for a relation given its candidate key(s) — single key (`2^{n-k}`) and multiple keys (inclusion–exclusion).
- **Identify prime vs. non-prime** attributes from given candidate keys (a normalization warm-up).
- **Minimal vs. minimum** candidate-key conceptual MCQs.
- **Maximum number of tuples / relation states** from domain sizes (`Π|dom|` and `2^{Π|dom|}`).
- **Properties-of-relations** MCQs (duplicates allowed? order significant? atomicity?), often contrasting the **model vs. SQL** (set vs. bag).
- **Which update operation violates which integrity constraint** (MSQ).
- **Formal-vs-informal terminology** matching (degree, cardinality, tuple, attribute).

---

## 7. Traps, pitfalls & common mistakes

- **"Candidate key = smallest key."** Wrong — it must be **minimal (irreducible)**, not **minimum (smallest)**. Different-sized candidate keys can coexist.
- **Forgetting `U` is always a superkey.** The set of all attributes is a superkey because relations have no duplicate tuples — handy sanity check.
- **Adding superkey counts directly with multiple keys.** You must use **inclusion–exclusion**; plain addition double-counts shared supersets.
- **Entity integrity applies to *all* unique keys.** No — only the **primary key** is forbidden from being NULL; other candidate/`UNIQUE` keys may permit a NULL (subject to the system's `UNIQUE` semantics).
- **Thinking the relational model allows duplicate rows.** The **model** does not (it's a set); **SQL** does by default. Don't conflate the two.
- **Confusing degree with cardinality.** Degree = #columns (schema property); cardinality = #rows (instance property).
- **NULL in a foreign key is "illegal."** A *fully* NULL FK is allowed (it simply means "no referenced tuple"); it's a *non-matching, non-NULL* value that violates referential integrity.
- **`DELETE` violating entity integrity.** It can't — deletion never inserts a NULL into a primary key. (`DELETE` endangers **referential** integrity only.)
- **Treating attribute order as always significant.** Under the standard named/mapping view, attribute order is **immaterial**; default to that unless a question explicitly uses the positional view.

---

## 8. Connection forward

The four constraints sketched in §4.6 are exactly what **Topic 3 — Integrity Constraints** formalizes and enforces: domain, key, **entity integrity** (no NULL primary key), and **referential integrity** (FK ↔ PK matching) with their reaction policies (`CASCADE`, `SET NULL`, `RESTRICT`) on `DELETE`/`UPDATE`. Beyond that, the **set-based, duplicate-free, order-independent** relation defined here is the object on which **relational algebra (Topic 4)** and **tuple calculus (Topic 5)** operate, **SQL (Topic 6)** is the practical (bag-relaxed) language for it, and the **prime / non-prime / candidate-key** vocabulary is the foundation of **Normalization (Topic 7)**.

---

## 9. One-line cheat-sheet summary

**A relation is a *set* of atomic-valued tuples (`r ⊆ dom(A₁)×…×dom(Aₙ)`) — so no duplicate rows and no row order; keys go superkey ⊇ candidate (minimal, *not* minimum) ⊇ primary; count superkeys by `2^{n−k}` (one key) or inclusion–exclusion (many); entity integrity ⇒ PK ≠ NULL, referential integrity ⇒ FK matches a PK or is NULL.**

---

*Next topic (logically): **Integrity Constraints** — turning the domain / key / entity-integrity / referential-integrity rules introduced here into the conditions a DBMS enforces on every insert, delete, and update.*

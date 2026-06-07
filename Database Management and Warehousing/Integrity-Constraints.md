# Topic 3 — Integrity Constraints

*Subject: Database Management and Warehousing (GATE DA)*

---

## 0. Topic header & syllabus map

- **Subject:** Database Management and Warehousing.
- **Cluster:** Data Modelling & the Relational Model (the **last** topic of this cluster).
- **Where it sits:** **Topic 3**, immediately **after Relational Model** and immediately **before Relational Algebra** (which starts a new cluster, *Querying*). The relational model *introduced* four constraints as static definitions; this topic makes them **precise and operational** — what each rule means, and what the DBMS does when an insert/delete/update would break it. After this, we stop *defining* the data and start *querying* it.

> **One-sentence placement:** *Integrity constraints are the guardrails that keep every database state not just well-typed but* **valid** *— so the data we are about to query can be trusted.*

---

## 1. Why this topic exists (motivation + connection)

- **The real problem it solves.** A table can be perfectly well-formed (right columns, right types) and still hold **nonsense**: an order pointing to a customer who doesn't exist, two students sharing one roll number, a primary key left blank. Integrity constraints (ICs) are **rules every legal database state must satisfy**, so that *structurally valid* also means *semantically consistent*.
- **What the Relational Model couldn't do (the bridge from Topic 2).** Topic 2 gave us the **structure** (relations, tuples) and the **vocabulary** (candidate key, primary key, foreign key), and it *named* domain / key / entity-integrity / referential-integrity constraints — but it treated them as static facts. It said nothing about the **dynamics**: what happens when you try to insert a child row with no parent, or delete a parent that children still point to. **This topic supplies the dynamics** — the conditions checked on every update and the system's reaction when a reference would dangle.
- **What the next topic builds on it.** **Relational Algebra (next, new chat)** and later **SQL** *retrieve and combine* data and **assume the relations are in valid states**. The **foreign-key relationships** pinned down here are exactly what those queries will **JOIN** on, and "**union-compatibility**" (a schema-level rule) is required before set operations. ICs guarantee the *inputs* to every query are consistent.

---

## 2. Prerequisites

- **Relational Model (Topic 2):** relations, tuples, domains; the key hierarchy (**superkey ⊇ candidate ⊇ primary**); the notion of a **foreign key**; **NULL** semantics.
- **ER-Model (Topic 1):** participation/cardinality and weak entities — they are the *source* of many constraints (total participation → `NOT NULL` foreign key; weak entity → FK that is part of the PK).

---

## 3. Core idea — intuition first

Think of a club's two notebooks: a **MEMBERS** notebook (one page per member, each with a unique ID) and a **DUES** notebook (each payment lists the member ID who paid).

Three commonsense rules keep them sane:

- **You can't write a payment for a member ID that isn't in MEMBERS.** → *referential integrity.*
- **Every member's page must have an ID filled in** (a blank ID identifies no one). → *entity integrity.*
- **No two members can share an ID, and an "age" entry must be a number, not a sentence.** → *key* and *domain* constraints.

And a *dynamics* rule: **what do you do when a member quits?** Tear out their dues pages too (*cascade*), blank out the member-ID on those pages (*set null*), or **refuse to let them quit** while unpaid dues exist (*restrict*). Integrity constraints are just these rules written precisely enough for a machine to enforce automatically.

---

## 4. Formal treatment

### 4.1 The three categories of constraints

| Category | Where it comes from | Examples | Enforced by |
|---|---|---|---|
| **Inherent / implicit** | the relational model itself | no duplicate tuples; atomic cell values (**1NF**); relations are sets | the model (free) |
| **Schema-based / explicit** | declared in the **DDL** | **domain**, **key / `UNIQUE`**, **`NOT NULL`**, **entity integrity**, **referential integrity**, **`CHECK`** | the DBMS, from your `CREATE TABLE` |
| **Application / semantic** | business rules the DDL can't express | "salary ≤ manager's salary", "≥ 2 authors per book" | **triggers / assertions / application code** |

The schema-based ones (below) are what GATE tests.

### 4.2 Domain constraint

- Every value of attribute `A` must be an **atomic** value drawn from **`dom(A)`** (or `NULL` if permitted).
- Declared via the **data type** (`INTEGER`, `VARCHAR(n)`, `DATE`, …) and optionally a **`CHECK`** (e.g., `Age INTEGER CHECK (Age BETWEEN 0 AND 150)`).

### 4.3 Key constraint (uniqueness) — `PRIMARY KEY` vs `UNIQUE`

- **Key constraint:** no two distinct tuples may agree on a candidate key. Declared with **`PRIMARY KEY`** (the chosen key) or **`UNIQUE`** (alternate keys).
- **The differences GATE loves:**

| | `PRIMARY KEY` | `UNIQUE` |
|---|---|---|
| How many per table | **at most one** | **many allowed** |
| NULLs | **forbidden** (entity integrity) | **allowed** (NULL ≠ NULL, so multiple NULLs typically permitted) |
| Role | tuple identifier; usual clustering key | enforces an alternate-key uniqueness |

### 4.4 `NOT NULL` constraint

- Forbids `NULL` in a column. **Primary-key columns are implicitly `NOT NULL`** (that *is* entity integrity).

### 4.5 Entity integrity constraint

```
Entity integrity:  the PRIMARY KEY value must never be NULL.
  • If the PK is composite, NO component attribute may be NULL.
  • Applies ONLY to the primary key. Other candidate keys / UNIQUE columns MAY contain NULL.
```

*Rationale:* the PK identifies tuples; a NULL in it would identify nothing and could not guarantee uniqueness (under 3-valued logic, `NULL = NULL` is *UNKNOWN*, not *TRUE*).

### 4.6 Referential integrity constraint (the most-tested rule)

A **foreign key (FK)** in a *referencing* relation `R₁` is an attribute set whose values reference the **primary key (or a `UNIQUE` candidate key)** of a *referenced* relation `R₂`.

```
Referential integrity (FK of R1 → PK of R2):
  for EVERY tuple t1 ∈ R1 :   t1[FK] = t2[PK] for some t2 ∈ R2     OR     t1[FK] is entirely NULL

Conditions / fine print:
  • FK and the referenced key are domain-compatible (same number of attributes, same domains).
  • The referenced attributes MUST be a PRIMARY KEY or a declared UNIQUE key in R2.
  • "entirely NULL" = every FK attribute is NULL. (A *partly* NULL composite FK is governed by the
    MATCH option: default MATCH SIMPLE lets it pass; MATCH FULL requires all-or-nothing.)
  • Self-reference is allowed (e.g., EMP.MgrID → EMP.EmpID).
  • An FK may be part of R1's own primary key (weak entities, M:N junction tables).
```

**What can break referential integrity, by operation:**

- **`INSERT` into `R₁` (child):** new `FK` must match an existing `R₂` PK or be NULL — else **violation**.
- **`DELETE` from `R₂` (parent):** if the deleted tuple is **still referenced** by some `R₁` tuple → **violation** (would dangle).
- **`UPDATE` of the PK in `R₂` (parent):** if the *old* PK value is referenced → **violation**.
- **`UPDATE` of the `FK` in `R₁` (child):** the *new* value must match an `R₂` PK or be NULL.

### 4.7 Referential triggered actions (the `ON DELETE` / `ON UPDATE` policies)

When a referenced **parent** tuple is **deleted** or its **PK updated** while children point to it, the FK declares the reaction:

```
NO ACTION / RESTRICT : REJECT the operation (the default).  Data unchanged, statement fails.
                       (RESTRICT checks immediately; NO ACTION checks at end-of-statement —
                        for GATE both mean "block it".)
CASCADE              : propagate — ON DELETE CASCADE deletes the child tuples;
                       ON UPDATE CASCADE rewrites child FK values to the new PK value.
                       → can CHAIN through further child tables.
SET NULL             : set the child FK to NULL  (requires the FK column to allow NULL).
SET DEFAULT          : set the child FK to its DEFAULT (which must be a valid parent PK, or NULL).
```

Declared per FK, e.g. `FOREIGN KEY (DNo) REFERENCES DEPT(DNo) ON DELETE CASCADE ON UPDATE SET NULL`.

> **CASCADE chains.** If `R₂ ← R₁` (cascade) and `R₁ ← R₃` (cascade), deleting from `R₂` can cascade through `R₁` into `R₃`. Counting total deletions across such a chain is a favourite NAT (WP1).

### 4.8 General / semantic constraints (brief)

- **`CHECK`:** a condition on one row's attributes, e.g., `CHECK (Salary > 0)` or `CHECK (EndDate >= StartDate)`.
- **Assertions** (`CREATE ASSERTION … CHECK (…)`): schema-level conditions spanning **multiple rows/tables**; powerful but rarely implemented.
- **Triggers:** procedural **event–condition–action** rules — the fallback when declarative constraints can't express the rule.

### 4.9 State vs. transition constraints (brief)

- **State (static) constraint:** must hold in **every** valid state (almost all constraints — e.g., `Age ≥ 0`).
- **Transition (dynamic) constraint:** restricts **how** the DB may move between states (e.g., "salary may only increase"). Needs **triggers**; not expressible as a plain `CHECK`.

### 4.10 How these appear in SQL DDL (grounding; full SQL is Topic 6)

```sql
CREATE TABLE DEPT (
  DNo   INT PRIMARY KEY,                 -- key + entity integrity (implicit NOT NULL)
  DName VARCHAR(40) UNIQUE NOT NULL      -- alternate key, no NULLs
);

CREATE TABLE EMP (
  ENo    INT PRIMARY KEY,
  EName  VARCHAR(40) NOT NULL,
  Salary INT CHECK (Salary > 0),         -- domain/semantic check
  DNo    INT,
  FOREIGN KEY (DNo) REFERENCES DEPT(DNo) -- referential integrity
       ON DELETE CASCADE ON UPDATE CASCADE
);
```

---

## 5. Worked problems (GATE-style)

**Common schema for WP1–WP2.**
`DEPT(DNo PK, DName)` · `EMP(ENo PK, EName, DNo FK→DEPT.DNo)` · `PROJECT(PNo PK, PName, ENo FK→EMP.ENo)`.
Data — `DEPT`: D1, D2 | `EMP`: E1(D1), E2(D1), E3(D2) | `PROJECT`: P1(E1), P2(E1), P3(E2), P4(E3).

### WP1 — Counting cascading deletes (chain) — NAT

> Both foreign keys use **`ON DELETE CASCADE`**. The statement `DELETE FROM DEPT WHERE DNo = D1;` runs. How many tuples are deleted **in total**, and how many from `PROJECT`?

**Solution.** Follow the cascade level by level:
- `DEPT`: D1 → **1** tuple.
- `EMP` referencing D1: E1, E2 → **2** tuples (E3 has D2, survives).
- `PROJECT` referencing E1 or E2: P1(E1), P2(E1), P3(E2) → **3** tuples (P4 has E3, survives).

Total `= 1 + 2 + 3 =` **6 tuples**; **`PROJECT` deletions = 3**.

### WP2 — Same delete under three different policies — MCQ/compare

> For `EMP.DNo → DEPT`, replace the policy and re-run `DELETE FROM DEPT WHERE DNo = D1;` (keep `PROJECT.ENo → EMP` as `CASCADE`). Effect under (a) `RESTRICT`, (b) `SET NULL`, (c) `CASCADE`?

**Solution.**
- **(a) `RESTRICT`/`NO ACTION`:** D1 is referenced by E1, E2 ⇒ the delete is **rejected**; **0 tuples change**, the statement fails.
- **(b) `SET NULL`:** D1 is deleted (1 tuple); E1, E2 **survive** with `DNo ← NULL` (requires `DNo` nullable); `PROJECT` untouched because E1, E2 still exist. **1 tuple deleted.**
- **(c) `CASCADE`:** as in WP1 ⇒ **6 tuples deleted**.

**Takeaway:** same intent, three very different data outcomes — read the declared policy carefully.

### WP3 — Which operation violates which constraint — MSQ

> Using `EMP(ENo PK, EName, DNo FK→DEPT.DNo)`, mark every TRUE statement.
> (i) Inserting an `EMP` whose `DNo` is absent from `DEPT` violates referential integrity.
> (ii) Inserting an `EMP` with `ENo = NULL` violates entity integrity.
> (iii) Deleting a `DEPT` row referenced by some `EMP`, with no cascade/set-null, is rejected by referential integrity.
> (iv) **Every** deletion of an `EMP` row violates referential integrity.
> (v) Updating `EName` to a too-long string violates referential integrity.

**Solution.**
- (i) **TRUE** — child FK with no matching parent.
- (ii) **TRUE** — NULL in the primary key.
- (iii) **TRUE** — the parent delete would leave dangling children; default `NO ACTION` blocks it.
- (iv) **FALSE** — deleting `EMP` only matters if **another** table references `EMP`; if nothing does, it's fine.
- (v) **FALSE** — that breaks the **domain** constraint, not referential integrity.

**Answer: (i), (ii), (iii).**

### WP4 — Entity integrity, composite PK, and `UNIQUE` NULLs — MSQ

> `ENROLL(StudID, CourseID, Grade)` with `PRIMARY KEY (StudID, CourseID)` and a separate `UNIQUE(Email)` column on a `STUDENT` table. Mark every TRUE statement.
> (i) A tuple with `StudID = NULL` is allowed in `ENROLL`.
> (ii) A `STUDENT` may have `Email = NULL`.
> (iii) Two `STUDENT` rows may both have `Email = NULL`.
> (iv) A table can declare more than one `PRIMARY KEY`.

**Solution.**
- (i) **FALSE** — `StudID` is part of the composite PK; **no PK component may be NULL** (entity integrity).
- (ii) **TRUE** — `UNIQUE` permits NULL.
- (iii) **TRUE** — under 3-valued logic `NULL = NULL` is *UNKNOWN*, so multiple NULLs don't violate `UNIQUE` (standard SQL behaviour).
- (iv) **FALSE** — at most **one** `PRIMARY KEY` per table (use additional `UNIQUE` constraints for alternate keys).

**Answer: (ii), (iii).**

---

## 6. Common GATE question types on this topic

- **Count tuples deleted/updated** under `ON DELETE/UPDATE CASCADE`, including multi-level **cascade chains**.
- **Compare referential-action policies** (`RESTRICT` vs `CASCADE` vs `SET NULL` vs `SET DEFAULT`) — which lets a parent delete succeed, and what happens to children.
- **Which operation (`INSERT`/`DELETE`/`UPDATE`) can violate which constraint** (MSQ).
- **Entity-integrity / composite-PK NULL** questions and **`PRIMARY KEY` vs `UNIQUE`** (NULLs, count per table).
- **Identify the constraint type** (domain / key / entity / referential / semantic) violated by a scenario.
- **Read SQL DDL** and decide whether a given `INSERT`/`DELETE` succeeds or is rejected.

---

## 7. Traps, pitfalls & common mistakes

- **Entity integrity ⇒ *every* unique key is non-NULL.** No — only the **primary key**. `UNIQUE`/other candidate keys **may** hold NULL.
- **A NULL foreign key is illegal.** A **wholly NULL** FK is legal (means "references nothing"). Only a **non-NULL, non-matching** value violates referential integrity.
- **`DELETE` can violate entity integrity.** It can't — deletion never inserts a NULL into a PK. `DELETE` endangers **referential** integrity only.
- **Assuming the default policy is `CASCADE`.** The default is **`NO ACTION`/`RESTRICT`** (block). Cascading happens **only** if explicitly declared.
- **Forgetting cascade chains.** A delete can ripple through several tables; count **all** levels, not just direct children.
- **`SET NULL` on a `NOT NULL` (or PK-member) FK.** Illegal — the column must allow NULL; you cannot `SET NULL` an FK that is part of the primary key.
- **Referencing a non-key column.** A foreign key must reference a **PRIMARY KEY or `UNIQUE`** column of the parent — not an arbitrary attribute.
- **Two NULLs violate `UNIQUE`.** They don't (NULL ≠ NULL under 3-valued logic), so multiple NULLs are allowed in a `UNIQUE` column.
- **Treating semantic rules as declarable.** "Salary ≤ manager's salary" or "salary may only increase" generally needs **triggers/assertions**, not a plain `CHECK`.
- **Domain vs. referential confusion.** A bad *value/type* breaks the **domain** constraint; a bad *reference* breaks **referential integrity** — different constraints.

---

## 8. Connection forward

With every database state now guaranteed **valid**, we move from *defining* data to *asking questions of it*. **Relational Algebra (next, new cluster)** is the formal query language whose operators consume these consistent relations; the **foreign keys** formalized here are precisely the columns we will **JOIN** on, and the **union-compatibility** requirement for set operators (`∪`, `∩`, `−`) is itself a schema constraint in the same spirit. Integrity constraints are also the consistency floor that **transactions** later protect under concurrency and failures — the "C" in ACID.

---

## 9. One-line cheat-sheet summary

**ICs keep states valid: domain (atomic, in-`dom`), key (`PRIMARY KEY` unique+`NOT NULL`; `UNIQUE` allows NULLs), entity integrity (no NULL in any PK component), referential integrity (FK = some parent PK *or* wholly NULL); on a referenced parent `DELETE`/`UPDATE` the FK reacts with `NO ACTION`/`RESTRICT` (block, default), `CASCADE` (propagate, can chain), `SET NULL`, or `SET DEFAULT`.**

---

*Next topic (logically, new chat): **Relational Algebra** — the first formal query language, operating on the now-consistent relations and joining on the foreign keys defined here.*

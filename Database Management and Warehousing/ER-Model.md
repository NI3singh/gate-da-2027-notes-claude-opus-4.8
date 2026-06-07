# Topic 1 — ER-Model (Entity–Relationship Model)

*Subject: Database Management and Warehousing (GATE DA)*

---

## 0. Topic header & syllabus map

- **Subject:** Database Management and Warehousing.
- **Cluster:** Data Modelling & the Relational Model (the very first cluster of the subject).
- **Where it sits:** This is **Topic 1** — it has *no predecessor* inside the subject; it is the entry point. Everything in DBMS design begins here. It leads **directly into Topic 2 (Relational Model)** through a mechanical "ER → relational" translation, and feeds **Topic 3 (Integrity Constraints)**, **Topic 7 (Normalization)** and the warehouse schema topics later.

> **One-sentence placement:** *The ER model is the human-readable blueprint of "what data we have and how it relates"; the relational model that follows is the machine-readable storage of that blueprint.*

---

## 1. Why this topic exists (motivation + connection)

- **The real problem it solves.** A real requirement sounds like: *"A customer can hold many accounts, every loan is sanctioned by exactly one branch, an employee may manage at most one department…"* This prose is vague, redundant, and ambiguous. Before we ever write a `CREATE TABLE`, we need a **precise, visual, implementation-independent picture** of the things we store and how they connect. The ER model is that picture.
- **What came before, and why it wasn't enough.** There is no earlier DBMS topic — the "previous state of the world" is unstructured requirements and ad-hoc file storage, where the same fact (a customer's address) gets duplicated in many files, with no notion of *which* fact identifies a thing or *which* associations are mandatory. The ER model is the **first disciplined abstraction**: it forces you to name entities, pin down what uniquely identifies each, and state the rules of every association — *without yet* committing to tables, keys, or SQL.
- **What the next topic builds on it.** The **relational model** is excellent for storage and querying but is **flat**: a table cannot natively say "this is an entity vs. that is a relationship," nor "this participation is compulsory." So we design conceptually in ER (easy for humans), then translate to relations (easy for machines). **ER answers *what to model*; the relational model answers *how to store and query it*.**

---

## 2. Prerequisites

- Basic **set theory**: sets, subsets, Cartesian product (relationships are conceptually subsets of a Cartesian product of entity sets).
- The everyday idea of a **record/field** (a row with attributes).
- That's all. **Keys, foreign keys, and the design vocabulary are introduced here** — you do not need them beforehand.

---

## 3. Core idea — intuition first

Think of modelling a small **university**. You only ever deal with three kinds of nouns:

| Building block | Plain meaning | University example | Diagram shape |
|---|---|---|---|
| **Entity** | a "thing" with independent existence | `STUDENT`, `COURSE`, `INSTRUCTOR` | rectangle |
| **Attribute** | a property of a thing | `RollNo`, `Name`, `Credits` | oval |
| **Relationship** | an association *between* things | `ENROLLS`, `TEACHES` | diamond |

An **ER diagram** is just a labelled graph of these shapes, with extra annotations on the connecting lines that say *how many* and *whether mandatory*.

**Type vs. set (intension vs. extension).** `STUDENT` the **entity type** is the schema/template (the columns every student has). The **entity set** is the *current collection* of actual students. Same distinction as *class* vs. *objects*. GATE phrases questions about both — read carefully which one is meant.

The whole game of this topic is: **draw the right shapes, annotate the lines correctly, then turn the picture into tables.**

---

## 4. Formal treatment

### 4.1 Entities and entity sets

- **Entity:** a distinguishable real-world object — *physical* (a car, a person) or *conceptual* (a course, a job).
- **Entity type / entity set:** a set of entities sharing the same attributes. (Type = schema; set = current instances.)
- **Notation:** rectangle (single line). A **weak** entity uses a **double rectangle** (see §4.6).

### 4.2 Attributes — the full taxonomy

Every attribute has a **value set (domain)** — the permitted values. The classifications below are **independent axes** (an attribute can be, e.g., composite *and* multivalued).

| Type | Meaning | Example | Notation |
|---|---|---|---|
| **Simple / atomic** | indivisible | `Age` | plain oval |
| **Composite** | splits into sub-parts | `Name` → First, Middle, Last; `Address` → Street, City, PIN | oval with sub-ovals hanging off it |
| **Single-valued** | one value per entity | `DateOfBirth` | plain oval |
| **Multivalued** | a *set* of values | `PhoneNumbers`, `Degrees` | **double oval** |
| **Stored** | physically kept | `DateOfBirth` | plain oval |
| **Derived** | computed from others | `Age` (from DOB), `#Employees` (count) | **dashed oval** |
| **Key attribute** | uniquely identifies entities | `RollNo` | **underlined** |

- **NULL** is a legal value meaning *unknown*, *not applicable*, or *missing* (three different reasons, same symbol).
- **Mapping consequence (preview):** a **composite** attribute is *flattened* — only its leaf components become columns. A **multivalued** attribute *cannot* be a column; it becomes its **own table** (§4.8). A **derived** attribute is usually *not stored*.

### 4.3 Keys (introduced here, reused everywhere)

For an entity set:

- **Superkey:** any attribute set that **uniquely identifies** each entity. (Adding more attributes to a superkey keeps it a superkey.)
- **Candidate key:** a **minimal** superkey — *no proper subset is itself a superkey*.
- **Primary key:** the **one** candidate key the designer chooses to identify entities (underlined in the diagram).
- An entity set may have **several candidate keys** but exactly **one** primary key.

**Containment:** every candidate key is a superkey; the primary key is a chosen candidate key.

$$\{\text{primary key}\} \subseteq \{\text{candidate keys}\} \subseteq \{\text{superkeys}\}$$

> These exact definitions reappear in the relational model and in normalization — learn them once, here.

### 4.4 Relationships

- **Relationship:** an association among **two or more** entities. **Relationship type/set** = the diamond; the *set* is the current collection of associations. Conceptually a relationship set is a subset of the Cartesian product of the participating entity sets.
- **Degree** = number of participating entity types:
  - **Unary / recursive (degree 1):** an entity set related to *itself*, e.g., `EMPLOYEE` *supervises* `EMPLOYEE`. You **must** label **role names** (supervisor, subordinate) to tell the two ends apart.
  - **Binary (degree 2):** the common case.
  - **Ternary (degree 3)** and general **n-ary**.
- **Relationship attributes:** a relationship can carry its own attributes, e.g., `WORKS_ON(EMPLOYEE, PROJECT)` with attribute `Hours`. *Where these attributes end up* in tables depends on cardinality (§4.8).

### 4.5 Constraints on relationships (the most tested part)

Two **independent** constraints describe every binary relationship.

**(a) Cardinality ratio (mapping cardinality)** — *how many* of one entity can relate to the other:

| Ratio | Meaning | Example |
|---|---|---|
| **1 : 1** | each A ↔ at most one B, and vice versa | `EMPLOYEE` *manages* `DEPARTMENT` |
| **1 : N** | one A relates to many B; each B to one A | `DEPARTMENT` *has* `EMPLOYEE` |
| **M : N** | each A to many B and each B to many A | `STUDENT` *enrolls* `COURSE` |

**(b) Participation constraint (existence dependency)** — *whether mandatory*:

- **Total participation:** **every** entity of the set must take part in ≥ 1 relationship. **Double line.**
- **Partial participation:** some entities need not take part. **Single line.**
- Example: if every employee *must* belong to a department → `EMPLOYEE` has **total** participation in `WORKS_FOR`.

> **Cardinality and participation are orthogonal.** A relationship can be 1:N *and* total on one side, or 1:N *and* partial — these are separate facts. GATE deliberately mixes them.

**(c) `(min, max)` "look-here" structural constraints.** Instead of arrows, write a pair `(min, max)` **on the edge from entity E to relationship R**, read as:

> *Each instance of E participates in **at least `min`** and **at most `max`** instances of R.*

- `min = 0` → **partial**;  `min ≥ 1` → **total** participation.
- `max = 1` → E connects to **one** of the other entity;  `max = N` → **many**.

**⚠ The classic inversion trap.** Because `(min,max)` counts *E's own* associations, the **max on E's edge tells how many of the OTHER entity each E connects to.** Worked out for `WORKS_FOR` between `DEPARTMENT` and `EMPLOYEE` (a dept has many employees; an employee has one dept):

| Edge | `(min,max)` | Reads as | So… |
|---|---|---|---|
| `EMPLOYEE` | **(1,1)** | each employee is in exactly one `WORKS_FOR` | employee → **one** dept |
| `DEPARTMENT` | **(1,N)** | each dept is in 1..N `WORKS_FOR` | dept → **many** employees |

Cardinality ratio is **DEPT : EMP = 1 : N**. Notice the **swap**: the entity carrying `(1,1)` (EMPLOYEE) is the **"many" (N) side**, and the entity carrying `(…,N)` (DEPARTMENT) is the **"one" (1) side**. Students who read "(1,1) ⇒ EMPLOYEE is the *one* side" get it backwards.

### 4.6 Weak entity sets

- A **weak entity set** has **no attribute(s) of its own that form a primary key**. It exists only in connection with a **owner / identifying entity set**, via an **identifying relationship**.
- It has a **partial key (discriminator)**: attribute(s) that distinguish weak entities **belonging to the same owner** (dashed underline).
- A weak entity **always has total participation** in its identifying relationship (it cannot exist without its owner). The owner→weak cardinality is typically **1 : N**.
- **Primary key of a weak entity = (primary key of owner) ⊕ (partial key).**
- **Notation:** double rectangle (weak entity), **double diamond** (identifying relationship).

**Example.** `DEPENDENT` is weak to `EMPLOYEE`. Two employees can each have a dependent named "Anil," so `DependentName` alone is not unique — it is the *partial key*. The full key is `(SSN, DependentName)` where `SSN` is the employee's primary key.

(*Strong / regular entity* = one that **does** have its own key.)

### 4.7 Enhanced ER (EER) features

GATE DBMS can touch these; know the definitions and the constraint vocabulary.

**Specialization vs. generalization**
- **Specialization (top-down):** define **subclasses** of a superclass by a distinguishing property, e.g., `EMPLOYEE` → {`ENGINEER`, `SECRETARY`, `MANAGER`}.
- **Generalization (bottom-up):** combine entity types with common features into a **superclass**, e.g., `CAR`, `TRUCK` → `VEHICLE`.
- **Attribute inheritance:** a subclass inherits *all* attributes and *all* relationships of its superclass, **plus** its own specific attributes. The link is an **IS-A** relationship.

**Two constraints on a specialization/generalization (4 combinations):**

| Axis | Options | Meaning |
|---|---|---|
| **Disjointness** | **disjoint (d)** vs **overlapping (o)** | can an entity be in **more than one** subclass? `d` = at most one; `o` = many |
| **Completeness** | **total** vs **partial** | must **every** superclass entity be in **some** subclass? total = yes (double line); partial = may be in none |

So the four kinds are: *disjoint-total, disjoint-partial, overlapping-total, overlapping-partial.* Subclass membership may be **predicate-defined** (a condition on an attribute, e.g., `JobType = 'Engineer'`) or **user-defined**.

**Aggregation**
- The ER model **forbids a relationship among relationships** directly. **Aggregation** treats an entire relationship (with its entities) as a **higher-level entity**, so it can participate in another relationship.
- *Example:* `WORKS_ON(EMPLOYEE, PROJECT)`; to record the `MANAGER` who supervises each *(employee, project)* assignment, aggregate `WORKS_ON` and relate it to `MANAGER`. Notation: a **box** drawn around the relationship and its entities.

**Category / union type (UNION)** — *brief.* A subclass that is a subset of the **union** of several *different* superclasses, e.g., `OWNER` over {`PERSON`, `BANK`, `COMPANY`}. Shown with a `∪` in a circle. (Rare in GATE — recognise it, don't over-study it.)

### 4.8 ER → Relational mapping (this *is* the bridge to Topic 2)

> No derivation is involved — this is a **rule-based algorithm**. (Stating that explicitly because GATE never asks you to "derive" the mapping; it asks you to *apply* it and to *count tables*.)

**Navathe's 7-step algorithm (condensed):**

1. **Strong entity** → one relation with all its **simple** attributes (flatten composites to leaves). Choose a primary key.
2. **Weak entity** → one relation = its attributes **+ owner's PK as a foreign key**; PK = owner's PK ⊕ partial key.
3. **Binary 1 : 1** → put the **other entity's PK as a foreign key** into **one** of the two relations (prefer the side with **total** participation). *No new table.* Relationship attributes go to that same side.
4. **Binary 1 : N** → put the **"1" side's PK as a foreign key** in the **"N" side** relation. *No new table.* Relationship attributes go to the N-side.
5. **Binary M : N** → **create a NEW relation** (junction): PKs of **both** entities as foreign keys; their **combination is the PK**; plus relationship attributes.
6. **Multivalued attribute** → **create a NEW relation**: the attribute **+ owner's PK** as FK; PK = both together.
7. **n-ary relationship (n ≥ 3)** → **create a NEW relation** with PKs of **all** participating entities as FKs + relationship attributes.

**Minimum number of tables — the GATE staple.** For a single binary relationship between **strong** entities `E1, E2` (atomic attributes; relationship attributes absorbed wherever the relationship is absorbed):

```
  CARDINALITY            PARTICIPATION           MIN #TABLES
  ─────────────────────────────────────────────────────────
  M : N                  any                     3   ← always (E1, E2, junction)
  1 : N  /  N : 1        any                     2   ← FK of "1" side → "N" side
  1 : 1                  at least one side partial   2   ← merge R into the TOTAL side (or either)
  1 : 1                  BOTH sides total            1   ← E1,E2 are in bijection → one combined table

  ADD-ONS
  each multivalued attribute              +1 table
  weak entity + identifying relationship  owner + weak = 2  (relationship absorbed, no extra)
  n-ary relationship (n ≥ 3), general     n entity tables + 1 relationship table
```

**Conditions you must keep in mind (GATE tests exactly these):**
- For **1 : N**, the count is **2 regardless of participation** — participation only decides whether the foreign-key column is allowed to be **NULL** (partial ⇒ NULLs possible; total ⇒ `NOT NULL`).
- **M : N is *always* 3** — neither side can store a single FK (both sides have many associations), so the junction table is unavoidable.
- The **"1:1 both-total ⇒ 1 table"** collapse assumes the two entities are *otherwise independent* (no other relationship forces them apart). It works because total + total + 1:1 means a perfect **bijection**.
- For a **weak entity**, never forget `PK(weak) = PK(owner) ⊕ partial_key`.

### 4.9 Notation cheat-table

| Shape | Means |
|---|---|
| Rectangle / **double** rectangle | entity / **weak** entity |
| Oval / **double** oval / **dashed** oval | attribute / **multivalued** / **derived** |
| **Underlined** / **dashed-underlined** attribute | primary key / **partial** key |
| Diamond / **double** diamond | relationship / **identifying** relationship |
| Single line / **double** line | partial / **total** participation |
| `1`, `N`, `M` labels or arrows; `(min,max)` | cardinality / structural constraint |
| Triangle/circle with subset symbol; `d` / `o` | IS-A (specialization); disjoint / overlapping |

---

## 5. Worked problems (GATE-style)

### WP1 — Minimum tables with mixed cardinalities *(the classic; GATE CS 2005 pattern)* — NAT

> `E1` and `E2` are two entity sets. `R1` is a **1 : N** relationship and `R2` is a **M : N** relationship between them. Neither relationship has any attribute of its own. What is the **minimum number of tables** to represent this in the relational model?

**Solution.**
- `E1` → 1 table, `E2` → 1 table.
- `R1` is **1 : N** → absorb the "1" side's PK as a foreign key in the "N" side. **No new table.**
- `R2` is **M : N** → **must** have its own junction table. **+1.**
- Total = `2 (entities) + 0 (R1) + 1 (R2) =` **3**.

**Answer: 3.** *(Trap: trying to also give `R1` a table → wrong answer 4.)*

### WP2 — 1 : 1 with participation reasoning — MCQ

> `MANAGES` is a **1 : 1** relationship between `EMPLOYEE` and `DEPARTMENT`. Every department has **exactly one** manager (DEPARTMENT total); an employee **may or may not** manage a department (EMPLOYEE partial). Minimum tables?

**Solution.** With 1:1, merge the relationship into the **total-participation** side so the FK is never NULL. Put `Mgr_SSN` (the employee's PK) as a `NOT NULL` foreign key inside `DEPARTMENT`. `EMPLOYEE` stays separate.
- `EMPLOYEE(SSN, …)`,  `DEPARTMENT(DNo, …, Mgr_SSN → EMPLOYEE.SSN)`.
- **Answer: 2 tables.**

**Variant (trap).** If instead **every** employee manages exactly one department **and** every department has exactly one manager (**both total**, 1:1 ⇒ bijection), the two can be combined into a **single** relation → **1 table**. *(Most students reflexively answer 2 here and lose the mark.)*

### WP3 — Weak entity mapping — write schemas + count — MSQ-style

> `EMPLOYEE(SSN [PK], Name)` is strong. `DEPENDENT` is **weak**, with attributes `DependentName`, `Birthdate`, `Relationship`, partial key `DependentName`, identifying relationship `DEP_OF` (EMPLOYEE **1 : N** DEPENDENT, DEPENDENT total). Give the relations and the count.

**Solution.**
- `EMPLOYEE(SSN, Name)` — PK = `SSN`.
- `DEPENDENT(SSN, DependentName, Birthdate, Relationship)` — **PK = {SSN, DependentName}**, with `SSN` a foreign key to `EMPLOYEE`.
- The identifying relationship needs **no** separate table (Step 2 absorbs it).

**Answer: 2 tables.** Key point: the weak entity's PK **includes the owner's PK**; `DependentName` alone is *not* a key.

### WP4 — Reading `(min, max)` (inversion trap) — MCQ

> In `WORKS_FOR` between `DEPARTMENT` and `EMPLOYEE`, the `EMPLOYEE` edge is labelled **(1,1)** and the `DEPARTMENT` edge **(0,N)**. State the cardinality ratio, the participations, and the minimum tables.

**Solution.**
- `EMPLOYEE (1,1)`: each employee is in **exactly one** `WORKS_FOR` ⇒ employee works for **one** department; `min = 1` ⇒ **EMPLOYEE total**.
- `DEPARTMENT (0,N)`: each department is in **0..N** ⇒ may have many or no employees; `min = 0` ⇒ **DEPARTMENT partial**.
- **Cardinality ratio = DEPT : EMP = 1 : N** (the entity with edge-max `N` is the **"one"** side — here DEPARTMENT).
- 1 : N ⇒ **2 tables**. FK `DNo` goes into `EMPLOYEE` (the N side) and is `NOT NULL` because EMPLOYEE is total.

**Answer: 1 : N, EMPLOYEE total / DEPARTMENT partial, 2 tables.** *(Trap: calling EMPLOYEE the "one" side because of its `(1,1)` label.)*

### WP5 — Full mapping: M : N + multivalued attribute — count — NAT

> `STUDENT(Roll [PK], Name)` also has a **multivalued** attribute `Phone`. `COURSE(CID [PK], Title)`. `ENROLLS` is **M : N** with attribute `Grade`. Minimum tables?

**Solution.** Apply the rules additively:
- `STUDENT(Roll, Name)` — entity ⇒ 1.
- `STUDENT_PHONE(Roll, Phone)`, PK `{Roll, Phone}`, `Roll` FK — multivalued ⇒ **+1**.
- `COURSE(CID, Title)` — entity ⇒ 1.
- `ENROLLS(Roll, CID, Grade)`, PK `{Roll, CID}`, FKs to both, `Grade` lives here — M : N ⇒ **+1**.

Count = `2 (entities) + 1 (multivalued) + 1 (M:N) =` **4**.

**Answer: 4 tables.** *(Trap: forgetting the multivalued attribute needs its own table → wrong answer 3.)*

---

## 6. Common GATE question types on this topic

- **"Minimum number of tables"** for a given ER diagram (most frequent) — apply the §4.8 rule block, remembering the M:N, multivalued, and weak-entity add-ons.
- **Interpret cardinality + participation** from a diagram or from `(min,max)` labels (the inversion trap).
- **Identify the weak entity** and state its **primary key** (`owner PK ⊕ partial key`).
- **Convert an ER fragment to relational schemas** — write tables, mark PKs and FKs, decide where relationship attributes go.
- **Count attributes / count relations** after mapping.
- **Conceptual MCQs:** degree of a relationship, role names in a recursive relationship, *why a ternary relationship is not three binaries*, disjoint vs. overlapping, total vs. partial specialization, what aggregation is for.
- **Distinguish** superkey / candidate key / primary key.

---

## 7. Traps, pitfalls & common mistakes

- **`(min,max)` inversion:** the entity whose edge-max is `1` is the **"many" (N)** side; the entity whose edge-max is `N` is the **"one" (1)** side. (`min = 0` ⇒ partial, `min ≥ 1` ⇒ total — never confuse the *first* number with cardinality.)
- **M : N is always 3 tables.** You cannot avoid the junction table; do not try to "absorb" it.
- **1 : 1 with both-total collapses to 1 table** (bijection). The reflex answer "2" is wrong in that special case.
- **1 : N is 2 tables irrespective of participation** — participation only changes whether the FK can be NULL.
- **Multivalued attributes need their own table** (+1). Easy to forget when counting.
- **Weak-entity PK includes the owner's PK.** The partial key alone is *never* a key.
- **Cardinality ≠ participation.** They are independent; a 1:N relationship can be total or partial on either side.
- **A ternary relationship is *not* equivalent to three binary relationships** — replacing it generally loses information or fabricates associations.
- **Composite attributes are flattened** in mapping (only leaf parts become columns); the composite "parent" is not a column.
- **Derived attributes are usually not stored** (dashed oval) — don't create a column for `Age` if `DOB` is present unless asked.
- **Specialization "total"** means *every superclass entity must be in some subclass* — it has nothing to do with "all attributes."
- **Candidate key is minimal; superkey need not be.** Don't call any uniquely-identifying set a "candidate" key.

---

## 8. Connection forward

The **output** of §4.8 — a set of **relations** with **primary keys** and **foreign keys** — is *exactly* the **input** to **Topic 2: Relational Model**, where we formalise tuples, relations, schemas, and what those keys mean operationally. **Topic 3 (Integrity Constraints)** then turns the ER constraints you drew (keys, total participation, FKs) into enforceable **entity-integrity** and **referential-integrity** rules, and **Topic 7 (Normalization)** repairs the redundancy that loose ER design — or a flattened M:N — can introduce. In short, the picture you just learned to draw becomes the tables everything else operates on.

---

## 9. One-line cheat-sheet summary

**ER = entities (rectangles) + attributes (ovals) + relationships (diamonds), annotated with cardinality and participation; map to tables by "M:N → +1 junction, multivalued → +1, 1:N → 2 (FK on N-side), 1:1 → 2 (or 1 if both total), weak-entity PK = owner-PK ⊕ partial-key," and beware the `(min,max)` inversion.**

---

*Next topic (logically): **Relational Model** — the formal home for the tables, keys, and foreign keys this ER mapping just produced.*

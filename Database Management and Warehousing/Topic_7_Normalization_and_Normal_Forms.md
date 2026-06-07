# Topic 7 — Normalization & Normal Forms (Functional Dependencies, 1NF–BCNF)

**Subject:** Database Management and Warehousing
**GATE DA Section:** Database Design & Storage (Chat 3)

---

## 0. Topic header & syllabus map

This topic opens the **Database Design & Storage** block. It sits *after* the querying topics (Relational Algebra, Tuple Calculus, SQL) and *before* **File Organization** (Topic 8) and **Indexing** (Topic 9).

The story so far: the **ER-model** (Topic 1) gave us a way to *capture* requirements; the **relational model** (Topic 2) turned them into tables; **integrity constraints** (Topic 3) and **SQL** (Topic 6) let us *enforce rules* and *query* the data. But none of those told us whether our table design is actually *good*. A schema can be perfectly legal (satisfies all integrity constraints, answers all SQL queries) and still be a maintenance disaster.

**Normalization is the theory of "is this schema well-designed, and if not, how do I fix it?"** — driven entirely by a single new concept, the **functional dependency**.

---

## 1. Why this topic exists (motivation + connection)

**The real problem:** badly designed tables store the same fact in many places. That redundancy causes three classic **anomalies**:

- **Update anomaly** — one fact lives in many rows; updating one row and forgetting the others makes the database self-contradictory.
- **Insertion anomaly** — you cannot record one fact without also inventing/knowing an unrelated fact (because they share a row).
- **Deletion anomaly** — deleting one fact silently destroys another, unrelated fact.

**What the previous topics couldn't do:** Integrity constraints (Topic 3) can stop *illegal* data (a null key, a dangling foreign key), but they cannot stop *redundant* data. SQL (Topic 6) lets you query whatever structure you have, but a query language does not improve a structure. So we needed a separate, formal tool to *diagnose* redundancy and *prove* that a re-structuring fixed it without losing information. That tool is the **functional dependency (FD)** plus the **normal forms** built on top of it.

**What the next topics build on this:** once a schema is normalized into many small tables, *how those tables are physically stored on disk* (File Organization, Topic 8) and *how we speed up lookups across them* (Indexing, Topic 9) become the next questions. Normalization decides the *logical* shape; storage and indexing make that shape *fast*.

---

## 2. Prerequisites

- **Relational model (Topic 2):** relation = set of tuples over attributes; the notion of a *schema* `R(A, B, C, …)`.
- **Keys (Topic 2/3):** super key, candidate key, primary key — revised and formalized below.
- **Integrity constraints (Topic 3):** especially key constraints; FDs are a generalization of key constraints.
- **Set basics:** subset (⊆), proper subset (⊂), union, intersection — used heavily for closures and decomposition.

---

## 3. Core idea — intuition first

A **functional dependency `X → Y`** ("X determines Y") just means: *if you know the value(s) of X, the value(s) of Y are fixed — there is exactly one possible Y for each X.*

Plain-language example. In `Student(RollNo, Name, Dept, DeptHead)`:

- `RollNo → Name` (each roll number belongs to one student name).
- `Dept → DeptHead` (each department has one head).

Now notice the trouble. If 500 students are in the CSE department, the fact "CSE's head is Dr. Rao" is repeated **500 times**. Change the head → 500 updates (update anomaly). Want to add a brand-new department with no students yet → you can't, because there's no row without a student (insertion anomaly). The last student of a department leaves → the department's head vanishes from the database (deletion anomaly).

**The cure is a re-structuring guided by the FDs:** every fact should live in exactly one place, "keyed" by what determines it. `Dept → DeptHead` is a fact about *departments*, so it belongs in a separate `Department(Dept, DeptHead)` table. That single move kills all three anomalies.

**Normal forms are a ladder of increasingly strict "every fact in its proper place" rules** (1NF → 2NF → 3NF → BCNF). Higher = less redundancy. The entire machinery — closures, keys, decomposition tests — exists just to let you *check which rung you're on* and *climb higher safely* (without losing data or losing the ability to enforce the original rules).

---

## 4. Formal treatment

### 4.1 Functional dependency — definition

> **Definition (FD).** For a relation schema `R` and attribute sets `X, Y ⊆ R`, the FD **`X → Y`** holds on `R` iff for every legal instance and every pair of tuples `t₁, t₂`:
> $$t_1[X] = t_2[X] \;\Longrightarrow\; t_1[Y] = t_2[Y].$$

An FD is a statement about **all allowed instances** (it is a constraint, like a business rule), *not* something you can confirm from one sample table. A single table can only *disprove* an FD (show a counterexample), never *prove* it.

**Classification of an FD `X → Y`:**

| Type | Condition | Note |
|---|---|---|
| **Trivial** | `Y ⊆ X` | Always holds; irrelevant to normal forms. |
| **Non-trivial** | `Y ⊄ X` (some attr of Y not in X) | These are what matter. |
| **Completely non-trivial** | `X ∩ Y = ∅` | Strongest non-triviality. |

### 4.2 Armstrong's axioms (inference rules)

To reason about *all* FDs logically implied by a given set, use Armstrong's axioms. They are **sound** (derive only true FDs) and **complete** (derive every implied FD).

**Primary rules:**

| Rule | Statement |
|---|---|
| **Reflexivity** | If `Y ⊆ X` then `X → Y`. |
| **Augmentation** | If `X → Y` then `XZ → YZ` (for any Z). |
| **Transitivity** | If `X → Y` and `Y → Z` then `X → Z`. |

**Derived rules (handy shortcuts):**

| Rule | Statement |
|---|---|
| **Union** | If `X → Y` and `X → Z` then `X → YZ`. |
| **Decomposition** | If `X → YZ` then `X → Y` and `X → Z`. |
| **Pseudotransitivity** | If `X → Y` and `WY → Z` then `WX → Z`. |
| **Composition** | If `X → Y` and `Z → W` then `XZ → YW`. |

> **GATE note on derivations:** GATE asks you to *apply* these (e.g., "which FD is implied?"), never to *prove soundness/completeness*. So those proofs are skipped here on purpose — they have never been required.

⚠️ **Decomposition works only on the RHS, not the LHS.** From `X → A` you may *not* conclude `(part of X) → A`. `AB → C` does **not** give `A → C`. This single confusion is behind a large fraction of normalization mistakes.

### 4.3 Attribute closure `X⁺`

> **Definition.** Given FD set `F`, the **closure** `X⁺` is the set of all attributes `A` such that `X → A` is implied by `F`.

**Algorithm (the workhorse of this entire topic):**

```
X⁺ := X
repeat:
    for each FD  (W → Z)  in F:
        if  W ⊆ X⁺  then  X⁺ := X⁺ ∪ Z
until X⁺ stops changing
```

**Three uses of closure (memorize):**

1. **Does `X → Y` hold?** ⇔ `Y ⊆ X⁺`.
2. **Is `X` a super key of `R`?** ⇔ `X⁺ = R` (X determines every attribute).
3. **Finding candidate keys** (next section).

### 4.4 Keys — formal definitions

| Term | Definition |
|---|---|
| **Super key** | A set `K ⊆ R` with `K⁺ = R`. (Determines everything; need not be minimal.) |
| **Candidate key** | A super key `K` such that **no proper subset** of `K` is a super key (a *minimal* super key). |
| **Primary key** | One candidate key chosen by the designer. |
| **Prime attribute** | An attribute that belongs to **at least one** candidate key. |
| **Non-prime (non-key) attribute** | An attribute in **no** candidate key. |

⚠️ To decide whether an attribute is prime, you need **all** candidate keys, not just one.

### 4.5 Finding ALL candidate keys (the L / R / B / N method)

Classify each attribute by where it appears across the FDs of `F`:

- **L** — appears **only on the left** of some FD (never on a right side).
- **R** — appears **only on the right** of some FD (never on a left side).
- **B** — appears on **both** sides.
- **N** — appears in **no** FD (neither side).

**Rules:**

1. Every attribute in **L** and **N** *must* be in **every** candidate key. (Nothing can determine them, so they can never be "produced" — they must be given.)
2. Every attribute in **R** can **never** be in any candidate key. (It is always derivable, so including it would break minimality.)
3. Compute `(L ∪ N)⁺`.
   - If `(L ∪ N)⁺ = R`, then **`L ∪ N` is the unique candidate key** — done.
   - Otherwise, augment `L ∪ N` with attributes from **B**, one/few at a time, computing closures, and collect the **minimal** sets whose closure equals `R`.

### 4.6 Counting super keys

A set `S` is a super key **iff it contains at least one candidate key**.

- **One candidate key** `K` with `|K| = k` in a relation with `n` attributes ⇒ number of super keys `= 2^(n−k)` (the k key attributes are fixed; each of the remaining `n−k` attributes is independently in or out).
- **Several candidate keys** ⇒ use **inclusion–exclusion** over "contains `K₁`", "contains `K₂`", …

### 4.7 Equivalence & minimal (canonical) cover

> **Equivalence.** FD sets `F` and `G` are **equivalent** (each "covers" the other) iff `F⁺ = G⁺`. Test: every FD of `F` is implied by `G` (i.e. `F ⊆ G⁺`) **and** every FD of `G` is implied by `F` (`G ⊆ F⁺`).

> **Minimal / Canonical cover `F_c`.** A set equivalent to `F` such that:
> 1. **Single attribute on each RHS.**
> 2. **No extraneous LHS attribute:** in `X → A`, attribute `B ∈ X` is extraneous if `A ∈ (X − B)⁺` (closure under the current set). Remove such `B`.
> 3. **No redundant FD:** `X → A` is redundant if `A ∈ X⁺` computed under `F − {X → A}`. Remove such FDs.

**Procedure:** (1) split RHSs → (2) strip extraneous LHS attributes → (3) drop redundant FDs.
The result is **not unique** (depends on processing order), but all valid minimal covers are equivalent to `F`. Minimal cover is the engine of the **3NF synthesis** algorithm (Section 4.10).

### 4.8 The normal forms (1NF → BCNF)

Throughout, a "violating FD" means a **non-trivial** FD (trivial FDs never cause violations).

> **1NF (First Normal Form).** Every attribute value is **atomic** — no multi-valued attributes, no composite attributes, no nested relations / repeating groups. (We assume 1NF for everything below; the relational model essentially demands it.)

> **2NF (Second Normal Form).** In 1NF **and** no **non-prime** attribute is **partially dependent** on any candidate key.
> *Partial dependency* = a non-prime attribute is determined by a **proper subset** of a candidate key.

> **3NF (Third Normal Form).** In 2NF **and** no non-prime attribute is **transitively dependent** on a candidate key.
> **Working definition (use this one):** for every non-trivial FD `X → A` (`A` a single attribute):
> $$\textbf{X is a super key} \quad\textbf{OR}\quad \textbf{A is a prime attribute.}$$

> **BCNF (Boyce–Codd Normal Form).** For **every** non-trivial FD `X → A`:
> $$\textbf{X is a super key.}$$
> (BCNF removes 3NF's "A is prime" escape hatch.)

**The ladder (strict containment):**

$$\text{1NF} \supseteq \text{2NF} \supseteq \text{3NF} \supseteq \text{BCNF}$$

Every BCNF relation is in 3NF, every 3NF in 2NF, etc. Higher rung = stricter = less redundancy.

| NF | Removes | One-line test |
|---|---|---|
| 1NF | repeating groups / non-atomic values | every cell holds a single value |
| 2NF | partial dependencies | no non-prime depends on *part* of a candidate key |
| 3NF | transitive dependencies | every FD: LHS is super key **OR** RHS is prime |
| BCNF | anomalies from overlapping/other keys | every FD: LHS is super key |

**Useful "free passes" (each saves time and is a frequent GATE shortcut):**

- A relation with **only two attributes** is **always in BCNF**.
- If **every attribute is prime**, the relation is **automatically in 3NF** (every RHS is prime).
- If **every candidate key is a single attribute**, the relation is **automatically in 2NF** (no proper subset of a 1-attribute key exists, so no partial dependency is possible).
- A relation whose **only candidate key is the full set of attributes** (or with no non-trivial FDs) is in **BCNF**.

> **Time-saving theorem (important):** to test **3NF** or **BCNF** it suffices to check only the FDs in the **given set `F`** — you do *not* need to enumerate `F⁺`. If no FD in `F` violates the form, no FD in `F⁺` does either. (For 2NF/partial-dependency checks, work from `F` plus the relevant closures as usual.)

### 4.9 Decomposition and its two desirable properties

Fixing a schema means **decomposing** `R` into `R₁, R₂, …`. A good decomposition needs:

**(a) Lossless-join (non-additive join)** — joining the pieces back returns *exactly* `R`, with no spurious extra tuples.

> **Binary test.** `R` split into `R₁, R₂` is lossless **iff** the common attributes form a super key of at least one piece:
> $$(R_1 \cap R_2) \to R_1 \quad\textbf{or}\quad (R_1 \cap R_2) \to R_2.$$
> Equivalently `(R₁ ∩ R₂) → (R₁ − R₂)` **or** `(R₁ ∩ R₂) → (R₂ − R₁)`.

⚠️ "at least **one**", not both. A lossy decomposition loses information — never acceptable.

> **General test — the Chase (tableau) algorithm** (for splitting into ≥ 3 relations):
> Build a table with one **row per `Rᵢ`** and one **column per attribute**. In row `i`, put the distinguished symbol `aⱼ` in every column of attribute `Aⱼ ∈ Rᵢ`, and a subscripted symbol `b_{ij}` elsewhere. Then repeatedly apply each FD `X → Y`: whenever two rows agree on all of `X`, force them to agree on `Y` (prefer the distinguished `a` symbol). If **any row becomes all `a`'s**, the decomposition is **lossless**; if the chase stabilizes with no such row, it is **lossy**.

**(b) Dependency preservation** — you can enforce all original FDs by checking each piece **independently**, without ever re-joining.

> **Definition.** Let `Fᵢ` = projection of `F` onto `Rᵢ` (all FDs `X → Y` of `F⁺` with `X, Y ⊆ Rᵢ`). The decomposition preserves dependencies iff
> $$(F_1 \cup F_2 \cup \dots \cup F_n)^+ = F^+.$$

**The single most-tested fact about decomposition:**

| Target form | Lossless join always achievable? | Dependency-preserving always achievable? |
|---|:---:|:---:|
| **3NF** | ✅ Yes | ✅ Yes |
| **BCNF** | ✅ Yes | ❌ **Not always** |

So BCNF may force you to sacrifice dependency preservation; 3NF lets you keep **both**. This trade-off is the reason 3NF is sometimes preferred in practice.

### 4.10 The two decomposition algorithms

**3NF synthesis (Bernstein) — lossless AND dependency-preserving:**

1. Compute a minimal cover `F_c`.
2. For each FD `X → A` in `F_c`, output a relation with attributes `X ∪ A`. **Group FDs with the same LHS** into one relation (`X → A₁, X → A₂` ⇒ relation `X A₁ A₂`).
3. If **no** output relation contains a candidate key of `R`, add one relation that *is* a candidate key of `R`.
4. Remove any relation whose attributes are a subset of another's.

Result: every relation in 3NF, lossless, dependency-preserving.

**BCNF decomposition — always lossless (dependency preservation not guaranteed):**

1. Find a non-trivial FD `X → Y` that **violates BCNF** in `R` (`X` not a super key).
2. Replace `R` with two relations:
   - `R₁ = X⁺` (peels off everything `X` determines), and
   - `R₂ = X ∪ (R − X⁺)`.
   (Simpler single-FD variant: `R₁ = X ∪ Y`, `R₂ = R − (Y − X)`.) Both share `X`, and `X` is a super key of `R₁`, so the split is lossless.
3. Recurse on any piece still not in BCNF.

> **(Beyond syllabus — one line only.)** Stricter forms exist — **4NF** removes *multi-valued dependencies* and **5NF** removes *join dependencies* — but GATE DA's syllabus stops at BCNF, so they are not developed here.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Attribute closure & super key (NAT/MCQ)

`R(A, B, C, D, E)`, `F = {A→B, BC→D, D→E, E→A}`. **(i)** Find `A⁺`. **(ii)** Is `AC` a super key?

**Solution.**
(i) `A⁺` = {A} → add B (A→B) → {A,B}. Need C for BC→D; not present → stop. **`A⁺ = {A, B}`.**
(ii) `(AC)⁺` = {A,C} → A→B ⇒ {A,B,C} → BC→D ⇒ {A,B,C,D} → D→E ⇒ {A,B,C,D,E}. Now `(AC)⁺ = R`. **Yes, `AC` is a super key.** (Is it a candidate key? `A⁺ = AB ≠ R`, `C⁺ = C ≠ R`, so neither proper subset works ⇒ `AC` is in fact a candidate key.)

---

### Problem 2 — Find all candidate keys (the rich one) (MSQ)

`R(A, B, C, D, E)`, `F = {A→B, BC→D, D→A, E→C}`. Find all candidate keys; name the prime and non-prime attributes; give the highest normal form.

**Solution.**
*Classify attributes.* RHS attributes: `B, D, A, C`. So `E` appears **only on the left** (L). `A, B, C, D` appear on both sides (B). No N attributes.

→ **`E` is in every candidate key.** `E⁺` = {E} → E→C ⇒ {E,C}. Stop (BC→D needs B). `E⁺ = EC ≠ R`, so `E` alone is not a key — augment with B-attributes.

- `(EA)⁺`: {E,A}→ A→B ⇒ {E,A,B}; E→C ⇒ {E,A,B,C}; BC→D ⇒ {E,A,B,C,D} = **R**. `{A,E}` ✔ (minimal: `A⁺=AB`, `E⁺=EC`).
- `(EB)⁺`: {E,B}; E→C ⇒ {E,B,C}; BC→D ⇒ +D; D→A ⇒ +A = **R**. `{B,E}` ✔.
- `(ED)⁺`: {E,D}; D→A ⇒ +A; A→B ⇒ +B; E→C ⇒ +C = **R**. `{D,E}` ✔.
- `(EC)⁺ = EC ≠ R`, so `C` with `E` is not enough; no other size-2 set qualifies.

**Candidate keys: `{A,E}`, `{B,E}`, `{D,E}`.** Prime = `A, B, D, E`. **Non-prime = `C`.**

*Highest normal form.* Check each FD:
- `A→B`: A super key? `A⁺=AB≠R` → no. B prime? yes → 3NF-OK; **BCNF-violated**.
- `BC→D`: `(BC)⁺ = BCDA ≠ R` → not super key. D prime? yes → 3NF-OK; BCNF-violated.
- `D→A`: `D⁺ = DAB ≠ R` → no. A prime? yes → 3NF-OK; BCNF-violated.
- `E→C`: `E⁺=EC≠R` → not super key. **C is non-prime** → **violates 3NF.**

So not 3NF. **2NF?** Non-prime `C` is determined by `E`, and `E` is a *proper subset* of every candidate key (`AE, BE, DE`) → `C` is **partially dependent** on a candidate key → **violates 2NF**.

**Highest normal form = 1NF only.** *(Lesson: never stop at "it satisfies 3NF for most FDs" — one offending FD, `E→C`, drops the whole relation to 1NF.)*

---

### Problem 3 — The classic 3NF-but-not-BCNF (MCQ trap)

`R(A, B, C)`, `F = {AB→C, C→B}`. Highest normal form?

**Solution.**
`A` appears only on the left ⇒ in every key. `(AB)⁺ = ABC = R` and `(AC)⁺`: {A,C}→C→B ⇒ {A,B,C}=R. **Candidate keys: `AB`, `AC`.** All of `A, B, C` are prime; no non-prime attributes.

- All attributes prime ⇒ **automatically 3NF** (check: `AB→C`, C prime ✔; `C→B`, B prime ✔).
- BCNF? `C→B`: is `C` a super key? `C⁺ = CB ≠ R` → **no**. So `C→B` violates BCNF.

**Answer: 3NF but not BCNF.** This overlapping-candidate-key pattern is the canonical "3NF ≠ BCNF" example — recognize it on sight.

---

### Problem 4 — Lossless-join via the Chase (MSQ)

`R(A, B, C, D)`, `F = {A→B, B→C, C→D}`. Is the decomposition `R₁(A,B)`, `R₂(B,C)`, `R₃(C,D)` lossless?

**Solution.** Tableau (`a` = distinguished):

| | A | B | C | D |
|---|---|---|---|---|
| R₁(A,B) | a₁ | a₂ | b₁₃ | b₁₄ |
| R₂(B,C) | b₂₁ | a₂ | a₃ | b₂₄ |
| R₃(C,D) | b₃₁ | b₃₂ | a₃ | a₄ |

- `A→B`: no two rows share `A` → no change.
- `B→C`: rows R₁, R₂ share `B = a₂` → unify their `C`: set R₁.C = `a₃`.
- `C→D`: now R₁, R₂, R₃ all have `C = a₃` → unify `D`: set all `D = a₄`.

Row R₁ is now `a₁, a₂, a₃, a₄` — **all distinguished** ⇒ **lossless ✔.**

---

### Problem 5 — BCNF decomposition that LOSES a dependency (concept MSQ)

`R(A, B, C)`, `F = {AB→C, C→B}` (candidate keys `AB`, `AC`). `C→B` violates BCNF. Decompose to BCNF and check dependency preservation.

**Solution.** Violating FD `C→B`: `C⁺ = {C,B}`.
- `R₁ = C⁺ = {B, C}` with FD `C→B`.
- `R₂ = C ∪ (R − C⁺) = {C} ∪ {A} = {A, C}`.

*Both BCNF?* `R₁(B,C)`: `C` is super key of `R₁` (`C→B`) ✔. `R₂(A,C)`: only two attributes ⇒ BCNF ✔.
*Lossless?* `R₁ ∩ R₂ = {C}`, and `C → B = R₁ − R₂` ⇒ lossless ✔.
*Dependency-preserving?* Projections: `F₁ = {C→B}`, `F₂ = {}` (no non-trivial FD on `{A,C}`). Union `F' = {C→B}`. Does `F'` imply `AB→C`? `(AB)⁺` under `F'` = `{A,B}` (the rule `C→B` never fires) → **`AB→C` is lost.** ❌

**Answer: lossless BCNF decomposition, but dependency `AB→C` is *not* preserved.** Concrete proof that BCNF can cost dependency preservation.

---

### Problem 6 — Minimal (canonical) cover (NAT-style)

`F = {A→BC, B→C, A→B, AB→C}`. Find a minimal cover.

**Solution.**
**Step 1 — single RHS:** `{A→B, A→C, B→C, AB→C}` (the duplicate `A→B` is dropped).
**Step 2 — extraneous LHS** (only `AB→C` has a composite LHS): is `A` extraneous? Check `C ∈ B⁺`? `B⁺ = BC` → yes ⇒ `A` extraneous, so `AB→C` collapses to `B→C` (already present) ⇒ drop it.
Now `{A→B, A→C, B→C}`.
**Step 3 — redundant FDs:**
- Remove `A→C`: under `{A→B, B→C}`, `A⁺ = ABC ⊇ C` → `A→C` is **redundant**, drop it.
- Remove `A→B`: under `{B→C}`, `A⁺ = A ⊉ B` → **keep**.
- Remove `B→C`: under `{A→B}`, `B⁺ = B ⊉ C` → **keep**.

**Minimal cover = `{A→B, B→C}`.**

---

### Problem 7 — Counting candidate keys and super keys (NAT)

`R(A, B, C, D, E)`, `F = {A→BCDE, BC→ADE}`. How many candidate keys and how many super keys?

**Solution.** `A⁺ = ABCDE = R` ⇒ `A` is a candidate key. `(BC)⁺ = BCADE = R` ⇒ `BC` is a candidate key. `D, E` are RHS-only ⇒ never in any key; `B⁺=B`, `C⁺=C`. **Candidate keys: `{A}` and `{B,C}` → 2 candidate keys.**

A set is a super key iff it contains `A` **or** contains `{B,C}`:
- contains `A`: `2⁴ = 16`,
- contains `{B,C}`: `2³ = 8`,
- contains both `A` and `{B,C}` (overcount): `2² = 4`.

Inclusion–exclusion: `16 + 8 − 4 = 20`. **20 super keys.**

---

### Problem 8 — Spotting 2NF vs 3NF violations from a schema (MCQ)

For each, give the highest normal form (assume the candidate key shown):

(a) `Enroll(StudentID, CourseID, StudentName, Grade)`; CK = `{StudentID, CourseID}`; FDs `StudentID→StudentName`, `{StudentID,CourseID}→Grade`.
(b) `Stu(StudentID, Zip, City)`; CK = `{StudentID}`; FDs `StudentID→Zip`, `Zip→City`.

**Solution.**
(a) Non-prime `StudentName` is determined by `StudentID`, a *proper subset* of the candidate key ⇒ **partial dependency** ⇒ **violates 2NF** ⇒ highest = **1NF**.
(b) Single-attribute key ⇒ automatically 2NF. But `StudentID → Zip → City`: `Zip` is not a super key (`Zip⁺ = {Zip, City} ≠ R`) and `City` is non-prime ⇒ **transitive dependency** ⇒ **violates 3NF** ⇒ highest = **2NF**.

---

## 6. Common GATE question types on this topic

1. **Compute `X⁺`** / decide whether a given FD is implied.
2. **Find all candidate keys** (L/R/B method) and **list prime attributes**.
3. **Count candidate keys** and **count super keys** (inclusion–exclusion).
4. **Identify the highest normal form** of a relation given `R` and `F`.
5. **Lossless-join test** — binary rule, or 3-way via the chase; true/false statements about a given decomposition.
6. **Dependency-preservation test** — does `(∪Fᵢ)⁺ = F⁺`?
7. **Compute a minimal/canonical cover.**
8. **Conceptual true/false** statements: "BCNF is always dependency-preserving" (false), "every 3NF is BCNF" (false), "a 2-attribute relation is always BCNF" (true), etc.
9. **3NF vs BCNF discrimination** on overlapping-candidate-key relations.

---

## 7. Traps, pitfalls & common mistakes

- **Decomposing on the LHS.** `AB → C` does **not** give `A → C`. Decomposition is RHS-only. ⚠️
- **Finding one candidate key and assuming it's the only one.** You need *all* candidate keys to know which attributes are prime — and prime-ness decides 3NF vs BCNF.
- **Forgetting the free passes:** all-prime ⇒ 3NF for free; single-attribute keys ⇒ 2NF for free; two-attribute relation ⇒ BCNF for free. Missing these wastes time and invites errors.
- **Lossless rule: "at least one," not "both."** Common attributes must be a super key of *at least one* piece. Many students wrongly demand it of both.
- **Treating trivial FDs as violations.** `AB → A` is trivial and never breaks any normal form. Only non-trivial FDs matter.
- **Believing BCNF is always achievable with dependency preservation.** It is **not** — Problem 5 is the counterexample. (3NF *is*.)
- **Confusing partial vs transitive dependency.** Partial = non-prime depends on *part* of a key (a 2NF issue). Transitive = non-prime depends on a non-key set that itself depends on the key (a 3NF issue).
- **Stopping the closure algorithm too early.** Re-scan the FD set after every addition; a newly added attribute can trigger more FDs (cascades).
- **Checking only some FDs for a normal form and concluding the relation satisfies it.** *One* violating FD lowers the entire relation's normal form (Problem 2: `E→C` alone forced 1NF).
- **Over-computing for 3NF/BCNF.** You need only the FDs in `F`, not all of `F⁺`. Conversely, for partial-dependency (2NF) reasoning, do use closures to find what subsets of keys determine.
- **Assuming a candidate key must be small.** Sometimes the only candidate key is the entire attribute set (then the relation is automatically BCNF).
- **Minimal cover is not unique** — don't panic if your valid answer differs from a key's; verify by checking equivalence (`F⁺ = F_c⁺`).

---

## 8. Connection forward

Normalization decides the **logical** schema: how many tables, and which attributes go where, so that each fact lives in exactly one place. Once you have many small, well-keyed relations, the next questions are physical: **how is each relation actually laid out and accessed on disk?** That is **File Organization (Topic 8)** — heap, sorted, and hashed files — which in turn sets up **Indexing (Topic 9)**, the structures that make lookups and joins across all these normalized tables fast. Logical design (this topic) → physical storage (next) → access speed (after).

---

## 9. One-line cheat-sheet summary

**FDs define what determines what; `X⁺` answers super-key/key questions; the ladder 1NF→2NF (no partial dep.)→3NF (every FD has super-key LHS *or* prime RHS)→BCNF (every FD has super-key LHS) shrinks redundancy — and 3NF decomposition is always lossless *and* dependency-preserving, while BCNF is always lossless but may sacrifice dependency preservation.**

---

### ▶ Next topic (logically next in order): **Topic 8 — File Organization** (heap / sequential / hashed files; how normalized relations are physically stored), which then leads into **Topic 9 — Indexing**.

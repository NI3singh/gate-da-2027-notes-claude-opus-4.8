# Topic 6 — SQL (Part 1 of 2): Data Definition, DML & Core Querying
**Subject:** Database Management and Warehousing
**Section:** Querying (Chat 2) — final topic of the section
**This part covers:** DDL + constraints, DML, `SELECT–FROM–WHERE`, `DISTINCT`/`ORDER BY`/pattern matching, set operations, **all join types (incl. outer joins)**, and **NULLs + three-valued logic**.
**Part 2 covers:** aggregation, `GROUP BY`/`HAVING`, subqueries (`IN`/`EXISTS`/`ANY`/`ALL`/correlated), the `NOT IN`-NULL trap, division ("for all"), and views.

---

## 0. Topic header & syllabus map

- **Topic:** SQL (Structured Query Language).
- **Subject:** Database Management and Warehousing → *Querying* cluster (the **last** topic of this section).
- **Where it sits:** SQL is the **practical, industry-standard** query language. It comes after relational algebra (procedural) and tuple relational calculus (declarative) and **realizes both**: its `SELECT–FROM–WHERE` describes results the way **calculus** does, while it also offers algebra-style operators (joins, set ops). Crucially, **SQL is *bag*-based** (keeps duplicates) where RA/TRC are *set*-based — the single biggest conceptual shift from the previous two topics. After SQL the syllabus moves to a **new chat: Normalization & Normal Forms** (good schema design).

---

## 1. Why this topic exists (motivation + connection)

Relational algebra and tuple calculus are **mathematical** languages — perfect for *reasoning* about queries, but nobody runs a business on `σ`, `π`, and `∃t`. We need a language that is (a) writable by ordinary users, (b) standardized across vendors, and (c) able to **define** the schema, **modify** the data, and **query** it — all in one. RA and TRC do none of (a)–(c): they only *query*, only on paper, and only over sets.

SQL fills every gap. It is **declarative** like calculus (you say *what* you want, the optimizer — using RA equivalences from Topic 4 — decides *how*), but it adds the things real systems need: a **data-definition** sublanguage (`CREATE/ALTER/DROP`), a **data-manipulation** sublanguage (`INSERT/UPDATE/DELETE`), **NULLs** for missing information (with a matching three-valued logic), **aggregation/grouping** (which pure RA/TRC *cannot* express — a §4.10-of-Topic-5 limit), and **views**. So SQL is where the abstract querying theory of this whole section becomes the language you will actually be tested on writing and tracing.

---

## 2. Prerequisites

- **Relational model:** relation/table, tuple/row, attribute/column, schema, domain/type, keys (Topics 1–3).
- **Integrity constraints:** primary key, candidate key, foreign key, referential integrity — SQL declares these directly.
- **Relational algebra (Topic 4):** selection ↔ `WHERE`, projection ↔ `SELECT`, join/product/set-ops have direct SQL counterparts; the optimizer rewrites SQL using RA equivalences.
- **Tuple calculus (Topic 5):** `∃`/`∀` ↔ `EXISTS`/`NOT EXISTS`; the "for all" pattern `¬∃…¬∃` ↔ `NOT EXISTS … NOT EXISTS`.
- **Propositional / three-valued logic:** AND/OR/NOT — extended here with **UNKNOWN** for NULLs.

---

## 3. Core idea — intuition first

The heart of SQL is one statement shape:

```sql
SELECT   <columns / expressions>      -- what to output  (≈ projection π)
FROM     <tables, joins>              -- where to get it (≈ × / ⋈)
WHERE    <row condition>              -- which rows      (≈ selection σ)
GROUP BY <columns>                    -- (Part 2) bucket rows
HAVING   <group condition>            -- (Part 2) which buckets
ORDER BY <columns>                    -- sort the output (not in RA)
```

You **describe** the result; the system picks the execution plan. Read it as: "From these tables, keep the rows satisfying `WHERE`, then output these columns."

**The one idea that changes everything from Topics 4–5: SQL keeps duplicates (bag semantics).** A plain `SELECT col FROM T` returns *as many rows as T has* — repeats included. RA's projection would have collapsed them. To get RA-style set behaviour you must write `SELECT DISTINCT`. This affects row counts, `COUNT`, and set operations, and is a constant source of GATE questions.

The second idea: **real data has holes.** SQL uses **NULL** to mean "value missing/unknown/inapplicable." NULL is *not* a value you can compare with `=`; comparisons involving it yield a third truth value, **UNKNOWN**. Mastering this three-valued logic (§4.8) is the difference between a careful SQL solver and one who keeps losing marks.

---

## 4. Formal treatment

### 4.1 Data Definition Language (DDL)

```sql
CREATE TABLE Student (
    sid     INT          PRIMARY KEY,
    name    VARCHAR(50)  NOT NULL,
    dept    VARCHAR(10)  DEFAULT 'NA',
    cgpa    DECIMAL(3,2) CHECK (cgpa >= 0 AND cgpa <= 10),
    advisor INT,
    UNIQUE (name, dept),
    FOREIGN KEY (advisor) REFERENCES Faculty(fid)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);

ALTER TABLE Student ADD COLUMN email VARCHAR(80);
ALTER TABLE Student DROP COLUMN email;
DROP TABLE Student;          -- removes table + data
```

**Constraints (memorize behaviours):**

| Constraint | Meaning / behaviour |
|---|---|
| `PRIMARY KEY` | **UNIQUE + NOT NULL**; at most one per table; may span multiple columns (composite) |
| `UNIQUE` | no duplicate values; **NULLs allowed** (standard SQL permits *multiple* NULLs, since `NULL = NULL` is UNKNOWN) |
| `NOT NULL` | column may not hold NULL |
| `CHECK (cond)` | row rejected if `cond` evaluates to **FALSE** (note: **UNKNOWN passes** — a CHECK is satisfied unless explicitly false) |
| `DEFAULT v` | value used when none supplied on insert |
| `FOREIGN KEY` | value must match a key in the referenced table **or be NULL** |

**Referential actions** (on the referenced row being deleted/updated):

| Action | Effect on child (referencing) rows |
|---|---|
| `ON DELETE CASCADE` | child rows are **deleted** too |
| `ON DELETE SET NULL` | child FK set to **NULL** |
| `ON DELETE SET DEFAULT` | child FK set to its **default** |
| `ON DELETE RESTRICT` / `NO ACTION` | **prevents** the delete if any child exists |

> GATE trap: a chain of `ON DELETE CASCADE` foreign keys can delete rows in *multiple* tables from one `DELETE`. Trace the cascade.

---

### 4.2 Data Manipulation Language (DML)

```sql
INSERT INTO Student (sid, name) VALUES (1, 'Asha');     -- unlisted cols → DEFAULT/NULL
INSERT INTO Student VALUES (2, 'Ben', 'CSE', 8.5, 10);  -- positional, all columns

UPDATE Student SET cgpa = cgpa + 0.1 WHERE dept = 'CSE';
DELETE FROM Student WHERE cgpa < 4.0;                   -- no WHERE ⇒ deletes ALL rows
```

A `DELETE`/`UPDATE` with **no `WHERE`** affects **every** row — a classic exam (and real-world) gotcha.

---

### 4.3 Basic `SELECT`

```sql
SELECT [DISTINCT] col1, col2, expr AS alias
FROM   T
WHERE  condition
ORDER BY col1 [ASC|DESC], col2 DESC;
```

- **`*`** = all columns. **`DISTINCT`** = remove duplicate result rows (turns the bag into a set).
- **Column alias** (`AS`) names an output column. ⚠️ An alias defined in `SELECT` **cannot** be used in `WHERE` (WHERE runs *before* SELECT), but **can** be used in `ORDER BY` (which runs *after* SELECT).
- **`ORDER BY`** sorts the final output; by default `ASC`. NULLs sort together (vendor-dependent: typically NULLs last in `ASC` on many systems — GATE rarely tests the exact NULL sort position).

---

### 4.4 `WHERE` predicates (single-table filtering)

| Predicate | Meaning |
|---|---|
| `a = b`, `<`, `>`, `<=`, `>=`, `<>`/`!=` | comparisons |
| `x BETWEEN lo AND hi` | `x >= lo AND x <= hi` (**inclusive** both ends) |
| `x IN (v1, v2, …)` | `x = v1 OR x = v2 OR …` |
| `x NOT IN (…)` | negation — **dangerous with NULL** (Part 2) |
| `s LIKE pattern` | string pattern match |
| `x IS NULL` / `x IS NOT NULL` | **the only correct NULL test** (returns TRUE/FALSE, never UNKNOWN) |

**`LIKE` wildcards:** `%` = any string of **zero or more** characters; `_` = **exactly one** character. e.g. `name LIKE 'A%'` (starts with A), `name LIKE '_a%'` (second letter a), `name LIKE '%@gmail.com'`. Use `ESCAPE` to match a literal `%`/`_`.

---

### 4.5 Set operations (combine two compatible queries)

```sql
<query1> UNION     [ALL] <query2>     -- ∪
<query1> INTERSECT [ALL] <query2>     -- ∩
<query1> EXCEPT    [ALL] <query2>     -- − (Oracle spells it MINUS)
```

- **Operands must be union-compatible:** same number of columns, compatible types (column *names* need not match; the **first** query's names label the result).
- **Default = set semantics** (duplicates **removed**). Add **`ALL`** to keep duplicates (bag semantics).
- Counting with bag math: if a row appears `m` times in Q1 and `n` times in Q2, then in
  - `UNION ALL` it appears `m + n`; in `UNION` it appears `1` (if `m+n ≥ 1`).
  - `INTERSECT ALL` it appears `min(m, n)`; in `INTERSECT` it appears `1` if both have it.
  - `EXCEPT ALL` it appears `max(m − n, 0)`; in `EXCEPT` it appears `1` if `m ≥ 1, n = 0`.

---

### 4.6 Joins (combine two tables) — the workhorse

```sql
-- 1. CROSS JOIN  (Cartesian product, ×)
SELECT * FROM R CROSS JOIN S;                 -- |R| × |S| rows

-- 2. INNER JOIN with ON  (theta / equi join)
SELECT * FROM R JOIN S   ON R.b = S.b;        -- keeps BOTH copies of b
SELECT * FROM R JOIN S   ON R.x < S.y;        -- theta join (any condition)

-- 3. NATURAL JOIN  (⋈: auto-match all same-named columns, keep ONE copy)
SELECT * FROM R NATURAL JOIN S;

-- 4. JOIN ... USING (col)  (match on named column, keep ONE copy of it)
SELECT * FROM R JOIN S USING (b);

-- 5. OUTER JOINS  (preserve unmatched rows, pad with NULL)
SELECT * FROM R LEFT  OUTER JOIN S ON R.b = S.b;   -- all R rows kept
SELECT * FROM R RIGHT OUTER JOIN S ON R.b = S.b;   -- all S rows kept
SELECT * FROM R FULL  OUTER JOIN S ON R.b = S.b;   -- all rows from both
```

**Key facts (heavily tested):**
- `ON` keeps **both** join columns; `NATURAL`/`USING` keep **one**. This changes the result's **column count**.
- **`NATURAL JOIN` with no common column ⇒ Cartesian product** (silent explosion); with a column named the same but meaning different things ⇒ wrong matches. Be deliberate.
- **Inner join drops "dangling" rows** (rows with no match). **Outer joins keep them, padding the other side with NULL.**
- **Self-join:** join a table to itself using **aliases** (the SQL form of relational-algebra *rename* — Topic 4):
  ```sql
  SELECT e1.name, e2.name
  FROM Employee e1 JOIN Employee e2 ON e1.mgr = e2.eid;   -- employee & their manager
  ```

**Row-count reasoning for joins:**
- `CROSS`: exactly `|R| · |S|`.
- `INNER` on equality: `0 ≤ rows ≤ |R| · |S|`; **if the join column is a key in `S` and a foreign key in `R`** (referential integrity, non-null), then **exactly `|R|`** (the Topic-4 result, unchanged).
- `LEFT OUTER`: `rows = (#matched R-rows × their matches) + (#unmatched R-rows)` ; always `≥ |R|`.
- `FULL OUTER`: `= (inner-join rows) + (unmatched R-rows) + (unmatched S-rows)`.

---

### 4.7 SQL ↔ Relational Algebra quick map

| SQL | Relational algebra |
|---|---|
| `SELECT DISTINCT a,b FROM R WHERE c` | `π_{a,b}(σ_c(R))` |
| `SELECT * FROM R, S WHERE R.b=S.b` | `R ⋈_{R.b=S.b} S` (equi-join, both columns kept) |
| `R NATURAL JOIN S` | `R ⋈ S` |
| `UNION` / `INTERSECT` / `EXCEPT` | `∪` / `∩` / `−` |
| `R LEFT OUTER JOIN S` | `R ⟕ S` |

(Note: plain `SELECT` without `DISTINCT` has **no** clean RA equivalent because RA is set-based — bag projection isn't an RA operator.)

---

### 4.8 NULLs and **three-valued logic (3VL)** ⭐ (the big trap engine)

NULL means "unknown/missing/inapplicable." Two rules drive everything:

**Rule 1 — any comparison with NULL is `UNKNOWN`.**
`NULL = NULL` → **UNKNOWN**. `NULL = 5` → **UNKNOWN**. `NULL <> 5` → **UNKNOWN**. `NULL > 0` → **UNKNOWN**. The *only* tests that return TRUE/FALSE are **`IS NULL`** and **`IS NOT NULL`**.

**Rule 2 — `WHERE`/`ON`/`HAVING` keep a row only when the predicate is exactly `TRUE`.** Rows evaluating to `UNKNOWN` or `FALSE` are **dropped**. (But a `CHECK` constraint *passes* unless the predicate is `FALSE` — UNKNOWN is accepted. Opposite convention — watch it.)

**3VL truth tables** (T = true, F = false, U = unknown):

```
 AND | T  F  U          OR  | T  F  U           NOT
-----+---------        -----+---------         -------
  T  | T  F  U           T  | T  T  T            T → F
  F  | F  F  F           F  | T  F  U            F → T
  U  | U  F  U           U  | T  U  U            U → U
```

**NULL behaviour cheat-list:**
- `x IN (1, 2, NULL)` → TRUE if `x` is 1 or 2; else **UNKNOWN** (not FALSE).
- `x NOT IN (1, 2, NULL)` → **never TRUE** (best case UNKNOWN) ⇒ a NULL in a `NOT IN` subquery silently kills the result. *(Full treatment + fix in Part 2.)*
- **Aggregates ignore NULLs** (except `COUNT(*)`). *(Part 2.)*
- **`GROUP BY` and `DISTINCT` and set ops treat all NULLs as equal** (they group/dedup together) — even though `NULL = NULL` is UNKNOWN in `WHERE`. This **inconsistency** is deliberate and testable.
- `COALESCE(x, y, …)` returns the first non-NULL argument; `NULLIF(a,b)` returns NULL if `a=b` else `a`. Use `COALESCE` to substitute defaults.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Outer-join row & NULL counting (NAT)

```
R(A,B):  (1,x) (2,y) (3,z)            S(B,C):  (x,p) (x,q) (w,r)
```
How many rows are produced by **(i)** `R LEFT OUTER JOIN S ON R.B=S.B`, and **(ii)** `R FULL OUTER JOIN S ON R.B=S.B`?

**Solution.**
- Matches: `R.(1,x)` matches `S.(x,p)` and `S.(x,q)` → 2 rows `(1,x,p),(1,x,q)`. `R.(2,y)` and `R.(3,z)` have no match.
- **(i) LEFT OUTER:** matched rows (2) + unmatched-left padded `(2,y,NULL),(3,z,NULL)` (2) = **4 rows**.
- **(ii) FULL OUTER:** the 4 above **plus** unmatched-right `(w,r)` padded as `(NULL,w,r)` = **5 rows**.

(For an `INNER JOIN` the answer would be just the **2** matched rows.)

---

### Problem 2 — Three-valued logic in `WHERE` (MCQ)

Table `Emp(name, age, sal)` has a row `('Ravi', NULL, 60000)`. Is this row returned by
`SELECT * FROM Emp WHERE age > 30 OR sal > 50000;`? What about `WHERE age > 30 AND sal > 50000;`?

**Solution.** `age > 30` → `NULL > 30` → **UNKNOWN**. `sal > 50000` → `60000 > 50000` → **TRUE**.
- `OR`: `UNKNOWN OR TRUE = TRUE` ⇒ predicate TRUE ⇒ **row returned.**
- `AND`: `UNKNOWN AND TRUE = UNKNOWN` ⇒ not TRUE ⇒ **row dropped.**

Trap: many assume a NULL row is *always* excluded — but `OR TRUE` rescues it.

---

### Problem 3 — Set operations, bag vs. set (NAT)

Query Q1 returns the multiset `{a, a, b}` and Q2 returns `{a, b, b, c}`. How many rows result from **(i)** `Q1 UNION Q2`, **(ii)** `Q1 UNION ALL Q2`, **(iii)** `Q1 INTERSECT Q2`, **(iv)** `Q1 EXCEPT ALL Q2`?

**Solution.** Counts per row: in Q1 — a:2, b:1; in Q2 — a:1, b:2, c:1.
- **(i) UNION** (distinct): `{a, b, c}` → **3 rows**.
- **(ii) UNION ALL**: `2+1` a's, `1+2` b's, `1` c = `3+3+1` = **7 rows**.
- **(iii) INTERSECT** (distinct): values present in both = `{a, b}` → **2 rows**.
- **(iv) EXCEPT ALL**: `max(m−n,0)` per value: a:`max(2−1,0)=1`, b:`max(1−2,0)=0`, c:`max(0−1,0)=0` → **1 row** (`a`).

---

### Problem 4 — Self-join (MCQ)

`Employee(eid, name, mgr)` where `mgr` is the `eid` of the employee's manager. Write SQL to list each employee's name alongside their manager's name, **including employees with no manager**.

**Solution.** Self-join with **aliases**; use a **LEFT OUTER JOIN** so managerless employees (NULL `mgr`) are kept:

```sql
SELECT e.name AS employee, m.name AS manager
FROM Employee e
LEFT OUTER JOIN Employee m ON e.mgr = m.eid;
```

An inner join here would silently **drop** the top boss (whose `mgr` is NULL, matching nobody). The outer join pads `manager` with NULL for that row.

---

## 6. Common GATE question types on this topic (Part-1 scope)

- **Trace a join's output** — number of rows / columns / NULLs for inner vs. outer joins; the foreign-key ⇒ exactly `|R|` case.
- **Three-valued logic** — given NULLs, decide which rows a `WHERE` returns; evaluate AND/OR/NOT with UNKNOWN.
- **Set operations** — row counts under `UNION`/`INTERSECT`/`EXCEPT` and their `ALL` variants (bag arithmetic).
- **`DISTINCT` vs. not** — how many rows a projection returns (bag vs. set).
- **`LIKE` pattern matching** — which strings match a `%`/`_` pattern.
- **DDL/constraint behaviour** — what a `CHECK`/`UNIQUE`/`FK` allows; what `ON DELETE CASCADE` does across tables.
- **Write a query** — translate English to `SELECT–FROM–WHERE` with joins; self-joins for "compare a table to itself."

---

## 7. Traps, pitfalls & common mistakes (Part-1 scope)

1. **Bag vs. set.** Plain `SELECT col` keeps duplicates; only `DISTINCT` removes them. Forgetting this mis-counts results and breaks the analogy with RA's projection.
2. **`NULL = NULL` is UNKNOWN, not TRUE.** Never test NULL with `=`/`<>`; use `IS NULL`/`IS NOT NULL`. Joining/filtering on a NULL column silently loses rows.
3. **`UNKNOWN` rows are dropped by `WHERE`** but **accepted by `CHECK`** (opposite conventions).
4. **Grouping/`DISTINCT`/set-ops treat NULLs as equal**, yet `WHERE` treats `NULL=NULL` as UNKNOWN — the same NULLs behave differently in different clauses.
5. **`NATURAL JOIN` surprises:** no common column ⇒ Cartesian explosion; an accidental same-named column ⇒ unintended matching. Prefer explicit `ON`/`USING`.
6. **`ON` keeps both join columns; `NATURAL`/`USING` keep one** — affects the column count of the result (off-by-one in degree questions).
7. **Inner join drops unmatched rows.** When the question says "include those without a match," you need an **outer** join.
8. **`BETWEEN` is inclusive** of both endpoints; `IN`/`NOT IN` expand to OR/AND of equalities (and `NOT IN` is NULL-fragile).
9. **`DELETE`/`UPDATE` without `WHERE`** hits **all** rows.
10. **Alias scope:** a `SELECT` alias is usable in `ORDER BY` but **not** in `WHERE` (clause evaluation order — detailed in Part 2).

---

## 8. Connection forward

This part gave SQL's skeleton: define tables, change data, filter and combine rows, and reason about NULLs. **Part 2** adds the features that pure relational algebra/calculus *cannot* express — **aggregation and grouping** (`COUNT/SUM/AVG/MIN/MAX`, `GROUP BY`, `HAVING`) — and the **subquery** machinery (`IN`, `EXISTS`, `ANY`, `ALL`, correlated subqueries) that implements tuple-calculus quantifiers, including the **division / "for all"** pattern via `NOT EXISTS … NOT EXISTS`. The notorious **`NOT IN`-with-NULL** trap (foreshadowed in §4.8) is fully dissected there. After both parts, the syllabus turns to **Normalization** — designing schemas so these queries run over clean, redundancy-free tables.

---

## 9. One-line cheat-sheet summary (Part 1)

**SQL is a *bag*-based, declarative language: `SELECT(π)–FROM(×/⋈)–WHERE(σ)`; `DISTINCT` restores set semantics; joins range from `CROSS` to `INNER` (`ON` keeps both keys) to `OUTER` (NULL-pads unmatched rows); and every comparison with NULL is `UNKNOWN`, so `WHERE` keeps only `TRUE` rows while `IS NULL` is the sole reliable NULL test.**

---

### ▶ Continue to **SQL Part 2** (aggregation, `GROUP BY`/`HAVING`, subqueries, `NOT IN`-NULL trap, division, views) — provided alongside this file. The topic after SQL begins a **new chat: Normalization & Normal Forms.**

# Topic 6 — SQL (Part 2 of 2): Aggregation, Grouping, Subqueries, Division & Views
**Subject:** Database Management and Warehousing
**Section:** Querying (Chat 2) — final topic of the section
**This part covers:** aggregate functions + `COUNT` semantics, `GROUP BY`/`HAVING`, the clause **evaluation order**, nested **subqueries** (`IN`/`EXISTS`/`ANY`/`ALL`/correlated), the **`NOT IN`-NULL trap**, **division ("for all")** in SQL, and **views**.
*(Read Part 1 first — it covers DDL/DML, `SELECT–FROM–WHERE`, joins, set operations, and the NULL three-valued-logic basics this part relies on.)*

---

## 0. Topic header & syllabus map

- **Topic:** SQL (Structured Query Language) — Part 2.
- **Subject:** Database Management and Warehousing → *Querying* cluster (final topic).
- **Where it sits:** Part 1 covered defining/changing data and basic-to-intermediate querying. Part 2 covers the two areas GATE tests **hardest**: (i) **aggregation + grouping**, which pure relational algebra/tuple-calculus *cannot* express, and (ii) **nested subqueries**, which realize tuple-calculus quantifiers (`∃`/`∀`) and the **division** "for all" pattern. After SQL, the syllabus opens a **new chat: Normalization & Normal Forms**.

---

## 1. Why this topic exists (motivation + connection)

Part 1's `SELECT–FROM–WHERE` answers "**which rows**?" — but enormous classes of real questions are about **summaries across rows**: *how many* students per department, *average* salary per team, *which* departments have more than 50 employees, *which* supplier supplies *every* part. Relational algebra and tuple calculus **literally cannot** compute averages, counts, or "for all" totals (recall the expressiveness limits from Topic 5 §4.10). Part-1 SQL can't either, on its own.

This part adds exactly those powers. **Aggregation** (`COUNT/SUM/AVG/MIN/MAX`) collapses a set of rows to a single value; **`GROUP BY`** does it per bucket; **`HAVING`** filters the buckets. **Subqueries** let one query's result feed another, and **correlated** subqueries (`EXISTS`/`NOT EXISTS`) re-evaluate per outer row — directly mirroring tuple calculus's `∃t(…)`. Together they let SQL express the universal "for all" query (relational-algebra **division**) that no earlier topic could write cleanly. So Part 2 is where SQL surpasses the abstract languages of this section — and where the subtlest exam traps (NULLs in aggregates, the `NOT IN`-NULL bug, the `SELECT`-list rule under `GROUP BY`) live.

---

## 2. Prerequisites

- **SQL Part 1:** `SELECT–FROM–WHERE`, joins, set operations, and especially **NULL three-valued logic** (UNKNOWN propagation) — aggregation and `NOT IN` hinge on it.
- **Tuple calculus (Topic 5):** `∃`/`∀`, and the equivalence `∀x(P⇒Q) ≡ ¬∃x(P ∧ ¬Q)` — the basis of `EXISTS`/`NOT EXISTS` and of SQL division.
- **Relational algebra (Topic 4):** the **division** operator `R ÷ S` (the "for all" query SQL re-expresses here); aggregation corresponds to extended-RA `γ`.

---

## 3. Core idea — intuition first

Two new mental models on top of Part 1.

**(A) Aggregation = "collapse many rows into one number," optionally per group.** Plain `SELECT COUNT(*) FROM Emp` gives one number for the whole table. Add `GROUP BY dept` and SQL first **sorts rows into buckets** (one per department), then computes the aggregate **inside each bucket**, emitting **one output row per bucket**. `WHERE` filters rows *before* bucketing; `HAVING` filters *buckets* after. The non-negotiable rule: once you `GROUP BY`, the `SELECT` list may contain **only** grouping columns and aggregates — nothing else.

**(B) A subquery is "a query used as a value/set/test inside another query."** It can return a single value (scalar), a column of values (for `IN`/`ANY`/`ALL`), or just a yes/no of "did any row come back?" (`EXISTS`). When the inner query **references the outer row** it is **correlated** — think of it as a loop: "for each outer row, run the inner query." That loop is exactly tuple calculus's `∃`, which is why **`EXISTS`/`NOT EXISTS` is the clean, NULL-safe way** to write existence and "for all" queries — and why the older `IN`/`NOT IN` style is **fragile when NULLs appear** (§4.5, §7).

---

## 4. Formal treatment

### 4.1 Aggregate functions & `COUNT` semantics ⭐

Five standard aggregates: **`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`**.

```sql
SELECT COUNT(*), COUNT(col), COUNT(DISTINCT col),
       SUM(col), AVG(col), MIN(col), MAX(col)
FROM   T;
```

**The NULL rule (memorize exactly):**

| Function | Behaviour with NULLs / duplicates |
|---|---|
| `COUNT(*)` | counts **all rows**, including rows that are all-NULL and duplicate rows |
| `COUNT(col)` | counts rows where `col` **IS NOT NULL** (skips NULLs; keeps duplicates) |
| `COUNT(DISTINCT col)` | counts **distinct non-NULL** values of `col` |
| `SUM / AVG / MIN / MAX(col)` | **ignore NULLs** entirely |

**Consequences that get tested:**
- `AVG(col) = SUM(col) / COUNT(col)` where **both ignore NULLs** ⇒ NULLs are excluded from the **denominator** too. `AVG` of `{10, NULL, 20}` = `30/2 = 15`, **not** `30/3` and **not** `(10+0+20)/3`.
- On a column that is **entirely NULL** (or an **empty** table/group): `SUM/AVG/MAX/MIN → NULL`, but `COUNT(col) → 0` and `COUNT(*) → 0`. (So `SUM` of nothing is **NULL**, while `COUNT` of nothing is **0** — a frequent trap.)
- `SUM(DISTINCT col)`, `AVG(DISTINCT col)` exist and operate on distinct non-NULL values.
- An aggregate with **no `GROUP BY`** treats the **whole table as one group** and returns **exactly one row** (even if the table is empty).

---

### 4.2 `GROUP BY` and `HAVING`

```sql
SELECT   dept, COUNT(*) AS n, AVG(sal) AS avg_sal
FROM     Emp
WHERE    sal IS NOT NULL          -- row filter (before grouping)
GROUP BY dept
HAVING   COUNT(*) > 5             -- group filter (after grouping)
ORDER BY avg_sal DESC;
```

**Rules (each a GATE favourite):**
- **`SELECT`-list rule:** every non-aggregated column in `SELECT` (and `HAVING`/`ORDER BY`) **must** appear in `GROUP BY`. Selecting a bare non-grouped column is an **error** in standard SQL.
- **One output row per group.** `GROUP BY a, b` groups by the **combination** `(a,b)`.
- **NULLs form a single group** (all NULL values of the grouping column are bucketed together — grouping uses "not distinct from", unlike `WHERE`'s `=`).
- **`WHERE` vs `HAVING`:** `WHERE` filters **individual rows before** grouping and **cannot** contain aggregates; `HAVING` filters **whole groups after** aggregation and **can** use aggregates. `HAVING` without `GROUP BY` treats the whole table as one group.

---

### 4.3 Logical clause evaluation order ⭐

SQL is *written* `SELECT … FROM … WHERE …` but *logically evaluated* in this order:

```
FROM/JOIN  →  WHERE  →  GROUP BY  →  HAVING  →  SELECT  →  DISTINCT  →  ORDER BY  →  LIMIT/OFFSET
```

This single fact explains many rules:
- **`WHERE` can't use aggregates** (grouping hasn't happened yet) — use `HAVING`.
- **`SELECT` aliases can't be used in `WHERE`** (SELECT runs later) but **can** in `ORDER BY` (which runs last).
- **`HAVING` can reference aggregates** (grouping already done).

---

### 4.4 Subqueries (nested queries)

**(a) Scalar subquery** — returns a single value, usable wherever a value is expected:
```sql
SELECT name FROM Emp WHERE sal > (SELECT AVG(sal) FROM Emp);
```

**(b) `IN` / `NOT IN` (set membership)** — subquery returns a column:
```sql
SELECT name FROM Emp WHERE dept IN (SELECT dept FROM Dept WHERE loc='Pune');
```

**(c) `EXISTS` / `NOT EXISTS` (correlated)** — tests whether the subquery returns **any** row:
```sql
SELECT name FROM Dept d
WHERE EXISTS (SELECT 1 FROM Emp e WHERE e.dept = d.dept);   -- depts having ≥1 employee
```
- `EXISTS` is **TRUE** iff the inner query yields ≥ 1 row; `NOT EXISTS` iff 0 rows.
- **NULL-safe:** `EXISTS` only checks *whether a row exists*, never compares values, so NULLs inside the subquery don't break it (unlike `IN`/`NOT IN`).
- **Correlated** = inner query references an outer column (`d.dept` above); conceptually re-run per outer row.

**(d) `ANY`/`SOME` and `ALL` (quantified comparison):**
```sql
x > ANY (subquery)   -- TRUE iff x exceeds at least one value  (⇔ x > MIN)
x > ALL (subquery)   -- TRUE iff x exceeds every value         (⇔ x > MAX)
x = ANY (subquery)   -- equivalent to  x IN (subquery)
x <> ALL (subquery)  -- equivalent to  x NOT IN (subquery)
```
- **Empty-subquery edge cases:** `> ALL (∅)` is **TRUE** (vacuously); `> ANY (∅)` is **FALSE**.
- `ANY`/`ALL` carry the **same NULL fragility** as `IN`/`NOT IN`.

**(e) Subquery in `FROM` (derived table)** — must be aliased:
```sql
SELECT dept, mx FROM (SELECT dept, MAX(sal) AS mx FROM Emp GROUP BY dept) AS t
WHERE mx > 90000;
```

---

### 4.5 The `NOT IN`-with-NULL trap ⭐⭐ (most famous SQL bug)

`x NOT IN (v1, v2, …, NULL)` expands to `x<>v1 AND x<>v2 AND … AND x<>NULL`. Since `x<>NULL` is **UNKNOWN**, the whole `AND` is at best **UNKNOWN** — **never TRUE**. Therefore:

> **If a `NOT IN` subquery returns even one NULL, the outer query returns *no rows* (or far fewer than intended).**

```sql
-- BUG: if any employee has NULL mgr (e.g. the CEO), this returns ZERO rows
SELECT eid FROM Emp WHERE eid NOT IN (SELECT mgr FROM Emp);

-- FIX 1: exclude NULLs from the subquery
SELECT eid FROM Emp WHERE eid NOT IN (SELECT mgr FROM Emp WHERE mgr IS NOT NULL);

-- FIX 2 (preferred): use NOT EXISTS, which is NULL-safe
SELECT eid FROM Emp e WHERE NOT EXISTS
   (SELECT 1 FROM Emp m WHERE m.mgr = e.eid);
```

`IN` (positive) is less dangerous — a NULL in the list just can't *match*, so it only fails to add rows; but `NOT IN` actively **suppresses** rows. **Prefer `EXISTS`/`NOT EXISTS` whenever NULLs are possible.**

---

### 4.6 Division — the "for all" query in SQL ⭐

This is relational-algebra **`R ÷ S`** (Topic 4) and tuple-calculus's `∀` (Topic 5), realized in SQL. Schema: `Student(sid)`, `Course(cid)`, `Enrolled(sid,cid)`. *"Find students enrolled in **every** course."*

**Method 1 — double `NOT EXISTS`** (direct translation of `¬∃c(¬∃e…)`):
```sql
SELECT s.sid FROM Student s
WHERE NOT EXISTS (                       -- there is no course ...
        SELECT 1 FROM Course c
        WHERE NOT EXISTS (               -- ... that the student is NOT enrolled in
                SELECT 1 FROM Enrolled e
                WHERE e.sid = s.sid AND e.cid = c.cid));
```
*Read:* "students for whom there is **no** course they are **not** enrolled in." NULL-safe and the canonical exam answer.

**Method 2 — counting** (works when `Enrolled.cid` only holds valid course ids):
```sql
SELECT sid FROM Enrolled
GROUP BY sid
HAVING COUNT(DISTINCT cid) = (SELECT COUNT(*) FROM Course);
```
*Read:* "students whose number of distinct courses equals the total number of courses." Simpler, but relies on referential integrity (no `Enrolled` rows pointing to non-existent courses) and on `DISTINCT` to avoid double-counting.

---

### 4.7 Views

```sql
CREATE VIEW HighEarners AS
   SELECT eid, name, sal FROM Emp WHERE sal > 100000;

SELECT * FROM HighEarners WHERE name LIKE 'A%';   -- query a view like a table
DROP VIEW HighEarners;
```

- A **view** is a **virtual table** — a stored query, computed on demand; it holds **no data of its own** (a *materialized view* does store results, refreshed periodically).
- **Benefits:** abstraction, security (expose only some columns/rows), query reuse.
- **Updatable-view conditions (roughly):** insert/update/delete through a view is allowed only when the system can map changes back to a single base table — typically the view is over **one table**, with **no aggregates, no `DISTINCT`, no `GROUP BY`/`HAVING`, no set operations**, and includes every `NOT NULL` (no-default) column of the base table. Views with aggregation/joins are generally **read-only**.

---

### 4.8 Beyond core (one-line awareness)

- **CTE / `WITH`:** names a subquery for readability; `WITH RECURSIVE` adds **recursion** (transitive closure) — the capability that pure RA/calculus *lacked* (Topic 5 §4.10).
- **Window functions** (`OVER(PARTITION BY … ORDER BY …)`, e.g. `RANK()`, `ROW_NUMBER()`): aggregate-style computations **without collapsing rows**. Useful to know exists; rarely the core of a GATE DA question.

---

## 5. Worked problems (GATE-style)

### Problem 1 — `COUNT` variants & `AVG` with NULL (NAT)

Table `Marks(sid, score)` has rows: `(1,80) (2,90) (3,NULL) (4,80) (5,NULL)`. Evaluate `COUNT(*)`, `COUNT(score)`, `COUNT(DISTINCT score)`, `SUM(score)`, `AVG(score)`.

**Solution.**
- `COUNT(*)` = all rows = **5**.
- `COUNT(score)` = non-NULL scores = `{80,90,80}` = **3**.
- `COUNT(DISTINCT score)` = distinct non-NULL = `{80,90}` = **2**.
- `SUM(score)` = `80+90+80` = **250** (NULLs ignored).
- `AVG(score)` = `SUM/COUNT(score)` = `250/3 ≈` **83.33** (NULLs excluded from **both** numerator and denominator — *not* `250/5 = 50`).

---

### Problem 2 — `GROUP BY` + `HAVING`, count the output rows (NAT)

```
Sales(region, product, amt):
 (N, X, 10) (N, Y, 20) (N, X, 30) (S, X, 40) (S, Z, 50) (W, X, 5)
```
How many rows does this return?
```sql
SELECT region, COUNT(*) FROM Sales GROUP BY region HAVING SUM(amt) >= 60;
```

**Solution.** Group by region → totals: **N** = 10+20+30 = 60, **S** = 40+50 = 90, **W** = 5. `HAVING SUM(amt) >= 60` keeps groups with total ≥ 60 → **N (60)** and **S (90)**; drops **W (5)**. Output = **2 rows**: `(N,3), (S,2)`.

---

### Problem 3 — The `NOT IN`-NULL trap (MCQ, classic)

`Emp(eid, mgr)` with rows `(1,NULL) (2,1) (3,1) (4,2)` (employee 1 is the CEO, no manager). What does this return, and why?
```sql
SELECT eid FROM Emp WHERE eid NOT IN (SELECT mgr FROM Emp);
```

**Solution.** Subquery `SELECT mgr` = `{NULL, 1, 1, 2}` — **contains NULL**. So `eid NOT IN (NULL,1,1,2)` ⇒ `eid<>NULL AND eid<>1 AND eid<>2`; the term `eid<>NULL` is **UNKNOWN**, making the whole condition **never TRUE**. **Result: 0 rows** — *not* the intended "non-managers" `{3,4}`. Fix with `WHERE mgr IS NOT NULL` in the subquery, or rewrite using `NOT EXISTS`.

---

### Problem 4 — Division: "supplies all parts" via `NOT EXISTS` (write-the-query)

`Supplier(sid)`, `Part(pid)`, `Catalog(sid,pid)`. Write SQL for *"suppliers who supply every part."*

**Solution.**
```sql
SELECT s.sid FROM Supplier s
WHERE NOT EXISTS (
        SELECT 1 FROM Part p
        WHERE NOT EXISTS (
                SELECT 1 FROM Catalog c
                WHERE c.sid = s.sid AND c.pid = p.pid));
```
*Reasoning:* a supplier qualifies iff there is **no** part that the supplier does **not** supply — the double-negation form of "for all parts, the supplier supplies it." (Equivalently: `GROUP BY sid HAVING COUNT(DISTINCT pid) = (SELECT COUNT(*) FROM Part)`.)

---

### Problem 5 — `ANY` vs `ALL` (MCQ)

`Emp(eid, dept, sal)`. What does each return?
**(i)** `SELECT eid FROM Emp WHERE sal > ALL (SELECT sal FROM Emp WHERE dept='CSE');`
**(ii)** `SELECT eid FROM Emp WHERE sal > ANY (SELECT sal FROM Emp WHERE dept='CSE');`

**Solution.**
- **(i) `> ALL`** ⇔ `sal > MAX(CSE salaries)` → employees paid **more than every** CSE employee (i.e., above the CSE maximum).
- **(ii) `> ANY`** ⇔ `sal > MIN(CSE salaries)` → employees paid **more than at least one** CSE employee (i.e., above the CSE minimum).
- **Edge case:** if no CSE employees exist, the subquery is empty: `> ALL (∅)` is **TRUE** (everyone qualifies), `> ANY (∅)` is **FALSE** (nobody qualifies).

---

## 6. Common GATE question types on this topic (Part-2 scope)

- **Compute aggregates with NULLs/duplicates** — especially `COUNT(*)` vs `COUNT(col)` vs `COUNT(DISTINCT)`, and `AVG` excluding NULLs from the denominator.
- **Count output rows of a `GROUP BY [HAVING]` query**; identify the number of groups.
- **`WHERE` vs `HAVING`** placement; the `SELECT`-list-must-be-grouped rule; clause evaluation order (alias scope, where aggregates are legal).
- **Subquery semantics** — `IN`/`EXISTS`/`ANY`/`ALL`; correlated vs uncorrelated; what a nested query returns.
- **The `NOT IN`-NULL trap** — "why does this query return no rows?"
- **Division / "for all"** — write or recognise the `NOT EXISTS … NOT EXISTS` (or `COUNT = COUNT`) pattern.
- **Views** — virtual vs materialized; when a view is updatable.
- **Equivalence/trace** — "which of these queries are equivalent?" or "what is the exact output?"

---

## 7. Traps, pitfalls & common mistakes (Part-2 scope)

1. **`NOT IN` + NULL ⇒ empty result.** The headline SQL bug. One NULL in the subquery suppresses *all* rows. Prefer `NOT EXISTS`, or filter `IS NOT NULL`.
2. **Aggregates ignore NULLs — except `COUNT(*)`.** `AVG` divides by the **non-NULL** count; `SUM`/`AVG`/`MIN`/`MAX` of an all-NULL or empty set is **NULL**, but `COUNT` is **0**.
3. **`COUNT(*)` ≠ `COUNT(col)`.** `*` counts every row (NULLs, duplicates included); `COUNT(col)` skips NULLs; add `DISTINCT` to dedup.
4. **`SELECT`-list rule under `GROUP BY`:** every non-aggregated selected column must be in `GROUP BY`, else it's an error. Don't select bare ungrouped columns.
5. **`WHERE` cannot hold aggregates; `HAVING` can.** `WHERE COUNT(*)>5` is illegal — use `HAVING`. And `WHERE` filters *rows before* grouping, `HAVING` filters *groups after*.
6. **Alias scope (evaluation order):** a `SELECT` alias is invalid in `WHERE` (SELECT runs after WHERE) but valid in `ORDER BY`.
7. **`> ALL`/`> ANY` reduce to MAX/MIN** — and **empty-subquery** flips them: `ALL(∅)=TRUE`, `ANY(∅)=FALSE`. `= ANY` is `IN`; `<> ALL` is `NOT IN` (with the same NULL hazard).
8. **`EXISTS` ignores the subquery's `SELECT` list** — `SELECT 1`, `SELECT *`, or `SELECT col` are all equivalent; only *whether a row exists* matters, so NULLs don't break it.
9. **Correlated subquery re-evaluates per outer row** — confusing it with an uncorrelated one changes the meaning (and the cost).
10. **Grouping puts all NULLs in one bucket**, even though `WHERE` would treat `NULL=NULL` as UNKNOWN — same NULLs, different rule by clause.
11. **Division via `COUNT = COUNT` needs `DISTINCT` and referential integrity**; without `DISTINCT`, duplicate enrolments inflate the count; with stray course ids, the equality breaks. The `NOT EXISTS` form is safer.
12. **A view stores no data** (unless materialized); querying it re-runs the underlying query, and most aggregated/joined views are **read-only**.

---

## 8. Connection forward

SQL completes the *Querying* section: you can now define data (Part 1), filter and join it (Part 1), and summarize, nest, and quantify over it (Part 2) — including the "for all" queries that pure relational algebra/calculus could only describe abstractly. But every query in this section silently **assumed the tables were well-designed**. They often aren't: a badly structured schema causes update/insert/delete **anomalies** and stores the same fact redundantly. The next chat — **Normalization & Normal Forms** — uses **functional dependencies** to *diagnose* such schemas and decompose them (1NF → 2NF → 3NF → BCNF) into clean, anomaly-free tables. In other words, querying told us how to *ask*; normalization tells us how to *store* so the asking stays correct and efficient.

---

## 9. One-line cheat-sheet summary (Part 2)

**Aggregates (`COUNT/SUM/AVG/MIN/MAX`) collapse rows and *ignore NULLs* — except `COUNT(*)`; `GROUP BY` emits one row per bucket (only grouped cols + aggregates in `SELECT`), `WHERE` filters rows before grouping while `HAVING` filters groups after; subqueries realize calculus quantifiers (`EXISTS`/`NOT EXISTS` are NULL-safe, `NOT IN` is *not* — one NULL empties it), and "for all"/division is written as `NOT EXISTS … NOT EXISTS`.**

---

### ▶ Next topic — **new chat: Topic 7 — Normalization & Normal Forms** (functional dependencies; 1NF, 2NF, 3NF, BCNF; decomposition, lossless join, dependency preservation). This closes the Querying section.

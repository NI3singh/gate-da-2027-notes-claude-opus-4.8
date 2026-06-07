# Topic 11 — Multidimensional Data Models
### (Schemas: Star/Snowflake/Fact-Constellation · Concept Hierarchies · Measures: Categorization & Computations)

*Subject: Database Management and Warehousing · GATE DA notes · **final topic of the subject***

---

## 0. Topic header & syllabus map

- **Topic:** Multidimensional data models — the **schema** for multidimensional data, **concept hierarchies**, and **measures** (their categorization and computation).
- **Subject:** Database Management and Warehousing.
- **Where it sits:** This is the **last topic** of the "Data Transformation & Warehousing" section, and of the whole subject. It comes right after **Data Transformation (Topic 10)**, which cleaned, scaled, discretized and reduced the data. Here that prepared data is finally **organised for analysis** — into data cubes, warehouse schemas, hierarchies, and aggregable measures. There is **no next topic in this subject**; this completes the warehousing arc.

> Mental model: Topic 10 was the **prep kitchen** (clean & cut the ingredients). This topic is the **plating** — arranging everything into the multidimensional structure that OLAP and data mining consume.

---

## 1. Why this topic exists (motivation + connection)

Operational databases (OLTP) are built for **transactions**: insert an order, update a balance, normalized to avoid anomalies (Topics 1–9). But analysts ask **aggregate, multi-dimensional questions**: "total sales of *electronics* in *Gujarat* per *quarter*." Answering that on a normalized OLTP schema means huge multi-way joins and `GROUP BY`s over millions of rows — slow and awkward.

A **data warehouse** instead stores data in a **multidimensional model** (a *data cube*) so these aggregate questions are natural and fast.

**What the previous topic (Data Transformation) could NOT do:** Topic 10 made individual attribute *values* clean, comparable and compact — but it did **not** decide how to *organise* the data for analytical querying. A z-scored, discretized table is still just a flat table; it can't roll up sales by region and time without an aggregation structure. The multidimensional model supplies that structure.

**Direct bridge from Topic 10:** the **set-grouping concept hierarchies** here (e.g., `price ∈ {1–10} → "cheap"`) are produced exactly by the **discretization/binning** you learned in Topic 10. Discretization is the *generator*; concept hierarchies are the *result used for roll-up/drill-down*.

**What this enables (forward, into other subjects):** the data cube and its measures are the substrate for **OLAP analysis and data mining** — association rules, classification, clustering — which appear in the Machine Learning / Data Mining portions of GATE DA.

---

## 2. Prerequisites

- **Relational model & SQL** (Topics 2, 6) — fact/dimension tables are relations; cube cells are `GROUP BY` aggregates.
- **Normalization (Topic 7)** — snowflake schemas are *normalized* dimension tables; star schemas are *denormalized*.
- **Discretization / binning (Topic 10)** — generates set-grouping concept hierarchies.
- Basic **aggregate functions**: `COUNT, SUM, MIN, MAX, AVG`, and statistics (variance, median, mode).
- **Counting / products** (for counting cuboids).

---

## 3. Core idea — intuition first

### 3.1 The data cube

Picture sales data along **three axes**: *time*, *item*, *location*. A single cell `(Q1, laptop, Ahmedabad)` holds a **measure** like `total_sales = ₹X`. That 3-axis grid is a **3-D data cube**. With more axes (supplier, customer, …) it becomes an **n-D cube** (a "hypercube" — we just can't draw it).

- **Dimensions** = the perspectives/axes you analyse by (*time, item, location*). Each dimension has a **dimension table** of descriptive attributes.
- **Facts / measures** = the numeric quantities being analysed (*sales amount, units sold*), stored in a **fact table**.
- A **cuboid** = the cube aggregated at one particular combination of dimensions. The **base cuboid** keeps all dimensions (most detailed); the **apex cuboid** (denoted `all`) aggregates everything into one grand total.

### 3.2 Schemas — three ways to lay the cube out in tables

- **Star schema:** one central **fact table** surrounded by **one table per dimension** (dimensions kept flat/denormalized). Diagram looks like a **star**.
- **Snowflake schema:** like a star, but dimension tables are **normalized** into sub-tables (hierarchies split out). Diagram branches like a **snowflake**.
- **Fact constellation (galaxy):** **multiple fact tables** that **share** dimension tables — for warehouses covering several subjects (sales *and* shipping).

### 3.3 Concept hierarchies — "zoom levels"

A **concept hierarchy** maps low-level concepts to higher ones: `street → city → state → country`. They give you the **zoom control**: **roll up** (zoom out, aggregate) and **drill down** (zoom in, detail).

### 3.4 Measures — and *why their type matters*

A measure is just a number per cube cell, but **how cheaply you can build the whole cube depends on the measure's type**:

- `SUM` of a big region = sum of sums of its parts → easy to combine (**distributive**).
- `AVG` = `SUM / COUNT` → combine two easy pieces (**algebraic**).
- `MEDIAN` of a region ≠ any simple combination of medians of parts → you essentially need all the raw values (**holistic**, expensive).

This three-way split (distributive / algebraic / holistic) is the single most testable idea of the topic.

---

## 4. Formal treatment

### 4A. The multidimensional model & the cuboid lattice

- A **data cube** allows data to be modelled and viewed in **n dimensions**.
- **Cuboids:** every possible "group-by" on a subset of the dimensions is one cuboid. They form a **lattice** from the **base cuboid** (all n dimensions) down to the **apex cuboid** `all` (0 dimensions).

**Counting cuboids (key NAT formula):**

$$
\boxed{\;T \;=\; \prod_{i=1}^{n}\,(L_i + 1)\;}
$$

- $n$ = number of dimensions; $L_i$ = number of **levels** in dimension $i$'s concept hierarchy (not counting the virtual `all`). The **"+1"** adds the `all`/apex level for that dimension.
- **Special case — no hierarchies** (every dimension a single level, $L_i = 1$):
  $$
  T = 2^{\,n}.
  $$
  (Each dimension is either *present* or *rolled up to `all`* → 2 choices per dimension.)
- **Conditions/assumptions:** the formula counts the **full** cube (all materializable cuboids). Real systems often materialize only *some* cuboids (partial materialization) to save space.

---

### 4B. Schemas — formal contents and trade-offs

**Fact table** holds: **foreign keys** to each dimension table (together forming the key) **+ the measures** (numeric facts). It is typically **large and "thin-but-tall."**

**Dimension table** holds: a **surrogate/primary key + descriptive attributes**, often arranged in a hierarchy.

| | **Star** | **Snowflake** | **Fact constellation (galaxy)** |
|---|---|---|---|
| Fact tables | 1 | 1 | **≥ 2, sharing dimensions** |
| Dimension tables | **Denormalized** (one per dimension) | **Normalized** (split into sub-tables) | shared dimension tables |
| Redundancy | higher | lower | depends |
| Joins per query | **fewer → faster** | more → can be slower | varies |
| Storage | more | **less** | most complex |
| Maintenance | simpler | easier to update consistently | complex |
| Use | most common, query-fast warehouses | storage-sensitive / cleanly hierarchical dims | multi-subject enterprise warehouses |

> **Star vs snowflake is exactly the denormalization↔normalization trade-off from Topic 7**, applied to dimension tables: snowflake normalizes (less redundancy, more joins); star denormalizes (faster reads, some redundancy).

*(Beyond-core, one line for awareness: warehouse **servers** are classified as **ROLAP** (relational back-end), **MOLAP** (array/cube store), or **HOLAP** (hybrid). GATE rarely goes past the definitions.)*

---

### 4C. Concept hierarchies — types & ordering

A **concept hierarchy** is a sequence of mappings from low-level to higher-level concepts on a dimension.

**By how they're defined:**
- **Schema hierarchy:** an order among **attributes** in the schema, e.g. `street < city < province_or_state < country`.
- **Set-grouping hierarchy:** groups **values** of one attribute into ranges/sets, e.g. `{1–10} < "inexpensive"`, `{Ahmedabad, Surat} < "Gujarat"`. **Generated by discretization (Topic 10).**
- **Operation-derived / rule-based:** defined by encoded rules or operations (e.g., split an email into user/domain).

**By ordering structure:**
- **Total order (linear):** each level rolls up to exactly one parent level —
  $$
  \text{street} < \text{city} < \text{province\_or\_state} < \text{country}.
  $$
- **Partial order (lattice):** a level may roll up to several —
  $$
  \text{day} < \{\text{week},\ \text{month} < \text{quarter} < \text{year}\}.
  $$
  Here `day` rolls up to both `week` and `month`, but `week` does **not** cleanly roll up to `month` → it's a lattice, not a line.

Hierarchies may be **provided manually** (experts/users) or **generated automatically** (via discretization, or by ordering categorical attributes by their **number of distinct values** — the attribute with the **fewest** distinct values sits **highest**, e.g., `country` (few) above `street` (many)).

---

### 4D. OLAP operations (navigation on the cube)

| Operation | What it does | Effect on dimensionality |
|---|---|---|
| **Roll-up** (drill-up) | Climb a concept hierarchy *or* drop a dimension → **aggregate, coarser** | reduces detail / dimensions |
| **Drill-down** (roll-down) | Reverse of roll-up: step **down** a hierarchy *or* add a dimension → **finer** | increases detail / dimensions |
| **Slice** | **Selection on one** dimension (fix it to a single value) | produces a **subcube of one fewer** dimension |
| **Dice** | **Selection on two or more** dimensions | produces a subcube |
| **Pivot (rotate)** | Rotate the axes for an alternative view (e.g., swap rows/columns) | presentation only |
| *Drill-across* | Query **across multiple fact tables** | — |
| *Drill-through* | Go from the cube down to the **back-end relational/raw** data | — |

Mnemonic: **roll-up = zoom out, drill-down = zoom in, slice = one cut, dice = a box, pivot = turn it.**

---

### 4E. Measures — categorization & computation

A **measure** is a numeric function evaluated at each cube cell; its value is obtained by **aggregating** the data mapped to that cell. Categorize by **how the aggregate composes over a partition** of the data:

**(1) Distributive.** Computable by partitioning the data, aggregating each part, then **combining the partial results with the same function** (no extra info needed).
$$
f(D) = g\bigl(f(D_1), f(D_2), \dots, f(D_k)\bigr)
$$
- `COUNT` (combine by `SUM` of counts), `SUM`, `MIN`, `MAX`.

**(2) Algebraic.** Computable by an **algebraic function of a bounded number $M$ of distributive measures**.
- `AVG = SUM / COUNT` (M = 2 distributive pieces).
- **Standard deviation / variance** — needs `SUM(x)`, `SUM(x²)`, `COUNT` (3 distributive pieces, combined algebraically):
  $$
  \sigma^2=\frac{\sum x_i^2}{N}-\left(\frac{\sum x_i}{N}\right)^2 .
  $$
- `MIN_N`, `MAX_N` (the N largest/smallest), `center_of_mass`.

**(3) Holistic.** **No constant bound** on the storage needed to describe a sub-aggregate — cannot be computed from a bounded set of distributive pieces; effectively needs the whole sub-population.
- `MEDIAN`, `MODE`, `RANK`.

**Why it matters:** distributive and algebraic measures let the cube be built **incrementally** from sub-cuboids (cheap, parallelizable). Holistic measures force (near-)full recomputation per cell → **expensive**; systems use approximations for them.

| Measure | `COUNT` | `SUM` | `MIN`/`MAX` | `AVG` | `VAR`/`SD` | `MIN_N`/`MAX_N` | `MEDIAN` | `MODE` | `RANK` |
|---|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| Class | Distrib. | Distrib. | Distrib. | **Algebraic** | **Algebraic** | **Algebraic** | **Holistic** | **Holistic** | **Holistic** |

---

## 5. Worked problems (GATE-style)

### Problem 1 — Counting cuboids, no hierarchy *(NAT)*
A data cube has **5 dimensions**, each with a single level. How many cuboids (including base and apex)?

$$
T = 2^{5} = 32.
$$
**Answer: 32.**

---

### Problem 2 — Counting cuboids with hierarchies *(NAT)*
A cube has 3 dimensions with concept-hierarchy levels: **time = 4**, **item = 2**, **location = 3** (levels excluding `all`). Total cuboids?

$$
T=(4{+}1)(2{+}1)(3{+}1)=5\times3\times4=60.
$$
**Answer: 60.** *(Trap: don't forget the "+1" per dimension for the `all` level.)*

---

### Problem 3 — Measure categorization *(MSQ)*
Classify each as distributive (D), algebraic (A), or holistic (H):
`COUNT, SUM, MAX, AVG, STANDARD_DEVIATION, MEDIAN, MODE, MIN_N`.

- `COUNT` → **D** · `SUM` → **D** · `MAX` → **D**
- `AVG` → **A** (= SUM/COUNT) · `STANDARD_DEVIATION` → **A** (sum, sum-of-squares, count) · `MIN_N` → **A**
- `MEDIAN` → **H** · `MODE` → **H**

**The trap pairs:** `AVG` and `STANDARD_DEVIATION` are **algebraic, not holistic**; `MEDIAN`/`MODE` are **holistic, not algebraic**.

---

### Problem 4 — Schema identification *(MCQ)*
A warehouse has **two fact tables**, `Sales` and `Shipping`, that **share** the `time`, `item`, and `location` dimension tables. This is a:
(a) Star (b) Snowflake (c) **Fact constellation (galaxy)** (d) Normalized OLTP schema

**Answer: (c)** — multiple fact tables sharing dimensions ⇒ fact constellation.

---

### Problem 5 — Star vs snowflake *(MCQ)*
Which statement is **correct**?
(a) Snowflake has more data redundancy than star.
(b) Star requires more joins than snowflake.
(c) **Snowflake normalizes dimension tables, reducing redundancy at the cost of more joins.**
(d) Fact constellation always has exactly one fact table.

**Answer: (c).** (a) and (b) are reversed; (d) is false by definition.

---

### Problem 6 — OLAP operations *(matching / MCQ)*
Identify each operation:
1. Viewing sales **by year** instead of **by quarter** → **roll-up** (climb time hierarchy).
2. Going from sales **per city** to **per street** → **drill-down**.
3. Fixing **time = "Q1"** and viewing the remaining (item × location) cube → **slice**.
4. Selecting **time ∈ {Q1,Q2}** AND **location ∈ {Gujarat, Maharashtra}** → **dice**.
5. Swapping the row and column axes of a report → **pivot**.

---

### Problem 7 — Concept-hierarchy ordering *(MCQ — trap)*
For dimension `time`, which is a **partial order (lattice)** rather than a total order?
(a) `street < city < state < country`
(b) **`day < {week, month < quarter < year}`**
(c) `second < minute < hour`
(d) `cheap < moderate < expensive`

**Answer: (b)** — `day` rolls up to both `week` and `month`, and `week` ↛ `month`, so it's a lattice. The others are linear chains.

---

### Problem 8 — Holistic vs algebraic reasoning *(NAT/conceptual trap)*
A cube is partitioned into 4 regions. For which measures can the **whole-cube value** be recovered using **only one stored number per region** (plus the same kind of value)?

- `SUM`: yes (one partial sum per region). **Distributive.**
- `MAX`: yes (one partial max per region). **Distributive.**
- `AVG`: **no** — one number per region (the average) is **insufficient**; you need *two* per region (sum and count). It's algebraic, **not** distributive.
- `MEDIAN`: **no** — even multiple summary numbers per region won't suffice in general; you need (almost) all raw values. **Holistic.**

**Takeaway:** "one summary number per partition suffices" ⇔ **distributive**; AVG needs a *bounded* set of such numbers ⇒ algebraic; MEDIAN needs an *unbounded* description ⇒ holistic.

---

## 6. Common GATE question types on this topic

- **Count the cuboids** for $n$ dimensions: $2^n$ (no hierarchy) or $\prod(L_i+1)$ (with levels) *(NAT)*.
- **Classify a measure** as distributive / algebraic / holistic *(MSQ)* — especially the AVG, variance/SD (algebraic) vs median, mode (holistic) traps.
- **Identify the schema** (star vs snowflake vs fact constellation) from a description or diagram.
- State **star↔snowflake trade-offs** (normalization, joins, redundancy, speed).
- **Name/identify the OLAP operation** (roll-up, drill-down, slice, dice, pivot) from a scenario.
- **Total vs partial order** in a concept hierarchy; which attribute sits highest (fewest distinct values).
- Identify **fact table vs dimension table** contents (keys + measures vs descriptive attributes).

---

## 7. Traps, pitfalls & common mistakes

- **Cuboid count "+1":** with hierarchies the formula is $\prod(L_i+1)$, **not** $\prod L_i$ — the `+1` is the `all` (apex) level per dimension. And $2^n$ is just the special case $L_i=1$.
- **AVG is algebraic, not holistic** — storing only the average per partition loses information; you need (sum, count). Many students wrongly call it holistic.
- **Variance/standard deviation are algebraic** (need sum, sum-of-squares, count — a *bounded* 3 distributive pieces), **not holistic**.
- **MEDIAN, MODE, RANK are holistic** — no constant-size summary suffices. Don't call median "algebraic."
- **Star vs snowflake reversed:** **snowflake = normalized** dimensions (fewer redundancies, **more** joins); **star = denormalized** (more redundancy, **fewer** joins, faster reads). Don't flip these.
- **Fact constellation ⇔ multiple fact tables sharing dimensions** — not "a bigger star."
- **Slice (one dimension) vs dice (two+ dimensions):** slice fixes a single dimension to one value → subcube of one fewer dimension; dice selects ranges/sets on **multiple** dimensions.
- **Roll-up vs drill-down direction:** roll-up = **up** the hierarchy = **more aggregated/coarser**; drill-down = **down** = **more detailed**.
- **Total vs partial order:** a hierarchy is a **partial order (lattice)** the moment one level has **more than one parent** (e.g., `day → week` and `day → month`).
- **Highest level = fewest distinct values** (when auto-generating hierarchies): `country` (few values) is above `street` (many) — students sometimes invert this.
- **Fact table key:** it's the **combination of the dimension foreign keys**, with measures alongside — not a single arbitrary id with all attributes inlined.
- **Don't confuse "dimension" with "level":** a *dimension* is an axis (time); its *levels* (day/month/year) are the hierarchy within that axis. The cuboid formula uses **levels per dimension**.

---

## 8. Connection forward

This **completes the Database Management & Warehousing subject.** The full arc was: **model the data** (ER → relational → integrity, Topics 1–3) → **query it** (relational algebra, tuple calculus, SQL, Topics 4–6) → **design its storage** (normalization, file organization, indexing, Topics 7–9) → **transform it for analysis** (normalization/scaling, discretization, sampling, compression, Topic 10) → **model it multidimensionally for analytics** (this topic). From here the data cube and its measures feed directly into **OLAP and data mining** (association rules, classification, clustering), which live in the **Machine Learning / Data Mining** portions of the GATE DA syllabus — so the warehouse you just modelled is the launchpad for everything analytical that follows.

---

## 9. One-line cheat-sheet summary

**Multidimensional model = data cube of dimensions × measures; cuboids number $2^n$ (no hierarchy) or $\prod(L_i+1)$ (with levels); lay it out as a star (denormalized dims, fast), snowflake (normalized dims, fewer redundancies), or fact constellation (shared dims, many fact tables); navigate with roll-up/drill-down/slice/dice/pivot along concept hierarchies (total = linear, partial = lattice; generated by discretization); and classify measures as distributive (count/sum/min/max), algebraic (avg, variance/SD, min_N), or holistic (median, mode, rank), since that decides how cheaply the cube computes.**

---

### ✅ Subject complete
This is the **final topic of Database Management and Warehousing** — there is no next topic in this subject. The natural continuation is the **Data Mining / Machine Learning** material, where these cubes and measures get analysed. Ask whenever you want to start that subject (or revisit any earlier DBMS/warehousing topic).

# Topic 10 — Data Types & Data Transformation
### (Normalization/Scaling · Discretization · Sampling · Compression)

*Subject: Database Management and Warehousing · GATE DA notes*

---

## 0. Topic header & syllabus map

- **Topic:** Data Types & Data Transformation — normalization/scaling, discretization, sampling, compression.
- **Subject:** Database Management and Warehousing.
- **Where it sits:** This is the **first topic of the "Data Transformation & Warehousing" section**. It comes right after **Indexing (Topic 9)**, which was about *physically storing and retrieving* data fast. It leads into **Multidimensional Data Models (Topic 11)**, where the cleaned/transformed data is organised into warehouse schemas, and into **Concept Hierarchies (Topic 12)**, which are literally built using the **discretization** machinery introduced here.

> Mental model: Topics 1–9 built and tuned the **operational (OLTP) database**. From here on, we *prepare data for analysis* (the **OLAP / data-mining** world). This topic is the "data preprocessing" station that everything downstream depends on.

---

## 1. Why this topic exists (motivation + connection)

Everything before this (ER → relational model → SQL → normalization → file organisation → indexing) made transactional data **correct** and **fast to fetch**. But a data warehouse and data-mining algorithms don't want raw transactional rows — they want data that is **clean, comparable, and the right size for analysis**.

Raw data straight from operational sources is hostile to analysis:

- Attributes live on wildly different scales (`age` 0–100 vs `income` 0–10⁶). Distance-based algorithms (k-NN, k-means, SVM, neural nets) get **dominated** by the large-magnitude attribute → fix with **normalization/scaling**.
- Some methods / concept hierarchies need **buckets, not raw real numbers** → fix with **discretization**.
- Datasets are too large to mine whole → take a **representative sample**.
- Storage and transmission cost too much → **compress / reduce** the data.

**What the previous topic (Indexing) could NOT do:** indexing speeds up *retrieval* but does nothing about the *values themselves* being on incomparable scales, being too voluminous, or being continuous when an algorithm needs discrete. Data transformation fixes the **values and the volume**, not the access path.

**What the next topics build on this:** typed, scaled, discretized, aggregated data is exactly the input to **multidimensional models (Topic 11)**, and **discretization here is the engine of concept hierarchies (Topic 12)**.

---

## 2. Prerequisites

- Basic descriptive statistics: **mean, variance, standard deviation, median, mode**.
- The notion of an **attribute / column** from the relational model (Topic 2).
- **Logarithms** (for log-transforms, decimal scaling, entropy).
- Basic **probability** (for sampling).
- (Helpful, not required) the idea of **entropy / information gain** — reused in supervised discretization and in compression.

---

## 3. Core idea — intuition first

### 3.1 Data types — "what am I legally allowed to do with this value?"

Before transforming or analysing, you must know an attribute's **type**, because the *type decides which operations are meaningful*. The classic four scales (Stevens):

- **Nominal** (categorical names): pure labels — `eye_colour ∈ {brown, blue}`, customer IDs, ZIP codes. Only **= / ≠** make sense. No order, no arithmetic. Center = **mode**.
- **Ordinal**: ordered categories, but the **gaps are unknown/unequal** — `{small, medium, large}`, exam grades, military rank, satisfaction `{poor < ok < good}`. **<, >** make sense; *differences don't*. Center = **median or mode**.
- **Interval**: numeric, equal-sized intervals, **no true zero** (zero is arbitrary) — temperature in **°C/°F**, calendar year/date. **+ , −** are meaningful; **ratios are NOT** ("20 °C is twice as hot as 10 °C" is **false**). Center = **mean** (and median, mode).
- **Ratio**: numeric, equal intervals, **a true zero exists** — length, weight, **age**, **income**, counts, **Kelvin** temperature. **All** arithmetic incl. **ratios** ("20 kg is twice 10 kg" is **true**). Center = mean, median, mode, **geometric mean**.

Two more distinctions GATE likes:

- **Discrete vs continuous:** discrete = countable (number of children, word counts); continuous = real-valued over a range (height, voltage).
- **Binary** = special nominal with 2 states.
  - **Symmetric binary:** both states equally important (gender M/F).
  - **Asymmetric binary:** one state is the "interesting"/rare one (disease test positive vs negative; positive matters more).

Grouping: **Nominal + Ordinal = qualitative / categorical.** **Interval + Ratio = quantitative / numeric.**

### 3.2 The four transformations, in one breath

- **Normalization / scaling:** squeeze attributes onto a common scale so no single attribute dominates due to its units.
- **Discretization:** turn a continuous attribute into a handful of **interval labels** (`age → {child, teen, adult, senior}`).
- **Sampling:** work on a **representative subset** because the full dataset is too big.
- **Compression / data reduction:** make the data **smaller** — either with *zero* information loss (lossless) or with *acceptable* loss (lossy).

> The broader "data transformation" family in the standard treatment also includes **smoothing, aggregation, generalization, and attribute construction**; the GATE syllabus singles out the four above ("such as normalization, discretization, sampling, compression"), so those are where we go deep.

---

## 4. Formal treatment

### 4A. Data types — meaningful-operations table

| Type | Distinguishes (= , ≠) | Order (<, >) | + / − meaningful | × / ÷ (ratios) meaningful | True zero | Valid center |
|---|:---:|:---:|:---:|:---:|:---:|---|
| **Nominal** | ✓ | — | — | — | — | mode |
| **Ordinal** | ✓ | ✓ | — | — | — | median, mode |
| **Interval** | ✓ | ✓ | ✓ | — | — | mean, median, mode |
| **Ratio** | ✓ | ✓ | ✓ | ✓ | ✓ | + geometric mean |

> Read the table top-to-bottom: each row **adds one capability**. That ladder (nominal → ordinal → interval → ratio) is the single most tested idea in this sub-part.

---

### 4B. Normalization / Scaling

Let attribute $A$ have observed minimum $\min_A$, maximum $\max_A$, mean $\bar A$, standard deviation $\sigma_A$.

**(1) Min–max normalization** — linear map of $[\min_A,\max_A] \to [\text{new\_min},\text{new\_max}]$:

$$
v' \;=\; \frac{v-\min_A}{\max_A-\min_A}\,(\text{new\_max}-\text{new\_min}) \;+\; \text{new\_min}
$$

- **Condition:** $\max_A \neq \min_A$ (a constant attribute → divide-by-zero).
- Most common target $[0,1]$: $\;v' = \dfrac{v-\min_A}{\max_A-\min_A}$.
- **Property:** preserves all relationships exactly (it's affine), **but is sensitive to outliers** — one huge max squashes everything else into a tiny band. A future value outside $[\min_A,\max_A]$ maps outside the target range.

**(2) Z-score (zero-mean) normalization:**

$$
v' \;=\; \frac{v-\bar A}{\sigma_A}
$$

- **Result:** mean $0$, standard deviation $1$; **range is unbounded**.
- **Use when** the true min/max are unknown or **outliers** are present (less sensitive than min–max).
- **Outlier-robust variant** — replace $\sigma_A$ by the **mean absolute deviation**
  $$
  s_A=\frac{1}{n}\sum_{i=1}^{n}\lvert v_i-\bar A\rvert,\qquad v'=\frac{v-\bar A}{s_A}.
  $$
  $s_A$ is *more robust* because deviations are **not squared**, so big outliers count less. If a problem says "mean absolute deviation," use **this**, not the standard deviation.

> **Population vs sample SD** (the #1 silent error): population $\sigma=\sqrt{\frac1n\sum(v_i-\bar A)^2}$, sample $s=\sqrt{\frac1{n-1}\sum(v_i-\bar A)^2}$. **Default to population (÷ n)** unless the question says "sample."

**(3) Decimal scaling** — move the decimal point:

$$
v' \;=\; \frac{v}{10^{\,j}}, \qquad j = \text{smallest integer with } \max\bigl(\lvert v'\rvert\bigr) < 1
$$

- Equivalently $j$ = **number of digits in $\max(|v|)$** (using the *absolute* max, so negatives count), with the **strict** condition $\dfrac{\max|v|}{10^{\,j}}<1$.
- Example: values in $[-986,\,917]$ → $\max|v|=986$ (3 digits) → $j=3$ → divide by $1000$ → range $(-0.986,\,0.917)$.
- Power-of-ten care: if $\max|v|=1000$, then $1000/10^3=1$ is **not** $<1$, so $j=4$ (and $1000$ does have 4 digits — consistent).

---

### 4C. Discretization & smoothing

**Goal:** replace numeric raw values by a small set of **interval labels**, optionally **smoothing out noise**.

**Binning (unsupervised) — two flavours:**

- **Equal-width (equal-interval / "distance") binning:** split $[\min,\max]$ into $k$ bins each of **width**
  $$
  W=\frac{\max-\min}{k}.
  $$
  Boundaries at $\min, \min+W, \min+2W,\dots,\max$. **Simple, but bad on skewed data / outliers** — some bins overflow, some sit empty.
- **Equal-frequency (equal-depth) binning:** each bin holds **≈ $n/k$ values**. Handles skew well; downside: identical values can be forced across a boundary.

**Smoothing after binning (noise reduction):**

- **By bin means:** replace every value in a bin by that **bin's mean**.
- **By bin medians:** replace by the **bin median**.
- **By bin boundaries:** replace each value by the **nearer** of its bin's min/max boundary.

**Other discretization methods:**

- **Histogram analysis** (unsupervised).
- **Clustering** (group then label by cluster).
- **Entropy / information-gain based (supervised):** choose split points that **minimise class entropy** (same machinery as decision trees) — needs class labels.
- **$\chi^2$-based, bottom-up (ChiMerge):** merge adjacent intervals whose class distributions are statistically similar (supervised).

> Discretization produces a **concept hierarchy** for a numeric attribute (raw → intervals → coarser intervals). For *categorical* attributes, hierarchies come from explicit schema order or from **#distinct values** (fewest distinct values → highest level). Full treatment in Topic 12.

---

### 4D. Sampling

Let the dataset $D$ have $N$ tuples; we want a sample of size $n$.

- **SRSWOR** — Simple Random Sample **Without Replacement:** draw $n$ **distinct** tuples, each equally likely; a chosen tuple is **not returned**.
- **SRSWR** — Simple Random Sample **With Replacement:** each of $n$ draws is from the **full** $D$, so the **same tuple can be picked more than once**.
- **Stratified sample:** partition $D$ into **strata** (homogeneous groups), then sample (usually **proportionally**) from each. **Guarantees representation of small/rare groups** — best for skewed data.
- **Cluster (group) sample:** partition $D$ into **clusters**, randomly pick some clusters, take **all** (or a sample) of the tuples inside (e.g., sampling whole disk pages of rows).

**Key facts / formulas:**

$$
\begin{aligned}
&\text{SRSWR, single draw:} && P(\text{a specific tuple}) = \tfrac{1}{N}\\
&\text{SRSWOR of size } n:&& P(\text{a specific tuple is in the sample}) = \tfrac{n}{N}\\
&\text{\# possible SRSWOR samples of size } n:&& \binom{N}{n}\\
&\text{\# ordered WR draws of length } n:&& N^{\,n}
\end{aligned}
$$

- **Why sampling helps:** its cost is **proportional to the sample size $n$, not to $N$** — i.e., **sublinear** in the data size.

---

### 4E. Compression / Data reduction

**Two regimes:**

- **Lossless:** the original is **perfectly** reconstructable. Examples: **Run-Length Encoding (RLE), Huffman coding, LZ/ZIP, string compression**.
- **Lossy:** only an **approximate** reconstruction, but smaller. Examples: **JPEG, wavelet (DWT) with coefficient truncation, PCA dimensionality reduction**.

**Major data-reduction techniques (standard warehousing taxonomy):**

- **Dimensionality reduction** (cut the number of *attributes*):
  - **PCA (Principal Component Analysis / Karhunen–Loève):** project onto the top-$k$ orthogonal directions ("principal components") that capture the most **variance**. *Lossy*.
  - **Wavelet transform (DWT):** transform the data, **keep only the largest coefficients**, zero the rest → lossy compression. For a given number of retained coefficients, DWT often gives **lower reconstruction error / better localised** compression than the DFT.
  - **Attribute subset selection (feature selection):** drop **irrelevant/redundant** attributes (e.g., via forward selection, backward elimination, decision-tree induction). *Lossless* if the dropped attributes were truly redundant.
- **Numerosity reduction** (cut the number of *data values*):
  - **Parametric:** fit a **model**, store only its **parameters** (e.g., regression, log-linear models) instead of the data.
  - **Non-parametric:** **histograms, clustering, sampling**.
- **Data-cube aggregation:** store **summarised cubes** at the level needed for analysis (directly feeds Topic 11).

**Compression math GATE may use:**

- **Entropy** of a source with symbol probabilities $p_i$:
  $$
  H \;=\; -\sum_i p_i \log_2 p_i \quad \text{bits/symbol}
  $$
  This is the **Shannon lower bound** on the average bits per symbol; **Huffman** coding approaches it.
- **Compression ratio** $=\dfrac{\text{original size}}{\text{compressed size}}$.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Min–max to [0,1] *(NAT)*
`income` ranges $[12000,\,98000]$. Normalise $v=73600$ to $[0,1]$.

$$
v'=\frac{73600-12000}{98000-12000}=\frac{61600}{86000}=0.7163.
$$

**Answer: ≈ 0.716.**

---

### Problem 2 — Min–max to a custom range [-1, 1] *(NAT)*
Same `income` $[12000,98000]$; map $73600$ to $[-1,1]$.

$$
v'=\underbrace{\frac{61600}{86000}}_{0.71628}\times\bigl(1-(-1)\bigr)+(-1)=0.71628\times 2-1=0.4326.
$$

**Answer: ≈ 0.433.** *(Trap: don't forget the $\times(\text{range})$ scale and the $+\,\text{new\_min}$ shift.)*

---

### Problem 3 — Z-score with given mean/SD *(NAT)*
Attribute has mean $54$, SD $16$. Normalise $v=70$.

$$
v'=\frac{70-54}{16}=\frac{16}{16}=1.0.
$$

**Answer: 1.0.**

---

### Problem 4 — Z-score, compute statistics first **(trap: population vs sample)** *(NAT)*
Data $\{2,4,6,8,10\}$. Z-score normalise $v=8$ using **population** SD.

Mean $\bar A=6$. Deviations $-4,-2,0,2,4$; squares $16,4,0,4,16$, sum $=40$.
Population variance $=40/5=8\Rightarrow\sigma=2.828$.

$$
v'=\frac{8-6}{2.828}=0.707.
$$

**Answer: ≈ 0.707.**
*If the question had said **sample** SD:* variance $=40/4=10$, $s=3.162$, $v'=2/3.162=\mathbf{0.632}$ — a classic "almost-right" distractor.

---

### Problem 5 — Decimal scaling *(NAT)*
Values lie in $[-4225,\,3891]$. After decimal scaling, what is the normalised value of $3891$?

$\max|v|=4225$ → 4 digits → $j=4$ → divide by $10000$.

$$
v'=\frac{3891}{10000}=0.3891.
$$

**Answer: 0.3891.** *(Trap: the max-magnitude comes from the **negative** end here.)*

---

### Problem 6 — Equal-width binning + smoothing *(NAT / descriptive)*
Sorted data: $\{4,8,15,21,21,24,25,28,34\}$. Use **3 equal-width** bins. Give bin contents, then smooth **by bin means** and **by bin boundaries** for the last bin.

Width $W=\dfrac{34-4}{3}=10$; boundaries $4,14,24,34$.

| Bin | Range | Values |
|---|---|---|
| 1 | $[4,14)$ | 4, 8 |
| 2 | $[14,24)$ | 15, 21, 21 |
| 3 | $[24,34]$ | 24, 25, 28, 34 |

**Smoothing by bin means:** bin1 → $6,6$; bin2 → $19,19,19$ (mean $57/3$); bin3 → $27.75,\,27.75,\,27.75,\,27.75$ (mean $111/4$).

**Smoothing bin 3 by boundaries** (min $24$, max $34$, replace by nearer): $24\to24$, $25\to24$ ($|25-24|=1<9$), $28\to24$ ($4<6$), $34\to34$ → $\{24,24,24,34\}$.

*(Contrast — equal-frequency, 3 bins of 3: $\{4,8,15\},\{21,21,24\},\{25,28,34\}$.)*

---

### Problem 7 — Classify the scale *(MSQ)*
Which of the following are **ratio-scaled**?
(a) Temperature in °C (b) Temperature in Kelvin (c) Age in years (d) Calendar year (e) Annual income

- (a) °C — **interval** (no true zero) ✗
- (b) Kelvin — **ratio** (0 K = absolute zero) ✓
- (c) Age — ratio ✓
- (d) Calendar year — **interval** (year 0 is arbitrary) ✗
- (e) Income — ratio ✓

**Answer: (b), (c), (e).**

---

### Problem 8 — Why a ratio statement fails *(MCQ)*
"20 °C is twice as warm as 10 °C." This is invalid because Celsius temperature is **\_\_\_\_** scaled.
**Answer: interval** (no true zero, so ratios are meaningless).

---

### Problem 9 — Sampling probability *(NAT)*
$D$ has $N=10000$ tuples; take an **SRSWOR** of size $n=200$. Probability a given tuple is in the sample?

$$
P=\frac{n}{N}=\frac{200}{10000}=0.02.
$$

**Answer: 0.02.** *(Trap: $1/N$ is the single-draw **with-replacement** probability, not this.)*

---

### Problem 10 — Sampling concepts *(MSQ)*
Match the property:
- Guarantees every subgroup is represented → **Stratified.**
- Can pick the same tuple twice → **SRSWR.**
- Picks whole pre-formed groups → **Cluster.**
- Cost grows with $n$, not $N$ → **true of all sampling.**

---

### Problem 11 — Entropy / minimum bits *(NAT)*
A source emits symbols with probabilities $\{0.5,\,0.25,\,0.125,\,0.125\}$. Minimum average bits/symbol?

$$
H=-\bigl[0.5\log_2 0.5 + 0.25\log_2 0.25 + 2(0.125\log_2 0.125)\bigr]
$$
$$
= 0.5(1)+0.25(2)+0.125(3)+0.125(3)=0.5+0.5+0.375+0.375=1.75.
$$

**Answer: 1.75 bits/symbol** (achievable exactly by Huffman, since all $p_i$ are powers of $\tfrac12$).

---

## 6. Common GATE question types on this topic

- Compute a **min–max / z-score / decimal-scaling** normalised value *(NAT)*.
- Given raw data, **compute mean & SD first** (watch population vs sample), then z-score.
- **Classify attributes** into nominal / ordinal / interval / ratio *(MCQ/MSQ)*.
- Identify the **valid center / operation** for a given data type (e.g., "median is meaningful but mean isn't" → ordinal).
- **Equal-width vs equal-frequency binning:** assign values, find boundaries/width, then smooth by **mean/median/boundary**.
- **Sampling:** probability a tuple appears (SRSWOR $=n/N$); distinguish WOR / WR / stratified / cluster.
- **Lossy vs lossless** classification; which technique is **dimensionality** vs **numerosity** reduction.
- **Entropy / minimum bits / compression ratio** *(NAT)*.

---

## 7. Traps, pitfalls & common mistakes

- **Population vs sample SD** in z-score: ÷ $n$ vs ÷ $(n-1)$. **Default population** unless told "sample." Biggest source of near-miss answers.
- **Min–max custom range:** apply **both** the $\times(\text{new\_max}-\text{new\_min})$ scaling **and** the $+\,\text{new\_min}$ shift — and in that order.
- **Decimal scaling $j$:** based on the **absolute** maximum (so negative extremes count), and the condition is **strict** $\max|v'|<1$. At exact powers of ten ($1000\to j=4$), recheck the strict inequality.
- **Interval vs ratio:** **°C, °F, calendar year/date are interval** (no true zero) → ratios meaningless; **Kelvin is ratio**. The "twice as hot" statement is the canonical trap.
- **Equal-width ≠ equal-frequency:** equal-width = equal *interval size* (counts may differ); equal-frequency = equal *counts* (widths may differ).
- **Smoothing by boundaries** = replace by the **nearer** boundary, not always the bin minimum.
- **Sampling cost is $\propto n$, not $N$** (sublinear). Don't claim it scales with the full dataset.
- **SRSWOR probability** that a specific tuple is in the sample $=n/N$ — not $1/N$.
- **Lossy vs lossless:** PCA, wavelet-with-truncation, JPEG = **lossy**; Huffman, RLE, ZIP = **lossless**; (clean) attribute selection = lossless.
- **Normalize using *training-set* statistics**, then apply to test data — so min–max can legitimately push future values **outside $[0,1]$** (conceptual but examinable).
- **MAD variant of z-score** ($\sum|v-\bar A|/n$) is **more outlier-robust** than the SD version — use it only when the problem says "mean absolute deviation."
- **Don't normalise nominal/ID attributes** — scaling a ZIP code or customer ID is meaningless; type-awareness gates which transform is even legal.

---

## 8. Connection forward

Clean, comparably-scaled, correctly-typed, optionally-summarised data is **precisely what a data warehouse ingests**. The **discretization** built here becomes the **concept hierarchies** of **Topic 12**, and the typed + aggregated data flows into the **multidimensional models — star, snowflake, fact-constellation schemas of Topic 11**, where **measures** are computed across **dimensions**. In short: this topic produces the raw material; the next two topics shape it into a warehouse.

---

## 9. One-line cheat-sheet summary

**Min–max maps to a target range, z-score $=(v-\mu)/\sigma$ (mean 0, sd 1), decimal scaling $=v/10^{\#\text{digits of }\max|v|}$; discretize by equal-width (equal size) or equal-frequency (equal count) bins + smooth by mean/median/boundary; sample via SRSWOR ($P=n/N$)/SRSWR/stratified/cluster (cost $\propto n$); reduce data losslessly (Huffman/RLE) or lossily (PCA/wavelet), with entropy $H=-\sum p\log_2 p$ as the min bits/symbol — and types nominal→ordinal→interval→ratio decide which operations are even legal.**

---

### ➡️ Next topic (in order)
**Topic 11 — Multidimensional Data Models & Schemas** (star schema, snowflake schema, fact constellation), which is where this transformed, typed, aggregated data gets organised for OLAP.

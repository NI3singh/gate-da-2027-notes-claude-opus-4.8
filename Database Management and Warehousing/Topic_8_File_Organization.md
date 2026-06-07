# Topic 8 — File Organization

**Subject:** Database Management and Warehousing
**GATE DA Section:** Database Design & Storage (Chat 3)

---

## 0. Topic header & syllabus map

File Organization is the **physical-storage** half of database design. It sits *after* **Normalization** (Topic 7), which decided the *logical* schema — how many tables and which attributes go where — and *before* **Indexing** (Topic 9), which adds auxiliary structures to speed up access.

The chain: Normalization (Topic 7) outputs many small, well-keyed relations. **Each relation is stored on disk as a *file* — a collection of *records* grouped into *blocks*.** This topic asks: *given a file, how do we arrange its records on disk so that searching, inserting, and deleting are cheap?* The answer is measured in **block accesses (disk I/Os)**, and the three classic answers are **heap, sorted, and hashed** organizations.

---

## 1. Why this topic exists (motivation + connection)

**The real problem:** databases are far too large to fit in main memory, so they live on **disk**, and a disk access is **roughly 100,000× slower than a memory access**. The single dominant cost of any database operation is therefore *the number of blocks it reads or writes from disk*. A logically perfect schema is useless if answering a query touches every block of a billion-row table.

**What the previous topic couldn't do:** Normalization (Topic 7) tells you the *correct logical shape* of the data — no redundancy, no anomalies — but says **nothing about speed**. Two databases with identical normalized schemas can differ by orders of magnitude in performance purely from how the rows are physically placed. We need a separate theory of *physical placement* to control I/O cost. That is file organization.

**What the next topic builds on this:** Sorted files give fast (`log₂`) search on one field but ruinous insertion; hashing gives O(1) equality lookups but cannot do ranges. **Indexing (Topic 9)** is the direct fix: keep a *small, separate, ordered structure* (a primary index, a B⁺-tree, a hash index) over the data file so you get fast search **and** manageable updates **and** range queries at once. Indexing is built entirely out of the ideas — ordered access, hashing, block-access cost — introduced here.

---

## 2. Prerequisites

- **Relational model (Topic 2):** a relation is a set of tuples; here each tuple becomes a **record**.
- **Normalization (Topic 7):** the relations being stored are the normalized tables; "organizing field" is usually a key.
- **Logarithms & ceilings/floors:** `log₂`, `⌈·⌉`, `⌊·⌋` appear in every cost formula.
- **Basic counting:** records-per-block, blocks-per-file arithmetic.

---

## 3. Core idea — intuition first

Think of a disk as a giant filing cabinet where the *only* unit you can pull out or put back is a **fixed-size drawer = a block (page)**, typically 512 B–8 KB. You never read "one record" from disk; you read the **whole block** that contains it. So the game is always: **minimize how many drawers you must open.**

A file is a pile of records packed into drawers. How you arrange them changes how many drawers you open to find something:

- **Heap (unordered):** toss each new record into the last drawer with free space. Adding is trivial (just append), but **finding** a specific record means opening drawers one by one — on average half of them.
- **Sorted (ordered):** keep records in sorted order of one field. Now you can **binary-search** the drawers (open ~`log₂` of them) — a massive speed-up *for that field*. But inserting a record in the middle forces you to shuffle everything down — very expensive.
- **Hashed:** run the key through a formula `h(K)` that tells you *exactly which drawer* the record is in. Finding it takes **one** drawer-open — the fastest possible for equality. But because the formula scatters records, you lose all sense of order, so "give me everyone with salary between 40k and 60k" becomes hopeless.

There is **no single best organization** — each wins for some operations and loses for others. The whole topic is learning the cost of each operation under each organization, so you can pick the right one (and so you can answer GATE's "how many block accesses?" questions).

---

## 4. Formal treatment

### 4.1 The storage hierarchy and the block (page)

From fast/small/expensive to slow/large/cheap: **CPU registers → cache → main memory (RAM) → SSD/flash → magnetic disk → tape**. Memory above the line is *volatile*; disk and below are *non-volatile* (survive power loss) — which is why the database itself lives on disk.

> **Block (page):** the fixed-size unit of transfer between disk and memory. **All disk I/O is in whole blocks.** A *block access* (one read or one write of a block) is the unit in which we measure all costs. The **buffer manager** caches recently used blocks in memory to avoid repeat I/O (assume cold cache unless told otherwise).

*(RAID — striping/mirroring across multiple disks for speed and reliability — is a storage-architecture topic outside GATE DA's file-organization scope; noted in one line only.)*

### 4.2 Disk access time (foundation for numericals)

To read one block from a magnetic disk, the head must (1) move to the right track, (2) wait for the right sector to rotate under it, (3) transfer the bytes:

> **Block access time** `= s + rd + btt`, where
> - **`s` = seek time** — time to move the arm to the target track.
> - **`rd` = rotational latency** — wait for the sector to arrive under the head. **Average `rd = ½ × (time for one rotation) = ½ × (60 / RPM)`** (an average request waits half a revolution).
> - **`btt` = block transfer time** `= (block size / track capacity) × (rotation time)`, equivalently `= block size / transfer rate`.

**Conditions/assumptions:** average rotational latency `= ½ rotation` assumes the target sector is uniformly random. Reading `k` *consecutive* blocks costs roughly `s + rd + k·btt` (one seek + one latency, then stream all `k`), which is why sequential access is far cheaper than `k` random reads (`k·(s+rd+btt)`).

### 4.3 Records and blocking

A **record** is the physical form of a tuple — a sequence of field values.

**Fixed-length records:** every record has the same byte size `R`. With block size `B`:

> **Blocking factor** (records per block): `bfr = ⌊B / R⌋`.
> **Number of blocks** for `r` records: `b = ⌈r / bfr⌉`.
> **Unused space per block** (unspanned): `B − (bfr × R)` bytes.

**Spanned vs unspanned storage:**

| | Unspanned | Spanned |
|---|---|---|
| Record crossing a block boundary? | **No** — record kept whole in one block | **Yes** — a record may continue in the next block, with a pointer at the block end |
| Wasted space | Up to `R−1` bytes per block | Minimal |
| Required when | `R ≤ B` | **`R > B`** (record larger than a block) |
| Access | Simpler, 1 block per record | A record may need 2 block reads |

**Variable-length records** arise from variable-size fields, optional (nullable) fields, or repeating groups. They are stored using length prefixes, field separators, or end-of-record markers, so `bfr` is then an *average*, not exact.

### 4.4 Heap (unordered / pile) file

Records are stored in **insertion order**; a new record is appended to the **last block** (or any block with free space).

| Operation | Cost (block accesses) | Why |
|---|---|---|
| **Insert** | **2** | Read the last block, add the record, write it back. (If a free-space directory points straight to a non-full block, still ~2.) |
| **Search**, key value, record exists | **`b/2`** (average) | Linear scan; stop when found. |
| **Search**, value may be absent, or non-key (many matches) | **`b`** | Must scan *every* block to be sure. |
| **Delete** | search cost **`+ 1`** | Locate, then mark/remove and rewrite that block. Leaves gaps → needs periodic **reorganization**. |
| **Read all records in sorted order** | `b` read + external sort `≈ b·log b` | Not sorted, so sorting is required. |

**Best for:** bulk loading and full-table scans. **Worst for:** selective retrieval.

### 4.5 Sequential (ordered / sorted) file

Records are physically sorted by an **ordering field**. If that field is also unique, it is the **ordering key**.

| Operation | Cost (block accesses) | Conditions |
|---|---|---|
| **Search on the ordering field (=)** | **`⌈log₂ b⌉`** | **Binary search** — the headline benefit. Requires sorting on *this* field. |
| **Search on a non-ordering field** | **`b/2` or `b`** | No benefit; same as heap. |
| **Range query on ordering field** | **`⌈log₂ b⌉ + (matching blocks)`** | Binary-search to the first match, then read consecutively. Very efficient. |
| **Insert** | **`≈ ⌈log₂ b⌉ + b/2`** ⇒ **O(b)** | Find position (`log₂ b`), then shift on average half the file to keep order — *expensive*. |
| **Delete** | search **`+ b/2`** (shift) or use deletion marker | Same shifting problem; markers + periodic reorganization mitigate it. |
| **Ordered retrieval** | trivial — already sorted | Just read sequentially. |

**The insertion problem and its fix:** because in-place insertion is O(b), real systems append new records to a small **overflow (transaction) file** and periodically **merge** it into the sorted main file. This makes insertion cheap but degrades search to *binary-search-the-main-file + linear-scan-the-overflow* until the next merge — which is exactly the limitation that motivates indexing.

### 4.6 Hashing — static (external) hashing

A **hash function** `h(K)` maps a key value `K` to a **bucket address** in `0 … M−1`. Each bucket is one (or a few) blocks. To find a record, compute `h(K)` and go **directly** to that bucket.

> **Search / insert / delete on the hash field ≈ 1 block access** (compute `h(K)`, read the bucket), provided there is no overflow. This O(1) equality lookup is hashing's headline benefit.

**Collisions and overflow** (two keys hash to the same bucket; bucket fills up) are handled by:

- **Open addressing (probing):** put the record in the next free slot (e.g., linear probing `h, h+1, h+2, …`).
- **Chaining (overflow buckets):** link extra overflow blocks to the primary bucket; search cost becomes `1 + (length of overflow chain)`.
- **Multiple hashing:** apply a second hash function on collision.

**Cost & limitations:**

| Operation | Cost | Note |
|---|---|---|
| Search / insert / delete on hash key | `≈ 1` (+ overflow chain) | The win. |
| Search on a **non-hash** field | `b` | No benefit — full scan. |
| **Range query** | `b` | **Hashing destroys order** — ranges are hopeless. ⚠️ |

> **The fatal flaw of *static* hashing:** the number of buckets `M` is **fixed**. As the file grows, overflow chains lengthen and the ~1-access guarantee collapses; as it shrinks, buckets sit empty. It does not adapt to size changes — motivating **dynamic hashing**.

The **load factor** `= (number of records) / (M × bucket capacity)` measures fullness; high load factor ⇒ many overflows.

### 4.7 Dynamic hashing I — Extendible hashing

Keeps a **directory** of `2^d` pointers, indexed by **`d` bits of the hash value** (`d` = **global depth**). Each *bucket* carries a **local depth `d′ ≤ d`** = how many hash bits actually determine membership in that bucket. (Convention used here: the **least-significant `d` bits**; some texts use the most-significant bits — fix one and stay consistent.)

**On bucket overflow:**

1. **If local depth `d′ < d` (global):** **split the bucket only** — create a sibling, redistribute its records by their `(d′+1)`-th bit, set both siblings' local depth to `d′+1`, and repoint the affected directory entries. **No directory growth.**
2. **If local depth `d′ = d` (global):** **double the directory** (`d → d+1`, copy pointers), **then** split the overflowing bucket as above.

> **Search / insert ≈ 1–2 block accesses** (one to consult the directory if not in memory, one for the bucket). No long overflow chains; the directory **grows and shrinks with the data.**

### 4.8 Dynamic hashing II — Linear hashing

Grows **one bucket at a time with *no directory*.** It uses a family of hash functions `hᵢ(K) = K mod (2ⁱ·M)` and a **split pointer `next`** marking the next bucket to split.

- When a split is triggered (commonly when the load factor exceeds a threshold — *not necessarily the bucket that overflowed*), the bucket at `next` is split using the finer function `h_{i+1}`, and `next` advances **round-robin**.
- Overflows before a bucket's turn are handled with temporary overflow chains.
- After all `2ⁱ·M` buckets have split, the round (level) increments: `i → i+1`, `next → 0`.

**Trade-off vs extendible:** linear hashing avoids the directory (saving space and a level of indirection) but can keep short overflow chains; extendible hashing avoids overflow chains but needs a directory that can double. Both give amortized ~1 access on the hash field; both still **cannot do range queries.**

### 4.9 Master cost comparison (the table to memorize)

For a file of **`b`** blocks, average block accesses (cold cache, uniform distribution, equality on the *organizing* field unless noted):

| Operation | **Heap** | **Sorted** | **Static Hash** |
|---|:---:|:---:|:---:|
| Search, organizing key (=) | `b/2` | `⌈log₂ b⌉` | `≈ 1` |
| Search, non-organizing field | `b` | `b` | `b` |
| **Range** query (organizing field) | `b` | `⌈log₂ b⌉` + range | `b` (bad) |
| Insert | `2` | `≈ ⌈log₂ b⌉ + b/2` | `≈ 2` |
| Delete | search `+ 1` | search `+ b/2` | `≈ 2` |
| Ordered scan | sort needed | trivial | sort needed |

**Reading the table (the design rule):** **heap** wins on insertion and full scans; **sorted** wins on range queries and ordered output; **hash** wins on equality lookups but is useless for ranges. There is no universal winner — and indexing (next topic) is what lets a single data file enjoy more than one of these strengths at once.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Blocking factor & number of blocks (NAT)

A file has **30,000** fixed-length records of **100 bytes** each. Block size = **1024 bytes**, **unspanned**. Find the blocking factor, number of blocks, and total wasted space.

**Solution.**
`bfr = ⌊1024/100⌋ = ⌊10.24⌋ = 10` records/block.
`b = ⌈30000/10⌉ = 3000` blocks.
Wasted/block `= 1024 − (10×100) = 24` bytes ⇒ total wasted `= 24 × 3000 = 72,000` bytes.

*Trap:* with `bfr=10`, the 11th record's-worth of space (the leftover 24 bytes) is **wasted**, not used for a partial record — that's the unspanned rule. (If spanned, `b = ⌈30000×100 / 1024⌉ = ⌈2929.7⌉ = 2930` blocks, ~no waste.)

---

### Problem 2 — Disk access time (NAT)

A disk spins at **7200 RPM**, average **seek = 8 ms**, transfer rate **40 MB/s**, block size **4 KB**. Find the average time to read **one random block**, and the time to read **100 consecutive blocks**.

**Solution.**
Rotation time `= 60/7200 s = 8.33 ms` ⇒ average rotational latency `rd = 8.33/2 = 4.17 ms`.
Transfer time `btt = 4096 / (40×10⁶) s = 1.024×10⁻⁴ s = 0.102 ms` (using 1 MB = 10⁶ B).
**One random block** `= s + rd + btt = 8 + 4.17 + 0.102 ≈ 12.27 ms`.
**100 consecutive blocks** `= s + rd + 100·btt = 8 + 4.17 + 10.24 ≈ 22.41 ms` — versus `100 × 12.27 = 1227 ms` if they were random. This ~55× gap is *why* sequential layout matters.

---

### Problem 3 — Heap vs sorted search (MCQ)

The file of Problem 1 has `b = 3000` blocks. Compare average block accesses to find a record by its key in (a) a heap file vs (b) a sorted file (sorted on that key).

**Solution.**
(a) Heap, average `= b/2 = 1500` block accesses.
(b) Sorted, binary search `= ⌈log₂ 3000⌉ = ⌈11.55⌉ = 12` block accesses.
A **125× reduction** — the value of ordering. *But* recall insertion into the sorted file now costs `≈ 12 + 1500 ≈ O(b)`, the price paid for that fast search.

---

### Problem 4 — Static hashing with overflow (NAT)

A static hash file has **M = 5** buckets, each holding **3** records, with chaining. Insert keys with `h(K) = K mod 5` for K = 12, 7, 22, 17, 2, 32, 27. Show bucket 2's contents and the worst-case search cost (in buckets) for a key hashing to bucket 2.

**Solution.** `h` values: 12→2, 7→2, 22→2, 17→2, 2→2, 32→2, 27→2 — *all seven* hash to bucket 2 (every key ≡ 2 mod 5).
Bucket 2 (capacity 3): primary block `{12, 7, 22}`; overflow block 1 `{17, 2, 32}`; overflow block 2 `{27}`.
**Worst-case search** = primary + 2 overflow blocks = **3 block accesses**. This degenerate clustering (all keys ≡ 2 mod 5) is exactly the static-hashing failure mode: a bad key distribution ⇒ long chains ⇒ the ~1-access guarantee is lost.

---

### Problem 5 — Extendible hashing walk-through (the GATE favorite)

Bucket capacity **= 2** records. Insert keys with the following hash bit-strings (use the **least-significant bits**), in order: **2 (…010), 3 (…011), 5 (…101), 7 (…111), 4 (…100), 8 (…1000)**. Track the directory (global depth `d`) and bucket local depths.

**Solution.** Start `d = 1`: directory `{0→B0, 1→B1}`, both local depth 1.

1. **Insert 2** (LSB 0) → `B0 = {2}`.
2. **Insert 3** (LSB 1) → `B1 = {3}`.
3. **Insert 5** (LSB 1) → `B1 = {3, 5}` (full).
4. **Insert 7** (LSB 1) → `B1` overflows; **local(B1)=1 = global d=1 ⇒ double directory to `d=2`** (entries `00,01,10,11`). `B0` keeps local depth 1, so `00` and `10` both point to `B0`. **Split B1** (local→2), redistribute `{3,5,7}` by **last 2 bits**: `5=…01→01`, `3=…11→11`, `7=…11→11`. Result: bucket@`01 = {5}` (LD 2); bucket@`11 = {3,7}` (LD 2, full).
   *State:* `d=2`; `00,10→B0(LD1)`, `01→{5}(LD2)`, `11→{3,7}(LD2)`.
5. **Insert 4** (last 2 bits `00`) → directory `00 → B0`; `B0 = {2,4}` (now full, no overflow yet).
6. **Insert 8** (last 2 bits `00`) → `B0` overflows; **local(B0)=1 < global d=2 ⇒ split *without* doubling.** Split `B0` (local→2), redistribute `{2,4,8}` by last 2 bits: `2=…10→10`, `4=…00→00`, `8=…00→00`. Result: bucket@`00 = {4,8}` (LD 2); bucket@`10 = {2}` (LD 2).
   *Final state:* `d=2`; `00→{4,8}`, `01→{5}`, `10→{2}`, `11→{3,7}` — all local depth 2.

**Takeaways for the exam:** step 4 shows **local = global ⇒ directory doubles**, step 6 shows **local < global ⇒ bucket splits, directory unchanged**. Count of directory doublings here = **1**; number of bucket splits = **2**.

---

### Problem 6 — Choose the organization (MSQ-style reasoning)

For each workload, name the best of {heap, sorted, static hash}:
(a) A log table that is only ever bulk-appended and occasionally fully scanned.
(b) Frequent equality lookups by a unique ID, almost no range queries, table size roughly constant.
(c) Frequent "salary between X and Y" range queries and reports printed in salary order.

**Solution.**
(a) **Heap** — O(1)-style appends (cost 2) and full scans are its strengths; no selective lookups needed.
(b) **Static hash** on the ID — ~1 block access per lookup; ranges not needed; stable size avoids the overflow problem.
(c) **Sorted** on salary — binary-search to the range start then sequential read, and ordered output is free; hashing would be useless here and heap too slow.

---

## 6. Common GATE question types on this topic

1. **Blocking factor / number of blocks** from record size, block size, record count (with spanned vs unspanned).
2. **Disk access time** numericals: seek + rotational latency (from RPM) + transfer; random vs sequential reads.
3. **Average block accesses** for search/insert/delete in heap vs sorted vs hash — plug into the master table.
4. **Binary search cost** `⌈log₂ b⌉` on a sorted file, and why a non-ordering field gets no benefit.
5. **Extendible hashing** directory walk: global/local depths, number of splits, number of directory doublings, final directory.
6. **Static hashing** overflow / load-factor / worst-case chain-length questions.
7. **Conceptual choice / true-false:** which organization suits a workload; "hashing supports range queries" (false); "sorted-file insertion is cheap" (false).

---

## 7. Traps, pitfalls & common mistakes

- **Counting record accesses instead of *block* accesses.** The unit is always the **block** — you read a whole block to get one record. ⚠️
- **`⌊B/R⌋` vs `⌈·⌉`.** Blocking factor uses **floor** (you can't store a partial record in unspanned files); number of blocks uses **ceiling**. Swapping them is a classic error.
- **Forgetting the leftover bytes are wasted** in unspanned files — they do *not* hold a partial record.
- **Using full rotation for latency.** Average rotational latency is **half** a rotation `= ½(60/RPM)`, not the full rotation time.
- **Assuming sorted-file search helps on *any* field.** Binary search works **only on the ordering field**; any other field is back to linear `b/2`.
- **Believing hashing supports range/ordered queries.** It does **not** — hashing scatters keys; ranges and ordered scans are `b` (full scan). ⚠️
- **Static-hashing "always ~1 access."** Only when there's *no overflow*. Bad key distribution or a growing file ⇒ long chains ⇒ cost blows up.
- **Extendible hashing — the split rule.** Overflow with **local depth = global depth ⇒ double the directory**; with **local depth < global depth ⇒ split the bucket only** (no doubling). Mixing these up is the #1 extendible-hashing mistake.
- **Mixing the bit convention** (LSB vs MSB) midway through an extendible-hashing problem — pick one and keep it for the whole walk.
- **Treating sorted-file insertion as cheap.** It is `O(b)` due to shifting; the overflow-file trick is what real systems use (and it degrades search until the next merge).
- **Sequential vs random read cost.** Reading `k` consecutive blocks ≈ `s + rd + k·btt`, *not* `k·(s+rd+btt)` — sequential is far cheaper.

---

## 8. Connection forward

File organization gives a data file **one** fast access path at best — a sorted file is fast on its ordering field only, a hash file on its hash field only, and a heap on nothing. Real queries need fast access on *several* fields, plus cheap updates, plus range support. **Indexing (Topic 9)** delivers all three by storing a *small, separate, ordered structure* — a primary/secondary index, or a **B⁺-tree** that solves the sorted file's insertion problem, or a **hash index** — pointing into the data file. Every cost we computed here (`log₂ b` search, the expense of ordered-file insertion, the ~1-access hash lookup) is exactly the baseline that indexing improves upon.

---

## 9. One-line cheat-sheet summary

**All cost is *block accesses*; `bfr=⌊B/R⌋`, `b=⌈r/bfr⌉`; heap = cheap insert (2) but `b/2` search, sorted = `⌈log₂ b⌉` search + fast ranges but `O(b)` insert, static hash = ~1-access equality but no ranges and overflow-prone — and extendible hashing fixes growth by doubling the directory when local depth = global depth, else splitting the bucket alone.**

---

### ▶ Next topic (logically next in order): **Topic 9 — Indexing** (primary / secondary / clustering indexes, dense vs sparse, multi-level indexes, B-trees and B⁺-trees, hash indexes), which builds directly on the ordered-file and hashing cost models above.

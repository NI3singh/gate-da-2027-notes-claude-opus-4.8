# Topic 9 — Indexing

**Subject:** Database Management and Warehousing
**GATE DA Section:** Database Design & Storage (Chat 3) — *final topic of this section*

---

## 0. Topic header & syllabus map

Indexing is the **last** topic of the Database Design & Storage section. It sits *after* **File Organization** (Topic 8) and is the structure that *fixes* file organization's limitations. After this, the syllabus opens a **new chat — Data Types & Data Transformation** (Topic 10), shifting from *how databases store data* to *how we prepare data for analysis and warehousing*.

The chain: File Organization (Topic 8) showed that a data file gets at most **one** fast access path — sorted files search in `log₂ b` but insert in `O(b)`; hash files are `~1` access for equality but useless for ranges; heaps are good for nothing selective. **An index is a small, separate, ordered (or hashed) structure built *over* the data file** so you can find records quickly without scanning, on more than one field, with manageable updates. The pinnacle is the **B⁺-tree**, the index structure essentially every real database uses.

---

## 1. Why this topic exists (motivation + connection)

**The real problem:** even the best file organization gives only one cheap access path, and it forces a bad trade-off. A sorted file searches a key in `log₂ b` block accesses — but every insertion in the middle shifts `O(b)` records. We want *fast search, cheap updates, and range queries at the same time, on possibly several fields.* No single file organization delivers all three.

**What the previous topic couldn't do:** File Organization (Topic 8) ties the *physical order* of records to one field. You can sort the data on `salary` *or* hash it on `emp_id`, but not both, and you can only have one physical order. So lookups on any other field collapse to a full `O(b)` scan, and keeping the order under insertions is ruinous.

**The fix (this topic):** Build a **separate, much smaller file of `(search-key, pointer)` entries** that points into the data file. Because the index is tiny compared to the data, searching it is cheap; because it is *separate*, you can build **many** indexes (one per field you query); and by making it a **balanced tree (B⁺-tree)** you get `log_fanout(N)` search *and* `log_fanout(N)` insertion *and* efficient range scans — all the things a plain sorted file could not give simultaneously.

**What the next topic builds on this:** indexing closes the DBMS storage-and-design arc. The next chat (**Data Types & Data Transformation**, Topic 10) leaves storage behind and starts on *preparing data values* — scaling, discretization, sampling, compression — the pre-processing that feeds analysis and the **data warehouse**. ⚠️ Heads-up: that topic also uses the word **"normalization,"** but there it means *scaling values to a range* — completely different from the schema normalization of Topic 7.

---

## 2. Prerequisites

- **File Organization (Topic 8):** ordered files, blocking factor `bfr = ⌊B/R⌋`, number of blocks `b = ⌈r/bfr⌉`, the `log₂ b` binary-search cost, and the block-access cost model. Indexing is measured in the same units.
- **Keys (Topic 2/3, 7):** ordering key, candidate/primary key — an index's *search key* may or may not be a database key.
- **Logarithms & ceilings:** `log₂`, `log_fanout`, `⌈·⌉`, `⌊·⌋`.

---

## 3. Core idea — intuition first

Think of the **index at the back of a textbook**. The book (the *data file*) is hundreds of pages; finding "B⁺-tree" by flipping pages is a linear scan. The index is two tidy pages of `(term → page number)` entries, *sorted alphabetically*. You binary-search the short index, get the page number, and jump straight there. Two key features make it work: the index is **far smaller** than the book, and it is **sorted** so you can search it fast.

A database index is exactly this: a small file of **`(search-key value, pointer)`** pairs. The pointer leads to the actual record (or to the block holding it). Search the small index → follow the pointer → one jump to the data.

Two refinements drive the whole topic:

- **How many index entries?** If you keep one entry for *every record* (a **dense** index, like indexing every word), the index is bigger but you can even answer "does this value exist?" from the index alone. If you keep one entry *per data block* (a **sparse** index, like indexing only the first word on each page), the index is much smaller — but this only works if the data is **sorted**, so that "the largest key ≤ what I want" lands you in the right block.
- **What if the index itself is too big to search fast?** Index the index. Repeat until the top fits in one block. That layered idea, made dynamic so insertions stay cheap, *is* the **B⁺-tree**.

That's the entire topic: pick what to index (primary / clustering / secondary), pick dense vs sparse, and when the index grows, go multilevel → B-tree → B⁺-tree.

---

## 4. Formal treatment

### 4.1 What an index is

> **Index:** an auxiliary access structure — a file of **index entries `⟨K, P⟩`**, where `K` is a **search-key** value and `P` is a pointer (to a record, or to a block). The index file is itself stored in blocks and is usually kept **sorted on `K`** (ordered index) or **hashed on `K`** (hash index).

- **Search key** = the field(s) the index is built on. It need *not* be a database key — it can be any attribute you query often. (Don't confuse "search key" with "primary key.")
- **Index blocking factor (fan-out)** `bfr_i = ⌊B / (K + P)⌋`, where `K` = search-key size, `P` = pointer size. Because `K + P` is tiny next to a full record, `bfr_i` is large, so the index has **far fewer blocks** than the data file — that's the whole point.

> **Dense index:** one index entry for **every search-key value present** (every record, if the key is unique).
> **Sparse index:** one index entry per **data block** (typically pointing to the *block anchor* — the first record of the block). Requires the **data file to be ordered** on the search key.

| | Dense | Sparse |
|---|---|---|
| Entries | one per record / value | one per block |
| Size | larger | smaller |
| Requires ordered data? | no | **yes** |
| Can answer "value exists?" from index alone | **yes** | no (must read the block) |

### 4.2 The three single-level ordered indexes (Elmasri classification)

| Index type | Built on | Data file ordered on it? | Field unique? | Density | # entries |
|---|---|:---:|:---:|---|---|
| **Primary index** | the **ordering key** field | **Yes** | **Yes** (key) | **Sparse** (one per block) | `b` (# data blocks) |
| **Clustering index** | an **ordering non-key** field | **Yes** | No (values repeat) | Sparse-ish | # **distinct values** |
| **Secondary index** | a **non-ordering** field | **No** | key *or* non-key | **Dense** (key) / via indirection (non-key) | `r` (key) or more |

**Primary index** — entries `⟨K(i), pointer-to-block-i⟩` where `K(i)` is the ordering-key value of the **first (anchor) record** of block `i`. Exactly one entry per data block ⇒ `b` entries.

**Clustering index** — the file is physically *clustered* by a non-key ordering field (e.g., all `dept = CSE` rows sit together). One entry per **distinct value**, pointing to the first block containing that value.

**Secondary index** — the data file is **not** ordered on this field.
- On a **secondary key** (unique): **dense** — one entry per record, `⟨K, pointer-to-record⟩`, so `r` entries.
- On a **non-key** (duplicates): either dense (one entry per record, values repeat) or one entry per distinct value pointing to a **block of record pointers** (extra level of indirection).

> **Crucial constraint:** a file can have **at most one** primary *or* clustering index (only one physical order is possible), but **many** secondary indexes (one per frequently-queried field). This is the structural reason indexing — not file organization — is what gives multi-field fast access.

### 4.3 Cost of searching with a single-level index

> With an ordered index of `b_i` blocks (`b_i = ⌈(#entries) / bfr_i⌉`):
> **Search cost `= ⌈log₂ b_i⌉ + 1`** block accesses — binary-search the index (`⌈log₂ b_i⌉`), then **one** access to the data block/record.

Since `b_i ≪ b`, this beats the `⌈log₂ b⌉` of searching the ordered data file directly, and crushes the `b/2` of a heap. (A secondary dense index is *larger* than a primary sparse one, so it costs a bit more — see Problems 1–2.)

### 4.4 Multilevel indexes → the road to trees

The index is itself an ordered file, so we can build a **primary (sparse) index over the index** = the *second level*, then a third level over that, and so on until the **top level fits in a single block**.

> Let **fan-out `fo = bfr_i`** (index blocking factor). First-level blocks `b₁ = ⌈(#entries)/fo⌉`; level-`t` blocks `b_t = ⌈b_{t-1}/fo⌉`.
> **Number of levels `= ⌈log_fo(b₁)⌉ + 1`** (build up until one block remains).
> **Search cost `= (#levels) + 1`** block accesses.

Because each level divides the block count by `fo` (often 50–200), only **3–4 levels** index *millions* of records. **But** a static multilevel index inherits the ordered file's curse: insertions/deletions force expensive reorganization at every level. The fix is to make the multilevel index **dynamic and balanced** — that is exactly a **B-tree / B⁺-tree.**

### 4.5 B-trees

A **B-tree of order `p`** is a balanced multilevel index where **search-key values and record pointers live in *all* nodes** (internal and leaf), and each search-key value appears **exactly once** in the whole tree.

**Node structure** (internal): `⟨P₁, ⟨K₁,Pr₁⟩, P₂, ⟨K₂,Pr₂⟩, …, P_{q}⟩` — `q` tree pointers `Pᵢ`, `q−1` keys `Kᵢ`, each key carrying a **record pointer** `Prᵢ`.

> **Constraints (order `p` = max child/tree pointers; "level" counts with root = level 1):**
> - Every node has at most `p` tree pointers and `p−1` keys.
> - Every node except the root has **at least `⌈p/2⌉` tree pointers** (so at least `⌈p/2⌉−1` keys).
> - The **root** has at least 2 tree pointers (1 key), unless it is the only node.
> - **All leaves are at the same level** (balanced); leaf tree pointers are null.
> - A node with `q` pointers has exactly `q−1` keys.

A B-tree search may **terminate at any level** (the key can sit in an internal node).

**Capacity (order `p`, with `t = ⌈p/2⌉` = minimum children of a non-root node):**

> **Maximum keys** in a B-tree of `L` levels: `p^L − 1` (every node full).
> **Minimum keys** in a B-tree of `L` levels: `2·t^{L−1} − 1` (root has 1 key, every other node minimally filled).

*(CLRS states the same via "minimum degree `t`": min children `t`, max `2t`; for tree height `h`, min keys `= 2t^h − 1`. Translate by `t = ⌈p/2⌉` and `L = h + 1`.)*

### 4.6 B⁺-trees (the one databases actually use)

A **B⁺-tree** moves **all search-key values and record pointers down to the leaf level**. Internal nodes hold **only routing keys (copies of some leaf keys) and tree pointers** — no record pointers. **Leaves are linked** in a sequence for fast ordered/range scans.

> **Internal node, order `p`:** up to `p` tree pointers and `p−1` keys; ≥ `⌈p/2⌉` tree pointers (root ≥ 2). **No record pointers.**
> **Leaf node, order `p_leaf`:** up to `p_leaf` entries `⟨K, Pr⟩` **plus one pointer to the next leaf**; ≥ `⌈p_leaf/2⌉` entries.

Because internal nodes carry no record pointers, they pack **more keys per block ⇒ higher fan-out ⇒ a shorter tree** than a B-tree on the same data (see Problem 3). And every search goes **all the way to a leaf**, giving a **uniform** search cost and (via the leaf links) **efficient range queries.**

**B⁺-tree capacity (internal order `p`, leaf order `p_leaf`, `L` levels, root = level 1, leaves at level `L`):**

> **Maximum keys** `= p^{L-1} · p_leaf` (full internal fan-out × full leaves).
> **Minimum keys** `= 2 · ⌈p/2⌉^{L-2} · ⌈p_leaf/2⌉` (for `L ≥ 2`; root has 2 children, others minimally filled).

### 4.7 Computing the order (fan-out) from block & pointer sizes — *the* GATE numerical

Let `B` = block size, `K` = search-key size, `P` = tree/block-pointer size, `Pr` = record-pointer size.

> **B-tree internal node** holds `p` tree pointers + `(p−1)` keys + `(p−1)` record pointers:
> $$p\cdot P + (p-1)(K + Pr) \le B \;\Longrightarrow\; \boxed{p \le \frac{B + K + Pr}{P + K + Pr}}, \text{ take } \lfloor\cdot\rfloor.$$
>
> **B⁺-tree internal node** holds `p` tree pointers + `(p−1)` keys (**no** record pointers):
> $$p\cdot P + (p-1)K \le B \;\Longrightarrow\; \boxed{p \le \frac{B + K}{P + K}}, \text{ take } \lfloor\cdot\rfloor.$$
>
> **B⁺-tree leaf node** holds `p_leaf` entries `⟨K, Pr⟩` + 1 next-leaf pointer:
> $$p_{leaf}\cdot(K + Pr) + P \le B \;\Longrightarrow\; \boxed{p_{leaf} \le \frac{B - P}{K + Pr}}, \text{ take } \lfloor\cdot\rfloor.$$

These three are deterministic and appear constantly. The B⁺-tree internal fan-out is always ≥ the B-tree fan-out on the same parameters — the formal reason B⁺-trees are shorter.

### 4.8 Insertion & deletion (overflow/underflow handling)

- **Insertion:** find the target **leaf**, insert in sorted order. If the leaf overflows (exceeds capacity), **split** it; in a B⁺-tree **copy** the split-key *up* (the key still exists in the leaf), in a B-tree **push** the median *up* (it leaves the node). Splits may **propagate** to the parent and, if the root splits, the **tree grows one level** (the only way height increases).
- **Deletion:** remove from the leaf; if it **underflows** (< minimum), **redistribute** (borrow from a sibling) or **merge** with a sibling, possibly propagating up; merging the children of the root can **shrink** the tree by a level.
- B-trees/B⁺-trees stay **perfectly balanced** through all updates — that is their defining advantage over a static multilevel index.

### 4.9 Hash indexes

A **hash index** stores `⟨K, P⟩` entries in **buckets** addressed by `h(K)` (Topic 8). Equality search/insert/delete on `K` ≈ **1 access**; but, like all hashing, it **cannot do range queries** and degrades with overflow unless dynamic (**extendible/linear hashing**). Use a hash index for equality-heavy fields, a B⁺-tree when you also need ranges/ordering.

### 4.10 B-tree vs B⁺-tree — the comparison table

| Feature | **B-tree** | **B⁺-tree** |
|---|---|---|
| Record pointers stored | in **all** nodes | in **leaf** nodes only |
| Search-key occurrences | each key **once** | internal keys are **copies** of leaf keys |
| Leaves linked? | No | **Yes** (sequential set) |
| Search cost | varies (may stop early) | **uniform** (always reaches a leaf) |
| Range / ordered scan | inefficient (in-order traversal) | **efficient** (scan linked leaves) |
| Fan-out / height | lower fan-out → **taller** | higher fan-out → **shorter** |
| Used by real DBMS | rarely | **almost always** |

### 4.11 Search-cost summary across Topics 8–9 (for a file with `b` data blocks, `N` records)

| Access path | Equality search cost (block accesses) |
|---|---|
| Heap (no index) | `b/2` |
| Ordered file (no index) | `⌈log₂ b⌉` |
| Single-level index | `⌈log₂ b_i⌉ + 1`, `b_i ≪ b` |
| Multilevel index / **B⁺-tree** | `(#levels) + 1 ≈ ⌈log_fo N⌉ + 1` (typically 3–4) |
| Hash index | `≈ 1` (no ranges) |

---

## 5. Worked problems (GATE-style)

### Problem 1 — Primary (sparse) index cost (NAT)

Data file: **30,000** records, **10 records/block** ⇒ `b = 3000` blocks, ordered on its key. Ordering-key size `K = 9` B, block-pointer `P = 6` B, block size `B = 512` B. Find the index size and search cost; compare to no index.

**Solution.**
Index fan-out `bfr_i = ⌊512/(9+6)⌋ = ⌊34.1⌋ = 34` entries/block.
Primary index is **sparse** ⇒ entries `= b = 3000`. Index blocks `b_i = ⌈3000/34⌉ = 89`.
**Search with index `= ⌈log₂ 89⌉ + 1 = 7 + 1 = 8`** block accesses.
**Without index** (binary search on data) `= ⌈log₂ 3000⌉ = 12`. The index saves 4 accesses *and* is only 89 blocks.

---

### Problem 2 — Secondary (dense) index, and why it's bigger (NAT/MCQ)

Same file, but now index a **non-ordering key** field (`K = 9` B, `P = 6` B, `B = 512` B). Find search cost and contrast with Problem 1.

**Solution.**
A secondary index on a key is **dense** ⇒ one entry per **record** `= 30,000` entries. With `bfr_i = 34`, index blocks `b_i = ⌈30000/34⌉ = 883`.
**Search `= ⌈log₂ 883⌉ + 1 = 10 + 1 = 11`** block accesses.
**Lesson:** the secondary (dense) index has ~10× the entries of the primary (sparse) index, so it is larger and a little slower — but it provides a fast path on a field the data is *not* ordered on, which the primary index cannot. (This is why multilevel/B⁺-trees matter: 883 index blocks is itself worth indexing.)

---

### Problem 3 — Order (fan-out) of B-tree vs B⁺-tree (NAT) — high-frequency

Block size `B = 1024` B, search key `K = 12` B, tree/block pointer `P = 8` B, record pointer `Pr = 8` B. Find: (a) B-tree order, (b) B⁺-tree internal order, (c) B⁺-tree leaf order.

**Solution.**
(a) **B-tree:** `p ≤ (B + K + Pr)/(P + K + Pr) = (1024 + 12 + 8)/(8 + 12 + 8) = 1044/28 = 37.28` ⇒ **`p = 37`** (37 pointers, 36 keys). Check: `37·8 + 36·20 = 296 + 720 = 1016 ≤ 1024` ✓.
(b) **B⁺-tree internal:** `p ≤ (B + K)/(P + K) = (1024 + 12)/(8 + 12) = 1036/20 = 51.8` ⇒ **`p = 51`** (51 pointers, 50 keys).
(c) **B⁺-tree leaf:** `p_leaf ≤ (B − P)/(K + Pr) = (1024 − 8)/(12 + 8) = 1016/20 = 50.8` ⇒ **`p_leaf = 50`**.

**Takeaway:** the B⁺-tree internal fan-out (51) beats the B-tree (37) on identical parameters — because B⁺ internal nodes drop the record pointers — so the B⁺-tree is shorter. *Trap:* the internal and leaf orders of a B⁺-tree are usually **different** (51 vs 50 here) — never assume they're equal.

---

### Problem 4 — Number of B⁺-tree levels and the I/O payoff (NAT)

A B⁺-tree (internal fan-out `51`, leaf capacity `50`) indexes **`N = 1,000,000`** unique keys. How many levels, and how many block accesses per equality search? Compare to an ordered file with 10 records/block.

**Solution.** (max-fill estimate)
Leaves: `⌈1,000,000 / 50⌉ = 20,000`. Build up with fan-out 51: `⌈20000/51⌉ = 393` → `⌈393/51⌉ = 8` → `⌈8/51⌉ = 1` (root).
Levels: root(1) → 8 → 393 → 20,000 leaves = **4 levels** ⇒ search `≈ 4 + 1 = 5` (or 4 if leaf holds the record pointer used directly).
**Ordered file:** `b = ⌈1,000,000/10⌉ = 100,000` blocks ⇒ `⌈log₂ 100000⌉ = 17`. **Heap:** `b/2 = 50,000`.
So ~**4 vs 17 vs 50,000** — the B⁺-tree turns a million-row lookup into a handful of I/Os. This `log_fanout` shrinkage is why every real database indexes with B⁺-trees.

---

### Problem 5 — Max / min keys for a given height (NAT)

A B-tree of **order 4** (max 4 children) has **3 levels**. Find the maximum and minimum number of keys it can hold.

**Solution.**
**Maximum** `= p^L − 1 = 4³ − 1 = 63` (every node full: 3 keys each; nodes `= 1+4+16 = 21`; keys `= 21×3 = 63` ✓).
**Minimum:** `t = ⌈4/2⌉ = 2` ⇒ `2·t^{L−1} − 1 = 2·2² − 1 = 7`. (Check: root 1 key/2 children; level 2 = 2 nodes × 1 key = 2; level 3 = 4 nodes × 1 key = 4; total `1+2+4 = 7` ✓.)
*Trap:* confusing "order/max children" with CLRS "minimum degree." State which convention the question uses; here order `p = 4` ⇒ min children `⌈p/2⌉ = 2`.

---

### Problem 6 — B⁺-tree insertion walk (order 3) (structure question)

Insert **1, 2, 3, 4, 5** into an empty B⁺-tree of **order 3** (max 3 pointers ⇒ max 2 keys per node; leaf holds 2 keys + next-leaf pointer). Convention: on overflow (3 keys), the **middle** key separates; leaves **copy** it up, internal nodes **push** it up.

**Solution.**
1. **1** → leaf `[1]`.
2. **2** → leaf `[1,2]` (full).
3. **3** → `[1,2,3]` overflows → split leaf into `[1]` | `[2,3]`, **copy up 2**. Root `[2]`; leaves `[1] → [2,3]`. *(2 levels)*
4. **4** → right leaf `[2,3,4]` overflows → split into `[2]` | `[3,4]`, **copy up 3**. Root `[2,3]`; leaves `[1] → [2] → [3,4]`. *(root now full)*
5. **5** → leaf `[3,4,5]` overflows → split into `[3]` | `[4,5]`, copy up 4 ⇒ root would become `[2,3,4]` (3 keys) → **root overflows** → split internal `[2,3,4]`: **push up 3**, left internal `[2]`, right internal `[4]`. New root `[3]`. **Height grows to 3.**

**Final tree:**
```
                 [3]
        ┌─────────┴─────────┐
       [2]                  [4]
     ┌──┴──┐             ┌───┴───┐
   [1]    [2]          [3]     [4,5]
   leaves linked:  [1] → [2] → [3] → [4,5]
```
All data keys `{1,2,3,4,5}` are at the **leaves**; `2,3` also appear as internal routers (copies). Note the height rose **only** when the **root** split — the standard fact GATE checks.

---

### Problem 7 — Identify the index type (MSQ)

Classify each as primary / clustering / secondary, and dense / sparse:
(a) Index on `emp_id` (unique) of a file physically sorted by `emp_id`.
(b) Index on `dept` (many employees per dept) of a file physically grouped by `dept`.
(c) Index on `email` (unique) of a file sorted by `emp_id` (not by email).

**Solution.**
(a) **Primary**, **sparse** (one entry per block; ordering key).
(b) **Clustering**, sparse-ish (one entry per **distinct `dept`**; ordering non-key field).
(c) **Secondary** on a key, **dense** (one entry per record; file not ordered on `email`). *(You may have many such secondary indexes, but only one of (a)/(b).)*

---

## 6. Common GATE question types on this topic

1. **Index sizing & search cost:** compute `bfr_i`, number of index blocks, `⌈log₂ b_i⌉ + 1`; compare primary vs secondary.
2. **Order/fan-out from block & pointer sizes** for B-tree, B⁺-tree internal, B⁺-tree leaf — the most frequent numerical.
3. **Number of levels / height** of a B⁺-tree for `N` keys, and resulting I/O.
4. **Max/min keys (or nodes)** in a B-tree/B⁺-tree of a given order and height.
5. **Insertion (and deletion):** resulting structure, number of splits, whether height changes.
6. **Dense vs sparse, primary vs clustering vs secondary** identification.
7. **B-tree vs B⁺-tree** conceptual differences (true/false).
8. **Hash index vs tree index** suitability (equality vs range).

---

## 7. Traps, pitfalls & common mistakes

- **"Search key" ≠ "primary key."** An index's search key is any indexed field; it need not be a database key.
- **Sparse needs ordered data.** A sparse (one-per-block) index only works when the file is **ordered** on the search key — so secondary indexes (unordered field) are **dense** (or use indirection), never one-per-block sparse.
- **Primary is sparse, secondary-on-key is dense.** Hence a secondary index has *more* entries (≈ `r`) than a primary one (≈ `b`); forgetting this gives wrong sizes. ⚠️
- **Only one primary/clustering index per file; many secondary.** Physical order can satisfy just one. A common false statement: "a table can have two primary indexes."
- **B-tree vs B⁺-tree fan-out.** B⁺-tree internal nodes hold **no record pointers**, so their fan-out is **larger** (use `(B+K)/(P+K)`), while B-tree uses `(B+K+Pr)/(P+K+Pr)`. Plugging record-pointer size into a B⁺-tree internal node is a classic error.
- **Internal order ≠ leaf order in a B⁺-tree.** Compute them separately; they're usually different numbers.
- **Floor vs ceiling.** Order/fan-out and `bfr` use **floor**; counts of blocks/levels/nodes use **ceiling**. Mixing them is a frequent slip.
- **"Order" terminology is ambiguous.** Order `p` (max children, Elmasri) vs minimum-degree `t` (CLRS, min children `t`, max `2t`) vs "order = max keys." State the convention; min children `= ⌈p/2⌉` and `t = ⌈p/2⌉` link them.
- **Height changes only at the root.** In B/B⁺-trees, height increases **only when the root splits** and decreases **only when the root's children merge** — not on ordinary leaf splits.
- **In a B⁺-tree, all data is at the leaves.** Internal keys are *copies/routers*; don't count them as separate data values, and remember search **always** descends to a leaf.
- **Hashing doesn't do ranges.** A hash index is ~1-access for equality only; for range/ordered queries you need a B⁺-tree.
- **Index maintenance has a cost.** Every secondary index must be updated on insert/delete/update of the data — fast reads are bought with slower writes (a conceptual point GATE sometimes probes).

---

## 8. Connection forward

Indexing completes the **Database Design & Storage** arc: Normalization (Topic 7) fixed the *logical* shape, File Organization (Topic 8) placed records on *disk*, and Indexing (Topic 9) gives **fast, multi-field, range-capable access** via small ordered structures and B⁺-trees. With that, the "store and retrieve relational data efficiently" story is done.

The next chat changes subject entirely — from *managing stored data* to *transforming data for analysis and warehousing*. **Topic 10 — Data Types & Data Transformation** covers scaling, discretization, sampling, and compression: the preprocessing that prepares raw values before they enter analytical models and the **multidimensional data warehouse**. ⚠️ Watch the vocabulary: "**normalization**" there means *rescaling a feature to a fixed range* (e.g., min–max to [0,1]) — a totally different idea from the schema normalization of Topic 7.

---

## 9. One-line cheat-sheet summary

**An index is a small `⟨key, pointer⟩` file over the data; primary = sparse on the ordering key (`b` entries), secondary-on-key = dense (`r` entries), search `≈ ⌈log₂ b_i⌉+1`; go multilevel when the index is large, and use a balanced B⁺-tree — internal fan-out `⌊(B+K)/(P+K)⌋`, leaf `⌊(B−P)/(K+Pr)⌋`, all data at linked leaves — for `log_fanout`-fast search, cheap updates, and efficient range queries.**

---

### ▶ Next topic (begins a NEW chat — Data Types & Data Transformation): **Topic 10 — Data Types & Data Transformation** (normalization/scaling, discretization, sampling, compression). Note: "normalization" there = feature rescaling, *not* the schema normalization of Topic 7.

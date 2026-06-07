# Topic 5 — Hash Tables (Hashing and Collision Resolution)

**Subject:** Programming, Data Structures and Algorithms (GATE DA)
*Section: Trees & Hash Tables — Topic 2 of 2 (final topic of this section)*

---

## 0. Topic header & syllabus map

- **Topic:** Hash Tables — hashing (hash functions, load factor) and collision resolution (separate chaining, open addressing: linear/quadratic probing, double hashing).
- **Subject:** Programming, Data Structures and Algorithms.
- **Where it sits:** This is the **last "basic data structure"** in the syllabus and the **fastest-on-average search structure**. It comes **after** Trees/BST (which gave ordered **O(log n)** search) and is the **O(1)-average** alternative. It **closes** the data-structures block; the next section begins a new chat with **Linear and Binary Search**, the comparison-based search baselines that hashing is meant to beat.

---

## 1. Why this topic exists (motivation + connection)

The real problem: **we want search/insert/delete in (near) constant time — faster than the O(log n) a balanced BST gives.**

**What the previous topic couldn't do:** A BST navigates by **comparisons** — at each node you ask "smaller or larger?" and descend. Even at its best the tree has height ≈ log₂ n, so every operation costs **O(log n)** comparisons. The comparisons are unavoidable as long as we *locate by comparing*.

**What hashing fixes:** Instead of *comparing our way* to a key's location, we **compute the location arithmetically** from the key itself via a **hash function** `h(k)`. Lookup becomes "compute one address, look there" — **O(1)** on average, with **no comparison-based descent at all**. The price: we **give up ordering** (no sorted scan, no successor/range queries — those still need a BST), we need **extra space**, and two keys can land in the same slot (a **collision**) which we must resolve.

**Connection in one sentence:** BST = *ordered, guaranteed* O(log n); hash table = *unordered, average* O(1) (but O(n) worst case). The **next section** (linear & binary search on arrays) completes the picture: linear search O(n) with no structure, binary search O(log n) needing sorted data, hashing O(1)-average needing a good hash function and space.

---

## 2. Prerequisites

- **Arrays** (random access in O(1)) — the hash table *is* an array `T[0…m−1]`; hashing decides *which index*.
- **Linked lists** (Topic 3): the chains in separate chaining are linked lists.
- **Modular arithmetic** (`k mod m`), **prime numbers**, **GCD/coprimality** — for hash functions and double hashing.
- **Big-O & expected-value reasoning**: results are *average-case*, under an assumption (simple uniform hashing).
- **BST/Trees** (Topic 4): for the ordered-vs-unordered contrast and the "why not just use a BST" comparison.
- **Logarithm / ln**: the open-addressing successful-search bound uses `ln`.

---

## 3. Core idea — intuition first

Imagine a cloakroom with **m numbered pegs** and far more coats than pegs. You need a rule that, given a coat, instantly tells you **which peg** — without searching. The **hash function** is that rule: feed it the coat's ticket number `k`, it spits out a peg number `h(k)` in `0…m−1`. To store: compute `h(k)`, hang the coat there. To retrieve: compute `h(k)` again, go straight to that peg. No scanning — **one computation, one look.**

Two issues make it interesting:

1. **The function maps a huge universe of keys into a small range**, so different keys can land on the **same peg** — a **collision**. The whole second half of this topic is *how to resolve collisions*.
2. **How full the cloakroom is** matters: the **load factor α = n/m** (coats ÷ pegs) governs how often collisions happen and therefore how fast things stay.

Two families of collision resolution:
- **Separate chaining:** each peg holds a small **linked list**; colliding coats just join the list at that peg.
- **Open addressing:** every coat lives **inside the peg array** itself; on a collision you **probe** a deterministic sequence of other pegs until you find a free one.

The headline guarantee — "O(1)" — is an **average** under the **simple uniform hashing assumption** (every key equally likely to hash to any slot, independently). In the **worst case** (a bad hash function or adversarial keys sending everything to one slot) a hash table degrades to **O(n)**. State this assumption whenever you quote the O(1); GATE loves the worst-case caveat.

---

## 4. Formal treatment

### 4.1 Setup, direct addressing, and load factor

- **Universe** `U` of possible keys; we store a subset of `n` keys in a table `T[0…m−1]` of `m` **slots/buckets**.
- **Direct-address table** (special case): if `U` is small, set `T[k] = ` the element with key `k`. O(1) everything, but wastes Θ(|U|) space — useless when `U` is huge or sparse.
- **Hashing** generalizes this: a **hash function** `h : U → {0,1,…,m−1}` maps the big universe into the small index range, so `T` only needs `m ≈ n` slots.
- **Collision:** `k₁ ≠ k₂` but `h(k₁) = h(k₂)`. Collisions are inevitable once `n > m` (pigeonhole), and likely far earlier.

```
Load factor:   α = n / m
   • separate chaining:  α may exceed 1  (chains absorb overflow)
   • open addressing:    α ≤ 1           (one key per slot ⇒ table can fill up)
```

`α` is the single most important quantity for performance — every cost formula below is written in terms of `α`.

### 4.2 Hash functions (and their conditions)

A **good hash function** is: **deterministic**, **fast (O(1))**, spreads keys **uniformly** (the *simple uniform hashing* ideal), uses the **whole key**, and **minimizes collisions**.

**Common construction methods:**

| Method | Formula | Condition / note |
|---|---|---|
| **Division** | `h(k) = k mod m` | Pick `m` **prime**, **not near a power of 2**. **Avoid `m = 2^p`** (then `h` uses only the low `p` bits) and `m = 10^p` for decimal keys. |
| **Multiplication** | `h(k) = ⌊ m · (k·A mod 1) ⌋`, `0<A<1` | `A ≈ (√5−1)/2 ≈ 0.6180339887` (Knuth) works well; `m` may be a power of 2. |
| **Mid-square** | square `k`, take the **middle** digits/bits | length of middle = `log₂ m`. |
| **Folding** | split `k` into equal parts, **add** them, take mod `m` | good for long keys. |

> **Universal hashing (one-line awareness):** to defeat worst-case/adversarial inputs, pick `h` **at random** from a family of hash functions; this makes the expected number of collisions small *for any fixed input*. Detailed family construction is at GATE-CS depth — know only the *idea*.

### 4.3 Collision resolution I — Separate chaining (open hashing)

Each slot `T[i]` points to a **linked list** of all keys with `h(k)=i`. Insert at the head → **O(1)** insert. Search/delete walk that one chain.

**Performance (simple uniform hashing):**

```
Expected chain length                         = α
Unsuccessful search (examine whole chain):    Θ(1 + α)   ← examines α elements on avg
Successful search   (examine half on avg):    Θ(1 + α)
   exact expected comparisons (CLRS) = 1 + α/2 − α/(2n) ≈ 1 + α/2
Worst case (all n keys in one chain):          Θ(n)
```

- Insertion is O(1) (no search needed if duplicates allowed); deletion is O(1) given the node, else O(1+α) to find it.
- Chaining tolerates **α > 1** gracefully; performance degrades **linearly** in α, not catastrophically.

### 4.4 Collision resolution II — Open addressing (closed hashing)

All keys live **in the array**; no external lists. On collision, follow a **probe sequence** `h(k,0), h(k,1), …, h(k,m−1)` (a permutation of all slots, ideally) until an empty slot is found (for insert) or the key/empty slot is reached (for search).

**(a) Linear probing**
```
h(k, i) = ( h'(k) + i ) mod m ,   i = 0,1,2,…,m−1
```
- Always covers all `m` slots. Simple, cache-friendly.
- **Primary clustering:** occupied slots form long **contiguous runs**; a key hashing anywhere into a run must scan to its end, and that lengthens the run — runs grow and merge, inflating probe counts. Keys with **different** home slots collide along the same stretch.

**(b) Quadratic probing**
```
h(k, i) = ( h'(k) + c₁·i + c₂·i² ) mod m      (simple form: h'(k) + i²)
```
- Reduces clustering vs linear, but has **secondary clustering:** keys with the **same** home slot `h'(k)` follow the **identical** probe sequence (the offset `c₁i+c₂i²` depends only on `i`, not on the key).
- **Coverage caveat:** the probe sequence may **not visit every slot**. For the form `(h'(k)+i²) mod m` with `m` **prime**, only the first **⌈m/2⌉ = (m+1)/2** probes are **distinct** ⇒ insertion is **guaranteed only while the table is less than half full (α < ½)**.

**(c) Double hashing**
```
h(k, i) = ( h₁(k) + i · h₂(k) ) mod m
```
- The **step size depends on the key** (via `h₂`), so even keys with the same `h₁` follow **different** sequences ⇒ **no primary or secondary clustering**; behaves closest to ideal uniform hashing.
- **Condition for full coverage:** `h₂(k)` must be **relatively prime to `m`** for every key. Standard recipes: take `m` **prime** and `h₂(k) = 1 + (k mod (m−1))`; or `m = 2^p` and design `h₂` to always be **odd**.

**Open-addressing performance (uniform hashing assumption, α < 1):**
```
Unsuccessful search / insertion:   expected probes ≤ 1 / (1 − α)
Successful search:                 expected probes ≤ (1/α) · ln( 1 / (1 − α) )
```
(The derivations are standard CLRS bounds; **derivation skipped — GATE only uses the results.**) Note the **explosion as α → 1**: at α = 0.9, unsuccessful search already needs ≤ 10 probes. This is why open-addressed tables are kept well below full (and quadratic probing below half).

**Deletion in open addressing — the subtlety GATE tests:**
You **cannot** simply blank a slot — that would prematurely halt the probe sequence of *other* keys that probed past it, making them "disappear." Instead mark the slot **DELETED** (a **tombstone**): **search** treats DELETED as "occupied, keep probing"; **insert** treats DELETED as "free, reuse it." Too many tombstones inflate search cost (α effectively rises) → eventually **rehash**.

### 4.5 Rehashing (table resizing)

When `α` crosses a threshold (typically ~0.5 for quadratic probing, ~0.7–0.75 generally), allocate a **larger** table (commonly the next prime ≈ 2×) and **re-insert every live key** by recomputing `h` with the new `m` (old indices are meaningless under a new modulus). Each rehash is Θ(n), but doubling makes insertion **amortized O(1)**. Rehashing also clears accumulated tombstones.

### 4.6 Chaining vs open addressing — the comparison table

| Aspect | Separate chaining | Open addressing |
|---|---|---|
| Storage | array + linked lists (extra pointers) | single array (no pointers) |
| Load factor α | can exceed 1 | must stay ≤ 1 (and `<½` for quadratic) |
| Collision handling | append to chain | probe sequence in the table |
| Deletion | easy (unlink node) | needs **tombstones** |
| Clustering | none | primary (linear) / secondary (quadratic) |
| Cache behavior | poorer (pointer chasing) | better (contiguous) |
| Worst case | Θ(n) (one long chain) | Θ(n) (long probe run) |
| Avg search | Θ(1 + α) | ≤ 1/(1−α) unsuccessful |

---

## 5. Worked problems (GATE-style)

### Q1 (NAT) — Separate chaining: chain lengths & expected search
*`m = 9`, `h(k) = k mod 9`, separate chaining. Insert: 9, 18, 27, 5, 14, 7. (a) Max chain length? (b) Expected comparisons for an unsuccessful search? (c) Expected comparisons for a successful search?*

**Solution.** Hashes: 9→0, 18→0, 27→0, 5→5, 14→5, 7→7.
Chains: slot 0 = {9,18,27} (len 3), slot 5 = {5,14} (len 2), slot 7 = {7} (len 1). `n=6, m=9 ⇒ α = 6/9 = 2/3`.
(a) **Max chain = 3.**
(b) Unsuccessful examines the whole chain at the hashed slot ⇒ expected = **α = 0.667** comparisons (time Θ(1+α)).
(c) Exact successful = `1 + α/2 − α/(2n) = 1 + 1/3 − (2/3)/12 = 1.278` (≈ `1 + α/2 = 1.33`).

### Q2 (NAT) — Open addressing: expected probes via 1/(1−α)
*An open-addressed table is 75% full. Under uniform hashing, give expected probes for (a) an unsuccessful search/insertion and (b) a successful search.*

**Solution.** α = 0.75.
(a) `1/(1−α) = 1/0.25 =` **4 probes.**
(b) `(1/α)·ln(1/(1−α)) = (1/0.75)·ln(4) = 1.333 × 1.386 =` **≈ 1.85 probes.**
*(Note how cheap a successful search stays even when an unsuccessful one is already 4 — successful searches stop early at the found key.)*

### Q3 (NAT/MCQ) — Linear probing insertion
*`m = 10`, `h(k) = k mod 10`, linear probing. Insert in order: 23, 13, 21, 14, 7, 8, 15, 99. (a) Final table. (b) How many keys collided (needed > 1 probe)? (c) Probes to insert 15?*

**Solution.** Step through (home → final):
- 23→3 (slot 3, 1 probe); 13→3 occ→4 (2); 21→1 (1); 14→4 occ→5 (2); 7→7 (1); 8→8 (1); 15→5 occ→6 (2); 99→9 (1).

| idx | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|---|---|---|---|---|---|---|---|---|---|---|
| key | – | 21 | – | 23 | 13 | 14 | 15 | 7 | 8 | 99 |

(b) Keys needing >1 probe: 13, 14, 15 ⇒ **3 collided.**
(c) 15 probes slot 5 (occupied by 14) then slot 6 (empty) ⇒ **2 probes.**

### Q4 (MCQ/NAT) — Quadratic probing & the half-full limit
*`m = 11` (prime), `h(k) = k mod 11`, quadratic probing `h(k,i) = (k mod 11 + i²) mod 11`. Insert 3, 14, 25, 36, 47 (all hash to 3) in this order. Where do they land, and how many distinct slots can this scheme ever reach?*

**Solution.** All have home 3. Probe offsets `i² mod 11`: i=0→0, 1→1, 2→4, 3→9, 4→5 ⇒ slots `3, 4, 7, (3+9)=12 mod 11 =1, (3+5)=8`.
- 3 → **3**; 14 → **4**; 25 → **7**; 36 → **1**; 47 → **8**.

Distinct reachable slots = first `(m+1)/2 = 6` offsets (i=0…5: 3,4,7,1,8,6), then they **repeat**. So at most **6 slots** are reachable ⇒ insertion is guaranteed **only while α < ½**.

### Q5 (NAT) — Double hashing
*`m = 13`, `h₁(k)=k mod 13`, `h₂(k)=1+(k mod 11)`. Insert 9, 22, 35 in order. Final positions?*

**Solution.** `h₁`: 9→9, 22→ (22 mod 13)=9, 35→(35 mod 13)=9 — all collide at 9.
- 9: slot 9 free → **9**.
- 22: 9 occ. `h₂(22)=1+(22 mod 11)=1+0=1`. probe `(9+1·1)=10` → **10**.
- 35: 9 occ. `h₂(35)=1+(35 mod 11)=1+2=3`. probe `(9+1·3)=12` → **12**.

Positions **9, 10, 12**. Note 22 and 35 used **different step sizes** (1 vs 3) despite the same home — that is exactly how double hashing avoids secondary clustering. (13 prime ⇒ every `h₂∈[1,12]` is coprime to 13, so full coverage is guaranteed.)

### Q6 (MCQ — trap) — Why open-addressing deletion needs a tombstone
*`m = 7`, `h(k)=k mod 7`, linear probing. Insert 7, 14, 21. Then physically empty the slot holding 14 and search for 21. What goes wrong?*

**Solution.** All three hash to 0: 7→slot 0, 14→slot 1, 21→slot 2 (table `0:7, 1:14, 2:21`). Naively deleting 14 leaves slot 1 **empty**. Searching 21: probe slot 0 (7 ≠ 21), probe slot 1 (**empty → search stops**, reports "not found") — **but 21 is sitting in slot 2.** The empty slot broke 21's probe chain. **Fix:** mark slot 1 **DELETED** (tombstone); search then keeps probing past it and finds 21.

### Q7 (MCQ — trap) — Reconstruct the insertion order (linear probing)
*`m = 7`, `h(k)=k mod 7`, linear probing produced this table:* `0:7, 1:14, 2:8, 3:9` *(others empty). Keys' homes: 7→0, 14→0, 8→1, 9→2. Which insertion order is consistent?*
(a) 7, 8, 14, 9  (b) 7, 14, 8, 9  (c) 14, 7, 8, 9

**Solution.** Deduce constraints from displacement:
- 7 is at its home 0 — fine anytime.
- 14 (home 0) sits at 1 ⇒ slot 0 was full when 14 arrived ⇒ **7 before 14**.
- 8 (home 1) sits at 2 ⇒ slot 1 was full when 8 arrived ⇒ **14 before 8**.
- 9 (home 2) sits at 3 ⇒ slot 2 was full when 9 arrived ⇒ **8 before 9**.

Required order: **7 → 14 → 8 → 9** ⇒ **(b).**
Quick check of (a) 7,8,14,9: 7→0, 8→1, 14→0,1 occ→2, 9→2 occ→3 ⇒ table `0:7,1:8,2:14,3:9` (≠ target). (c) 14,7,8,9: 14→0, 7→1, 8→2, 9→3 ⇒ `0:14,1:7,…` (≠ target). Only **(b)** works.

### Q8 (MSQ) — True/false on hashing
*Which are TRUE?*
(i) In open addressing the load factor can exceed 1.
(ii) Double hashing suffers from primary clustering.
(iii) Using `h(k)=k mod m` with `m = 2^p` is a poor choice.
(iv) For separate chaining, average unsuccessful-search time is Θ(1 + α).

**Solution.**
- (i) **False** — open addressing stores ≤ 1 key/slot, so α ≤ 1.
- (ii) **False** — the per-key step `h₂(k)` eliminates primary (and secondary) clustering.
- (iii) **True** — `mod 2^p` keeps only the low `p` bits, ignoring the rest of the key ⇒ poor spread; prefer a prime `m`.
- (iv) **True** — that is the standard chaining bound.

**Answer: (iii) and (iv).**

---

## 6. Common GATE question types on this topic

- **Insert a key list with `h(k)=k mod m`** under chaining / linear / quadratic / double hashing; produce the **final table**.
- **Count collisions / total probes / probes for one key**; identify the slot a given key ends in.
- **Compute expected probes** from α: chaining `Θ(1+α)` (and `1+α/2`); open addressing `1/(1−α)` (unsuccessful) and `(1/α)ln(1/(1−α))` (successful).
- **Reconstruct a valid insertion order** from a final linear-probing table (the displacement-constraint trap).
- **Clustering identification:** which scheme has primary vs secondary clustering.
- **Coverage conditions:** quadratic needs α < ½ (prime m); double hashing needs `h₂` coprime to `m`.
- **Deletion correctness** in open addressing (tombstones).
- **Choosing/critiquing a hash function** (why prime `m`, why not `2^p`).
- **Chaining vs open addressing** comparison (space, α range, deletion, cache).
- **Worst-case vs average-case** statements (the O(1) vs O(n) caveat).

---

## 7. Traps, pitfalls & common mistakes

1. **"O(1)" is average, under simple uniform hashing — worst case is O(n).** Always pair the claim with the assumption; GATE answers hinge on this.
2. **α > 1 is possible for chaining but impossible for open addressing.** Mixing this up flips MSQ truth values.
3. **Quadratic probing can fail to find a free slot even when one exists** — only `(m+1)/2` slots reachable (prime `m`) ⇒ keep **α < ½**.
4. **Double hashing needs `h₂(k)` coprime to `m`** (else the probe sequence cycles before covering the table). Pick `m` prime or force `h₂` odd with `m=2^p`.
5. **Never physically delete in open addressing** — use a **tombstone**; otherwise later searches stop early and "lose" keys.
6. **Primary vs secondary clustering:** primary = **linear** probing (different homes collide along runs); secondary = **quadratic** (same home ⇒ same sequence). **Double hashing has neither.**
7. **`mod 2^p` (or `mod 10^p`) is a bad hash** — it discards high-order bits/digits. Prefer prime `m` away from powers of 2 for the division method.
8. **Insertion order changes the open-addressed table** — the same key-set can yield different layouts; a "final table" determines only a *partial order* of insertions.
9. **Hash indices are modulus-specific** — after rehashing to a new `m` you **must recompute** `h(k)`; old slot numbers carry no meaning.
10. **Don't confuse the two cost formulas** — chaining uses `1+α`; open addressing uses `1/(1−α)`. They behave very differently as α→1 (linear vs blow-up).
11. **Off-by-one in `(h'(k)+i)` probing:** `i` starts at **0** (so the first probe is the home slot). Counting probes as `i+1`.
12. **Successful vs unsuccessful** are different costs — a successful search stops at the found key (cheaper), an unsuccessful one runs to an empty slot/chain end.

---

## 8. Connection forward

This closes the data-structures block, and it sets the stage for the **Searching & Sorting** section (next chat). We now hold the **complete search-cost landscape**:

- **Linear search** — O(n), needs **no structure** (works on any array/list) — *formalized next.*
- **Binary search / BST** — O(log n), needs **sorted/ordered** data — *binary search formalized next.*
- **Hashing** — **O(1) average** (O(n) worst), needs a **good hash function + extra space**, and gives **no ordering** (no range/successor queries).

The very next topic, **Linear and Binary Search**, makes rigorous the two comparison-based baselines we have been informally invoking — including binary search's hard requirement of a **sorted array** (the array analogue of a BST's sorted inorder) and its O(log n) recurrence, which then leads into sorting (how to *produce* that sorted order) and divide-and-conquer.

---

## 9. One-line cheat-sheet summary

> **Hash table:** compute index `h(k)`; **load factor α = n/m**. **Chaining:** lists per slot, α can exceed 1, search Θ(1+α) (successful ≈ 1+α/2). **Open addressing** (α ≤ 1, one key/slot): linear `(h'+i)`→**primary clustering**; quadratic `(h'+i²)`→**secondary clustering**, needs **α<½** (prime m); double `(h₁+i·h₂)`→no clustering, needs **h₂ coprime to m**. Open-addressing expected probes: **1/(1−α)** unsuccessful, **(1/α)ln(1/(1−α))** successful. **Delete via tombstones**, not blanking. Use **prime m**, never `2^p`, for `k mod m`. **O(1) is average** (worst case O(n)); rehash near the α threshold.

---

*This is the final topic of the **Trees & Hash Tables** section. The next chat begins a new section — **Searching & Sorting**, starting with **Linear and Binary Search** — which formalizes the O(n) and O(log n) comparison-based search methods that hashing's O(1)-average is meant to outperform. Open a new chat and ask for it when ready.*

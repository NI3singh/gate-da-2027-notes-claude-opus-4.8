# Topic 7 — Basic Sorting (Selection, Bubble, Insertion)

**Subject:** Programming, Data Structures and Algorithms (GATE DA)
**Section:** Searching & Sorting (this is the second topic; it follows search)

---

## 0. Topic header & syllabus map

- **Topic:** Basic comparison sorts — **Selection Sort, Bubble Sort, Insertion Sort**.
- **Subject:** Programming, Data Structures and Algorithms.
- **Where it sits:** Comes right after **Linear and Binary Search**. Search could *find* an element in sorted data but could not *create* the order; these algorithms produce that order. All three run in **Θ(n²)**, which is exactly why the next topic, **Divide and Conquer** (mergesort, quicksort), exists — to break the n² barrier and reach Θ(n log n).

---

## 1. Why this topic exists (motivation + connection)

**The problem it solves.** Put a collection into order. Sorted data powers binary search, deduplication, grouping/aggregation, finding medians and order statistics, and is a preprocessing step for countless algorithms.

**What the previous topic couldn't do (connection backward).** Binary search gave you O(log n) lookups — *but only on sorted data*, and it had no way to **produce** that sortedness. Search **reads**; it never **rearranges**. Sorting is the missing piece: it manufactures the precondition binary search depends on. (Neat tie-in: a *binary insertion sort* even uses binary search to locate each insertion point.)

**What the next topic builds (connection forward).** These three sorts are simple and in-place, but all are **Θ(n²)** on average — they choke on large inputs. **Divide and Conquer** recursively splits the array to reach **Θ(n log n)**. Two direct bridges: (i) **insertion sort reappears as the base case** inside fast hybrid sorts (it's unbeatable on tiny subarrays), and (ii) **counting inversions** — the quantity that governs bubble/insertion cost — can be done in Θ(n log n) with a modified mergesort.

---

## 2. Prerequisites

- **Arrays, 0-based indexing, and swapping** two elements (`a[i], a[j] = a[j], a[i]`).
- **Asymptotic notation** (best/worst/average; Θ, O).
- **The summation** `1 + 2 + … + (n−1) = n(n−1)/2` — every count in this topic reduces to it.
- **Four definitions** used throughout (formalized in §4): *comparison sort, in-place, stable, adaptive/online*.
- **Inversions** (defined in §4.4) — the bridge between "swaps/shifts" and "how sorted the input is".

---

## 3. Core idea — intuition first

All three repeatedly grow a **sorted region** until it covers the whole array; they differ in *how* they extend it.

- **Selection sort — "pick the smallest, place it."** Look through the entire unsorted part, find the minimum, and swap it to the front of the unsorted part. Like scanning a shelf for the shortest book and moving it to the left end, then repeating on the rest. It always scans everything, so it never "notices" that the data might already be sorted.

- **Bubble sort — "let big values float right."** Walk left to right comparing each adjacent pair; swap if out of order. The largest element gets carried to the far right in one pass, like a bubble rising. Repeat; each pass parks one more big element at the end. If a whole pass makes no swaps, the array is already sorted and you can stop.

- **Insertion sort — "sort a hand of cards."** Keep the left part sorted. Pick the next card and slide it leftward, shifting larger cards right, until it drops into its slot. This is what most people actually do with playing cards, and it's blazing fast when the hand is *almost* sorted.

Tiny feel for the difference on `[3, 1, 2]`:
- **Selection:** scan → min is `1`, swap to front → `[1, 3, 2]`; scan rest → min is `2`, swap → `[1, 2, 3]`.
- **Bubble:** `(3,1)` swap → `[1, 3, 2]`; `(3,2)` swap → `[1, 2, 3]`; next pass no swaps → stop.
- **Insertion:** insert `1` before `3` → `[1, 3, 2]`; insert `2` between `1` and `3` → `[1, 2, 3]`.

---

## 4. Formal treatment

**Four definitions (GATE tests these directly):**
- **Comparison sort:** orders elements only by comparing pairs (`<`, `>`). All three here are comparison sorts.
- **In-place:** uses O(1) extra memory beyond the input array. All three are in-place.
- **Stable:** equal keys keep their original relative order after sorting.
- **Adaptive:** runs faster when the input is already partially sorted. **Online:** can sort data as it arrives, one element at a time.

### 4.1 Selection Sort

```python
def selection_sort(a):
    n = len(a)
    for i in range(n - 1):              # n-1 passes
        min_idx = i
        for j in range(i + 1, n):       # scan the unsorted part fully
            if a[j] < a[min_idx]:
                min_idx = j
        a[i], a[min_idx] = a[min_idx], a[i]   # one swap per pass
```

*Invariant:* after pass `i`, `a[0..i]` holds the `i+1` **smallest** elements, in final sorted position.

> **Selection sort — comparisons**
> **Always exactly n(n−1)/2** comparisons.
> *Condition:* the inner loop runs fully every pass regardless of input ⟹ best = worst = average. Selection sort is **not adaptive**: a sorted array costs the same as a reversed one.

> **Selection sort — swaps**
> **≤ n−1 swaps** (exactly n−1 in the version that always swaps) ⟹ **Θ(n)** data moves.
> *Significance:* this is the **fewest writes** of the three — ideal when writing is expensive (e.g. flash/EEPROM).

> **Time:** Best = Worst = Average = **Θ(n²)**. **Stable? No.** In-place? Yes.

*Why unstable:* `[4a, 4b, 1]` → pass 1 swaps the min `1` into index 0, giving `[1, 4b, 4a]` — the two equal `4`s are now reversed.

### 4.2 Bubble Sort (with the standard early-exit optimization)

```python
def bubble_sort(a):
    n = len(a)
    for i in range(n - 1):                 # up to n-1 passes
        swapped = False
        for j in range(n - 1 - i):         # adjacent compares (tail is sorted)
            if a[j] > a[j + 1]:
                a[j], a[j + 1] = a[j + 1], a[j]
                swapped = True
        if not swapped:                    # no swaps ⟹ already sorted ⟹ stop
            break
```

*Invariant:* after pass `i`, the last `i+1` elements are the **largest**, in final position.

> **Bubble sort — comparisons**
> Optimized: best = **n−1** (one clean pass on sorted input); worst = **n(n−1)/2**.
> *Without the `swapped` flag (plain bubble):* comparisons are **always n(n−1)/2** — plain bubble is **not adaptive**. The Θ(n) best case **requires** the early-exit flag.

> **Bubble sort — swaps**
> **Number of swaps = number of inversions** in the array (each adjacent swap removes exactly one inversion).
> Best = **0** (sorted); worst = **n(n−1)/2** (reverse sorted) — the **most swaps** of the three.

> **Time:** Best = **Θ(n)** (optimized), Worst = Average = **Θ(n²)**. **Stable? Yes** (equals never cross — swap only on strict `>`). In-place? Yes.

### 4.3 Insertion Sort

```python
def insertion_sort(a):
    for i in range(1, len(a)):
        key = a[i]
        j = i - 1
        while j >= 0 and a[j] > key:   # shift larger elements right
            a[j + 1] = a[j]
            j -= 1
        a[j + 1] = key                 # drop key into its slot
```

*Invariant:* before iteration `i`, `a[0..i-1]` is sorted (a sorted prefix that grows by one each step).

> **Insertion sort — comparisons & shifts**
> Best (already sorted): **n−1** comparisons, **0** shifts ⟹ **Θ(n)**.
> Worst (reverse sorted): **n(n−1)/2** comparisons and shifts ⟹ **Θ(n²)**.
> **Total work = Θ(n + I)**, where **I = number of inversions** = number of shifts. So insertion sort is **fast on nearly-sorted data**.

> **Time:** Best = **Θ(n)**, Worst = Average = **Θ(n²)**. **Stable? Yes.** **Online? Yes** (uniquely among the three — can absorb elements as they arrive). In-place? Yes.

**Binary insertion sort (nuance, connection backward).** Replace the linear "find the slot" scan with **binary search** → comparisons drop to **Θ(n log n)**, but **shifts stay Θ(n²)**, so overall time is still **Θ(n²)**. Useful when comparisons are far costlier than moves.

### 4.4 Inversions — the unifying sub-concept

An **inversion** is a pair of indices `(i, j)` with `i < j` but `a[i] > a[j]` (an out-of-order pair).

> **Facts on inversions** (let `I` = number of inversions)
> - `I = 0` ⟺ array sorted ascending; `I = n(n−1)/2` (the **maximum**) ⟺ array reverse-sorted.
> - **Bubble-sort swaps = insertion-sort shifts = I.**
> - Brute-force counting of `I` is O(n²); a **modified mergesort counts `I` in Θ(n log n)** (preview of Divide & Conquer).

### 4.5 The GATE comparison table

| Property | Selection | Bubble (optimized) | Insertion |
|---|---|---|---|
| Best-case time | Θ(n²) | **Θ(n)** | **Θ(n)** |
| Worst-case time | Θ(n²) | Θ(n²) | Θ(n²) |
| Average-case time | Θ(n²) | Θ(n²) | Θ(n²) |
| Comparisons (best) | n(n−1)/2 (**always**) | n−1 | n−1 |
| Swaps / moves (worst) | **≤ n−1** (Θ(n)) | n(n−1)/2 (**most**) | n(n−1)/2 shifts |
| Swaps relate to inversions? | No | **= I** | shifts **= I** |
| Stable? | **No** | Yes | Yes |
| Adaptive? | **No** | Yes (needs flag) | Yes |
| Online? | No | No | **Yes** |
| Extra space | O(1) | O(1) | O(1) |

*Reading the table:* **fewest writes → Selection. Nearly-sorted / small / streaming data → Insertion. Stable + simple, can detect "already sorted" → Bubble.** Reverse-sorted is the worst case for all three.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Selection sort on a *sorted* array (NAT, trap)

An array of **8 distinct integers is already sorted in ascending order**. How many **element comparisons** does selection sort perform?

**Solution.** Selection sort's comparison count is **input-independent**: the inner loop always runs fully.
`n(n−1)/2 = 8·7/2 = 28`.

**Answer: 28.**
**Trap:** "It's already sorted, so 0 (or very few)" — *wrong.* Sortedness saves selection sort nothing; it still does all 28 comparisons (and ≤ 7 swaps). Only **bubble (optimized)** and **insertion** detect sortedness and finish in Θ(n).

---

### Problem 2 — Swaps = inversions (NAT)

For `A = [2, 5, 1, 4, 3]`, how many **swaps** does (optimized) bubble sort perform, and how many **shifts** does insertion sort perform?

**Solution.** Both equal the number of **inversions**. Count out-of-order pairs:
`(2,1)`, `(5,1)`, `(5,4)`, `(5,3)`, `(4,3)` → **5 inversions**.

Quick verification by tracing bubble sort:
- Pass 1: `(5,1)`swap, `(5,4)`swap, `(5,3)`swap → `[2,1,4,3,5]` (3 swaps)
- Pass 2: `(2,1)`swap, `(4,3)`swap → `[1,2,3,4,5]` (2 swaps)
- Pass 3: no swaps → stop.

Total swaps = `3 + 2 = 5`. ✓

**Answer: 5 swaps (bubble) and 5 shifts (insertion).**

---

### Problem 3 — Identify the algorithm from a snapshot (MCQ)

The array `[6, 5, 3, 1, 8, 7, 2, 4]` is being sorted **ascending**. After **two complete passes** the array is:
`[3, 1, 5, 6, 2, 4, 7, 8]`. Which algorithm is it?

(A) Selection sort  (B) Bubble sort  (C) Insertion sort  (D) Quicksort

**Solution — read the signature of each after 2 passes.**
- **Selection (first k smallest fixed):** `[1, 2, 3, 6, 8, 7, 5, 4]` → leftmost two would be `1, 2`. ✗
- **Bubble (last k largest fixed):** `[3, 1, 5, 6, 2, 4, 7, 8]` → rightmost two are the two **largest** `7, 8`, in place; the rest is only partly ordered. ✓
- **Insertion (first k+1 sorted among themselves):** the first three would be sorted, e.g. `[3, 5, 6, …]`. ✗

The two largest elements sitting sorted at the **end** is the bubble-sort fingerprint.

**Answer: (B) Bubble sort.**
**How to remember:** *Selection fills the front with smallest; Bubble fills the back with largest; Insertion keeps a sorted prefix that isn't yet in final positions.*

---

### Problem 4 — True/False properties (MSQ)

Select all correct statements.

1. Selection sort is stable.
2. With the early-exit optimization, bubble sort runs in Θ(n) on an already-sorted array.
3. Insertion sort is an online algorithm.
4. The number of comparisons made by selection sort depends on the input order.

**Solution.**
1. **False** — the long-distance swap can reorder equal keys.
2. **True** — one pass with no swaps triggers the early exit ⟹ Θ(n).
3. **True** — insertion sort can place each new element as it arrives.
4. **False** — selection sort always does exactly `n(n−1)/2` comparisons.

**Answer: 2 and 3.**

---

## 6. Common GATE question types on this topic

- **Exact comparison counts** — especially *selection = n(n−1)/2 always*.
- **Exact swap / move counts** — usually reducible to **number of inversions** (bubble swaps, insertion shifts).
- **Best / worst / average complexity** of each algorithm, and *which is which*.
- **Stability** — which of the three are stable (Bubble, Insertion = yes; Selection = no).
- **Adaptive / online** properties.
- **Snapshot / pass-count questions** — "after k passes the array is …, which algorithm?" or "how many passes until sorted?".
- **Inversions** — max/min, and "swaps required" = inversion count.
- **Selection criteria** — fewest swaps (Selection), best for nearly-sorted/streaming/small (Insertion), best when writes are expensive (Selection).
- **Python output / fill-in-the-blank** — predict the array after partial execution, or complete a loop bound.

---

## 7. Traps, pitfalls & common mistakes

- **Selection sort comparisons are input-independent.** Sorted, reversed, or random — always `n(n−1)/2`. This is the single most-tested trap. (Its *swaps* can be 0, but comparisons cannot.)
- **Selection sort is NOT stable.** Don't assume "simple sort ⟹ stable". Bubble and insertion are stable; selection is not.
- **Plain bubble sort is NOT adaptive.** The famous Θ(n) best case **requires** the `swapped` early-exit flag. Without it, comparisons stay `n(n−1)/2` even on sorted input.
- **All three average Θ(n²)** — none is asymptotically better than another on average. Differences are in constants, swaps, stability, and best case.
- **Best case is Θ(n), not Θ(1).** Bubble/insertion still scan once (`n−1` comparisons) to confirm sortedness.
- **Stability hinges on a strict comparison.** The `a[j] > key` / `a[j] > a[j+1]` use strict `>`; changing it to `>=` would make equal elements cross and **break stability** — a subtle exam point.
- **"Fewest swaps" ≠ "fewest comparisons".** Selection has fewest *swaps* (Θ(n)) but a fixed, large number of *comparisons*. Bubble has the *most* swaps in the worst case (`n(n−1)/2`).
- **Snapshot signatures must not be mixed up:** after k passes — Selection fixes the **k smallest at the front**, Bubble fixes the **k largest at the back**, Insertion has the **first k+1 sorted among themselves but not in final positions**.
- **Passes vs comparisons vs swaps** are three different counts; read exactly what's asked.
- **Off-by-one in loop bounds:** `n−1` passes; bubble's inner loop shrinks (`range(n-1-i)`); insertion's `while j >= 0`. Forgetting these causes index errors or extra/missed work.
- **Reverse-sorted is the worst case for all three** (maximum inversions `n(n−1)/2`).

---

## 8. Connection forward

Every algorithm here is bounded by **Θ(n²)** on average — perfectly fine for small or nearly-sorted inputs, but hopeless at scale. The next topic, **Divide and Conquer (mergesort, quicksort)**, splits the array recursively to reach **Θ(n log n)**, which is the best any comparison sort can do (the comparison-sort lower bound is **Ω(n log n)**). Two threads from this topic feed directly into it: **insertion sort becomes the base case** for tiny subarrays in real hybrid sorts, and the **inversion count** that drove bubble/insertion cost is itself computed in **Θ(n log n)** by a modified mergesort. Sorting also finally *delivers* the sorted array that **binary search** (previous topic) needs.

---

## 9. One-line cheat-sheet summary

All three are in-place, average **Θ(n²)**. **Selection:** always `n(n−1)/2` comparisons, ≤ n−1 swaps, **unstable, not adaptive**. **Bubble:** swaps = inversions, **stable & adaptive** (with flag), Θ(n) best. **Insertion:** comparisons/shifts = inversions, **stable, adaptive, online**, Θ(n) best — the go-to for small/nearly-sorted data.

---

*Next topic (logically): **Divide and Conquer** — mergesort and quicksort.*

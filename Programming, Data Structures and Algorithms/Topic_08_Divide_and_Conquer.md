# Topic 8 — Divide and Conquer (Merge Sort and Quick Sort)

**Subject:** Programming, Data Structures and Algorithms (GATE DA)
**Section:** Searching & Sorting (final topic of this section)

---

## 0. Topic header & syllabus map

- **Topic:** The **Divide and Conquer** paradigm, applied to the two optimal comparison sorts — **Merge Sort** and **Quick Sort** — together with the **Master Theorem** used to analyze them.
- **Subject:** Programming, Data Structures and Algorithms.
- **Where it sits:** Comes right after **Basic Sorting**, whose algorithms were all Θ(n²). Divide and conquer breaks that barrier and reaches **Θ(n log n)** — the best any comparison sort can achieve. This **closes the Searching & Sorting section**; the next chat opens a new area, **Introduction to Graph Theory**.

---

## 1. Why this topic exists (motivation + connection)

**The problem it solves.** Sort large inputs *fast*. The elementary sorts are fine for tiny or nearly-sorted data, but Θ(n²) is unusable at scale (a million elements ⇒ ~10¹² operations).

**What the previous topic couldn't do (connection backward).** Selection, bubble, and insertion sort all grow the sorted region **one element at a time**, doing local, incremental work — that is *why* they cost Θ(n²) comparisons. **Divide and conquer** changes the strategy: split the array into **independent subproblems**, solve each recursively, then **combine**. Because each level processes all n elements in Θ(n) total and there are only Θ(log n) levels, the cost drops to **Θ(n log n)**. (Insertion sort doesn't vanish, though — it returns as the **base case** for tiny subarrays inside real hybrid sorts.)

**What this enables next (connection forward).** With merge/quick sort we now own the **Θ(n log n)** frontier, matching the **Ω(n log n)** lower bound for comparison sorts — the sorting story is complete. Two threads carry on: merge sort's **merge** step is the engine for **external sorting** and for **counting inversions in Θ(n log n)** (the open thread from Topic 7), and the divide-and-conquer mindset plus **recurrence analysis** (Master Theorem, recursion trees) are reusable tools you'll lean on throughout algorithms, including the graph chapter ahead.

---

## 2. Prerequisites

- **Recursion** — base case + recursive calls, and how the call stack grows.
- **Basic Sorting (Topic 7)** — for the Θ(n²) contrast, and insertion sort as a base case.
- **Asymptotic notation, `log₂`, and summations** — `1+2+…+(n−1)=n(n−1)/2`.
- **Recurrence relations** — what `T(n)=2T(n/2)+n` *means* and how to solve it.
- **Inversions (Topic 7)** — the quantity merge sort can count in Θ(n log n).
- **Stable / in-place** definitions (Topic 7) — used to separate the two sorts.

---

## 3. Core idea — intuition first

**Divide and conquer in three steps:** **Divide** the problem into smaller subproblems → **Conquer** each by recursion (until a trivial base case) → **Combine** the sub-answers into the full answer. The cleverness lives in *where the real work happens*:

- **Merge sort — work in the COMBINE step.** Splitting in half is trivial (just an index). The effort is **merging** two already-sorted halves into one sorted whole. Think of merging two sorted piles of exam papers: repeatedly take the smaller top card. Because each half is pre-sorted, one linear pass merges them.

- **Quick sort — work in the DIVIDE step.** Here splitting is the hard part: pick a **pivot** and **partition** so that everything smaller sits left of it and everything larger sits right — the pivot lands in its final position. After that, the two sides are independent and need no combining at all. Like sorting a room of people by height: pick someone, send shorter people left and taller people right, then sort each group.

The trade-off this sets up: merge sort's split is cheap but its merge needs **extra space**; quicksort's combine is free and **in-place**, but a *bad pivot* can make the split lopsided and ruin the running time.

---

## 4. Formal treatment

### 4.1 The Master Theorem (the analysis tool for both sorts)

For a recurrence of the form `T(n) = a·T(n/b) + f(n)` with `a ≥ 1`, `b > 1`, let the **critical exponent** be `c* = log_b a` (so `n^{c*} = n^{log_b a}`).

> **Master Theorem (three cases)**
> Compare `f(n)` with `n^{log_b a}`:
> - **Case 1 (recursion dominates):** if `f(n) = O(n^{log_b a − ε})` for some `ε > 0` ⟹ **T(n) = Θ(n^{log_b a})**.
> - **Case 2 (balanced):** if `f(n) = Θ(n^{log_b a} · log^k n)` for some `k ≥ 0` ⟹ **T(n) = Θ(n^{log_b a} · log^{k+1} n)**.
> - **Case 3 (combine dominates):** if `f(n) = Ω(n^{log_b a + ε})` for some `ε > 0` **and** the regularity condition `a·f(n/b) ≤ c·f(n)` holds for some `c < 1` and large `n` ⟹ **T(n) = Θ(f(n))**.
> *Caveats:* Cases 1 and 3 need a **polynomial gap** (the `ε`). Case 3 **must** also pass the regularity check. The theorem does **not** apply to recurrences like `T(n)=T(n−1)+f(n)` (not of the form `aT(n/b)`) or to "gap" cases with no polynomial separation (e.g. `f(n)=n/log n`).

**Recursion-tree view (why merge sort is Θ(n log n)):** with `T(n)=2T(n/2)+cn`, each level does `cn` total work, and the tree has `⌈log₂ n⌉ + 1` levels (root at n, leaves at 1). Total `= cn · (log₂ n + 1) = Θ(n log n)`.

### 4.2 Merge Sort

```python
def merge_sort(a):
    if len(a) <= 1:
        return a
    mid   = len(a) // 2
    left  = merge_sort(a[:mid])     # divide + conquer
    right = merge_sort(a[mid:])
    return merge(left, right)       # combine

def merge(L, R):
    out, i, j = [], 0, 0
    while i < len(L) and j < len(R):
        if L[i] <= R[j]:            # '<=' takes from LEFT on ties -> STABLE
            out.append(L[i]); i += 1
        else:
            out.append(R[j]); j += 1
    out.extend(L[i:]); out.extend(R[j:])   # copy the leftover tail
    return out
```

> **Merge sort — recurrence & time**
> `T(n) = 2T(n/2) + Θ(n)`. Master Theorem Case 2 (`a=2, b=2, n^{log_b a}=n`, `k=0`) ⟹ **Θ(n log n)**.
> *Condition:* holds for **every** input — **Best = Worst = Average = Θ(n log n)**. Standard merge sort is **not adaptive** (it splits in half regardless), but it is also **never quadratic**.

> **Merge sort — comparisons**
> Merging two sorted runs of sizes `m` and `n`: **minimum = min(m, n)**, **maximum = m + n − 1**.
> Total (for `n` a power of 2, worst case): **n·log₂ n − n + 1** comparisons → Θ(n log n).

> **Merge sort — space & stability**
> **Auxiliary space Θ(n)** (the merge needs a temporary buffer) ⟹ **not in-place** (array version). Recursion depth **Θ(log n)**.
> **Stable: Yes** (ties resolved in favour of the left run). On a **linked list**, merge sort needs **no random access** and can run with **O(1)** extra space — it is the **preferred sort for linked lists and for external sorting** (multi-way merge of data too big for memory).

### 4.3 Quick Sort

**Lomuto partition** (pivot = last element) — the standard teaching scheme:

```python
def partition(a, low, high):        # Lomuto; pivot = a[high]
    pivot = a[high]
    i = low - 1                     # boundary of the '<= pivot' region
    for j in range(low, high):      # n-1 comparisons for a block of size n
        if a[j] <= pivot:
            i += 1
            a[i], a[j] = a[j], a[i]
    a[i + 1], a[high] = a[high], a[i + 1]   # put pivot in its final slot
    return i + 1                    # pivot's final index

def quicksort(a, low, high):
    if low < high:
        p = partition(a, low, high)
        quicksort(a, low, p - 1)    # left side (< pivot)
        quicksort(a, p + 1, high)   # right side (> pivot)
```

*(Hoare partition uses two pointers moving inward; it does fewer swaps on average but does **not** place the pivot at its final index. GATE problems usually specify Lomuto.)*

> **Quick sort — recurrences & time**
> - **Best case (balanced split, pivot ≈ median):** `T(n) = 2T(n/2) + Θ(n)` ⟹ **Θ(n log n)**.
> - **Worst case (most unbalanced, pivot is smallest/largest every time):** `T(n) = T(n−1) + Θ(n)` ⟹ **Θ(n²)**. *Master Theorem does not apply here* — solve by summation: `Σ Θ(k) = Θ(n²)`.
> - **Average case (random permutation / random pivot):** **Θ(n log n)**, with expected comparisons **≈ 1.39 · n log₂ n ≈ 2n ln n**.

> **Quick sort — comparisons, space, stability**
> Worst-case comparisons `= n(n−1)/2` (blocks of size n, n−1, …, each doing size−1 comparisons).
> **In-place: Yes** — extra space is just the recursion stack: **Θ(log n) average**, **Θ(n) worst**. **Stable: No** (partitioning swaps far-apart equal keys).

**Beating the worst case (pivot strategies).**
- **Randomized quicksort:** choose the pivot uniformly at random ⟹ **expected Θ(n log n) on *every* input** (no single input is reliably bad).
- **Median-of-three:** pivot = median of first/middle/last ⟹ avoids the sorted-input trap cheaply (still Θ(n²) on rare adversarial inputs).
- **Median-of-medians (deterministic select):** guarantees a good pivot ⟹ **Θ(n log n) worst case**, but the constant is large, so it's rarely used in practice.
- **Stack safety:** recurse on the **smaller** side and loop on the larger (tail-call elimination) ⟹ stack depth **Θ(log n)** even in the worst case.

### 4.4 Merge Sort vs Quick Sort — the decision table

| Property | Merge Sort | Quick Sort |
|---|---|---|
| Best / Average time | Θ(n log n) / Θ(n log n) | Θ(n log n) / Θ(n log n) |
| **Worst-case time** | **Θ(n log n)** | **Θ(n²)** |
| Typical recurrence | `2T(n/2)+Θ(n)` | best `2T(n/2)+Θ(n)`; worst `T(n−1)+Θ(n)` |
| Auxiliary space | **Θ(n)** | **Θ(log n)** avg, Θ(n) worst (stack) |
| In-place? | No (array) | **Yes** |
| Stable? | **Yes** | No |
| Worst-case trigger | none (always n log n) | **sorted input + first/last pivot** |
| Linked lists / external sort | **preferred** | poor (needs random access) |
| Constants / cache locality | more data movement | **excellent → fastest in practice** |

*Reading it:* **need guaranteed Θ(n log n), stability, linked-list or external sorting → Merge Sort. Need raw in-memory speed and minimal extra space → Quick Sort (randomized).**

---

## 5. Worked problems (GATE-style)

### Problem 1 — Solving the two sorting recurrences (NAT / short answer)

Solve: (a) `T(n) = 2T(n/2) + cn` (merge sort), and (b) `T(n) = T(n−1) + cn` (quicksort worst case).

**Solution.**
(a) Master Theorem: `a=2, b=2 ⟹ n^{log_b a} = n^1 = n`. Here `f(n)=cn=Θ(n^1 log⁰ n)` ⟹ **Case 2** with `k=0` ⟹ `T(n) = Θ(n log n)`.

(b) **Not** of the form `aT(n/b)` ⟹ Master Theorem doesn't apply. Unroll:
`T(n) = cn + c(n−1) + c(n−2) + … + c·1 = c·n(n+1)/2 = Θ(n²)`.

**Answers: (a) Θ(n log n); (b) Θ(n²).** Lesson: balanced division gives `log n` levels (n log n); shaving off one element per call gives `n` levels of Θ(n) work (n²).

---

### Problem 2 — One Lomuto partition (NAT)

Run a **single Lomuto partition** (pivot = last element) on `A = [5, 3, 8, 4, 9, 1, 7]`. Give the array after partitioning and the **pivot's final index**.

**Solution.** Pivot `= 7`. Start `i = −1`, scan `j = 0…5`:

| j | a[j] | a[j] ≤ 7? | action | array |
|---|---|---|---|---|
| 0 | 5 | yes | i=0, swap(0,0) | [5,3,8,4,9,1,7] |
| 1 | 3 | yes | i=1, swap(1,1) | [5,3,8,4,9,1,7] |
| 2 | 8 | no | — | [5,3,8,4,9,1,7] |
| 3 | 4 | yes | i=2, swap(2,3) | [5,3,4,8,9,1,7] |
| 4 | 9 | no | — | [5,3,4,8,9,1,7] |
| 5 | 1 | yes | i=3, swap(3,5) | [5,3,4,1,9,8,7] |

Finally swap `a[i+1]=a[4]` with the pivot `a[6]`: → `[5, 3, 4, 1, 7, 8, 9]`.

**Answer: array `[5, 3, 4, 1, 7, 8, 9]`, pivot index = 4** (four elements ≤ 7 to its left, two > 7 to its right). ✓

---

### Problem 3 — Merge comparison counts (NAT)

Two **sorted** arrays have sizes 5 and 7. What are the **minimum** and **maximum** numbers of comparisons to merge them into one sorted array?

**Solution.**
- **Maximum = m + n − 1 = 5 + 7 − 1 = 11.** Happens when elements interleave so the merge runs until only one element of one array is left.
- **Minimum = min(m, n) = 5.** Happens when one array's values are all smaller (or all larger): once the smaller-sized run of 5 is exhausted after 5 comparisons, the remaining 7 are copied with no further comparisons.

**Answer: min = 5, max = 11.** Trap: it is **not** always `m + n`; the last block is copied without comparison.

---

### Problem 4 — Properties (MSQ)

Select all correct statements.

1. Merge sort's worst-case running time is Θ(n log n).
2. Quick sort with the last element as pivot runs in Θ(n²) on an already-sorted array.
3. Merge sort is stable; quick sort (standard) is not.
4. Randomized quick sort has Θ(n²) expected running time.

**Solution.**
1. **True** — merge sort is Θ(n log n) on every input.
2. **True** — sorted input + last-element pivot ⟹ maximally unbalanced partitions ⟹ `T(n)=T(n−1)+Θ(n)=Θ(n²)`.
3. **True** — merge keeps ties left (stable); partitioning swaps distant equals (unstable).
4. **False** — random pivots make the **expected** time **Θ(n log n)** regardless of input; Θ(n²) is only a low-probability worst case.

**Answer: 1, 2, and 3.**

---

## 6. Common GATE question types on this topic

- **Solve a recurrence** via Master Theorem; recognize when it **doesn't apply** (`T(n−1)+…`, gap cases) and switch to summation/recursion-tree.
- **Identify the Master-Theorem case** for a given `a, b, f(n)` and state the Θ-bound.
- **Trace a partition** (Lomuto/Hoare): array after partition, pivot's final index, number of comparisons/swaps.
- **Best/worst/average complexity** of merge and quick sort, and the **exact inputs** triggering quicksort's worst case.
- **Merge comparison counts** — min `min(m,n)`, max `m+n−1`; total merge-sort comparisons.
- **Space and stability** comparisons; recursion depth (`Θ(log n)` vs `Θ(n)` worst for quicksort).
- **Pivot strategies / randomization** and their effect on worst-case vs expected time.
- **"Which sort for this scenario"** — linked list, external/huge data, stability required, minimal extra space.
- **Counting inversions in Θ(n log n)** with modified merge sort (links to Topic 7).
- **Python output / fill-in-the-blank** for `merge`, `partition`, or the recursive driver.

---

## 7. Traps, pitfalls & common mistakes

- **Quicksort's worst case is Θ(n²) on SORTED input** (ascending or descending) when the pivot is the first or last element — the single most-tested trap. **Reverse-sorted is just as bad.**
- **Randomization / median-of-three fixes the *expected* time**, not the absolute worst; only **median-of-medians** guarantees Θ(n log n) worst case (with a big constant). Don't claim random pivots make the worst case impossible — they make it *improbable*.
- **Master Theorem misuse:** it requires the `aT(n/b)` form. `T(n)=T(n−1)+n` is **not** eligible (⟹ Θ(n²) by summation). Case 3 also needs the **regularity condition** — forgetting it is a common error. Cases 1/3 need a **polynomial gap** (the `ε`); "in-between" functions like `n/log n` are gap cases the theorem can't resolve.
- **Merge sort is never quadratic** — Best = Worst = Average = Θ(n log n). It is *not adaptive*, but there is **no bad input** for it.
- **Space:** merge sort needs **Θ(n)** auxiliary (not in-place); quicksort is **in-place** with only an **O(log n)** average stack (**O(n)** worst). Mixing these up is frequent.
- **Stability:** merge = stable, quicksort = **not** stable. Don't assume "faster sort ⟹ stable."
- **Merge comparison bounds:** minimum `min(m,n)`, maximum `m+n−1` — **not** `m+n`. The trailing block is copied free.
- **Recursion depth / stack overflow:** unbalanced quicksort recurses `Θ(n)` deep on sorted input. Recursing on the **smaller** side first caps depth at `Θ(log n)`.
- **Pivot logic belongs to quicksort only** — merge sort has no pivot. Don't transplant pivot reasoning into merge sort.
- **Base case in practice:** real implementations switch to **insertion sort** for tiny subarrays (constants beat Θ(n log n) there) — a direct callback to Topic 7.
- **Lomuto vs Hoare** give different swap counts and Hoare doesn't place the pivot at its final index — answer for the scheme the question names.
- **Off-by-one in partition/merge bounds** (`range(low, high)`, `low < high`, the final pivot swap) corrupts both the result and the index returned.

---

## 8. Connection forward

This topic completes the comparison-sort story: merge and quick sort hit the **Θ(n log n)** optimum (the comparison-sort lower bound is **Ω(n log n)**, provable via decision trees). Just as importantly, you now carry two reusable tools — the **divide-and-conquer template** (divide → recurse → combine) and **recurrence analysis** (Master Theorem + recursion trees) — which reappear across algorithms, including pieces of the graph material in the next chat. Merge sort's **merge** also powers **external sorting** and **counting inversions in Θ(n log n)** (closing the thread from Topic 7), and together with binary search it completes the classic **sort-then-search** pipeline that began this section. With searching and sorting in hand, the syllabus turns to a new structure entirely: **graphs**.

---

## 9. One-line cheat-sheet summary

**Divide & conquer:** split → recurse → combine; analyze with the Master Theorem (`T=aT(n/b)+f(n)`, compare `f(n)` vs `n^{log_b a}`). **Merge sort:** `2T(n/2)+Θ(n)=Θ(n log n)` **always**, **stable**, **Θ(n)** aux, best for linked lists/external. **Quick sort:** avg **Θ(n log n)**, worst **Θ(n²)** on sorted input with first/last pivot, **in-place**, **unstable**; randomize or use median-of-three, and recurse on the smaller side for `O(log n)` stack.

---

*Next topic (new chat): **Introduction to Graph Theory** — the start of the Graphs section.*

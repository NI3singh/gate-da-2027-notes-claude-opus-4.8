# Topic 6 — Linear and Binary Search

**Subject:** Programming, Data Structures and Algorithms (GATE DA)
**Section:** Searching & Sorting (this is the *first* topic of that section)

---

## 0. Topic header & syllabus map

- **Topic:** Linear Search and Binary Search.
- **Subject:** Programming, Data Structures and Algorithms.
- **Where it sits:** Comes right after the *data-structures* block (stacks, queues, linked lists, trees, hash tables). It opens the **Searching & Sorting** section by asking the most basic algorithmic question of all — *"is element x in this collection, and where?"* — directly on a plain array. It leads into **Basic Sorting** (selection, bubble, insertion), because binary search only works on *sorted* data, so sorting is the natural next step.

---

## 1. Why this topic exists (motivation + connection)

**The problem it solves.** Almost every program eventually asks: *does value `x` occur in this list, and at which index?* That single operation — **search** — underlies lookups, membership tests, joins, deduplication, and is a building block of countless larger algorithms.

**What the previous topics couldn't do (the connection backward).** The data-structure topics each gave you *one* way to search:
- A **hash table** gives O(1) *average* lookup, but it costs extra memory, must be *built* first, has **no ordering** (you cannot ask "what's the next-larger key?" or do a range query), and degrades to O(n) in the worst case.
- A **balanced BST** searches in O(log n), but you must *maintain a tree of pointers*.

Very often, though, your data is just a **flat array** with no auxiliary structure attached. So we now study search *as an algorithm on arrays directly.* **Linear search** works on *any* array (sorted or not). **Binary search** is the array twin of BST search: it exploits **sorted order** to reach O(log n) **without building any extra structure** — only the array, kept sorted, is needed.

**What the next topic builds (the connection forward).** Binary search's O(log n) is *conditional on the array being sorted.* That single precondition is exactly what motivates the next topic, **Basic Sorting** — and later, the "divide the problem in half" idea you meet here reappears, generalized, in **divide & conquer** (mergesort, quicksort).

---

## 2. Prerequisites

- **Arrays and 0-based indexing** — `a[0] … a[n-1]`, and the fact that array access `a[i]` is **O(1)** (random access).
- **Asymptotic notation** — Big-O, Θ, Ω; how to read "best / worst / average case".
- **`log₂` and the floor function `⌊·⌋`** — binary search complexity is stated in base-2 logs.
- **Basic loops and recursion** — binary search has both an iterative and a recursive form.
- **The idea of sorted vs unsorted data** — and why "sorted" is a precondition, not a detail.
- (Helpful) **BST search** from the Trees topic — binary search *is* the same idea on an array.

---

## 3. Core idea — intuition first

**Linear search** = looking for a friend's name by reading a guest list *top to bottom*, one name at a time, until you hit it or reach the end. No assumption about order — that's its strength (works on anything) and its weakness (you may read everything).

**Binary search** = looking up a word in a paper **dictionary**. You don't start at "aardvark". You flip to the *middle*, see whether your word comes *before* or *after* that page, and **throw away half the dictionary** in one step. Repeat on the surviving half. Each step halves the search space, so even a million-page dictionary needs only ~20 flips.

Two things make the dictionary trick work, and both are non-negotiable:
1. The dictionary is **sorted** (otherwise "before/after" is meaningless).
2. You can **open to any page instantly** (random access). A dictionary chained page-to-page like a **linked list** would force you to walk page by page to reach the middle — and the trick collapses (see the trap in §7).

Tiny trace — search `23` in `[2, 5, 8, 12, 16, 23, 38, 56, 72, 91]` (indices 0–9):
- Look at the middle, index 4 → value `16`. Since `23 > 16`, discard the left half; search indices 5–9.
- New middle, index 7 → value `56`. Since `23 < 56`, discard the right half; search indices 5–6.
- New middle, index 5 → value `23`. **Found.** Three probes instead of up to ten.

---

## 4. Formal treatment

### 4.1 Linear (sequential) search

**Definition.** Scan `a[0], a[1], …` left to right; return the first index `i` with `a[i] == key`, or `-1` if none.

```python
def linear_search(a, key):
    for i in range(len(a)):
        if a[i] == key:
            return i        # found at index i
    return -1               # not found
```

> **Linear search comparison counts** (on `n` elements)
> - Best case: **1** comparison (key at index 0).
> - Worst case: **n** comparisons (key at last index, or absent).
> - Average (successful): **(n + 1)/2** comparisons.
> - Average (unsuccessful): **n** comparisons.
> *Conditions:* the average-successful figure assumes the key **is present** and each of the `n` positions is **equally likely**. If the key is present with probability `p`, expected comparisons = `p·(n+1)/2 + (1−p)·n`.

> **Time complexity (linear search):** Best **Θ(1)**, Worst/Average **Θ(n)**. *Condition:* none — works on **sorted or unsorted** arrays, and even on linked lists (each `next` is one step).

### 4.2 Binary search

**Definition.** On a **sorted** array, repeatedly examine the middle element of the current `[low, high]` window and discard the half that cannot contain the key.

**Iterative form:**
```python
def binary_search(a, key):
    low, high = 0, len(a) - 1
    while low <= high:                  # window non-empty
        mid = low + (high - low) // 2   # see overflow note in §7
        if a[mid] == key:
            return mid                  # found
        elif a[mid] < key:
            low = mid + 1               # key is in right half
        else:
            high = mid - 1              # key is in left half
    return -1                           # not found
```

**Recursive form** (same logic; recurrence below):
```python
def binary_search_rec(a, low, high, key):
    if low > high:
        return -1
    mid = low + (high - low) // 2
    if a[mid] == key:
        return mid
    elif a[mid] < key:
        return binary_search_rec(a, mid + 1, high, key)
    else:
        return binary_search_rec(a, low, mid - 1, key)
```

**Loop invariant (why it's correct).** *If the key is anywhere in the array, it lies within `a[low..high]`.* Each iteration shrinks the window while preserving this, so when the loop ends either `a[mid] == key` was found, or `low > high` means the window is empty and the key was never there.

> **Binary search — worst-case probes / comparisons** (on `n` elements)
> **C_max(n) = ⌊log₂ n⌋ + 1**
> *Conditions:* array **sorted**, **random access** (true arrays, not linked lists), and one "comparison" = one examination of `a[mid]`. Holds whether the key is present or not, and with or without duplicates. Best case = **1** probe (key at the first `mid`).

> **Binary search recurrence** (worst case)
> **T(n) = T(⌊n/2⌋) + 1, T(1) = 1  ⟹  T(n) = ⌊log₂ n⌋ + 1**
> By the **Master Theorem** with `a = 1, b = 2, f(n) = Θ(1)`: since `n^{log_b a} = n^0 = 1` matches `f(n)`, this is **Case 2** ⟹ **T(n) = Θ(log n)**.

> **Time complexity (binary search):** Best **Θ(1)**, Worst/Average **Θ(log n)**. Average successful ≈ `log₂ n − 1` (only about one probe fewer than the worst case, because roughly half the elements sit at the deepest level). Unsuccessful search is also **O(log n)**.

**Decision-tree view (GATE favourite).** Binary search on `n` elements is a binary **decision tree**: internal (round) nodes are the elements you may land on (successful outcomes), and external (square) "null" nodes are the gaps between/around them (unsuccessful outcomes). The tree's **height is ⌊log₂ n⌋**, so the longest root-to-node path uses `⌊log₂ n⌋ + 1` comparisons — which is exactly `C_max`.

### 4.3 Useful variants (all O(log n), all assume a sorted array)

These appear constantly in GATE-style and coding questions:

- **First (leftmost) occurrence** with duplicates — on a match, *don't stop*; keep searching the **left** half.
- **Last (rightmost) occurrence** — on a match, keep searching the **right** half.
- **Count of a value** = `last − first + 1`.
- **`lower_bound`** = first index with `a[i] ≥ key`; **`upper_bound`** = first index with `a[i] > key`. (Python: `bisect.bisect_left` / `bisect.bisect_right`.)
- **Search in a rotated sorted array** — one half of every window is still sorted; decide which half contains the key. (Classic GATE CS variant.)
- **"Binary search on the answer"** — when the *answer* is monotonic (e.g. "smallest capacity that works"), binary-search the answer range, not an array. (Mention-level for DA.)

Leftmost-occurrence template:
```python
def first_occurrence(a, key):
    low, high, ans = 0, len(a) - 1, -1
    while low <= high:
        mid = low + (high - low) // 2
        if a[mid] == key:
            ans = mid          # record, then keep going left
            high = mid - 1
        elif a[mid] < key:
            low = mid + 1
        else:
            high = mid - 1
    return ans
```

Python's standard library does this for you:
```python
import bisect
i = bisect.bisect_left(a, key)        # leftmost insertion point
present = i < len(a) and a[i] == key  # True iff key is in the sorted list a
```

### 4.4 Linear vs Binary at a glance

| | Linear search | Binary search |
|---|---|---|
| Needs sorted data? | **No** | **Yes** |
| Needs random access? | No (works on linked lists) | **Yes** (useless on linked lists) |
| Worst-case comparisons | `n` | `⌊log₂ n⌋ + 1` |
| Best-case comparisons | `1` | `1` |
| Time complexity | Θ(n) | Θ(log n) |
| Extra space | O(1) | O(1) iterative; O(log n) stack if recursive |
| Good when | data unsorted, or `n` small, or one-off search | data already sorted, many searches |

---

## 5. Worked problems (GATE-style)

### Problem 1 — Maximum comparisons (NAT)

A sorted array holds **1000 distinct integers**. Using standard binary search, what is the **maximum number of key comparisons (probes)** needed to search for an element (present or absent)?

**Solution.**
`C_max(n) = ⌊log₂ n⌋ + 1`. For `n = 1000`, `log₂ 1000 ≈ 9.97`, so `⌊log₂ 1000⌋ = 9`.
`C_max = 9 + 1 = 10`.

**Answer: 10.** (For contrast, the *minimum* — best case — is **1**, and linear search would need up to **1000**.)

---

### Problem 2 — Which elements are compared? (MCQ — trap variety)

For the sorted array below (0-indexed), binary search is run for `key = 53` using `mid = low + (high − low)//2`. Which elements does it compare the key against, in order?

```
index:  0   1   2   3   4   5   6   7   8   9
value:  3   9  14  19  25  31  42  47  53  60
```

(A) 25, 47, 53  (B) 31, 47, 53  (C) 25, 53  (D) 25, 47, 53, 60

**Solution (full trace).**
- `low=0, high=9` → `mid = 0 + (9−0)//2 = 4` → `a[4] = 25`. `53 > 25` ⟹ `low = 5`.
- `low=5, high=9` → `mid = 5 + (9−5)//2 = 7` → `a[7] = 47`. `53 > 47` ⟹ `low = 8`.
- `low=8, high=9` → `mid = 8 + (9−8)//2 = 8` → `a[8] = 53`. **Match** → return 8.

Elements compared: **25, 47, 53** → **Answer (A).**

**Why the distractors are the trap.** (B) is what you get if you wrongly use the **ceiling** midpoint `(low+high+1)//2` at the first step (`(0+9+1)//2 = 5 → a[5]=31`). (D) confuses binary search with continuing toward the end. The exact comparison sequence depends on the **mid convention**, so always trace with the convention the question states — floor division here.

---

### Problem 3 — Binary search on a linked list (MCQ — trap variety)

What is the **worst-case time complexity** of performing binary search on a **sorted singly linked list** of `n` nodes?

(A) O(log n)  (B) O(n)  (C) O(n log n)  (D) O(1)

**Solution.** Binary search still makes only `O(log n)` *comparisons*, but a linked list has **no random access** — reaching the "middle" node means *walking* there from a known node. First mid ≈ `n/2` steps; then `n/4`; then `n/8`; … Total traversal `≈ n/2 + n/4 + n/8 + … = O(n)`. The pointer-walking **dominates**, so the whole thing is **Θ(n)**.

**Answer (B) O(n).** Lesson: binary search gives **no asymptotic advantage** over linear search on a linked list — it needs the O(1) indexing that only an **array** provides. (This is exactly why the variant templates in §4.3 all assume an array.)

---

## 6. Common GATE question types on this topic

- **Count comparisons** — maximum (`⌊log₂ n⌋ + 1`), minimum (`1`), or average; for successful *and* unsuccessful search.
- **Trace it** — given an array and a key, list the elements/indices compared (watch the mid convention).
- **Time complexity** — best/worst/average of linear and binary, and which is which.
- **Recurrence** — recognize `T(n) = T(n/2) + 1` ⟹ `Θ(log n)`; solve via Master Theorem or unrolling.
- **Preconditions / when it breaks** — sorted required, random access required (the **linked-list trap**).
- **Trade-off reasoning** — linear search vs *"sort once, then binary-search many times"* (see §7).
- **Variants** — first/last occurrence, count of a value, `lower_/upper_bound`, **rotated sorted array**.
- **Python output / fill-in-the-blank** — predict the return value, or complete a `while low <= high` loop / a `bisect` call.

---

## 7. Traps, pitfalls & common mistakes

- **Sorted is mandatory for binary search.** On an unsorted array the answer is garbage. If the array is sorted **descending**, the `<` / `>` branches must be **swapped**.
- **Random access is mandatory.** On a linked list, binary search degrades to **O(n)** (Problem 3). Don't claim O(log n) just because the list is sorted.
- **Loop condition must be `while low <= high`** (not `<`). With `<` you miss the case where the window narrows to a single element — a real off-by-one bug.
- **Update past the mid: `low = mid + 1`, `high = mid - 1`.** Writing `low = mid` or `high = mid` can leave the window unchanged ⟹ **infinite loop**.
- **Mid computation — overflow.** In C/C++/Java, `mid = (low + high) / 2` can **overflow** for large indices; the safe form is `mid = low + (high − low) / 2`. In **Python this overflow cannot happen** (ints are arbitrary-precision), but `low + (high - low)//2` is still the recommended habit — and GATE may test the C/Java version, so know *why* it overflows.
- **Floor vs ceil mid changes the trace.** `(low+high)//2` and `(low+high+1)//2` pick different middles when the window has even length, altering the comparison sequence — a favourite MCQ trap.
- **Comparison-count off-by-one.** Linear search worst case is exactly **n** (not `n−1`, not `n+1`). Binary search worst case is **`⌊log₂ n⌋ + 1`** — don't forget the `+1`.
- **Duplicates:** plain binary search returns **some** matching index, **not necessarily the first or last**. Use the leftmost/rightmost variant when the question wants a specific one.
- **Average cases get mixed up.** Linear: successful `(n+1)/2`, unsuccessful `n`. Binary: ≈ `log₂ n − 1` (close to the worst case, **not** half of it).
- **Single-search trade-off (worked illustration).** Suppose the array is **unsorted** and you must do **one** search. Sorting first costs `O(n log n)` and then binary search adds `O(log n)` — total `O(n log n)`, which is **worse** than a single linear search's `O(n)`. But for **k** searches: `sort once + k·binary = O(n log n + k log n)` beats `k · O(n) = O(kn)` once `k` is large. So *"sort then binary-search"* pays off **only when you search many times**. Picking the wrong side of this is a classic exam mistake.

---

## 8. Connection forward

Binary search just proved that **sorted order buys you O(log n) lookups** — but it *assumes* the data is already sorted. That assumption is the entire reason the **next topic, Basic Sorting** (selection, bubble, insertion), exists: it's how you *create* the precondition binary search depends on. Moreover, the halve-the-problem trick you used here is the seed of **divide & conquer** (mergesort, quicksort), where the same "split in two" idea organizes the *sorting* itself.

---

## 9. One-line cheat-sheet summary

**Linear search:** any array, `Θ(n)`, worst = `n` comparisons. **Binary search:** needs a **sorted, random-access array**, `Θ(log n)` with `⌊log₂ n⌋ + 1` worst-case probes; use `mid = low + (high−low)//2` and `while low <= high`; on a linked list it collapses to `O(n)`.

---

*Next topic (logically): **Basic Sorting** — selection sort, bubble sort, and insertion sort.*

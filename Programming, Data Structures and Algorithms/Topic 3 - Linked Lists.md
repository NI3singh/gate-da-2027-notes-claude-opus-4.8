# Topic 3 — Linked Lists
### Subject: Programming, Data Structures and Algorithms (GATE DA)

---

## 0. Topic header & syllabus map

- **Topic:** Linked Lists
- **Subject:** Programming, Data Structures and Algorithms
- **Where it sits:** The **third** topic and the **last** of the "Python & linear data structures" group. It **comes after Stacks and Queues** (array-backed stacks/queues had fixed capacity and circular-queue index math; linked lists remove both) and **leads into Trees** — the next chat — where a node simply gains *several* outgoing references instead of one. Linked lists are the 1‑dimensional ancestor of every pointer-connected structure that follows (trees, graph adjacency lists, hash-table chains).

> GATE DA tests this topic mainly through **(1)** precise **operation complexities** (which operation is O(1) vs O(n), SLL vs DLL, with/without a tail pointer), **(2)** **array vs linked-list** trade-offs, and **(3)** **pointer-manipulation reasoning** — reversing, finding the middle, the *n*-th from the end, and **Floyd's cycle detection**. Everything below targets those.

---

## 1. Why this topic exists (motivation + connection)

A Python `list` (a dynamic array) stores elements **contiguously**, which buys instant random access — `a[i]` is O(1) — but punishes structural edits: inserting or deleting anywhere except the end forces every later element to **shift**, which is O(n). Array-backed structures also have a notion of fixed capacity.

A **linked list** makes the opposite trade. Elements live in separate **nodes** scattered in memory, each holding its data plus a **reference** to the next node. You **give up random access** (to reach element *k* you must walk *k* links, O(n)), but in return:

- inserting or deleting a node, *once you're positioned there*, is just **re-wiring a couple of references → O(1)**, with **no shifting**, and
- the structure **grows and shrinks naturally**, with no capacity ceiling.

- **What the previous topic couldn't do:** array-backed stacks/queues had a **fixed capacity** (hence overflow and the circular-queue `% S` juggling). A linked list gives the *same* O(1) end operations with **unbounded** size and **no** wrap-around arithmetic. A stack is an SLL with push/pop at the **head**; a queue is an SLL with **head = front** and **tail = rear**.
- **What the next topic builds:** a **tree** node is just a node with *multiple* next-pointers (e.g. `left`, `right`). All the reference-rewiring skill here transfers directly; trees generalise the chain into branching.

---

## 2. Prerequisites

- **Python's reference / object model (Topic 1):** variables are *name tags on objects*; `node.next = other` is a **reference assignment**, and `None` is the "no node" terminator. This is the heart of linked lists.
- Basic classes/objects in Python.
- Stack/queue ADT idea (Topic 2) — linked lists give an alternative backing store.
- Time-complexity vocabulary (O(1) vs O(n)).

---

## 3. Core idea — intuition first

Think of a **treasure hunt of clues**. Each clue card (a **node**) has two things: a piece of treasure (the **data**) and directions to the **next** card (a **reference**). The very last card says "no more" (**`None`**). The only way to reach the 5th card is to start at the first and follow the directions four times — there is **no shortcut** to jump straight there (that's why indexing is O(n)). But slipping a brand-new card into the chain is trivial: you just change the "next" directions on the card before it and point the new card onward (that's why insertion, *once positioned*, is O(1)).

Contrast the **array**: numbered lockers in a row. You can open **any** locker instantly (random access, O(1)), but to insert a new locker in the middle you must shove everyone down one place (O(n)).

So the mental model is: **arrays = fast lookup, slow edits; linked lists = slow lookup, fast edits** — and the "fast edit" only applies *once you already hold a reference to the spot.*

---

## 4. Formal treatment

There are no theorems to derive; GATE tests the **exact behaviour and cost** of each operation plus a few **standard pointer algorithms**. Conditions are stated alongside.

### 4.1 Node structure and the reference model (Python)

```python
class Node:
    def __init__(self, data):
        self.data = data      # the payload
        self.next = None      # reference to the next Node, or None at the end

# A "linked list" is simply a reference to its first node (the head):
head = Node(10)
head.next = Node(20)
head.next.next = Node(30)     # 10 -> 20 -> 30 -> None
```

There are **no raw pointers** in Python — `self.next` is an ordinary object reference (Topic 1). `None` plays the role of the null terminator. The list is identified entirely by its **`head`**; lose the head reference and the whole list is unreachable (garbage-collected).

### 4.2 Types of linked lists

- **Singly linked list (SLL):** each node has **one** link (`next`). Traversal is forward-only.
- **Doubly linked list (DLL):** each node has **two** links (`prev` and `next`). Allows backward traversal and O(1) deletion of a known node.
- **Circular linked list:** the last node links **back to the head** (instead of `None`). Singly- or doubly-circular. Used for round-robin scheduling.
- **Header / sentinel list:** a dummy node before the real first node, present so that insertion/deletion at the head and the empty-list case need **no special-casing**.

### 4.3 Core operations on a singly linked list

**Traverse / search (O(n)):**
```python
cur = head
while cur is not None:
    # visit cur.data
    cur = cur.next
```

**Insert at head (O(1)):**
```python
def insert_head(head, data):
    node = Node(data)
    node.next = head
    return node              # the new head
```

**Insert at tail:** O(n) without a tail pointer (walk to the last node); **O(1)** if a `tail` reference is maintained.

**Insert *after* a node you already hold (O(1)):**
```python
node = Node(data)
node.next = p.next
p.next = node                # splice in after p — no shifting
```
> **Order matters:** set `node.next` **before** overwriting `p.next`, or you lose the rest of the list.

**Insert *before* a given node:** O(n) in an SLL (you must find the **predecessor**), O(1) in a DLL.

**Delete at head (O(1)):** `head = head.next`.

**Delete at tail:** **O(n)** in an SLL **even with a tail pointer** — you must update the *second-to-last* node's `next` to `None`, and finding it requires a full walk (an SLL has no back-link). **O(1)** in a DLL.

**Delete a node given only a reference to it:** O(n) in an SLL (need the predecessor) **unless** the copy trick (see Problem 6) is used — that is O(1) but **cannot delete the last node**. **O(1)** in a DLL.

### 4.4 Complexity — Singly Linked List

| Operation | No tail pointer | With tail pointer |
|---|---|---|
| Access *i*-th element | O(n) | O(n) |
| Search (unsorted) | O(n) | O(n) |
| Insert at head | **O(1)** | **O(1)** |
| Insert at tail | O(n) | **O(1)** |
| Delete at head | **O(1)** | **O(1)** |
| Delete at tail | O(n) | **O(n)** ← still! |
| Insert after a held node | **O(1)** | **O(1)** |
| Insert before a held node | O(n) | O(n) |
| Delete a held node | O(n) (or O(1)\*) | O(n) (or O(1)\*) |

\* via the data-copy trick, which fails for the last node.

### 4.5 Doubly linked list — what it fixes

Node: `data`, `prev`, `next`. Each link costs an **extra reference per node** (more memory), but the back-link removes the SLL's worst cases:

```python
class DNode:
    def __init__(self, data):
        self.data = data
        self.prev = None
        self.next = None
```

**Delete a known node `x` in O(1):**
```python
if x.prev: x.prev.next = x.next
if x.next: x.next.prev = x.prev
```

| Operation (with head & tail refs) | DLL |
|---|---|
| Insert at head / tail | **O(1) / O(1)** |
| Delete at head / **tail** | **O(1) / O(1)** ← DLL fixes the SLL tail case |
| Insert before / after a held node | **O(1)** |
| **Delete a held node** | **O(1)** ← DLL fixes this too |
| Backward traversal | **possible**, O(n) |
| Search | O(n) |

> The whole reason DLLs exist: **O(1) deletion of the tail and of any node you already hold, plus backward traversal** — at the price of one extra pointer per node.

### 4.6 Circular linked list

The last node's `next` points to the head, so there is **no `None`**; traversal must stop when it returns to the starting node:
```python
cur = head
while True:
    # visit cur.data
    cur = cur.next
    if cur is head:
        break
```
**Useful property — a circular SLL kept by its *tail* pointer alone** gives O(1) access to **both** ends: the tail is `tail`, and the head is `tail.next`. Hence **insert at head and insert at tail are both O(1)** with a single reference. Round-robin schedulers exploit this.

### 4.7 Array vs Linked List (the comparison GATE loves)

| Aspect | Array / Python `list` (dynamic array) | Linked list |
|---|---|---|
| Memory layout | Contiguous | Scattered nodes |
| Access *i*-th element | **O(1)** random access | **O(n)** sequential |
| Search (unsorted) | O(n) | O(n) |
| Insert/delete at **end** | O(1) amortised (`append`/`pop`) | O(1) with tail ref (SLL insert; DLL also delete) |
| Insert/delete at **beginning** | **O(n)** (shift all) | **O(1)** |
| Insert/delete in **middle** | O(n) (shift) | **O(1) to splice**, but O(n) to *find* the spot |
| Resize / capacity | Amortised re-allocation | Grows naturally, no ceiling |
| Memory overhead per element | Low (data only) | High (1 pointer SLL, 2 pointers DLL) |
| Cache locality | Good | Poor |

> **Critical nuance:** "linked-list insert/delete is O(1)" is true **only if you already hold a reference** to the node (or its predecessor). If you must **search by value or index first**, total cost is O(n) to find + O(1) to splice = **O(n)**. This find-vs-splice split is a favourite trap.
>
> (Python tie-in to Topic 2: a `list` has O(1) `append`/`pop()` at the end but **O(n)** `insert(0,x)`/`pop(0)` at the front — exactly why a queue uses `deque`, not a list.)

### 4.8 Classic pointer algorithms

**(a) Reverse a singly linked list — iterative, 3 pointers, O(n) time, O(1) space:**
```python
def reverse(head):
    prev = None
    cur = head
    while cur is not None:
        nxt = cur.next      # save next
        cur.next = prev     # flip the link
        prev = cur          # advance prev
        cur = nxt           # advance cur
    return prev             # new head
```
Recursive version (O(n) time, **O(n) stack** space):
```python
def reverse_rec(node):
    if node is None or node.next is None:
        return node
    new_head = reverse_rec(node.next)
    node.next.next = node
    node.next = None
    return new_head
```

**(b) Find the middle — slow/fast (tortoise–hare), O(n) time, O(1) space:**
```python
slow = fast = head
while fast and fast.next:
    slow = slow.next          # 1 step
    fast = fast.next.next     # 2 steps
# slow is the middle
```
- **Odd length:** `slow` lands on the exact middle.
- **Even length:** with this loop, `slow` lands on the **second** of the two middle nodes (convention-dependent — use `while fast.next and fast.next.next` to stop at the first middle).

**(c) *n*-th node from the end — two pointers with a gap of `n`, one pass:**
```python
fast = head
for _ in range(n):
    fast = fast.next          # move fast n ahead first
slow = head
while fast:
    slow = slow.next
    fast = fast.next
# slow is the n-th node from the end
```

**(d) Cycle detection — Floyd's algorithm (tortoise–hare), O(n) time, O(1) space:**
```python
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
    if slow is fast:          # references coincide -> cycle
        break
```
- If a cycle exists, the two pointers **must meet inside it** (the fast pointer gains 1 node per step on the slow one, so the gap closes). If `fast` reaches `None`, there is **no** cycle.
- **Start of the cycle:** after they meet, reset one pointer to the **head** and advance **both one step at a time**; they meet again exactly at the **cycle's entry node**.
  - *Why:* let μ = nodes before the cycle, λ = cycle length, and let them meet `k` nodes into the cycle. Slow travelled `μ+k`, fast travelled `2(μ+k)`, and fast did some whole loops: `μ+k+mλ = 2(μ+k) ⇒ μ = mλ − k`. Walking μ steps from the head reaches the entry; walking the same μ steps from the meeting point lands at `k + (mλ − k) = mλ ≡ 0 (mod λ)` — the entry. (Full proof is light; GATE wants the **result**.)
- **Cycle length λ:** keep one pointer at the meeting node and walk another around until it returns; count the steps.
- A list **with** a cycle has **no well-defined length** — naive traversal loops forever, which is the whole reason Floyd's algorithm is needed.

**(e) Merge two sorted linked lists — O(m + n):** repeatedly splice the smaller current node from the two lists into the result.

### 4.9 Variants / awareness (one line each)

- **Sentinel/header node:** removes head/empty-list special cases — common in clean implementations.
- **XOR linked list:** a DLL storing `prev XOR next` in one field to save a pointer per node — a GATE-CS curiosity, rarely tested in DA; just recognise the name.

---

## 5. Worked problems (GATE-style)

### Problem 1 (MCQ) — head-insertion order
Starting from an empty SLL, `insert_head` is called with `10, 20, 30, 40` in that order (each new node becomes the head). Traversing from the head prints:
(A) `10 20 30 40`  (B) `40 30 20 10`  (C) `40 10 20 30`  (D) `10 40 30 20`

**Solution.** Inserting at the head reverses arrival order: after 10 → `10`; after 20 → `20→10`; after 30 → `30→20→10`; after 40 → `40→30→20→10`. **Answer: (B).** (Head-insert builds a stack/LIFO — Topic 2 link.)

---

### Problem 2 (descriptive) — reverse trace
Run the iterative `reverse` on `1 → 2 → 3 → None`. Give the list after each iteration and the returned head.

**Solution.** (`prev`, `cur`) evolve as:
- start: prev=None, cur=1
- iter 1: save nxt=2; `1.next=None`; prev=1, cur=2 → `1→None`
- iter 2: save nxt=3; `2.next=1`; prev=2, cur=3 → `2→1→None`
- iter 3: save nxt=None; `3.next=2`; prev=3, cur=None → `3→2→1→None`
- loop ends; **return prev = node 3**. Final list: **`3 → 2 → 1 → None`**. (The trick is saving `nxt` *before* flipping `cur.next`.)

---

### Problem 3 (NAT/short) — Floyd's cycle detection
An SLL has 7 nodes `n1…n7` and `n7.next = n3` (so `n3,n4,n5,n6,n7` form a cycle). Using Floyd's algorithm: **(a)** is a cycle detected, **(b)** at which node do slow and fast first meet, **(c)** what is the cycle's start, **(d)** what is the cycle length?

**Solution.** The cyclic order after `n7` is `…n7 → n3 → n4 → …`.
Trace (both start at `n1`):

| step | slow | fast |
|---|---|---|
| 0 | n1 | n1 |
| 1 | n2 | n3 |
| 2 | n3 | n5 |
| 3 | n4 | n7 |
| 4 | n5 | n4 (n7→n3→n4) |
| 5 | **n6** | **n6** (n4→n5→n6) |

- **(a)** Yes — they coincide ⇒ cycle exists.
- **(b)** First meeting at **n6**.
- **(c)** Reset one pointer to `n1`, advance both by 1: `n1→n2→n3` and `n6→n7→n3` — meet at **n3** = cycle start (μ = 2 prefix nodes, consistent).
- **(d)** Walk from `n3` around: `n3,n4,n5,n6,n7` back to `n3` ⇒ **length 5**.

---

### Problem 4 (MSQ) — complexity (select all true)
(I) Deleting the **tail** of an SLL is O(1) if a tail pointer is kept.
(II) Deleting the **tail** of a DLL with head & tail pointers is O(1).
(III) Accessing the *i*-th element of a linked list is O(1).
(IV) Inserting a node *immediately after* a node you already hold is O(1) in an SLL.

**Solution.**
- (I) **False** — you still need the *second-to-last* node's reference, which costs a full walk in an SLL.
- (II) **True** — `tail.prev` gives the new tail in O(1).
- (III) **False** — no random access; it's O(n).
- (IV) **True** — pure pointer splice, no shifting.

**Correct: (II) and (IV).**

---

### Problem 5 (MCQ) — slow/fast on an even-length list
A list has 6 nodes `a→b→c→d→e→f`. With `while fast and fast.next: slow=slow.next; fast=fast.next.next`, which node is `slow` at the end?
(A) `c`  (B) `d`  (C) `e`  (D) `b`

**Solution.** Trace: (slow,fast) = (a,a)→(b,c)→(c,e)→(d, f.next.next). After third iteration `fast` becomes `None` (from `e`, two steps overshoot past `f`); `slow = d`. For even length this standard loop returns the **second middle**. **Answer: (B).**

---

### Problem 6 (descriptive — the classic "delete with only a node reference") 
In an SLL you are given **only** a reference to a node `p` that is **not** the last node. Show how to delete `p` in **O(1)**, and state why it fails for the last node.

**Solution.** You can't reach `p`'s predecessor, but you *can* overwrite `p` so its *value* disappears and the chain stays intact:
```python
p.data = p.next.data        # copy successor's data into p
p.next = p.next.next        # bypass (and drop) the successor node
```
This removes the node *logically* in **O(1)** — the value originally at `p` is gone and the list length drops by one. **It fails if `p` is the last node**, because there is no `p.next` to copy from or to bypass. (In a **DLL** this restriction vanishes — true O(1) deletion of any held node via `prev`/`next`.)

---

### Problem 7 (MCQ) — circular SLL with only a tail pointer
A **circular** singly linked list is maintained using **only a `tail` reference**. Which pair of operations are **both** O(1)?
(A) insert-at-head and delete-at-tail
(B) insert-at-head and insert-at-tail
(C) search and insert-at-tail
(D) delete-at-tail and access-*i*-th

**Solution.** With circularity, `tail.next` **is** the head, so the head is reachable in O(1). Inserting a new node after `tail` (and pointing it to `tail.next`) handles **both** "at head" (point tail to it appropriately) and "at tail" in O(1). Delete-at-tail still needs the predecessor (O(n)), and access-*i*-th/search are O(n). **Answer: (B).**

---

### Problem 8 (MCQ) — Python reference semantics (Topic 1 tie-in)
```python
class N:
    def __init__(self, d):
        self.d = d
        self.next = None
a = N(1); b = N(2)
a.next = b
c = a
c.next.d = 99
print(a.next.d)
```
(A) `2`  (B) `99`  (C) `None`  (D) error

**Solution.** `c = a` makes `c` and `a` reference the **same** node (no copy). `c.next` **is** `b`, so `c.next.d = 99` mutates `b`. Then `a.next.d` reads `b.d` = `99`. **Answer: (B).** (Linked lists are pure reference manipulation — aliasing rules from Topic 1 apply verbatim.)

---

## 6. Common GATE question types on this topic

- **Operation complexity:** classify operations as O(1)/O(n) — SLL vs DLL, with/without a tail pointer; the "insert/delete tail" gotchas.
- **Array vs linked list trade-offs** (random access, shifting, memory overhead, cache).
- **Pointer-manipulation trace:** what is the final list after a given sequence of splices/deletes; "what does this code do?"
- **Reverse / middle / *n*-th-from-end** via the standard pointer idioms.
- **Floyd's cycle detection:** detect a cycle, find its **start** and **length**, identify the meeting node.
- **The "delete given only a node reference" trick** and its last-node limitation.
- **Circular-list properties** (tail-only access to both ends; traversal termination).
- **Reference-semantics output** (Python aliasing inside nodes).
- **Merging / counting** in O(m+n) / O(n).

---

## 7. Traps, pitfalls & common mistakes

1. **Save `next` before re-wiring.** In reversal/splicing, overwriting `cur.next` first **loses the rest of the list**. Always stash `nxt = cur.next` first.
2. **Delete-tail is O(n) in an SLL even with a tail pointer** — you need the *predecessor*. Only a **DLL** makes it O(1).
3. **"O(1) insert/delete" assumes you already hold the node** (or predecessor). With a prior **search by value/index**, the total is **O(n)**.
4. **No random access:** indexing a linked list is **O(n)**, never O(1). (Reverse confusion: arrays index in O(1) but shift in O(n).)
5. **Insert *before* a node is O(n) in an SLL** (predecessor needed), O(1) only in a DLL.
6. **Even-length "find middle"** returns the **second** middle with the common `while fast and fast.next` loop — know your convention.
7. **Cycle ⇒ no length / infinite traversal.** A naive node count loops forever; you *must* use Floyd's (or a visited-set, which is O(n) extra space).
8. **Floyd's "find start" requires resetting to the head** and stepping **both by one** — not by two. Re-using two-step movement after the meeting is wrong.
9. **The data-copy delete trick fails on the last node** (no successor to copy) — state this caveat whenever you use it.
10. **Circular list traversal needs a return-to-start check**, not a `None` check — an SLL-style `while cur is not None` loops forever on a circular list.
11. **Losing the head reference loses the whole list** (it becomes unreachable / garbage-collected). Always return/update the head after head insert/delete.
12. **DLL deletion must update *both* neighbours' links** (`prev.next` *and* `next.prev`); updating one corrupts the list.
13. **Memory overhead:** SLL = 1 extra pointer/node, DLL = 2 — relevant when a question weighs space against the O(1) DLL operations.
14. **Aliasing inside nodes:** assigning one node reference to another (or to a list variable) does **not** copy the chain — Topic 1's reference rules apply.

---

## 8. Connection forward

You now manipulate **reference-connected nodes** fluently — splicing, reversing, two-pointer scanning, and cycle detection — and you understand the **array-vs-pointer** trade-off precisely. The next chapter, **Trees**, keeps the identical node-and-reference machinery but lets each node carry **more than one** outgoing link (`left`, `right`, …), turning the single chain into a **branching** structure. Linked lists are exactly the degenerate, one-child-per-node case of a tree; the pointer reasoning you built here is what makes BST insertion, traversal, and rotation feel routine. (And recall: a **stack** = SLL with head push/pop, a **queue** = SLL with head-front/tail-rear — and graph **adjacency lists** and **hash-table chains** are linked lists too.)

---

## 9. One-line cheat-sheet summary

**Nodes connected by references (`next`, plus `prev` in a DLL), terminated by `None`: O(n) access/search but O(1) insert/delete *once positioned* (no shifting) — DLL adds O(1) tail-delete, O(1) delete-of-any-held-node, and backward walks; master reverse (3 pointers), middle/*n*-th-from-end (slow–fast), and Floyd's cycle detection (meet → reset to head → meet at cycle start).**

---

### ➡️ Next topic (in order): **Topic 4 — Trees** (begins the new chat: *Trees & hash tables*).

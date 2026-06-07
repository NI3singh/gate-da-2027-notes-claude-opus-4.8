# Topic 4 — Trees (Binary Trees, BST and Traversals)

**Subject:** Programming, Data Structures and Algorithms (GATE DA)
*Section: Trees & Hash Tables — Topic 1 of 2*

---

## 0. Topic header & syllabus map

- **Topic:** Trees — Binary Trees, Binary Search Trees (BST), and Traversals.
- **Subject:** Programming, Data Structures and Algorithms.
- **Where it sits:** This is the **first hierarchical (non-linear) data structure** in the syllabus. It comes **after** linear structures — Python, stacks & queues, linked lists — and **leads into** Hash Tables. The traversal machinery here reuses **stacks** (for DFS) and **queues** (for BFS) from Topic 2, and it sets up the **search/sort** section (Topic 6 onwards): a BST is essentially "binary search made into a data structure."

---

## 1. Why this topic exists (motivation + connection)

The real problem: **we want a collection that supports fast search, insertion, and deletion simultaneously.**

- An **array** gives O(1) random access but O(n) insertion/deletion (you must shift elements).
- A **sorted array** gives O(log n) search (binary search) but still O(n) insertion (shifting).
- A **linked list** (Topic 3) gives O(1) insertion *if you already hold the node*, but **O(n) search** because you can only walk sequentially — there is no way to "skip half the list."

**What the previous topic couldn't do:** Linked lists are *linear* — every element has at most one successor, so to find something you may have to scan all n nodes. There is no branching, hence no way to discard half the remaining data in one comparison.

**What trees fix:** A **binary search tree** branches at every node into a "smaller" side and a "larger" side. One comparison eliminates an entire subtree, giving **O(height)** search — and if the tree stays balanced, height is O(log n). So a BST combines the *ordered, halving* search of a sorted array with the *cheap pointer-based insert/delete* of a linked list. Trees also model naturally hierarchical data (file systems, parse trees, decision trees in ML).

**What the next topic builds on this:** Hash tables push the goal further — **O(1) average** search by giving up order. The contrast is exactly the exam-relevant trade-off: **BST = ordered, O(log n), supports range/successor queries; Hash table = unordered, O(1) average, no ordering queries.**

---

## 2. Prerequisites

- **Linked lists** (Topic 3): a tree node is a struct with a key plus *references* to child nodes — same pointer mechanics, just two links instead of one.
- **Recursion**: traversals, height, insert/delete are most naturally defined recursively.
- **Stacks and Queues** (Topic 2): DFS traversals use a stack (often the call stack); level-order uses a queue.
- **Big-O notation** and basic **logarithms** (log₂): height-vs-nodes reasoning.
- **Counting / factorials / binomial coefficients**: for "number of trees" (Catalan) questions.
- **Binary search idea** (helpful, formalized later): a BST is binary search "frozen" into structure.

---

## 3. Core idea — intuition first

Think of an **org chart** or a **family tree**. One person at the top (the **root**). Each person points to a few people directly below them (**children**). Follow the arrows down and you never come back up or form a loop — that "no cycles, one way down" shape is a **tree**.

A **binary tree** just limits each person to **at most two** direct reports — a *left* one and a *right* one (and left vs right is meaningful; they are not interchangeable).

A **binary search tree** adds one rule that makes searching fast: **everything in your left subtree is smaller than you, everything in your right subtree is larger.** So searching for a number is like the "guess the number" game — compare with the current node, and based on smaller/larger, jump entirely into the left or right subtree, throwing away the other half. If the tree is bushy and balanced, you halve the search space each step → log₂ n steps.

**One picture to remember the three depth-first traversals** — at each node you do three things: *visit yourself (N)*, *go Left (L)*, *go Right (R)*. The only question is **when you visit yourself**:
- **Pre**order = visit **before** children → **N** L R
- **In**order = visit **between** children → L **N** R
- **Post**order = visit **after** children → L R **N**

("Pre/In/Post" tells you where the **N** sits.) For a BST, **inorder gives the keys in sorted order** — that single fact answers a large fraction of BST exam questions.

---

## 4. Formal treatment

### 4.1 Tree terminology (state these precisely — GATE tests definitions)

- **Tree:** a connected, acyclic structure of nodes. A **rooted tree** designates one **root**; every other node has exactly **one parent**. A tree with **n nodes has exactly n − 1 edges**.
- **Edge:** a parent→child link.
- **Parent / child / sibling:** direct relations; siblings share a parent.
- **Leaf (external node):** a node with **no children** (degree 0).
- **Internal node:** a node with **at least one** child.
- **Degree of a node:** number of **children** it has (in DS context the parent link is *not* counted). **Degree of the tree:** maximum node degree.
- **Path & path length:** sequence of nodes joined by edges; length = number of **edges**.
- **Ancestor / descendant:** nodes on the path up to the root / down into a subtree.
- **Subtree:** a node together with all its descendants.

**Depth / Level / Height — and the conventions GATE flips on you:**

| Term | Definition (default convention used here) | Single-node tree | Empty tree |
|---|---|---|---|
| **Depth of a node** | # edges from **root** to the node | root depth = 0 | — |
| **Level** | usually = depth, so **root is at level 0** | root level = 0 | — |
| **Height of a node** | # edges on the **longest** path down to a leaf | leaf height = 0 | — |
| **Height of tree** | height of the root = max depth in the tree | **0** | **−1** |

> ⚠️ **Convention trap.** Some books put the **root at level 1** (then "max nodes at level i" becomes 2^(i−1)), and some define height by **counting nodes** (then a single node has height 1, and an n-node skewed tree has height n). GATE questions are usually self-consistent, but **read the question's convention first.** All formulas below use **root at level 0** and **edge-counted height** (single node → height 0, empty → −1).

### 4.2 Binary tree

A **binary tree** is a finite set of nodes that is either **empty**, or consists of a root plus two **disjoint, ordered** binary trees called the **left subtree** and **right subtree**. "Ordered" ⇒ a left-only child differs from a right-only child.

**Types (and the differences that get confused):**

| Type | Defining rule |
|---|---|
| **Full / Strict / Proper** | every node has **0 or 2** children (never exactly 1) |
| **Complete** | all levels full **except possibly the last**, which is filled **left→right** |
| **Perfect** | all internal nodes have 2 children **and** all leaves are at the **same level** |
| **Balanced** | height is **O(log n)** (e.g., AVL, Red-Black) |
| **Degenerate / Skewed / Pathological** | every node has **one** child ⇒ behaves like a linked list |

Relationships: **Perfect ⟹ Complete** and **Perfect ⟹ Full**, but not conversely. A complete tree need not be full (its last internal node may have a single left child), and a full tree need not be complete.

### 4.3 Core properties (memorize — each with its condition)

> **Convention:** root at level 0; height *h* counted in **edges**.

```
P1.  Max nodes at level i           = 2^i                         (i ≥ 0)
P2.  Max nodes in a tree of height h = 2^(h+1) − 1                 (perfect tree)
P3.  Min nodes in a tree of height h = h + 1                       (skewed: 1 node/level)
P4.  Min height for n nodes          = ⌈log₂(n+1)⌉ − 1  = ⌊log₂ n⌋ (as bushy as possible)
P5.  Max height for n nodes          = n − 1                       (skewed tree)
P6.  Leaf vs degree-2 count:   n₀ = n₂ + 1                         (ANY binary tree)
P7.  #NULL (empty) child links       = n + 1                       (n-node binary tree)
P8.  Perfect tree of height h:  leaves = 2^h,  internal = 2^h − 1,  total = 2^(h+1) − 1
P9.  Complete tree with n nodes:  height = ⌊log₂ n⌋,
                                  leaves = ⌈n/2⌉,  internal = ⌊n/2⌋
P10. #structurally-distinct binary trees with n nodes = Catalan Cₙ
        Cₙ = (1/(n+1))·C(2n, n) = (2n)! / ((n+1)! · n!)
        Recurrence: Cₙ = Σ_{i=0}^{n−1} Cᵢ · C_{n−1−i}
```

**Proof of P6 (n₀ = n₂ + 1)** — appears repeatedly, so know the argument:
Let n₀, n₁, n₂ = number of nodes with 0, 1, 2 children. Total nodes n = n₀ + n₁ + n₂.
Count **edges** two ways. Every node except the root has exactly one parent edge ⇒ edges = n − 1.
Every edge is "a child of someone" ⇒ edges = 0·n₀ + 1·n₁ + 2·n₂.
Equate: n − 1 = n₁ + 2n₂ ⇒ (n₀ + n₁ + n₂) − 1 = n₁ + 2n₂ ⇒ **n₀ = n₂ + 1.** ∎

**Proof of P7 (#NULL links = n + 1):** each node has 2 child slots ⇒ 2n slots total. The filled slots are exactly the edges = n − 1. So NULL slots = 2n − (n − 1) = **n + 1.** (Equivalently, an extended binary tree has n + 1 external nodes.) ∎

**Catalan table** (keep handy):

| n | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|---|
| Cₙ | 1 | 1 | 2 | 5 | 14 | 42 | 132 |

The recurrence reasoning (worth understanding, not just memorizing): fix the root; if the **left subtree** has *i* nodes, the **right subtree** has the remaining *n − 1 − i* nodes; sum over all splits i = 0…n−1. The closed-form derivation (generating functions) is **beyond GATE — skipped**; you only need the recurrence and formula to *compute* values.

**Complete-binary-tree array indexing** (1-indexed array `A[1…n]`):
```
parent(i) = ⌊i / 2⌋ ,   left(i) = 2i ,   right(i) = 2i + 1
```
(For 0-indexed: left = 2i+1, right = 2i+2, parent = ⌊(i−1)/2⌋.) This compact storage is exactly how **heaps** represent a complete binary tree without pointers.

### 4.4 Traversals

A **traversal** visits every node exactly once ⇒ **O(n) time** for all of them. They split into depth-first (use a stack / recursion) and breadth-first (use a queue).

**Depth-first (DFS):** difference is *when the node is visited* (see §3).
```
preorder(x):  if x: visit(x); preorder(x.left); preorder(x.right)   # N L R
inorder(x):   if x: inorder(x.left); visit(x); inorder(x.right)     # L N R
postorder(x): if x: postorder(x.left); postorder(x.right); visit(x) # L R N
```
- **Space:** O(h) recursion-stack (h = height) — O(log n) if balanced, O(n) if skewed.

**Breadth-first (BFS) = level-order:** visit level 0, then level 1, … each left→right, using a **queue**.
```
levelorder(root):
    if root is None: return
    Q ← queue with [root]
    while Q not empty:
        x ← Q.dequeue();  visit(x)
        if x.left:  Q.enqueue(x.left)
        if x.right: Q.enqueue(x.right)
```
- **Space:** O(w) where w = maximum width (≈ n/2 for a near-perfect last level).

**Key traversal facts:**
1. **Inorder of a BST = keys in ascending sorted order.** (Reverse-inorder = descending.)
2. **Expression-tree bridge** (connects to stacks, Topic 2): for an arithmetic expression tree, **preorder = prefix (Polish)**, **inorder = infix**, **postorder = postfix (Reverse Polish)**. Evaluating postfix with a stack is the postorder use-case.

**Reconstruction — which traversal pairs determine a unique tree?**

| Given | Unique tree? |
|---|---|
| Inorder + Preorder | **Yes** |
| Inorder + Postorder | **Yes** |
| Inorder + Level-order | **Yes** |
| Preorder + Postorder | **No** in general (ambiguous when a node has 1 child); **unique only if the tree is a full binary tree** |
| Preorder + Level-order (no inorder) | **No** in general |

**Why inorder is the magic ingredient:** preorder/postorder/level-order each tell you the **root** of a (sub)tree, but only **inorder splits the remaining keys into "left subtree" vs "right subtree"** (everything before the root vs after it). Without inorder you cannot tell where one subtree ends and the next begins.

**Reconstruction recipe (Inorder + Preorder):**
1. First element of preorder = **root**.
2. Locate root in **inorder**: keys to its **left** form the left subtree's inorder; keys to its **right** form the right subtree's inorder.
3. The **left subtree size k** = (# keys left of root in inorder). Take the next **k** preorder elements as the left subtree's preorder; the rest as the right subtree's preorder.
4. **Recurse** on each side.

> **Special case for BSTs:** since inorder is just the sorted keys, **a single traversal of a BST (preorder OR postorder OR level-order) is enough** to rebuild it — pair it with "sorted = inorder." This is why BST questions often give *only* a preorder.

### 4.5 Binary Search Tree (BST)

**Definition (BST property):** a binary tree in which, for **every** node `x`,
```
all keys in left-subtree(x)  <  key(x)  <  all keys in right-subtree(x)
```
(assuming **distinct** keys; duplicate-handling is by convention and rarely tested).
Equivalently: **inorder traversal is strictly increasing.** Note the condition is on **entire subtrees**, not merely the immediate children — a common mistake.

**Operations (all are O(h)):**

*Search* — compare and descend:
```python
def search(root, key):
    while root and root.key != key:
        root = root.left if key < root.key else root.right
    return root            # node or None
```

*Insert* — search to a NULL slot, attach as a new **leaf** (structure depends on insertion order):
```python
def insert(root, key):
    if root is None:
        return Node(key)
    if   key < root.key: root.left  = insert(root.left,  key)
    elif key > root.key: root.right = insert(root.right, key)
    return root            # key == root.key → ignore (no duplicates)
```

*Find-min / Find-max:* leftmost / rightmost node.

*Inorder successor* of x (next-larger key): if x has a right subtree → **min of the right subtree**; else → the lowest ancestor of which x lies in the **left** subtree. *Predecessor* is symmetric (max of left subtree, etc.).

*Delete* — **three cases**:
1. **Leaf (0 children):** just remove it.
2. **One child:** splice the node out — replace it by its single child.
3. **Two children:** replace the node's key by its **inorder successor** (min of right subtree) *or* **inorder predecessor** (max of left subtree); then delete that successor/predecessor node — which has **at most one child**, so it reduces to case 1 or 2.

```python
def delete(root, key):
    if root is None: return None
    if   key < root.key: root.left  = delete(root.left,  key)
    elif key > root.key: root.right = delete(root.right, key)
    else:                                   # found node to delete
        if root.left  is None: return root.right   # cases 1 & 2
        if root.right is None: return root.left     # cases 1 & 2
        succ = root.right                           # case 3: inorder successor
        while succ.left: succ = succ.left
        root.key   = succ.key
        root.right = delete(root.right, succ.key)
    return root
```

**Complexities:**

| Operation | Balanced BST | Skewed BST (worst) | Average (random keys) |
|---|---|---|---|
| Search / Insert / Delete | O(log n) | O(n) | O(log n) |
| Find min/max, succ/pred | O(log n) | O(n) | O(log n) |
| Inorder (sorted output) | O(n) | O(n) | O(n) |

**Counting BSTs:** the number of **distinct BSTs on n distinct keys = Cₙ** (Catalan). Reason: choose any of the Cₙ tree **shapes**; the BST property then forces a **unique** key-to-node labeling. Contrast: the number of **labeled binary trees** on n distinct labels = **Cₙ · n!** (shape × arbitrary labeling), because there ordering is unconstrained.

> **Insertion-order vs shape:** there are **n!** possible insertion orders but only **Cₙ** distinct resulting BSTs, so many different insertion sequences produce the *same* tree. Separately, the number of BSTs of **maximum height (n − 1)** is **2^(n−1)** (the tree is a path; at each step the next key extends left or right).

### 4.6 Self-balancing trees & heaps — scope note (read once)

To guarantee O(log n) even in the worst case, **self-balancing BSTs** (AVL, Red-Black) perform **rotations** after insert/delete to keep height ≈ log n. A **heap** is a *complete* binary tree with the heap-order property, stored in an array via P-indexing, supporting O(log n) insert/extract-min. **Rotation mechanics, AVL balance factors, and heapsort are at the GATE-CS depth and are not part of the core GATE-DA "trees" syllabus** (DA sorting is limited to selection/bubble/insertion/merge/quick). Know the *purpose* ("balancing keeps height O(log n)"); you are unlikely to be asked to perform AVL rotations in DA. They are flagged here only for completeness.

---

## 5. Worked problems (GATE-style)

### Q1 (NAT) — Leaf count via n₀ = n₂ + 1
*A binary tree has 25 nodes in total. Exactly 6 nodes have one child. How many leaf nodes does it have?*

**Solution.** n = n₀ + n₁ + n₂ = 25, with n₁ = 6, so n₀ + n₂ = 19. By **P6**, n₀ = n₂ + 1. Substitute: (n₂ + 1) + n₂ = 19 ⇒ 2n₂ = 18 ⇒ n₂ = 9 ⇒ **n₀ = 10 leaves.**
*(Trap: you cannot use n₀ = n₂ + 1 alone — you needed n₁ to pin down the totals.)*

### Q2 (NAT) — Max nodes / height conventions
*(a) Maximum number of nodes in a binary tree of height 4? (b) A binary tree has 100 nodes; what is its minimum possible height?* (Root at level 0, height in edges.)

**Solution.** (a) **P2**: 2^(4+1) − 1 = **31**.
(b) **P4**: ⌊log₂ 100⌋ = ⌊6.64⌋ = **6** (check: a perfect tree of height 6 holds up to 2⁷−1 = 127 ≥ 100, height 5 holds only 63 < 100). 
*(Trap: if the problem said "5 levels" with root at level 1, levels are 1…5 → height 4 in edges → 31 nodes. Always translate the convention first.)*

### Q3 (NAT) — Counting trees (Catalan) vs labeled trees
*(a) How many structurally distinct binary trees have 4 nodes? (b) How many distinct BSTs can be built from the keys {10, 20, 30, 40}? (c) How many distinct binary trees have 4 labeled (distinct) nodes?*

**Solution.** (a) C₄ = **14**. (b) BSTs on n distinct keys = Cₙ = C₄ = **14** (shape determines the labeling). (c) shapes × labelings = C₄ · 4! = 14 · 24 = **336**.
*(Trap: do **not** multiply (b) by 4!. The BST property removes the labeling freedom.)*

### Q4 (MCQ) — Traversal outputs
Tree:
```
            A
          /   \
         B     C
        / \      \
       D   E      F
```
*Give preorder, inorder, postorder, and level-order.*

**Solution.**
- **Preorder (N L R):** A, B, D, E, C, F → **A B D E C F**
- **Inorder (L N R):** D, B, E, A, C, F → **D B E A C F**
- **Postorder (L R N):** D, E, B, F, C, A → **D E B F C A**
- **Level-order:** A, B, C, D, E, F → **A B C D E F**

### Q5 (NAT/MCQ) — Reconstruct from Preorder + Inorder, then output Postorder
*Preorder = A B D E C F ; Inorder = D B E A C F. Find the postorder.*

**Solution.**
1. Preorder[0] = **A** = root. In inorder, split at A: left = `D B E`, right = `C F`.
2. Left subtree (3 nodes): its preorder = next 3 of preorder = `B D E`, inorder = `D B E`. Root **B**; inorder split at B → left `D`, right `E`. So B has children D (left), E (right).
3. Right subtree (2 nodes): preorder = `C F`, inorder = `C F`. Root **C**; C is first in its inorder → no left child, right child **F**.
4. Reassemble → exactly the tree in Q4. **Postorder = D E B F C A.**
*(This is the Q4 tree run backwards — note the same answer.)*

### Q6 (MSQ) — Why Preorder + Postorder is not enough
*Preorder = `x y`, Postorder = `y x`. How many distinct binary trees fit? Which traversal pair would have made it unique?*

**Solution.** Root must be `x` (first in preorder, last in postorder), and `y` is its only other node — but `y` could be the **left** child **or** the **right** child; both give preorder `x y` and postorder `y x`. So **2 trees** fit. Adding **inorder** disambiguates: inorder `y x` ⇒ y is left child; inorder `x y` ⇒ y is right child.
*(General rule: pre+post is ambiguous exactly when some node has only one child; it is unique only for **full** binary trees.)*

### Q7 (NAT) — BST build + height + inorder
*Insert into an empty BST in this order: 50, 30, 70, 20, 40, 60, 80. (a) Draw it. (b) Its height? (c) Its inorder?*

**Solution.**
```
            50
          /    \
        30      70
       /  \    /  \
      20  40  60   80
```
(a) As above. (b) **Height = 2** (perfect tree of 7 nodes; P2: 2³−1 = 7). (c) Inorder of a BST is sorted: **20 30 40 50 60 70 80.**
*Contrast — insert 20, 30, 40, 50, 60, 70, 80 (already sorted) → fully right-**skewed**, **height = 6**, search becomes O(n). Same key-set, different order, very different tree.*

### Q8 (MCQ) — BST deletion (two-children case)
*From the BST of Q7, delete the root 50. Show the resulting tree using the inorder-successor rule.*

**Solution.** Node 50 has two children → replace by its **inorder successor** = min of right subtree = leftmost of {70,60,80} = **60**. Put 60 at the root, delete original 60 (a leaf):
```
            60
          /    \
        30      70
       /  \       \
      20  40       80
```
Inorder check: 20 30 40 60 70 80 ✓ (still sorted, BST property preserved). *(Using the inorder **predecessor** = 40 instead would be equally valid and gives a different but correct tree.)*

### Q9 (MSQ) — Valid BST search path
*In a BST containing the keys 1…10, while searching for key 6 the sequence of nodes examined was claimed. Which sequences are **impossible**?*
(i) 9, 2, 8, 5, 6  (ii) 10, 1, 9, 3, 7, 6  (iii) 2, 7, 3, 8, 6

**Solution.** A search path must be **monotone toward the target**: once you turn left at a node larger than 6, every later node must stay < that node, and vice-versa. Track the live interval for 6.
- (i) 9(go left, <9) → 2(go right, in (2,9)) → 8(go left, in (2,8)) → 5(go right, in (5,8)) → 6 ✓ — each bound consistent. **Possible.**
- (ii) 10→1: but 6 > 1 means after visiting 1 we go right, all later keys must be **> 1** *and* still **< 10**; 9 ok, then 9→left so < 9, 3 ok, 3→right so in (3,9), 7 ok, 7→left so in (3,7), 6 ✓. **Possible.**
- (iii) 2→7: from 2 we go right (>2), reach 7; 6<7 so go left, all later keys must be **< 7**; next is 3 (<7 ok, and >2), from 3 go right (>3) so later keys in (3,7); next is **8** — but 8 ∉ (3,7). **Impossible.** ✗

So **only (iii) is impossible.**
*(Technique: maintain a shrinking (low, high) interval; each visited key must lie inside the current interval, then it tightens one bound.)*

### Q10 (NAT) — Complete-tree indexing & NULL links
*(a) A complete binary tree is stored 1-indexed. What are the children of node at index 6, and the parent of node 11? (b) A binary tree has 15 nodes; how many NULL child pointers does it have?*

**Solution.** (a) children of 6 = 2·6, 2·6+1 = **12 and 13**; parent of 11 = ⌊11/2⌋ = **5**. (b) **P7**: NULL links = n + 1 = **16**.

---

## 6. Common GATE question types on this topic

- **Counting via P6:** given some of {leaves, one-child nodes, two-child nodes, total}, find another (n₀ = n₂ + 1).
- **Max/min nodes ↔ height** (P1–P5); **#NULL links = n+1** (P7).
- **Number of distinct binary trees / BSTs** = Catalan Cₙ; *labeled* trees = Cₙ·n! (distinguish the two).
- **Traversal output** for a drawn tree (pre/in/post/level).
- **Reconstruction** from two traversals; or *deciding* whether a given pair is sufficient (the pre+post trap).
- **BST operations:** resulting tree / height after a sequence of inserts; deletion (esp. two-children case); inorder = sorted.
- **Valid vs invalid BST search path** (shrinking-interval check).
- **Complete-tree array indexing** (parent/child formulas), leaf/internal counts (P9).
- **Identify the tree type** (full vs complete vs perfect) from a figure.
- **Complexity** of operations under balanced vs skewed assumptions.

---

## 7. Traps, pitfalls & common mistakes

1. **Convention mismatches** — root at level 0 vs 1; height counted in **edges** vs **nodes**; empty tree height = −1. A single misread shifts every formula by one. **Translate the question's convention before plugging in.**
2. **n₀ = n₂ + 1 is about *children*, not depth**, and it **ignores n₁**. You usually need the count of one-child nodes (or total) to finish.
3. **Full ≠ Complete ≠ Perfect.** Full = "0 or 2 children"; Complete = "filled left-to-right, last level may be partial"; Perfect = "all leaves same level." Don't swap definitions.
4. **Preorder + Postorder do NOT determine a unique tree** (ambiguous for any one-child node); unique only for **full** binary trees. To reconstruct in general you **need inorder**.
5. **But a BST needs only one traversal** — students wrongly insist on "two traversals" for BSTs. Since inorder = sorted keys, a single preorder/postorder/level-order suffices.
6. **BST property is over whole subtrees, not just immediate children.** A node can satisfy its direct children yet violate a distant ancestor's bound.
7. **Catalan counting:** #BSTs on n keys = Cₙ (do **not** multiply by n!); #labeled binary trees = Cₙ·n! (do multiply). Mixing these is the most common counting error.
8. **BST deletion:** replacement for a two-children node must be the **inorder successor (min of right) or predecessor (max of left)** — not an arbitrary descendant — to keep the order intact.
9. **Skewed trees:** worst-case BST height = **n − 1**, so search/insert/delete degrade to **O(n)**. Inserting **sorted** input is the usual cause.
10. **Edges vs nodes:** n nodes ⇒ **n − 1 edges** and **n + 1 NULL links**; max height n − 1 (edges). Off-by-one here is frequent.
11. **Level-order needs a queue, not a stack** (a stack would give a depth-first-ish order). Don't conflate the BFS/DFS auxiliary structures.
12. **Same key-set, different insertion order ⇒ different tree.** The resulting BST is *not* determined by the set alone.

---

## 8. Connection forward

Trees set up two threads. **(a) Toward Hash Tables (next topic):** the BST gives **ordered O(log n)** search and supports range/successor queries; hash tables will trade ordering away for **O(1) average** lookup — the exam-favorite comparison. **(b) Toward Searching & Sorting (Topic 6–8):** a BST is "binary search turned into a structure," its inorder yields a **sorted sequence** (a BST-based sort), and balanced/complete-tree ideas reappear in heaps and in the divide-and-conquer recursion trees of mergesort/quicksort. The **graph** section (Topic 9–10) then generalizes trees by allowing cycles and multiple parents, and BFS/DFS reappear there as graph traversals.

---

## 9. One-line cheat-sheet summary

> **Binary tree:** n nodes ⇒ n−1 edges, n+1 NULL links, **n₀ = n₂+1**, max nodes at level i = 2ⁱ, max nodes for height h = 2^(h+1)−1, #shapes = Catalan Cₙ. **Traversals:** pre=NLR, in=LNR (=**sorted for a BST**), post=LRN, level=BFS(queue); reconstruct uniquely **only with inorder** (pre+post needs a *full* tree; a BST needs just one traversal). **BST:** left<node<right, search/insert/delete O(h) (O(log n) balanced, O(n) skewed), #BSTs on n keys = Cₙ (labeled trees = Cₙ·n!), delete-2-children → replace by inorder successor/predecessor.

---

*Next topic (logically): **Hash Tables** — hashing, hash functions, load factor, and collision resolution (chaining, open addressing), i.e., how to get O(1)-average lookup once we drop the ordering that a BST gave us. Ask for it when ready.*

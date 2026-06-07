# Topic 2 — Stacks and Queues
### Subject: Programming, Data Structures and Algorithms (GATE DA)

---

## 0. Topic header & syllabus map

- **Topic:** Stacks and Queues
- **Subject:** Programming, Data Structures and Algorithms
- **Where it sits:** This is the **second** topic. It **comes after Programming in Python** (you now have lists, `append`/`pop`, indexing, modulo, and the call-stack idea from recursion) and **leads into Linked Lists** (which give an alternative, pointer-based way to build these same structures without a fixed-size array). Stacks and queues are the **first Abstract Data Types (ADTs)** in the course: you specify *what* the operations do, then choose *how* to store them.

> GATE DA tests this topic in three repeating ways: **(1)** trace/operation problems (what is popped? final state? valid pop sequence?), **(2)** **expression conversion & evaluation** (infix↔postfix↔prefix, the single most common stack application), and **(3)** **circular-queue index arithmetic** (front/rear/count, full vs empty). Everything below is aimed at those.

---

## 1. Why this topic exists (motivation + connection)

A plain list lets you touch *any* element at *any* time. But a huge number of real problems need **restricted, policy-driven access**, and enforcing that policy is exactly what prevents bugs:

- **"Most recent first" (LIFO):** function calls (the call stack), undo, backtracking, matching brackets, and parsing/evaluating expressions all need the *last* thing handled *first* → a **stack**.
- **"First come first served" (FIFO):** CPU/print scheduling, buffering, and breadth-first search all need the *earliest* item handled *first* → a **queue**.

- **What the previous topic couldn't do:** raw Python lists give *unrestricted* access; nothing stops you from violating the order discipline these problems require. Stacks and queues *restrict* the list interface (to `push`/`pop` at one end, or `enqueue` at one end and `dequeue` at the other) to **guarantee** the policy and keep every operation O(1).
- **What the next topic builds:** array-backed stacks/queues have a **fixed capacity** (hence overflow and the circular-queue index gymnastics). **Linked Lists** (next) let these structures **grow dynamically** with O(1) end operations and no wrap-around arithmetic — the same ADTs, a cleaner backing store.

---

## 2. Prerequisites

- Python lists: `append`, `pop`, `pop(0)`, indexing, `[-1]` (from Topic 1).
- Modulo arithmetic `%` (for circular-queue wrap-around).
- Operator precedence & associativity (for expression conversion).
- Recursion / the call-stack idea (a stack you already used implicitly).
- Basic time-complexity vocabulary (O(1) vs O(n)).

---

## 3. Core idea — intuition first

- **Stack = a pile of plates.** You add and remove only from the **top**. The last plate placed is the first removed — **LIFO** (Last-In, First-Out).
- **Queue = a line at a ticket counter.** People join at the **rear** and leave from the **front**. The first to arrive is the first served — **FIFO** (First-In, First-Out).

**ADT mindset.** A stack is *defined* by its operations (`push`, `pop`, `peek`, `isEmpty`), not by whether it's built from an array, a Python list, or a linked list. Two implementations that obey the same operation contract are the "same" stack. This separation of *interface* from *implementation* is the conceptual leap of this topic.

---

## 4. Formal treatment

There are no theorems to derive here; what GATE tests is the **precise operational behaviour** of each operation and the **standard algorithms** built on top. Conditions are stated alongside each.

### 4.1 Stack ADT — operations and complexity

| Operation | Effect | Error condition | Time |
|---|---|---|---|
| `push(x)` | put `x` on top | overflow (array full) | O(1) |
| `pop()` | remove & return top | **underflow** (empty) | O(1) |
| `peek()` / `top()` | read top without removing | empty | O(1) |
| `isEmpty()` | is the stack empty? | — | O(1) |
| `size()` | number of elements | — | O(1) |

- **LIFO invariant:** the element returned by `pop` is always the one most recently `push`ed and not yet popped.
- Space: **O(n)**.

### 4.2 Stack implementation

**Python (idiomatic):** a `list`, pushing/popping at the **end**.

```python
s = []
s.append(10)     # push
s.append(20)
top = s[-1]      # peek  -> 20
x = s.pop()      # pop   -> 20  (LIFO)
empty = (len(s) == 0)
```

All these are **O(1)** (amortised for `append`). *Never* push/pop at index 0 for a stack — `s.pop(0)`/`s.insert(0,x)` are O(n).

**Array implementation (fixed capacity `S`):** keep a `top` index initialised to `-1`.
- `push`: if `top == S-1` → **overflow**; else `top += 1; A[top] = x`.
- `pop`: if `top == -1` → **underflow**; else `x = A[top]; top -= 1`.

**Linked implementation:** push/pop at the **head** of a singly linked list (detailed in Topic 3); O(1), no capacity limit.

### 4.3 Stack applications (with the algorithms GATE asks)

**(a) Function call stack / recursion.** Each call pushes an activation record (parameters, locals, return address); returning pops it. Recursion *is* an implicit stack — this is why deep recursion causes a stack overflow (Topic 1's `RecursionError`).

**(b) Balanced parentheses / valid bracket sequence.**
```
scan left→right:
  if opening bracket  -> push it
  if closing bracket  -> if stack empty OR top isn't the matching opener -> NOT balanced
                          else pop
at end: balanced iff stack is empty
```

**(c) Expression notations.** For an operation with operator `op` and operands `a`, `b`:
- **Infix:** `a op b` (operator *between*) — what humans write; needs precedence rules and parentheses.
- **Postfix (Reverse Polish):** `a b op` (operator *after*) — no parentheses needed, evaluated left→right with a stack.
- **Prefix (Polish):** `op a b` (operator *before*) — no parentheses needed.

> Postfix/prefix exist because they are **unambiguous without parentheses or precedence rules**, so a machine can evaluate them in one linear pass with a single stack.

**Operator precedence & associativity (used for all conversions):**

```
^  (exponentiation)   highest    RIGHT associative
*  /                  middle     LEFT  associative
+  -                  lowest     LEFT  associative
(  )                  parentheses override everything
```

**(d) Infix → Postfix (shunting-yard):**
```
scan left→right:
  operand          -> append to output
  '('              -> push
  ')'              -> pop to output until '(' ; discard the '('
  operator o1      -> while stack not empty and top is an operator o2 such that
                         prec(o2) > prec(o1)  OR  (prec(o2)==prec(o1) and o1 is LEFT-assoc):
                            pop o2 to output
                      then push o1
end                -> pop all remaining operators to output
```
Right-associative `^` is the exception: with equal precedence you **do not pop** (so `A^B^C → A B C ^ ^`, i.e. `A^(B^C)`). Left-associative `-` does pop on equal precedence (so `A-B-C → A B - C -`, i.e. `(A-B)-C`).

**(e) Postfix evaluation (one stack, left→right):**
```
scan left→right:
  operand   -> push
  operator  -> pop RIGHT operand, then pop LEFT operand, compute LEFT op RIGHT, push result
end         -> the single remaining value is the answer
```
> **Operand-order trap:** in postfix, the **first** value popped is the **right** operand. For `a b -` you pop `b` then `a` and compute `a − b`.

**(f) Prefix evaluation (one stack, right→left):**
```
scan right→left:
  operand   -> push
  operator  -> pop LEFT operand, then pop RIGHT operand, compute LEFT op RIGHT, push
```
> Mirror of postfix: here the **first** popped is the **left** operand.

**(g) Infix → Prefix (foolproof method).** Fully parenthesise by precedence, then write each operator *before* its two operands. Example: `A + B * C` → `(A + (B*C))` → **`+ A * B C`**. (A common alternative recipe: reverse the infix, swap `(`↔`)`, convert to postfix treating `^` associativity carefully, then reverse the result.)

**(h) DFS uses a stack; backtracking uses a stack.** (Connects forward to Graphs — Topic 10.)

### 4.4 Stack permutations (a GATE favourite)

Push `1, 2, …, n` in that fixed order, interleaving `pop`s freely. **Which output (pop) sequences are achievable?**

- **Count:** exactly the **n-th Catalan number**
  ```
  C_n = (1 / (n+1)) * C(2n, n)
  C_1=1, C_2=2, C_3=5, C_4=14, C_5=42
  ```
  (Derivation via the reflection principle is **beyond GATE scope** — memorise the small values.)

- **Characterisation (shortcut):** a permutation is achievable **iff it avoids the pattern `3‑1‑2`** — i.e. there are **no** output positions `i < j < k` with `out[i] > out[k] > out[j]` (a large value, then a small, then a medium). The single forbidden order for `n=3` is `3,1,2`.

- **Reliable method (always works): simulate.** To output a value `v`, every smaller value still inside the stack is trapped *below* later, larger values; you cannot reach `v` until those on top are popped. Just replay push/pop and see if the target order is producible.

### 4.5 Queue ADT — operations and complexity

| Operation | Effect | Error condition | Time |
|---|---|---|---|
| `enqueue(x)` | add `x` at **rear** | overflow (full) | O(1) |
| `dequeue()` | remove & return **front** | **underflow** (empty) | O(1) |
| `front()` | read front | empty | O(1) |
| `rear()` | read rear | empty | O(1) |
| `isEmpty()` | empty? | — | O(1) |

- **FIFO invariant:** `dequeue` returns the element that has waited longest.
- Space: **O(n)**.

### 4.6 Linear (simple) queue and its flaw → "false overflow"

In a fixed array with `front` and `rear` indices that only ever **increase**, after several enqueues/dequeues `rear` reaches the last cell while early cells (before `front`) sit **empty but unusable**. The queue reports "full" though space exists — **false overflow**. Fixing this is the entire reason the **circular queue** exists.

### 4.7 Circular queue (wrap-around)

Indices wrap using modulo: after the last index you go back to `0`. Two standard conventions appear in exams — **state which one a problem uses.**

**Convention A — "sacrifice one cell" (most common):** `front` = index of the first element; `rear` = index where the **next** element will be inserted.
```
array size = S
empty   :  front == rear
full    :  (rear + 1) % S == front
count   :  (rear - front + S) % S
capacity:  S - 1     (one cell is always kept empty to distinguish full from empty)
enqueue :  A[rear] = x ;  rear = (rear + 1) % S
dequeue :  x = A[front] ; front = (front + 1) % S
```

**Convention B — `rear` = index of the *last* element (needs a separate count or full-flag):**
```
count (when non-empty) = (rear - front + S) % S + 1
capacity = S   (all cells usable, but you must track count to tell full from empty)
```

> The `+ S` before `% S` is essential — it keeps the result non-negative when `rear < front`.

**Worked index trace (Convention A, `S = 5`):** start `front=rear=0` (empty).
Enqueue a,b,c,d → `front=0, rear=4`; `(rear+1)%5 = 0 = front` ⇒ **full** (holds `S−1 = 4`). Dequeue → `front=1`; count `=(4−1+5)%5 = 3`. Enqueue e → placed at index 4, `rear=(4+1)%5=0`; now `front=1, rear=0`, count `=(0−1+5)%5 = 4`, and `(rear+1)%5 = 1 = front` ⇒ full again.

### 4.8 Queue implementation in Python — use `deque`

```python
from collections import deque
q = deque()
q.append(10)      # enqueue (rear)
q.append(20)
f = q[0]          # front peek -> 10
x = q.popleft()   # dequeue (front) -> 10   (FIFO)   O(1)
```

> **Major trap:** implementing a queue with a plain `list` and `list.pop(0)` for dequeue is **O(n)** per dequeue (every later element shifts left). `collections.deque` gives **O(1)** at both ends and is the correct choice.

### 4.9 Deque and other variants

- **Deque (double-ended queue):** insert/remove at **both** ends — `append`, `appendleft`, `pop`, `popleft`, all O(1). A deque can act as *either* a stack or a queue.
- **Priority queue:** items dequeued by **priority**, not arrival order (typically backed by a heap). *Heaps are not in the DA syllabus*, so this is a one-line awareness note only.

### 4.10 Building one from the other (conceptual, occasionally asked)

- **Queue using two stacks:** `enqueue` → push to `inStack`. `dequeue` → if `outStack` empty, pop everything from `inStack` into `outStack` (reversing order), then pop `outStack`. **Amortised O(1)** per operation.
- **Stack using two queues:** `push` → enqueue into `q1`. `pop` → move all but the last element from `q1` to `q2`, return/remove the last, then swap names. **O(n)** per pop.

### 4.11 Linked-list backing (preview)

A stack pushes/pops at the **head** of a singly linked list; a queue keeps **head = front** (dequeue) and **tail = rear** (enqueue). Both stay O(1) with no fixed capacity and no wrap-around arithmetic — the payoff detailed in Topic 3.

---

## 5. Worked problems (GATE-style)

### Problem 1 (MCQ) — valid stack output sequence
Elements `1,2,3,4` are pushed onto a stack **in that order**, with pops allowed in between. Which sequence can **NOT** be a sequence of popped values?
(A) `1,2,3,4`  (B) `2,1,4,3`  (C) `4,3,2,1`  (D) `3,4,1,2`

**Solution.** Use the 3‑1‑2 test (and confirm by simulation).
- (D) `3,4,1,2`: positions (0,2,3) give values `3,1,2` → `out[0]=3 > out[3]=2 > out[2]=1` is a 3‑1‑2 pattern ⇒ **not achievable**. Simulate: output 3 ⇒ push1,2,3, pop3, stack `[1,2]`; output 4 ⇒ push4,pop4, stack `[1,2]`; now the only reachable next value is `2` (top), never `1`. So `3,4,1,…` cannot continue to `1`. Confirmed impossible.
- (A),(B),(C) all simulate cleanly. **Answer: (D).**

---

### Problem 2 (NAT) — number of distinct stack outputs
How many distinct pop sequences are possible when `1,2,3,4` are pushed in order?

**Solution.** This count is the Catalan number `C_4 = C(8,4)/5 = 70/5 = 14`. **Answer: 14.**

---

### Problem 3 (descriptive→NAT) — Infix → Postfix, then evaluate
Convert `(A + B) * C - D / E` to postfix, then evaluate it for `A=1, B=2, C=3, D=8, E=4`.

**Solution — conversion (shunting-yard):**

| Token | Action | Output | Stack |
|---|---|---|---|
| `(` | push | | `(` |
| `A` | output | `A` | `(` |
| `+` | top is `(` → push | `A` | `( +` |
| `B` | output | `A B` | `( +` |
| `)` | pop to `(` | `A B +` | (empty) |
| `*` | push | `A B +` | `*` |
| `C` | output | `A B + C` | `*` |
| `-` | `*` higher → pop `*`; push `-` | `A B + C *` | `-` |
| `D` | output | `A B + C * D` | `-` |
| `/` | `-` lower → push `/` | `A B + C * D` | `- /` |
| `E` | output | `A B + C * D E` | `- /` |
| end | pop all | `A B + C * D E / -` | (empty) |

Postfix = **`A B + C * D E / -`**.

**Evaluation (push operands; on operator pop right then left):**
`1 2 + → 3`; `3 3 * → 9`; `8 4 / → 2`; `9 2 - → 7`. **Answer: 7.** (Cross-check infix: `(1+2)*3 − 8/4 = 9 − 2 = 7`.) ✓

---

### Problem 4 (NAT) — evaluate a postfix expression
Evaluate the postfix expression `5 6 2 + * 12 4 / -`.

**Solution.** Push operands; on each operator pop **right** then **left**:
- `+`: pop `2,6` → `6+2 = 8` → stack `[5, 8]`
- `*`: pop `8,5` → `5*8 = 40` → `[40]`
- `12,4` pushed → `[40,12,4]`
- `/`: pop `4,12` → `12/4 = 3` → `[40, 3]`
- `-`: pop `3,40` → `40−3 = 37` → `[37]`

**Answer: 37.**

---

### Problem 5 (MCQ) — right-associative `^` in conversion
The postfix form of `A ^ B ^ C` is:
(A) `A B ^ C ^`  (B) `A B C ^ ^`  (C) `A B C ^ ^` reversed  (D) `A B C * +`

**Solution.** `^` is **right associative**, so `A^B^C = A^(B^C)`. With equal precedence and right-associativity the algorithm **does not** pop the stacked `^`, giving `A B C ^ ^`. **Answer: (B).** (Contrast: left-associative `A − B − C → A B − C −`.)

---

### Problem 6 (NAT) — circular queue count
A circular queue uses an array of size `10`. Convention: `rear` holds the index of the **last** element, `front` the index of the **first** element. Currently `front = 7`, `rear = 2`, and the queue is non-empty. How many elements does it hold?

**Solution.** Convention B: `count = (rear − front + S) % S + 1 = (2 − 7 + 10) % 10 + 1 = 5 % 10 + 1 = 5 + 1 = 6`. **Answer: 6.** (The wrap is real: occupied indices are `7,8,9,0,1,2` → six cells.)

---

### Problem 7 (NAT) — minimum stack size to evaluate postfix
What is the **minimum** stack size (maximum number of operands ever on the stack) needed to evaluate `A B C * + D E F G * - + *`?

**Solution.** Track stack height (operand pushes `+1`, each binary operator nets `−1`):
`A`→1, `B`→2, `C`→3, `*`→2, `+`→1, `D`→2, `E`→3, `F`→4, `G`→**5**, `*`→4, `-`→3, `+`→2, `*`→1.
Peak height = **5**. **Answer: 5.**

---

### Problem 8 (MSQ) — properties / complexity (which are true?)
Select all correct statements.
(I) Implementing a queue with a Python `list` using `pop(0)` for dequeue is O(1).
(II) In an array circular queue (sacrifice-one-cell convention) of size `S`, at most `S−1` elements can be stored.
(III) DFS is naturally implemented with a queue and BFS with a stack.
(IV) For push order `1..n`, the number of achievable pop sequences is the Catalan number `C_n`.

**Solution.**
- (I) **False** — `list.pop(0)` shifts all remaining elements ⇒ O(n); use `deque.popleft` for O(1).
- (II) **True** — one cell is left empty to distinguish *full* from *empty*.
- (III) **False** — it's the reverse: **DFS uses a stack, BFS uses a queue.**
- (IV) **True** — exactly `C_n` distinct sequences.

**Correct: (II) and (IV).**

---

## 6. Common GATE question types on this topic

- **Operation trace:** given a script of push/pop or enqueue/dequeue, find the output or final state.
- **Valid pop sequence / count:** "is this a possible stack output?" and "how many outputs are possible?" (Catalan + 3‑1‑2 / simulation).
- **Expression conversion:** infix → postfix / prefix (precedence & associativity, especially right-assoc `^`).
- **Expression evaluation:** evaluate a given postfix/prefix expression (watch the operand order).
- **Minimum stack depth** to evaluate a postfix expression / **max stack size** during conversion.
- **Circular-queue arithmetic:** front/rear after ops, element count, full vs empty, capacity (`S` vs `S−1`).
- **Complexity MSQs:** O(1) end operations; the `list.pop(0)` O(n) trap; deque vs list.
- **Variant construction:** queue from two stacks, stack from two queues.
- **Application identification:** which structure underlies recursion/DFS (stack) vs scheduling/BFS (queue).

---

## 7. Traps, pitfalls & common mistakes

1. **Postfix operand order:** the **first** popped is the **right** operand. `a b -` = `a − b`, **not** `b − a`. (Prefix is the mirror: first popped = left operand.)
2. **Right-associative `^`** in conversion: with equal precedence, **don't** pop the stacked `^`. `A^B^C → A B C ^ ^` (means `A^(B^C)`), unlike left-associative `−`/`/`.
3. **Circular-queue capacity:** sacrifice-one-cell convention stores **`S−1`**, not `S`. Forgetting the reserved empty cell is the #1 error.
4. **Empty vs full ambiguity:** in a circular queue `front == rear` can mean *both* unless you reserve a cell **or** keep a count. Know which convention the problem states.
5. **Negative modulo in count:** always use `(rear − front + S) % S` — the `+ S` keeps it non-negative when `rear < front`.
6. **`list.pop(0)` is O(n):** a queue built on a Python list with `pop(0)`/`insert(0,…)` is *not* O(1). Use `collections.deque`.
7. **Stack vs queue for DFS/BFS:** **DFS → stack, BFS → queue.** Reversing these is a classic distractor.
8. **`pop` returns a value; check before popping:** popping an empty stack/queue is **underflow** (raises `IndexError` on a Python list/deque).
9. **`{}` is a dict, not a set; `[]` is a list (your stack), but a *queue* should not use it directly** — these subtle type/structure choices change complexity.
10. **Counting distinct outputs:** it's the Catalan number `C_n`, **not** `n!`. (`n!` counts all permutations; only `C_n` are stack-realizable.)
11. **3‑1‑2, not 2‑3‑1:** the forbidden pattern for "pushed in increasing order" outputs is **3‑1‑2** (large, small, medium). When unsure, **simulate** — it never lies.
12. **Prefix evaluation direction:** scan **right→left** (postfix is left→right). Scanning the wrong way flips the result for non-commutative operators.
13. **Min stack size = peak operand count**, computed by running the postfix and tracking the height (each binary op is a net −1). Don't just count operands.
14. **Parentheses in conversion** are popped only by a matching `)`, never by an operator; `(` behaves as the lowest precedence on the stack.

---

## 8. Connection forward

You can now reason about LIFO/FIFO discipline and the O(1) end operations that power them. The array backing, though, forced **fixed capacity** and the circular-queue index juggling. **Linked Lists (Topic 3)** remove both limits: pushing/popping at a list **head**, or enqueuing at a **tail** while dequeuing at a **head**, gives the *same* O(1) stack/queue behaviour with **unbounded** size and **no** wrap-around math — and they set up trees and graph adjacency lists later. (Your stack also resurfaces as the engine of **DFS**, and your queue as the engine of **BFS**, in Topic 10.)

---

## 9. One-line cheat-sheet summary

**Stack = LIFO (`append`/`pop` at the end, O(1)); Queue = FIFO (use `deque`: `append`/`popleft`, O(1)) — master infix→postfix/prefix (right-assoc `^`!), postfix eval (first pop = right operand), circular-queue count `(rear−front+S)%S` with capacity `S−1`, and that valid stack outputs number `C_n` (avoid the 3‑1‑2 pattern).**

---

### ➡️ Next topic (in order): **Topic 3 — Linked Lists** (Chat 1, Python & linear data structures).

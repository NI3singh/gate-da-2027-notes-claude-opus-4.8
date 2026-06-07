# Uninformed Search
### GATE DA — Subject: Artificial Intelligence · Section: Search (Topic 1 of 8)

---

## 0. Topic header & syllabus map

- **Topic:** Uninformed (blind) search
- **Subject:** Artificial Intelligence
- **Where it sits:** This is the **first** algorithmic topic of the *Search* section. It comes right after the idea of a **problem-solving agent** (formulating a problem as a state space) and **leads directly into Informed Search**, where heuristics make these same algorithms goal-directed and far cheaper.

The whole Search section is one story:

```
Problem formulation → UNINFORMED search (this topic, no clues)
                    → Informed search (use a heuristic to aim at the goal)
                    → Adversarial search (an opponent fights back)
```

---

## 1. Why this topic exists (motivation + connection)

**The real problem it solves.** Many AI tasks reduce to: *"I'm in some starting situation; I want to reach a goal situation; what sequence of actions gets me there?"* — route finding, puzzle solving (8-puzzle, Rubik's cube), planning, theorem proving. Each situation is a **state**; each action moves you between states. Finding a solution means finding a **path** from the start state to a goal state through this (often gigantic) state space.

**What came before, and what it couldn't do.** Before any search *algorithm*, all you have is a **problem definition**: an initial state, a way to generate successors, a goal test, and step costs. That definition by itself does **not** tell you *which* successor to try first — it only tells you the rules of the world, not the way to the goal. Uninformed search is the most basic answer: explore the state space *systematically*, using **only** the problem definition and **no extra knowledge** about which states look "closer" to the goal. Because it's the baseline, it's the thing every later, smarter algorithm is measured against.

**What the next topic builds on this.** Uninformed search is *blind*: it spreads out in all directions because it has no idea where the goal is, so its cost is **exponential** in the worst case. **Informed search** (next topic) fixes exactly this: it adds a **heuristic** `h(n)` that estimates remaining distance to the goal, so the search is pulled *toward* the goal instead of expanding everything. But informed search reuses the very machinery — frontier, expansion, graph-search bookkeeping, completeness/optimality reasoning — that you build here. Master this and informed search is "the same engine, plus a heuristic."

---

## 2. Prerequisites

- **Graphs and trees:** nodes, edges, directed/undirected, paths, cycles, branching factor.
- **Data structures:** FIFO **queue**, LIFO **stack**, **priority queue** / min-heap.
- **Recursion** and the call stack (for the DFS / IDS implementation view).
- **Asymptotic notation** (`O(·)`) for comparing time and space.
- **Problem-as-state-space formulation** (the 5 components below) — revise if shaky.

---

## 3. Core idea — intuition first

Imagine you're lost in a building and want to reach the exit, but you have **no map and no signs** — you can only see the doors of the room you're currently in.

- **Breadth-First (BFS):** Explore *cautiously and evenly*. Check every room one door away, then every room two doors away, and so on. You'll find the **nearest** exit (fewest doors), but you must remember a **huge** set of rooms you've half-explored.
- **Depth-First (DFS):** *Plunge ahead*. Pick a door, keep going deeper down one corridor until you hit a dead end, then back up and try the next door. You barely have to remember anything (just your current path), but you might wander far down the wrong corridor — possibly forever.
- **Uniform-Cost (UCS):** Some corridors are long, some short. Instead of "fewest doors," always expand the room reachable at the **lowest total walking distance so far**. This finds the genuinely **cheapest** route even when steps cost different amounts.
- **Iterative Deepening (IDS):** Use DFS's tiny memory but BFS's guarantee of finding the nearest exit, by repeatedly doing a depth-limited DFS with limit 1, then 2, then 3… It re-walks the early rooms each time, but that waste is small.

The common thread: **you never use a hunch about which direction the exit is.** You just have a *systematic* rule for which half-explored room to open next. That rule (FIFO / LIFO / cheapest-first) is the *only* difference between these algorithms.

---

## 4. Formal treatment

### 4.1 Problem formulation (the 5 components)

A search problem is the tuple:

| Component | Meaning |
|---|---|
| **Initial state** `s₀` | Where the agent starts. |
| **Actions / successor fn** | `Actions(s)` = legal actions in state `s`. |
| **Transition model** | `Result(s, a)` = state reached by doing `a` in `s`. |
| **Goal test** | `IsGoal(s)` → true/false. (May be a single state or a property.) |
| **Step (action) cost** | `c(s, a, s′) ≥ 0`. **Path cost** `g(n)` = sum of step costs from `s₀` to `n`. |

The **state space** is the (implicit) graph of all states reachable from `s₀`. A **solution** is a path from `s₀` to a goal state; an **optimal solution** has minimum path cost.

### 4.2 State vs. node — a distinction GATE loves

- A **state** is a configuration of the world (e.g., a board position). The same state can be reached by many paths.
- A **search node** is a bookkeeping object on the search tree. It stores: `state`, `parent`, `action` (that produced it), `path-cost g(n)`, and `depth`.

Two different nodes can hold the **same state**. (This is exactly why we need an *explored set* — see below.)

### 4.3 Key parameters (used in every complexity formula)

| Symbol | Meaning |
|---|---|
| `b` | **Branching factor** — max number of successors of any node. |
| `d` | **Depth of the shallowest goal** (solution depth). |
| `m` | **Maximum depth** of the state space / search tree (can be ∞). |
| `ε` | **Minimum step cost** (ε > 0), used for UCS. |
| `C*` | **Cost of the optimal solution**, used for UCS. |

Always: `d ≤ m`. Confusing `d` with `m` is the #1 complexity mistake (Section 7).

### 4.4 Frontier, explored set, and the two search templates

- **Frontier** (a.k.a. *fringe* / *open list*): nodes generated but not yet expanded. The **search strategy = which node we remove from the frontier next.**
- **Explored set** (a.k.a. *closed list*): states already expanded, kept so we don't re-expand them.

**Tree-Search** (no memory of explored states — can loop on cycles):

```
function TREE-SEARCH(problem):
    frontier ← { node(s₀) }
    loop:
        if frontier is empty: return failure
        node ← REMOVE(frontier)          # order depends on strategy
        if IsGoal(node.state): return SOLUTION(node)
        frontier ← frontier ∪ EXPAND(node)   # add all children
```

**Graph-Search** (keeps an explored set — needed for completeness on graphs with cycles):

```
function GRAPH-SEARCH(problem):
    frontier ← { node(s₀) };  explored ← { }
    loop:
        if frontier is empty: return failure
        node ← REMOVE(frontier)
        if IsGoal(node.state): return SOLUTION(node)
        add node.state to explored
        for each child in EXPAND(node):
            if child.state ∉ explored and child.state ∉ frontier:
                frontier ← frontier ∪ { child }   # (UCS: also replace if cheaper)
```

> The **only** thing that changes between BFS, DFS, and UCS is the `REMOVE` rule:
> **BFS → FIFO queue**, **DFS → LIFO stack**, **UCS → priority queue ordered by `g(n)`**.

### 4.5 The algorithms

#### (a) Breadth-First Search (BFS)
- **Data structure:** FIFO queue. Expands shallowest unexpanded node first.
- **Goal test timing:** at **generation** (as soon as a node is created) — fine here because in a unit-cost space every node at depth `d` is equally good.
- **Complete?** **Yes**, if `b` is finite.
- **Optimal?** **Yes — but only if all step costs are equal** (e.g., all 1). With unequal costs BFS finds the *fewest-edge* path, **not** the cheapest. ⚠
- **Time:** `O(b^d)`  (caveat: `O(b^(d+1))` if goal-tested at expansion instead of generation).
- **Space:** `O(b^d)` — every generated node is stored. **This is BFS's real weakness.**

#### (b) Uniform-Cost Search (UCS) — Dijkstra, made goal-directed
- **Data structure:** priority queue ordered by **path cost `g(n)`**. Expands the lowest-`g` node first.
- **Goal test timing:** at **expansion (when popped), NOT at generation.** Two reasons: (1) the first goal *generated* may lie on a costlier path; (2) we may later find a cheaper path to a frontier node and must keep the cheaper one. Testing at generation can return a **sub-optimal** answer. ⚠ (Classic trap.)
- **Complete?** **Yes**, provided every step cost `≥ ε > 0` (a strictly positive minimum stops infinite zero-cost sequences).
- **Optimal?** **Yes**, for **non-negative** step costs. (Negative edges break it, exactly like Dijkstra.) ⚠
- **Time & Space:** `O(b^(1 + ⌊C*/ε⌋))`. When all steps cost the same `ε` and the goal is at depth `d`, `C*/ε = d`, so this is `O(b^(d+1))` — *slightly worse* than BFS because UCS can explore many cheap deep steps before checking a goal.

#### (c) Depth-First Search (DFS)
- **Data structure:** LIFO stack (or recursion via the call stack). Expands deepest unexpanded node first.
- **Complete?**
  - **Tree-search DFS:** **No** — loops forever in infinite spaces *and* in finite spaces that contain cycles.
  - **Graph-search DFS (with explored set):** **Yes in finite state spaces** (it eventually expands every reachable node). Still **No** in infinite spaces.
- **Optimal?** **No** — it returns the first solution it stumbles on, not the shallowest/cheapest.
- **Time:** `O(b^m)`  (depends on max depth `m`, which can be ≫ `d`, or ∞).
- **Space:** `O(b·m)` for the standard frontier version; `O(m)` for a recursive backtracking variant that generates one successor at a time. **This tiny memory footprint is DFS's whole reason to exist.**

#### (d) Depth-Limited Search (DLS)
- DFS with a hard depth cutoff `ℓ`: never expand below depth `ℓ`.
- **Complete?** **Only if `ℓ ≥ d`.** If `ℓ < d`, the goal is below the limit and is missed.
- **Optimal?** **No.**
- **Time:** `O(b^ℓ)`. **Space:** `O(b·ℓ)`.
- Purpose: tames DFS's infinite-loop problem; the building block for IDS.

#### (e) Iterative Deepening Search (IDS / IDDFS)
- Run DLS with limit `ℓ = 0, 1, 2, …` until a goal is found.
- **Complete?** **Yes**, if `b` is finite.
- **Optimal?** **Yes, if all step costs are equal** (same condition as BFS).
- **Time:** `O(b^d)` — *the regeneration overhead is only a constant factor* (see derivation 4.7). ⚠ (Trap: people think repeating work makes it asymptotically slower than BFS — it does not.)
- **Space:** `O(b·d)` — DFS-like, **much** smaller than BFS's `O(b^d)`.
- **Why it's the favourite uninformed method:** it combines **BFS's completeness + optimality** with **DFS's linear space**.

#### (f) Bidirectional Search
- Search **forward from `s₀`** and **backward from the goal** at the same time (both BFS); stop when the two frontiers **meet**.
- **Time & Space:** `O(b^(d/2))`. Since `b^(d/2) + b^(d/2) ≪ b^d`, this is a massive saving.
- **Complete & Optimal?** **Yes** if both directions use BFS and step costs are uniform.
- **Catches:** you must be able to **generate predecessors** (search backward), and you need an explicit goal state (or a small set) to start the backward search from; the frontier-intersection test also costs memory.

### 4.6 Master comparison table (memorize this)

| Criterion | BFS | UCS | DFS | DLS | IDS | Bidirectional |
|---|---|---|---|---|---|---|
| **Complete?** | Yes¹ | Yes¹ ² | No⁵ | No⁶ | Yes¹ | Yes¹ ⁴ |
| **Time** | `O(b^d)` | `O(b^(1+⌊C*/ε⌋))` | `O(b^m)` | `O(b^ℓ)` | `O(b^d)` | `O(b^(d/2))` |
| **Space** | `O(b^d)` | `O(b^(1+⌊C*/ε⌋))` | `O(b·m)` | `O(b·ℓ)` | `O(b·d)` | `O(b^(d/2))` |
| **Optimal?** | Yes³ | Yes | No | No | Yes³ | Yes³ ⁴ |
| **Structure** | FIFO queue | Priority queue (g) | LIFO stack | LIFO + limit | repeated DLS | two BFS |

**Conditions (these are the marks):**
1. complete **only if `b` is finite**
2. complete **only if step costs `≥ ε > 0`**
3. optimal **only if all step costs are equal**
4. **both directions BFS** (and uniform costs)
5. DFS: tree-search is **not** complete; graph-search **is** complete in **finite** spaces
6. DLS complete **only if `ℓ ≥ d`**

### 4.7 The two derivations GATE actually asks for

**(i) Why IDS time is `O(b^d)` despite re-expansion.**
In IDS to limit `d`, the **bottom** level (depth `d`) is generated **once**, depth `d−1` **twice**, …, depth `1` is generated `d` times. So the number of generated nodes is:

```
N(IDS) = d·b + (d−1)·b² + … + 2·b^(d−1) + 1·b^d
```

Compare with BFS, which generates each level once:

```
N(BFS) = b + b² + … + b^d
```

Because the **last term `b^d` dominates** the geometric sum, `N(IDS) = O(b^d)` — same order as BFS. The asymptotic overhead factor is only about `b/(b−1)`.

*Numeric sanity check (b = 10, d = 5):*
```
N(IDS) = 50 + 400 + 3000 + 20000 + 100000 = 123,450
N(BFS) = 10 + 100 + 1000 + 10000 + 100000 = 111,110
Overhead ≈ 11%  (≈ b/(b−1) = 10/9)
```
*(Convention note: this counts generated successor nodes from depth 1. Some books also count the root once per iteration; the difference is negligible. Use whatever level-counting the question implies.)*

**(ii) Why UCS is optimal (sketch).** UCS always pops the frontier node with the **smallest `g`**. When it pops a goal node, every other frontier node already has `g ≥` this goal's cost, and (with non-negative steps) every *unexplored* path can only get more expensive. Hence the popped goal is the cheapest — *provided the goal test is applied at pop time, not at generation.*

---

## 5. Worked problems (GATE-style)

### Problem 1 — BFS vs DFS expansion order and frontier (MCQ-style)

Consider this search **tree** (children expanded **left to right**); the goal is **H**.

```
            A
          / | \
         B  C  D
        /|     |\
       E F     G H
```
Adjacency: `A→{B,C,D}`, `B→{E,F}`, `D→{G,H}`; `C, E, F, G` are leaves.

**(a)** Give the order in which nodes are **expanded** by BFS until the goal is found.
**(b)** Give the order for DFS.
**(c)** Just **after expanding C**, what does the BFS frontier (queue) contain?

**Solution.**

**(a) BFS (FIFO).** Expand by increasing depth, left to right:
`A, B, C, D, E, F, G, H` → finds **H** (goal test at generation would actually detect H the moment D is expanded, but as an *expansion* sequence the answer is A, B, C, D and then the depth-2 nodes).
Standard expansion answer: **A, B, C, D, E, F, G, H**.

**(b) DFS (LIFO).** Go deep down the leftmost branch first:
`A, B, E, F, C, D, G, H` → finds **H**.
Standard expansion answer: **A, B, E, F, C, D, G, H**.

**(c)** Start: dequeue `A`, enqueue `B,C,D` → queue `[B,C,D]`.
Dequeue `B`, enqueue `E,F` → `[C,D,E,F]`.
Dequeue `C` (a leaf, no children) → **frontier = `[D, E, F]`**. ✔

**Takeaway:** BFS and DFS visit the *same nodes* but in a *different order*; the data structure (queue vs stack) is the entire difference.

---

### Problem 2 — Uniform-Cost Search & the "BFS is not optimal" trap (NAT + reasoning)

Weighted directed graph (edge costs in parentheses). Start = **S**, goal = **G**.

```
S →(1) A        A →(3) B        B →(3) D
S →(12) G       A →(1) C        C →(1) D        D →(3) G
```

**(a)** Find the **optimal path** and its cost using UCS.
**(b)** List the **order of node expansion** by UCS (graph search).
**(c)** What path would **BFS** return, and why is it wrong here?

**Solution.**

Enumerate the candidate paths to G:
- `S→G` = 12
- `S→A→B→D→G` = 1+3+3+3 = 10
- `S→A→C→D→G` = 1+1+1+3 = **6**  ← cheapest

**UCS trace** (priority queue keyed by `g`; pop smallest; goal-test at pop):

| Pop (g) | Generate (g) | Frontier after |
|---|---|---|
| S (0) | A(1), G(12) | A:1, G:12 |
| A (1) | B(1+3=4), C(1+1=2) | C:2, B:4, G:12 |
| C (2) | D(2+1=3) | D:3, B:4, G:12 |
| D (3) | G(3+3=6) → keep 6 over 12 | B:4, G:6 |
| B (4) | D(4+3=7) → D already expanded, **discard** | G:6 |
| G (6) | **goal popped → stop** | — |

**(a)** Optimal path **S→A→C→D→G**, cost **6**.
**(b)** Expansion order: **S, A, C, D, B, G**.
**(c)** BFS ignores edge weights and returns the **fewest-edge** path, `S→G`, with cost **12** — it is the shortest in *number of edges* but **not** the cheapest. This is the canonical demonstration that **BFS is optimal only for equal step costs.** ⚠

**Trap inside the trap:** if UCS had goal-tested `G` at *generation* (when first produced via `S→G` at g=12, or via `D` at g=6), naïve early termination on the *first* generated goal could return the wrong cost. UCS is safe **only** because it tests at **pop** time.

---

### Problem 3 — IDS node count vs BFS (NAT)

A search tree has branching factor `b = 3`. The shallowest goal is at depth `d = 4`. Assuming the goal test happens at generation and counting generated nodes from depth 1:

**(a)** How many nodes does **BFS** generate (worst case, to reach depth 4)?
**(b)** How many does **IDS** generate?
**(c)** What is the percentage overhead of IDS over BFS?

**Solution.**

`b = 3`: levels have `3, 9, 27, 81` nodes (depths 1–4).

**(a) BFS:** `3 + 9 + 27 + 81 = 120`.

**(b) IDS:** depth `k` is regenerated `(d − k + 1)` times:
```
N(IDS) = 4·3 + 3·9 + 2·27 + 1·81
       = 12 + 27 + 54 + 81 = 174
```

**(c) Overhead** = `(174 − 120)/120 = 54/120 = 45%`.

**Sanity check with the rule of thumb** `b/(b−1) = 3/2 = 1.5` (i.e., ~50% overhead): for small `b` the overhead is larger (here 45%, close to 50%); for `b = 10` it drops to ~11%. **The point:** the overhead is a **constant factor**, so `N(IDS) = O(b^d)` — same order as BFS. ⚠

---

### Problem 4 — True/False statements (MSQ, "trap" style)

Mark each as TRUE or FALSE.

1. DFS using graph search (with an explored set) is complete in any **finite** state space.
2. BFS always returns the minimum-cost path.
3. IDS uses asymptotically more **time** than BFS.
4. UCS with all step costs equal expands nodes in essentially the same order as BFS.
5. UCS remains optimal even if some edge costs are negative.

**Solution.**

1. **TRUE.** A finite space has finitely many states; the explored set prevents re-expansion, so DFS terminates and finds a goal if one exists.
2. **FALSE.** Only when step costs are equal. With unequal costs BFS minimizes *edges*, not *cost* (see Problem 2).
3. **FALSE.** Both are `O(b^d)`; IDS's extra work is a constant factor only.
4. **TRUE.** With equal costs, "lowest `g`" = "shallowest," so UCS behaves like BFS (minor difference: UCS goal-tests at pop, BFS at generation, so UCS may expand a few extra nodes / tie-breaking can differ).
5. **FALSE.** Negative edges break UCS's optimality, exactly like Dijkstra; you'd need a different algorithm.

(TRUE: **1, 4** · FALSE: **2, 3, 5**.)

---

## 6. Common GATE question types on this topic

- **Trace the expansion order** of BFS / DFS / UCS / IDS on a given graph or tree.
- **State the contents of the frontier** (queue / stack / priority queue) after `k` steps.
- **Find the optimal path & its cost** with UCS on a weighted graph (and contrast with BFS's wrong answer).
- **Count nodes generated / expanded** (especially IDS vs BFS) — NAT numericals.
- **Fill the completeness / optimality / time / space table** — and identify the **exact condition** under which each property holds (the footnotes are the marks).
- **Pick the right algorithm** for a constraint ("limited memory" → DFS/IDS; "need cheapest path, varying costs" → UCS; "huge but shallow goal, must be optimal" → IDS/Bidirectional).
- **Spot the false statement** in an MSQ about properties (the trap set in Problem 4).

---

## 7. Traps, pitfalls & common mistakes

- **`d` vs `m` mix-up.** BFS/IDS use `d` (shallowest goal); DFS/DLS use `m`/`ℓ` (max depth / limit). Plugging `m` into BFS or `d` into DFS is the most common complexity error.
- **"BFS is optimal."** Only with **equal step costs.** Add weights and it breaks. The single most-tested trap.
- **UCS goal test at generation.** Must test at **pop**, not at generation, or you can return a sub-optimal solution. Same logic: don't stop UCS the instant a goal is *generated*.
- **Negative edges.** UCS (and Dijkstra) assume **non-negative** costs; negatives destroy optimality.
- **"IDS is slow because it repeats work."** Wrong — repetition is only a constant factor; IDS is `O(b^d)`, same order as BFS, with far less memory.
- **DFS completeness blanket claim.** Tree-search DFS is **not** complete (loops on cycles / infinite depth). Only **graph-search DFS in finite spaces** is complete. State the qualifier.
- **DLS with `ℓ < d`.** Returns failure even though a goal exists. Completeness needs `ℓ ≥ d`.
- **Confusing space culprit.** It's **space**, not time, that kills BFS (`O(b^d)` memory). DFS's whole advantage is `O(bm)` space.
- **State vs node.** Two nodes can hold the same state; the explored set works on **states**, not nodes — forgetting this leads to wrong frontier traces on graphs with multiple paths.
- **Bidirectional preconditions.** Needs a **predecessor function** and an explicit goal to search backward from; don't claim `O(b^(d/2))` without "both BFS."
- **Tie-breaking.** When two frontier nodes tie, GATE usually fixes an order (alphabetical / left-to-right / FIFO). Read the question's stated rule; otherwise your trace order may differ from the key.

---

## 8. Connection forward

Uninformed search is **blind** — it expands in every direction because nothing tells it where the goal lies, which is why its cost is exponential. **Informed (heuristic) search** — the next topic — keeps this exact frontier-expansion machinery but adds a **heuristic `h(n)`** that estimates remaining cost to the goal, so the priority queue is ordered to head *toward* the goal. In fact:

> **UCS ordered by `g(n)` + a heuristic `h(n)` = A\*** (orders the frontier by `f(n) = g(n) + h(n)`).

So Informed Search isn't a new engine — it's **UCS with a sense of direction.** Every completeness/optimality argument you practiced here returns, now phrased in terms of *admissible* and *consistent* heuristics.

---

## 9. One-line cheat-sheet summary

**Uninformed search explores a state space using only the problem definition (no heuristic):** BFS (FIFO, `O(b^d)` time & space, optimal only for equal costs), UCS (priority queue on `g`, optimal for non-negative costs, goal-test at pop), DFS (LIFO, `O(bm)` space, not optimal/complete in general), IDS (DFS space `O(bd)` + BFS guarantees, `O(b^d)` time) — and `d ≤ m` always.

---

*Next topic (logically): **Informed Search** — heuristics, Greedy Best-First, and A\* (= this topic's UCS, now guided by `h(n)`).*

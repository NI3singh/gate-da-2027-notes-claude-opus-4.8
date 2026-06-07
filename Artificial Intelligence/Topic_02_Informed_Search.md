# Informed Search
### GATE DA — Subject: Artificial Intelligence · Section: Search (Topic 2 of 8)

---

## 0. Topic header & syllabus map

- **Topic:** Informed (heuristic) search — Greedy Best-First, A\*, and memory-bounded variants (IDA\*).
- **Subject:** Artificial Intelligence
- **Where it sits:** Second topic of the *Search* section. It comes **after Uninformed Search** (which explored blindly) and **leads into Adversarial Search** (where the heuristic idea becomes the *evaluation function* for game positions).

```
Uninformed search (blind) → INFORMED search (this topic: add a heuristic h(n))
                          → Adversarial search (heuristic → board evaluation, plus an opponent)
```

The headline relationship to remember:

> **A\* = Uniform-Cost Search + a heuristic.** UCS ordered the frontier by `g(n)`; A\* orders it by `f(n) = g(n) + h(n)`. Set `h ≡ 0` and A\* *is* UCS.

---

## 1. Why this topic exists (motivation + connection)

**The real problem it solves.** Uninformed search wastes enormous effort because it has **no idea where the goal is** — BFS/UCS spread out evenly in all directions, paying `O(b^d)`. But in most real problems we *do* have a cheap estimate of "how far is this state from the goal" (e.g., straight-line distance for route finding, number of misplaced tiles for the 8-puzzle). Informed search uses that estimate to **aim** the search at the goal, often shrinking the explored region from exponential to nearly linear along the solution path.

**What the previous topic couldn't do.** In Topic 1, the *only* inputs were the problem definition (states, actions, costs, goal test). That tells you the rules of the world but **not which successor looks promising.** So uninformed methods must, in the worst case, examine essentially the whole frontier at each depth. There was no way to say "expand *this* node next because it's heading toward the goal." Informed search adds exactly that missing ingredient: a **heuristic function `h(n)`** estimating the cheapest cost from `n` to a goal.

**The bridge from Topic 1.** Best-first search is "UCS with a tunable ordering function `f(n)`." Choosing `f = g` gives back UCS; choosing `f = h` gives Greedy; choosing `f = g + h` gives A\*. Every property you proved for UCS (frontier mechanics, the pop-time goal test, graph-search bookkeeping) carries over — A\* just changes the key the priority queue sorts by.

**What the next topic builds on this.** Adversarial Search (next) drops the assumption that *you alone* control every move. An opponent now actively works against you, so you can't simply walk a path to a goal. But when a game tree is too deep to search to the end, games use an **evaluation function** to estimate "how good is this position" — which is precisely a heuristic, generalized from "distance to a goal" to "value of a state." So `h(n)` is the seed of the static evaluation function in minimax.

---

## 2. Prerequisites

- **Uninformed search (Topic 1):** frontier / explored set, tree-search vs graph-search, UCS and its priority-queue + pop-time goal test, the `b, d, m, ε, C*` parameters, completeness/optimality reasoning.
- **Priority queue / min-heap** and **graphs with weighted edges**.
- **Path cost `g(n)`** (cost from start to `n`) — directly from UCS.
- **Asymptotic notation** and basic induction (for the consistency ⇒ admissibility argument).

---

## 3. Core idea — intuition first

Back to the building-with-no-map analogy, but now someone hands you a **rough compass** that points roughly toward the exit and whispers an *estimate* of how far it is. That estimate is the **heuristic `h(n)`**.

Two ways to use it:

- **Greedy Best-First:** trust the compass completely — always step into the room with the **smallest estimated remaining distance `h`**, ignoring how far you've already walked. Fast, but the compass can fool you into a long, expensive corridor that *looks* close. **Not optimal.**
- **A\*:** balance both — order rooms by `f = (distance already walked) + (estimated distance remaining) = g + h`. This refuses to be tricked: it won't commit to a "looks-close" room if getting there was already very expensive. **Optimal**, provided the compass never *overestimates* the remaining distance.

The single condition that makes A\* trustworthy is **admissibility**: the estimate must be **optimistic** — never larger than the true remaining cost. An optimistic estimate can make A\* explore a bit more, but it can *never* make A\* discard the true cheapest path, because that path's `f` stays a lower bound on its true cost.

A stronger, very convenient property is **consistency** (the heuristic obeys a triangle inequality: stepping to a neighbour can drop the estimate by at most the step's cost). Consistency guarantees that the **first** time A\* reaches any state, it already has the cheapest way there — so it never has to revisit a settled state.

---

## 4. Formal treatment

### 4.1 Heuristic function

For a node `n` with state `s`:

- `h(n)` = **estimated** cheapest cost from `s` to a goal. By definition `h(goal) = 0`, and `h(n) ≥ 0`.
- `h*(n)` = the **true** cheapest cost from `s` to a goal (often unknown — that's why we estimate).
- `g(n)` = actual cost of the path from start to `n` found so far.

### 4.2 Best-First Search (the general framework)

Run graph-search, but `REMOVE` from the frontier the node with the **minimum evaluation `f(n)`** (priority queue keyed by `f`). The choice of `f` defines the algorithm:

| Algorithm | Evaluation function `f(n)` | Idea |
|---|---|---|
| Uniform-Cost (Topic 1) | `g(n)` | cheapest so far |
| **Greedy Best-First** | `h(n)` | looks closest to goal |
| **A\*** | `g(n) + h(n)` | cheapest *estimated total* through `n` |

### 4.3 Greedy Best-First Search — `f = h`

Expands the node that *appears* closest to the goal.

- **Complete?**
  - **Tree-search:** **No** — can loop forever (oscillates between two "close-looking" states). *(Classic Romania example: Iaşi→Neamţ looks closer to Fagaras but is a dead end; greedy bounces back and forth.)*
  - **Graph-search (explored set):** **Yes in finite state spaces.**
- **Optimal?** **No.** It ignores `g`, so it happily takes an expensive route that ends near the goal. ⚠
- **Time:** `O(b^m)` worst case; a good heuristic can cut this dramatically.
- **Space:** `O(b^m)` — keeps all generated nodes (it's best-first, not depth-first).

### 4.4 A\* Search — `f = g + h`

Expands the node with the lowest **estimated total solution cost** through it: `f(n) = g(n) + h(n)`.

- Uses a **priority queue keyed by `f`**, and (as in UCS) applies the **goal test at pop time**, and **replaces a frontier node if a cheaper path to it is found**.
- **Complete?** **Yes**, provided `b` is finite *and* there are finitely many nodes with `f(n) ≤ C*` (guaranteed when every step cost `≥ ε > 0`).
- **Optimal?**
  - **A\* tree-search is optimal iff `h` is admissible.**
  - **A\* graph-search is optimal iff `h` is consistent** (or `h` admissible *with re-opening* of closed nodes). ⚠ This tree-vs-graph condition is a top GATE trap.
- **Optimally efficient:** for a given **consistent** heuristic, no other optimal algorithm that extends paths from the root and uses the same `h` is guaranteed to expand fewer nodes than A\* (up to tie-breaking among nodes with `f = C*`).
- **Time & Space:** **exponential in the worst case.** Space (it stores every generated node, like UCS/BFS) is the binding constraint — this is what motivates IDA\* (§4.8).

**The node-expansion characterization of A\*** (heavily tested):

> A\* expands **every** node with `f(n) < C*`, expands **some** nodes with `f(n) = C*`, and **never** expands a node with `f(n) > C*`.

### 4.5 Admissibility vs Consistency (the two definitions GATE tests most)

**Admissible** (never overestimates):
```
h(n) ≤ h*(n)   for every node n        (and h(goal) = 0)
```
*Optimistic.* Guarantees A\* **tree-search** optimality.

**Consistent / monotone** (triangle inequality):
```
h(n) ≤ c(n, a, n') + h(n')   for every node n and every successor n'   (and h(goal) = 0)
```
*The estimate drops by at most the step cost.* Guarantees A\* **graph-search** optimality.

**Key facts:**

1. **Consistency ⇒ admissibility** (but **not** the reverse). *Proof by induction on the number of steps from `n` to the goal along an optimal path:* base case `h(goal)=0=h*(goal)`. Inductive step: let `n'` be the next node on the optimal path from `n`, so `h*(n) = c(n,n') + h*(n')`. Then by consistency and the inductive hypothesis,
   `h(n) ≤ c(n,n') + h(n') ≤ c(n,n') + h*(n') = h*(n)`. ∎
2. **Consistency ⇒ `f` is non-decreasing along any path** (monotonicity): `f(n') = g(n)+c(n,n')+h(n') ≥ g(n)+h(n) = f(n)`. Therefore the first time A\* pops a state, it has the optimal `g` for it ⇒ **no re-opening of closed nodes is ever needed.** This is *why* consistency is the right condition for graph search.
3. **Admissible-but-inconsistent heuristics exist** (see Worked Problem 1). With such an `h`, graph-search A\* may settle a state via a costlier path first and would need node re-opening to stay optimal.

*(Compact reason A\* tree-search is optimal with admissible `h`: a suboptimal goal `G₂` on the frontier has `f(G₂) = g(G₂) > C*`, while any frontier node `n` on the optimal path has `f(n) = g(n)+h(n) ≤ g(n)+h*(n) = C*`. So `f(n) ≤ C* < f(G₂)` and A\* expands `n` before `G₂`, never returning the suboptimal goal first.)*

### 4.6 Dominance — comparing heuristics

If `h₂(n) ≥ h₁(n)` for **all** `n` and **both are admissible**, then `h₂` **dominates** `h₁`. Consequence:

> A\* with `h₂` expands **no more** nodes than A\* with `h₁` (every node A\* with `h₂` expands has `f₂ < C*`, hence `f₁ < C*` too). A larger admissible heuristic is **better-informed**, never worse.

If several admissible heuristics `h₁,…,hₖ` are available, then `h(n) = max(h₁(n),…,hₖ(n))` is **admissible and dominates all of them** — use the max.

### 4.7 Generating heuristics

- **Relaxed problems.** Drop a constraint on the actions. The optimal solution cost of the *relaxed* problem is an **admissible (and consistent) heuristic** for the original — because the relaxed problem allows *more* moves (a supergraph), so its optimal cost ≤ the original's, and being a true cost it obeys the triangle inequality.
  - **8-puzzle illustration:**
    - `h₁` = **number of misplaced tiles** (relax: a tile may jump anywhere in one move).
    - `h₂` = **sum of Manhattan distances** (relax: a tile may move to *any* adjacent square, ignoring the blank).
    - Both admissible; `h₂(n) ≥ h₁(n)` always ⇒ **`h₂` dominates `h₁`** ⇒ A\* with `h₂` expands fewer nodes.
- **Pattern databases:** precompute exact optimal costs of sub-problems and store them; these costs are admissible.
- **Combining:** take the `max` of several admissible heuristics (above).
- **Trade-off:** a sharper heuristic may be costly to compute; the net win is *(nodes saved) × (per-node cost) − (heuristic cost)*.

**Effective branching factor `b*`** (how to *measure* heuristic quality): if A\* expands `N` nodes to find a solution at depth `d`, then `b*` solves
```
N + 1 = 1 + b* + (b*)² + ⋯ + (b*)^d
```
A good heuristic gives `b*` close to **1**.

### 4.8 Memory-bounded informed search

A\*'s memory is the problem, so:

- **IDA\* (Iterative-Deepening A\*).** Like IDS, but the cutoff is the **`f`-value**, not depth. Start cutoff `= f(root) = h(root)`; do an `f`-bounded DFS pruning any node with `f > cutoff`; if it fails, set the new cutoff to the **smallest `f` that exceeded** the old one; repeat.
  - **Space `O(bd)`** (DFS-like). **Optimal** with admissible `h`; complete under A\*'s conditions.
  - **Best when few distinct `f`-values** (e.g., unit step costs). With many distinct `f`-values it adds barely one node per iteration and degrades.
- **RBFS (Recursive Best-First Search).** Linear space `O(bd)`; mimics best-first by tracking the best alternative `f` and backing it up; can re-expand nodes. Optimal with admissible `h`.
- **SMA\* (Simplified Memory-Bounded A\*).** Uses all available memory; when full, drops the highest-`f` leaf (remembering its value in the parent). Complete/optimal if the solution path fits in memory.

*(For GATE DA, master IDA\*; know RBFS/SMA\* exist and that all three trade time for space against A\*.)*

### 4.9 Comparison table (memorize)

| Criterion | Greedy Best-First | A\* (tree) | A\* (graph) | IDA\* |
|---|---|---|---|---|
| `f(n)` | `h(n)` | `g+h` | `g+h` | `g+h`, with `f`-cutoff |
| **Complete?** | No (tree) / Yes (finite graph) | Yes¹ | Yes¹ | Yes¹ |
| **Optimal?** | **No** | Yes, if `h` **admissible** | Yes, if `h` **consistent**² | Yes, if `h` **admissible** |
| **Time** | `O(b^m)` | exp. (worst) | exp. (worst) | exp. |
| **Space** | `O(b^m)` | exp. — stores all nodes | exp. | **`O(bd)`** |

1. needs `b` finite **and** step costs `≥ ε > 0` (finitely many nodes with `f ≤ C*`).
2. or admissible `h` **with re-opening** of closed nodes.

---

## 5. Worked problems (GATE-style)

For Problems 2–3, use this weighted directed graph (Start **S**, Goal **G**). Edge costs in parentheses; heuristic `h` shown per node.

```
Edges:  S→A(1)  S→B(4)  A→B(2)  A→C(5)  B→C(2)  B→G(6)  C→G(3)
h:      h(S)=7  h(A)=6  h(B)=4  h(C)=2  h(G)=0
```

---

### Problem 1 — Is the heuristic admissible? Consistent? (MSQ/short)

Graph (Start **S**, Goal **G**): `S→A(1), S→B(2), A→B(1), A→G(5), B→G(2)`, with `h(S)=4, h(A)=3, h(B)=1, h(G)=0`.

**(a)** Is `h` admissible?  **(b)** Is `h` consistent?  **(c)** What does this imply for A\*?

**Solution.**

First compute the true costs `h*` (cheapest path to G from each node):
- `h*(G)=0`
- `h*(B)= B→G = 2`
- `h*(A)= min(A→B→G = 1+2, A→G = 5) = 3`
- `h*(S)= min(S→A→B→G = 1+1+2, S→A→G = 1+5, S→B→G = 2+2) = 4`

**(a) Admissibility** (`h ≤ h*`): `4≤4, 3≤3, 1≤2, 0≤0` — all hold ⇒ **admissible.** ✔

**(b) Consistency** (`h(n) ≤ c + h(n')` on every edge):

| Edge | Check | OK? |
|---|---|---|
| S→A | `4 ≤ 1+3 = 4` | ✔ |
| S→B | `4 ≤ 2+1 = 3` | ✗ |
| A→B | `3 ≤ 1+1 = 2` | ✗ |
| A→G | `3 ≤ 5+0 = 5` | ✔ |
| B→G | `1 ≤ 2+0 = 2` | ✔ |

Two edges violate the triangle inequality ⇒ **not consistent.**

**(c)** `h` is **admissible but not consistent** — a perfectly legal combination. So **A\* tree-search is still optimal here**, but **A\* graph-search needs re-opening of closed nodes** to guarantee optimality (consistency was the condition that let graph search skip re-opening). This proves *admissibility does **not** imply consistency.* ⚠

---

### Problem 2 — A\* trace: optimal path, expansion order, `f`-values (NAT)

Using the graph at the top of §5, run **A\* graph-search**. (Note this `h` is **consistent** — verify if you wish.) Find the path returned, its cost, and the expansion order.

**Solution.** Priority queue keyed by `f = g + h`; pop smallest; goal test at pop; replace a frontier node when a cheaper `g` is found.

| Pop (g, f) | Successors generated (g, f) | Frontier after pop |
|---|---|---|
| S (g0, f7) | A(g1, f7), B(g4, f8) | A:f7, B:f8 |
| A (g1, f7) | B via A(g3, f7) **↓ replaces B**, C(g6, f8) | B:f7(g3), C:f8(g6) |
| B (g3, f7) | C via B(g5, f7) **↓ replaces C**, G(g9, f9) | C:f7(g5), G:f9(g9) |
| C (g5, f7) | G via C(g8, f8) **↓ replaces G** | G:f8(g8) |
| G (g8, f8) | **goal popped → stop** | — |

- **Optimal path:** trace parents `G←C←B←A←S` ⇒ **S→A→B→C→G**, **cost 8**.
- **Expansion order:** **S, A, B, C, G**.
- **Popped `f`-values:** 7, 7, 7, 7, 8 — **non-decreasing**, exactly as a consistent heuristic guarantees (monotonicity). Every node with `f < C* = 8` (S, A, B, C) was expanded; the goal at `f = 8` was popped; no node with `f > 8` was expanded — matching the node-expansion rule in §4.4.

---

### Problem 3 — Greedy Best-First on the same graph: the sub-optimality trap (MCQ)

Run **Greedy Best-First** (`f = h`) on the §5 graph. What path does it return, and how does it compare to A\*?

**Solution.** Pop the node with the smallest **`h`** (ignore `g`):

| Pop (h) | Successors (h) | Pick (min h) |
|---|---|---|
| S (7) | A(6), B(4) | B (4) |
| B (4) | C(2), G(0) | G (0) |
| G (0) | goal | — |

- **Path returned:** **S→B→G**, **cost = 4 + 6 = 10**.
- **A\*'s optimal cost was 8** (Problem 2) ⇒ **Greedy is sub-optimal here.** It chased the low `h(B)=4` straight into the *expensive* edges `S→B(4)` and `B→G(6)`, never noticing the cheaper `S→A→B→C→G`.
- Greedy expanded only **3** nodes vs A\*'s **5** — *faster but worse*. This is the canonical Greedy-vs-A\* trade-off: speed without guarantees. ⚠

---

### Problem 4 — Property statements (MSQ, "trap" style)

Mark TRUE/FALSE.

1. Every **consistent** heuristic is **admissible**.
2. Every **admissible** heuristic is **consistent**.
3. Greedy best-first search is optimal whenever `h` is admissible.
4. With `h(n) = 0` for all `n`, A\* behaves exactly like **Uniform-Cost Search**.
5. If `h₁, h₂` are admissible, then `h(n) = max(h₁(n), h₂(n))` is admissible and dominates both.
6. A\* never expands a node whose `f(n) > C*`.

**Solution.**

1. **TRUE** (proved by induction, §4.5).
2. **FALSE** — counterexample in Problem 1 (admissible but inconsistent). ⚠
3. **FALSE** — Greedy ignores `g`; admissibility says nothing about it (Problem 3). ⚠
4. **TRUE** — `f = g + 0 = g`, the UCS key. *(This is the formal statement of "A\* = UCS + heuristic.")*
5. **TRUE** — the max of admissible heuristics is admissible and ≥ each, so it dominates.
6. **TRUE** — node-expansion characterization (§4.4).

(TRUE: **1, 4, 5, 6** · FALSE: **2, 3**.)

---

## 6. Common GATE question types on this topic

- **Trace A\* / Greedy** on a weighted graph: give the **expansion order**, the **`f`/`g`/`h` table**, the **returned path** and its **cost**.
- **Check admissibility / consistency** of a given `h` (compute `h*`, test the triangle inequality on every edge) — and conclude which optimality (tree vs graph) is guaranteed.
- **Count nodes A\* expands** using the rule "all with `f < C*`, some with `f = C*`, none with `f > C*`."
- **Dominance:** given two admissible heuristics, decide which expands fewer nodes; or build a better one via `max`.
- **Effective branching factor `b*`** from `(N, d)`.
- **Greedy-vs-A\*-vs-UCS contrasts:** which is optimal, which is fastest, what `h ≡ 0` reduces A\* to.
- **Memory-bounded:** identify when IDA\* is preferable (limited memory / few distinct `f`-values) and its space `O(bd)`.
- **MSQ traps** on the exact conditions (the false statements in Problem 4).

---

## 7. Traps, pitfalls & common mistakes

- **Tree vs graph optimality condition.** Admissibility ⇒ A\* *tree-search* optimal; you need **consistency** for A\* *graph-search* optimality (or admissibility **with re-opening**). Stating "admissible ⇒ A\* always optimal" unqualified is the most common error. ⚠
- **Admissible ⇏ consistent.** They are *different*; consistency is strictly stronger (Problem 1). The converse (consistent ⇒ admissible) *is* true.
- **Greedy is not optimal and not (tree-)complete.** It ignores `g`. Don't confuse it with A\*.
- **Forgetting to add `g`.** A\* sorts by `f = g + h`, **not** by `h` alone. Dropping `g` silently turns A\* into Greedy and changes the answer.
- **Goal test timing.** As in UCS, test at **pop** time, not generation — and **update a frontier node** when a cheaper path appears. Otherwise you can report a sub-optimal cost. ⚠
- **Overestimating heuristic (inadmissible).** If `h` ever exceeds `h*`, A\* may **miss the optimal solution** (it can prune the true best path). Watch for this in "is this `h` safe?" questions.
- **Negative step costs.** A\* (like UCS/Dijkstra) assumes **non-negative** edge costs. Negatives break the analysis.
- **Re-opening with a consistent heuristic.** **Not needed** — consistency makes `f` monotone, so the first pop of a state is optimal. Claiming consistent-A\* must re-open closed nodes is false (Problem 4.6... item 6 logic).
- **IDA\* with many distinct `f`-values.** It then adds ~one node per iteration and behaves almost like plain DFS — IDA\* shines with **few** distinct `f`-values (unit costs).
- **`d` vs `m` again.** Greedy's worst case uses `m` (max depth); IDA\* space uses `d` (solution depth). Same mix-up as Topic 1.
- **Dominance needs *both* admissible.** "`h₂ ≥ h₁`" only implies fewer expansions when **both are admissible**; otherwise the comparison is meaningless.

---

## 8. Connection forward

Informed search assumes a **single agent** moving through a **static** world that it fully controls — you choose every action toward your own goal, guided by `h(n)`. **Adversarial Search** (next) removes that luxury: a **second agent actively opposes you** (chess, tic-tac-toe, Nim). You can no longer plan a one-sided path; you must assume the opponent replies with *their* best move at every turn — the **minimax** principle, refined by **α-β pruning** to skip provably irrelevant branches (the same "don't expand what can't matter" instinct as A\*'s `f > C*` pruning).

Crucially, the **heuristic carries over**: when a game tree is too deep to search to a terminal state, players cut off the search and apply a **static evaluation function** that estimates "how good is this position" — exactly `h(n)`, generalized from *distance-to-goal* to *value-of-a-state*. So this topic's central idea, **estimating quality without solving to the end**, is precisely what makes adversarial search tractable.

---

## 9. One-line cheat-sheet summary

**Informed search guides exploration with a heuristic `h(n)`:** Greedy (`f=h`, fast, **not optimal**), A\* (`f=g+h`, **optimal** with **admissible** `h` for tree-search / **consistent** `h` for graph-search; **consistent ⇒ admissible**, not vice-versa; expands all `f<C*`, none `f>C*`), bigger admissible `h` **dominates**, `h≡0` makes A\* = UCS, and **IDA\*** gives A\*-optimality in `O(bd)` space.

---

*Next topic (logically): **Adversarial Search** — game trees, Minimax, and α-β pruning, where this topic's heuristic becomes the static evaluation function.*

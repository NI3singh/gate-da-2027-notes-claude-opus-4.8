# Topic 10 — Graph Traversals (BFS, DFS) & Shortest Path (Dijkstra)

*Subject: Programming, Data Structures and Algorithms (GATE DA)*

---

## 0. Topic header & syllabus map

- **Topic:** Graph Traversals (Breadth-First Search, Depth-First Search) & Shortest Path (Dijkstra)
- **Subject:** Programming, Data Structures and Algorithms
- **Where it sits:** This is the **final** topic of the *Graphs* section and the **last topic of the DSA subject**. It comes immediately **after Topic 9 (Introduction to Graph Theory)**, which gave us the *static* objects — vertices, edges, weights, adjacency lists/matrices, paths, distance, connectivity, trees. This topic gives us the **procedures** that actually *compute* those things by systematically walking the graph.

> **Scope note for GATE DA.** The named, high-value algorithms are **BFS, DFS, and Dijkstra** — these get full depth (traces, complexity, edge classification, applications, traps). Natural neighbours that recur in GATE CS — **Bellman–Ford, Floyd–Warshall, SCCs, articulation points/bridges** — are included compactly and marked **[lower priority / context]** so you can triage on the last day. A one-line **priority-queue / min-heap** refresher is built in, since Dijkstra needs it and DA has no separate heaps topic.

---

## 1. Why this topic exists (motivation + connection)

Topic 9 told us *what* a path, a cycle, a component, and a distance **are** — but it gave no **procedure** to find them. From the definitions alone you cannot answer the questions that actually matter:

- *Is the graph connected? How many components?*
- *What is the shortest route from A to B* (fewest hops, or least total weight)?
- *Does the graph contain a cycle? Is this directed graph acyclic?*
- *In what order can I perform tasks that depend on one another?*

**What the previous topic couldn't do.** Topic 9 is descriptive: it defines distance but never computes it; it defines "connected" but never tests it. A definition is not an algorithm. To answer the questions above you must **explore the graph in a disciplined order**, marking what you have seen so you neither miss nor re-process a vertex.

**This topic supplies exactly two exploration disciplines and one greedy refinement:**
- **BFS** explores **level by level** (a queue) → gives **shortest paths in unweighted graphs**, components, bipartiteness.
- **DFS** explores **as deep as possible, then backtracks** (a stack/recursion) → gives **cycle detection, topological sort, edge classification**, components.
- **Dijkstra** upgrades shortest paths to **weighted graphs with non-negative weights** by greedily finalizing the closest vertex each step.

These reuse Topic 2's **queue and stack** and Topic 9's **adjacency-list** representation, and they are the workhorses behind countless higher-level methods (route finding, dependency resolution, state-space search in AI).

---

## 2. Prerequisites

- **Topic 9 (Graph Theory):** vertices/edges, directed vs undirected, **weighted** graphs, **adjacency list vs matrix**, path/walk/cycle, **distance**, **connected components**, **trees/spanning trees**. (BFS/DFS build a spanning tree of each component.)
- **Topic 2 (Stacks & Queues):** **queue (FIFO)** drives BFS; **stack / recursion (LIFO)** drives DFS.
- **Asymptotic analysis (Topics 6–8):** to read the $O(V+E)$, $O(V^2)$, $O(E\log V)$ costs.
- **Priority queue / min-heap (refresher here):** a structure supporting `insert`, `extract-min`, `decrease-key`, each in $O(\log n)$ for a binary heap. Dijkstra repeatedly asks for the **closest unfinalized vertex** — that is an extract-min.

---

## 3. Core idea — intuition first

**Marking is the whole game.** Both traversals do the same skeleton: start somewhere, look at a vertex's neighbours, and **mark vertices as visited** so you never enter the same vertex twice. The *only* difference is **which pending vertex you expand next**:

- **BFS — "ripples in a pond."** Drop a stone at the source; the wave reaches all distance-1 vertices, then all distance-2 vertices, and so on. You expand the **oldest** pending vertex first → a **queue**. Because the wave reaches things in order of distance, the first time you touch a vertex is along a **shortest** (fewest-edge) route.
- **DFS — "exploring a maze with a string."** Walk down a corridor as far as you can; when you hit a dead end, **backtrack** along your string to the last junction with an unexplored corridor. You expand the **newest** pending vertex first → a **stack** (or recursion). The order you *enter* and *leave* rooms (discovery/finish times) reveals the graph's structure — cycles, ancestry, a valid task order.
- **Dijkstra — "BFS for roads of different lengths."** Now edges have lengths. You can no longer trust hop-count, so instead you always **finalize the unfinalized vertex with the smallest known total distance**, then relax its outgoing roads (maybe shortening your estimate to its neighbours). As long as no road is negative, once a vertex is finalized its distance can never improve, so the greedy choice is safe.

Two intuitions GATE leans on:

1. **BFS = shortest path, but only when all edges count the same.** Add weights and BFS's hop-count is wrong; that is *why Dijkstra exists*.
2. **DFS timestamps are structure.** The pair (discovery, finish) tells you who is whose ancestor and whether an edge closes a loop.

---

## 4. Formal treatment

### 4.1 Breadth-First Search (BFS)

**Idea.** From source $s$, visit all vertices in increasing order of distance (number of edges) using a **FIFO queue**, recording each vertex's `level` (= distance from $s$) and `parent`.

```
BFS(G, s):
  for each v: visited[v] = false; dist[v] = ∞; parent[v] = NIL
  visited[s] = true; dist[s] = 0; enqueue(Q, s)
  while Q not empty:
     u = dequeue(Q)
     for each neighbour v of u:
        if not visited[v]:
           visited[v] = true
           dist[v] = dist[u] + 1
           parent[v] = u
           enqueue(Q, v)
```

**Facts (high-yield):**
- **Time:** $\Theta(V+E)$ with an **adjacency list**; $\Theta(V^2)$ with an **adjacency matrix** (scanning a full row per vertex). **Space:** $O(V)$ (queue + arrays).
- The `parent` pointers form a **BFS tree** of $s$'s component; its **tree edges** number $(\text{component size})-1$.
- **`dist[v]` is the shortest-path distance from $s$ in an *unweighted* graph** (this is *the* shortest-path use of BFS).
- **Undirected non-tree edges connect vertices whose levels differ by at most 1** (no edge skips a level). An edge within the *same* level ⇒ an **odd cycle** ⇒ **not bipartite** — so BFS gives a 2-colour bipartite test (colour = level parity).
- Restarting BFS from each unvisited vertex enumerates **connected components**; the number of restarts = number of components (undirected).

### 4.2 Depth-First Search (DFS)

**Idea.** Plunge deep along one path, backtrack at dead ends, using **recursion (an implicit stack)**. Stamp each vertex with a **discovery time `d[v]`** (when first seen, coloured gray) and a **finish time `f[v]`** (when fully explored, coloured black). White = undiscovered.

```
DFS(G):
  time = 0; for each v: color[v]=WHITE; parent[v]=NIL
  for each v in V:                     # restart → DFS forest
     if color[v]==WHITE: DFS-Visit(v)

DFS-Visit(u):
  time += 1; d[u] = time; color[u] = GRAY
  for each neighbour v of u:
     if color[v]==WHITE: parent[v]=u; DFS-Visit(v)   # tree edge
  color[u] = BLACK; time += 1; f[u] = time
```

**Facts (high-yield):**
- **Time:** $\Theta(V+E)$ (adjacency list); $\Theta(V^2)$ (matrix). **Space:** $O(V)$ recursion stack (worst case a path of all $V$ vertices).
- **Parenthesis theorem.** For any $u,v$, the intervals $[d[u],f[u]]$ and $[d[v],f[v]]$ are **either disjoint or nested** — they **never partially overlap**. One contains the other **iff** one vertex is an ancestor of the other in the DFS forest.
- **White-path theorem.** $v$ is a descendant of $u$ in the DFS forest **iff** at time $d[u]$ there is a path $u\!\rightsquigarrow\! v$ of entirely **white** vertices.

#### Edge classification (directed DFS) — when exploring edge $(u,v)$:

| Type | Condition on $v$ | Timestamp signature |
|---|---|---|
| **Tree** | $v$ is **white** | new tree edge |
| **Back** | $v$ is **gray** (ancestor) | $d[v] < d[u] < f[u] < f[v]$ |
| **Forward** | $v$ **black**, a descendant | $d[u] < d[v] < f[v] < f[u]$ |
| **Cross** | $v$ **black**, not related | $d[v] < f[v] < d[u] < f[u]$ |

> ⚠ **In an *undirected* graph, DFS produces only TREE and BACK edges — never forward or cross edges.** This is a classic GATE fact.

#### What DFS computes
- **Cycle detection.** A graph has a cycle **iff DFS finds a back edge**. (Directed: edge to a **gray** vertex. Undirected: edge to a visited vertex that is **not the immediate parent**.) Hence a **directed graph is a DAG ⇔ DFS yields no back edge.**
- **Topological sort (DAGs only).** A linear order of vertices so that every edge $(u,v)$ has $u$ before $v$. Two methods:
  - **DFS method:** output vertices in **decreasing finish time** $f[\cdot]$.
  - **Kahn's algorithm:** repeatedly remove a vertex of **in-degree 0** (queue-based); if fewer than $V$ vertices are output, a **cycle** exists.
  - A topological order **exists ⇔ the graph is a DAG**; it need **not be unique**. Cost $\Theta(V+E)$.
- **[lower priority]** **Strongly Connected Components (SCCs):** Kosaraju (two DFS passes: order by finish time, then DFS on the transpose $G^{T}$) or Tarjan — both $\Theta(V+E)$. **Articulation points / bridges:** one DFS with `low[]` values, $\Theta(V+E)$.

### 4.3 BFS vs DFS at a glance

| | **BFS** | **DFS** |
|---|---|---|
| Frontier structure | **Queue (FIFO)** | **Stack / recursion (LIFO)** |
| Visiting order | level by level (by distance) | deep then backtrack |
| Shortest path (unweighted)? | **Yes** (`dist` = #edges) | **No** |
| Natural extra output | levels, bipartite test | d/f times, cycles, topo-sort, SCC |
| Time / Space | $\Theta(V+E)$ / $O(V)$ | $\Theta(V+E)$ / $O(V)$ |

Both visit every vertex once and traverse every edge a constant number of times (each undirected edge appears in two adjacency lists), giving the shared $\Theta(V+E)$.

### 4.4 Shortest paths — the weighted problem

Let $w(u,v)$ be edge weights and $\delta(s,v)$ the minimum total weight from source $s$ to $v$.

- **Unweighted (or all weights equal):** use **BFS** → $\delta$ = number of edges, in $\Theta(V+E)$.
- **Non-negative weights:** use **Dijkstra**.
- **The relaxation step** (heart of every SSSP algorithm):
$$\textbf{if } d[u] + w(u,v) < d[v] \textbf{ then } d[v] \leftarrow d[u]+w(u,v),\ \ \text{pred}[v]\leftarrow u.$$
  Shortest paths obey the **triangle inequality** $\delta(s,v) \le \delta(s,u) + w(u,v)$, and relaxation only ever *decreases* an over-estimate toward $\delta$.

### 4.5 Dijkstra's algorithm

**Idea (greedy).** Keep a set $S$ of vertices whose shortest distance is **final**. Repeatedly pick the unfinalized vertex $u$ with the **smallest tentative `d[u]`** (an **extract-min** from a min-priority-queue), finalize it, and **relax** all its outgoing edges.

```
Dijkstra(G, w, s):
  for each v: d[v]=∞; pred[v]=NIL
  d[s]=0; build min-PQ keyed by d
  while PQ not empty:
     u = extract-min(PQ)            # finalize u
     for each edge (u,v):
        if d[u] + w(u,v) < d[v]:    # relax
           d[v] = d[u] + w(u,v); pred[v] = u; decrease-key(PQ, v)
```

> **Required condition: all edge weights are non-negative.** With a negative edge the greedy "finalize the current minimum" can be wrong (see §5, Problem 6).

**Why it is correct (1-line):** when $u$ is extracted, every *other* unfinalized vertex already has tentative distance $\ge d[u]$; since all remaining edges are non-negative, no future path can reach $u$ more cheaply, so $d[u]=\delta(s,u)$ is final.

**Complexity (memorize the table):**

| Min-PQ implementation | Time | Best when |
|---|---|---|
| Array / linear scan for min | $\Theta(V^2)$ | **Dense** graphs ($E\approx V^2$) |
| **Binary heap** + adjacency list | $\Theta((V+E)\log V) = O(E\log V)$ | **Sparse** graphs |
| Fibonacci heap | $\Theta(E + V\log V)$ | theoretical optimum |

⚠ **Density tradeoff (GATE favourite):** for dense graphs, array-Dijkstra $\Theta(V^2)$ beats binary-heap $\Theta(V^2\log V)$; for sparse graphs the heap wins. There is no single "fastest" answer — it depends on $E$ vs $V$.

- Dijkstra produces a **shortest-path tree** (via `pred`); follow `pred` back from $v$ to recover the actual path.

### 4.6 Other shortest-path facts **[lower priority / context]**

- **Negative edges (no negative cycle):** **Bellman–Ford**, $\Theta(VE)$. It relaxes *all* edges $V-1$ times; a successful relaxation on a $V$-th pass **detects a negative-weight cycle** (then "shortest path" is undefined, $-\infty$).
- **DAG shortest path:** relax edges in **topological order** → $\Theta(V+E)$, and it **handles negative weights** (a DAG has no cycles).
- **All-pairs shortest paths:** **Floyd–Warshall** (dynamic programming, $\Theta(V^3)$, allows negative edges, detects negative cycles via a negative diagonal).

---

## 5. Worked problems (GATE-style)

Use the convention that **neighbours are visited in increasing vertex order** unless stated otherwise (GATE usually fixes this).

### Problem 1 — BFS order and distances (NAT/MCQ)
*Undirected graph, vertices $1..6$, edges: $1\text-2,\ 1\text-3,\ 2\text-4,\ 3\text-4,\ 4\text-5,\ 3\text-6$. Run BFS from vertex 1. Give the visiting order and `dist[5]`.*

Queue evolution (neighbours in increasing order):
- Visit **1** (dist 0); enqueue 2, 3.
- Visit **2** (dist 1); enqueue 4.
- Visit **3** (dist 1); enqueue 6 (4 already queued).
- Visit **4** (dist 2); enqueue 5.
- Visit **6** (dist 2).
- Visit **5** (dist 3).

**Order:** $1,2,3,4,6,5$. **`dist[5] = 3`** (path $1\!-\!2\!-\!4\!-\!5$ or $1\!-\!3\!-\!4\!-\!5$, both length 3). Tree edges: $1\text-2,1\text-3,2\text-4,3\text-6,4\text-5$ (5 edges $= n-1$).

### Problem 2 — DFS timestamps & edge type (MCQ)
*Directed graph, edges: $1\!\to\!2,\ 1\!\to\!3,\ 2\!\to\!4,\ 4\!\to\!1,\ 3\!\to\!4$. Run DFS from 1 (neighbours increasing). Classify edge $4\!\to\!1$, and state whether the graph is a DAG.*

Trace (`time` increments on discovery and on finish):
- d[1]=1 → d[2]=2 → d[4]=3. From 4 explore edge $4\!\to\!1$: vertex 1 is **gray** (still on the stack) ⇒ **back edge**. 4 has no other unvisited neighbour ⇒ f[4]=4. Back up: f[2]=5.
- Back at 1, explore $1\!\to\!3$: d[3]=6 → edge $3\!\to\!4$: 4 is **black**, and $d[3]=6>d[4]=3$ ⇒ **cross edge**. f[3]=7. f[1]=8.

$4\!\to\!1$ is a **back edge** ⇒ the graph **contains a cycle** ($1\!\to\!2\!\to\!4\!\to\!1$) ⇒ **not a DAG**, so **no topological order exists**.

### Problem 3 — Dijkstra trace (NAT)
*Undirected weighted graph, source $A$. Edges: $A\text-B=4,\ A\text-C=1,\ C\text-B=2,\ C\text-D=5,\ B\text-D=1,\ B\text-E=7,\ D\text-E=3$. Find all shortest distances from $A$ and the finalization order.*

| Step | Finalize | $d[B]$ | $d[C]$ | $d[D]$ | $d[E]$ | comment |
|---|---|---|---|---|---|---|
| init | — | 4 | 1 | ∞ | ∞ | from $A$: $B{=}4,\ C{=}1$ |
| 1 | **C (1)** | $\min(4,1{+}2){=}3$ | — | $1{+}5{=}6$ | ∞ | relax via C |
| 2 | **B (3)** | — | — | $\min(6,3{+}1){=}4$ | $3{+}7{=}10$ | relax via B |
| 3 | **D (4)** | — | — | — | $\min(10,4{+}3){=}7$ | relax via D |
| 4 | **E (7)** | — | — | — | — | done |

**Finalization order:** $A, C, B, D, E$. **Distances:** $A{=}0,\ C{=}1,\ B{=}3,\ D{=}4,\ E{=}7$. (Note $B$ improved $4\to3$ and $E$ improved $10\to7$ via relaxation — exactly what GATE tests.)

### Problem 4 — Counting topological orders (NAT)
*DAG with edges $1\!\to\!2,\ 1\!\to\!3,\ 2\!\to\!4,\ 3\!\to\!4$. How many distinct topological orderings exist?*

Vertex 1 must be first; vertex 4 must be last; 2 and 3 are mutually independent and both lie strictly between. So the orders are $1,2,3,4$ and $1,3,2,4$ → **2 orderings**.

### Problem 5 — Complexity selection **(trap, MCQ)**
*You must run single-source shortest paths on a graph with $V=1000$ vertices and $E\approx 500{,}000$ edges (dense, non-negative weights). Which is asymptotically faster, **array-Dijkstra** $\Theta(V^2)$ or **binary-heap-Dijkstra** $\Theta(E\log V)$?*

$\Theta(V^2)=10^6$. $\Theta(E\log V)=5\times10^5\times\log_2 1000 \approx 5\times10^5\times10 = 5\times10^6$. So the **array version is faster here** — the graph is dense ($E$ near $V^2/2$). *Trap:* "heap is always faster" is false; the heap only wins for **sparse** graphs.

### Problem 6 — Dijkstra fails on negative edges **(trap, MSQ)**
*Directed graph, source $s$, edges $s\!\to\!a=2,\ s\!\to\!b=3,\ b\!\to\!a=-2$. What does Dijkstra report for $d[a]$, and what is the true shortest distance?*

Dijkstra: extract $s$ (0), relax → $d[a]=2,\ d[b]=3$. Extract $a$ (min $=2$) and **finalize** it. Later extract $b$ and relax $b\!\to\!a$: $3+(-2)=1<2$ — but $a$ is **already finalized**, so Dijkstra keeps the wrong $d[a]=2$. The **true** shortest path is $s\!\to\!b\!\to\!a = 1$. ⇒ **Dijkstra gives 2 (wrong); correct is 1.** Since there is no negative cycle, **Bellman–Ford** would correctly return 1.

### Problem 7 — Which DFS edges can appear? **(concept MSQ)**
*True/False: In a DFS of an **undirected** connected graph, a cross edge can occur.*

**False.** Undirected DFS yields only **tree** and **back** edges; forward and cross edges are impossible. (Any non-tree edge necessarily reaches an ancestor.)

---

## 6. Common GATE question types on this topic

- **BFS/DFS order:** "which of the following is a valid BFS/DFS sequence from vertex $x$?" or list the order given a neighbour-ordering rule; compute `dist[]`/levels.
- **DFS timestamps:** assign $d[v], f[v]$; apply the **parenthesis** / **white-path** theorems; classify a named edge as tree/back/forward/cross.
- **Edge-type facts:** "undirected DFS has only tree & back edges"; "a back edge exists ⇔ there is a cycle"; "no back edge ⇔ DAG."
- **Topological sort:** produce a valid order, **count** the valid orders, or decide if one exists (DAG test).
- **Dijkstra trace:** finalization order, final distances, the value of a label after $k$ relaxations; recover the shortest-path tree.
- **Complexity:** $\Theta(V+E)$ vs $\Theta(V^2)$ for BFS/DFS by representation; **array vs heap Dijkstra** density tradeoff; $\Theta(VE)$ Bellman–Ford; $\Theta(V^3)$ Floyd–Warshall.
- **Algorithm choice:** unweighted → BFS; non-negative weights → Dijkstra; negative edges → Bellman–Ford; DAG → topological relaxation; all-pairs → Floyd–Warshall.
- **Bipartite check** via BFS level parity; **components** via restart count.
- **[lower priority]** SCC counting; articulation points / bridges.

---

## 7. Traps, pitfalls & common mistakes

- **BFS gives shortest paths only for *unweighted* graphs.** Apply it to a weighted graph and the hop-count is wrong — that's why Dijkstra exists.
- **Dijkstra requires non-negative weights.** A single negative edge can break it (Problem 6). Do not "fix" it by adding a constant to all edges — that changes which path is shortest (longer paths gain more).
- **Undirected DFS ⇒ only tree & back edges.** Claiming a forward/cross edge in an undirected graph loses marks.
- **Cycle test in undirected DFS ignores the parent.** An edge back to the *immediate* parent is the tree edge itself, not a cycle; a back edge to *any other* visited vertex signals a cycle.
- **Topological sort needs a DAG.** If there's a cycle, no valid order exists; Kahn's algorithm outputs fewer than $V$ vertices.
- **Traversal/topological orders are not unique** — they depend on start vertex and neighbour ordering. Don't assume "the" order.
- **Complexity by representation:** BFS/DFS are $\Theta(V+E)$ on adjacency **lists** but $\Theta(V^2)$ on the **matrix** (you scan a full row per vertex).
- **"Heap Dijkstra is always faster" is false.** For dense graphs, array $\Theta(V^2)$ beats heap $\Theta(V^2\log V)$ (Problem 5).
- **Recursion-stack space.** DFS uses $O(V)$ stack in the worst case (a long path); deep graphs can overflow a naive recursive implementation.
- **Levels skip / don't skip:** in *undirected* BFS, a non-tree edge connects vertices of **equal or adjacent** levels only — never two levels apart.
- **Negative cycle ⇒ no shortest path.** Distances become $-\infty$; only algorithms that *detect* it (Bellman–Ford $V$-th pass, Floyd–Warshall negative diagonal) report correctly. Dijkstra/BFS are not applicable.
- **`extract-min` finalizes — don't relax a finalized vertex.** Re-opening a black vertex is the exact error a negative edge would force.

---

## 8. Connection forward

This completes the **Graphs** section and the **entire DSA subject**. You now have, end to end, the data-structure toolkit GATE DA expects: linear structures and Python (Topics 1–3), trees and hashing (Topics 4–5), searching, sorting and divide-and-conquer (Topics 6–8), and now graph modelling plus the core graph algorithms (Topics 9–10).

These algorithms are not just exam items — they reappear across the rest of GATE DA: **BFS/DFS underlie state-space search in the AI section**; **topological sort models task/dependency scheduling**; **shortest-path and graph-distance ideas surface in graph-based ML and network analysis**; and the **queue/stack/heap reasoning** transfers directly to algorithm-design questions elsewhere. With the DSA subject closed, the natural next move is to consolidate cross-subject links (e.g., the **linear-algebra** view of the adjacency matrix, or **probability** on graphs) and drill mixed past-year sets.

---

## 9. One-line cheat-sheet summary

**BFS (queue, $\Theta(V+E)$) = level order + unweighted shortest paths + bipartite test; DFS (stack, $\Theta(V+E)$) = discovery/finish times → cycles (back edge), DAG test, topological sort, and *undirected ⇒ only tree+back edges*; Dijkstra (greedy, non-negative weights only) finalizes the min-tentative vertex and relaxes, costing $\Theta(V^2)$ with an array (dense) or $\Theta(E\log V)$ with a heap (sparse) — negative edges need Bellman–Ford $\Theta(VE)$.**

---

✅ **This completes the Programming, Data Structures and Algorithms subject (Topics 1–10).** Logical next step: review cross-subject bridges and attempt mixed GATE DA past-year problem sets, or begin the next subject in your plan.

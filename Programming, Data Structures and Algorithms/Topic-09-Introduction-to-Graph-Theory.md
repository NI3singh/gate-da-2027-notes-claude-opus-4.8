# Topic 9 — Introduction to Graph Theory

*Subject: Programming, Data Structures and Algorithms (GATE DA)*

---

## 0. Topic header & syllabus map

- **Topic:** Introduction to Graph Theory
- **Subject:** Programming, Data Structures and Algorithms
- **Where it sits:** This is the **first** topic of the *Graphs* section. It comes **after** Divide & Conquer / Searching & Sorting (Topics 6–8), which all worked on *linear* data, and it leads directly into **Topic 10: Graph Traversals (BFS, DFS) & Shortest Path (Dijkstra)**. This topic builds the vocabulary, structures, and representations; Topic 10 runs algorithms on them.

> **Scope note for GATE DA.** Graph theory in the DA syllabus lives under *Data Structures & Algorithms*, so the **representations, degree/connectivity facts, trees, and counting arguments** are the high-value core. A few classic discrete-math results (Euler/Hamiltonian, planarity, coloring, Cayley's formula) are included because they recur in GATE CS and are cheap to learn — they are marked **[lower priority for DA]** so you can triage on the last day.

---

## 1. Why this topic exists (motivation + connection)

A huge class of real problems is about **relationships between things**, not about a single ordered list of things: cities joined by roads, web pages joined by links, people joined by friendships, courses joined by prerequisites, variables joined by dependencies. The essential data is *which pairs are connected* (and possibly *how strongly*).

**What the previous topics couldn't do.** Arrays, stacks, queues, and linked lists (Topics 1–3) model data laid out in a **line**; sorting and searching (Topics 6–8) operate on such linear collections. Trees (Topic 4) were our first *branching* structure, but a tree is a very **restricted** shape: it is connected, has exactly one path between any two nodes, has no cycles, and has a single root with a parent→child hierarchy. Real networks break all of those rules — there can be many routes between two cities, loops in a road map, no natural "root," and disconnected pieces. None of the earlier structures can store "any item may relate to any other item, possibly in a loop."

**Graphs generalize all of this.** A graph drops every tree restriction: any pair of vertices may be joined, cycles are allowed, multiple paths are allowed, and the whole thing may even be in several disconnected pieces. (In fact a **tree is just a special graph** — connected and acyclic — which is the precise bridge from Topic 4.) Once we can *represent* such networks, **Topic 10** will systematically explore them (BFS/DFS) and find best routes through them (shortest paths).

---

## 2. Prerequisites

- **Trees (Topic 4)** — a tree is the special "connected + acyclic" graph; many tree facts reappear as graph facts.
- **Arrays & linked lists (Topics 1, 3)** — used to build the two standard graph representations (adjacency matrix vs. adjacency list).
- **Stacks & queues (Topic 2)** — not needed *here*, but they power the traversals in Topic 10; keep them fresh.
- **Basic counting & summation** — handshaking, edge counts, and "is this degree sequence possible?" are pure counting arguments.
- **Sets & relations, matrices** — a graph is essentially a binary relation on a vertex set; the adjacency matrix is that relation written as a 0/1 matrix, and we will multiply such matrices.

---

## 3. Core idea — intuition first

Picture **dots and lines**. Each dot is a **vertex** (node); each line joining two dots is an **edge**. That's the whole object.

- **Cities-and-roads analogy.** Vertices = cities, edges = direct roads. If a road runs both ways, the edge is **undirected**. If it's one-way, the edge has an arrow — a **directed** edge. If roads have lengths/tolls, the edges carry numbers — a **weighted** graph.
- The **degree** of a city is just *how many roads touch it*. A city with many roads is "high degree."
- A **path** is a way to drive from city A to city B without revisiting a city. A **cycle** is a round trip that comes back to where it started.
- The map is **connected** if you can drive from every city to every other; if it splits into separate clusters of cities with no road between clusters, each cluster is a **connected component**.

Two warnings your intuition must absorb early, because GATE exploits them:

1. **A picture is not the graph.** The same graph can be drawn in many ways; what matters is *which pairs are joined*, not the geometry. Two differently-drawn graphs that join the same pairs are the **same** graph (isomorphic).
2. **Direction changes everything.** In an undirected graph "A connected to B" automatically means "B connected to A." In a directed graph it does **not** — a one-way street from A to B says nothing about getting back.

Everything formal below is just making "dots, lines, arrows, weights, reachability" precise enough to compute with.

---

## 4. Formal treatment

### 4.1 Graphs, and the main types

> **Definition (graph).** A graph is a pair $G = (V, E)$ where $V$ is a finite non-empty set of **vertices** and $E$ is a set of **edges**. Let $n = |V|$ and $m = |E|$.

- **Undirected graph:** each edge is an *unordered* pair $\{u, v\}$.
- **Directed graph (digraph):** each edge is an *ordered* pair $(u, v)$ — an arrow from tail $u$ to head $v$.
- **Weighted graph:** a function $w: E \to \mathbb{R}$ assigns a weight/cost to each edge.
- **Simple graph:** no **self-loops** (an edge from a vertex to itself) and no **parallel/multiple edges** (two edges joining the same pair). *Unless stated otherwise, "graph" in GATE means simple.*
- **Multigraph:** parallel edges allowed. **Pseudograph:** self-loops *and* parallel edges allowed.
- $u, v$ are **adjacent** (neighbours) if joined by an edge; that edge is **incident** to $u$ and to $v$.

### 4.2 Degree and the Handshaking Lemma

> **Degree** $\deg(v)$ = number of edge-endpoints at $v$. A **self-loop adds 2** to the degree. A vertex of degree 0 is **isolated**; degree 1 is a **leaf/pendant**.
> For a **directed** graph: **in-degree** $\deg^-(v)$ = arrows entering, **out-degree** $\deg^+(v)$ = arrows leaving.

$$\boxed{\sum_{v \in V} \deg(v) = 2m} \quad\text{(Handshaking Lemma, undirected)}$$

*Condition:* any undirected graph (loops counted twice). *Why:* each edge has exactly two endpoints, so summing degrees counts every edge twice.

**Corollary (parity):** the number of **odd-degree** vertices is **even**. *(The total $2m$ is even; even-degree vertices contribute an even amount, so the odd-degree vertices' contribution must also be even, forcing an even count of them.)*

$$\boxed{\sum_{v} \deg^{+}(v) = \sum_{v} \deg^{-}(v) = m} \quad\text{(directed)}$$

⚠ For digraphs the total is $m$, **not** $2m$ — a very common GATE trap.

- A graph is **$k$-regular** if every vertex has degree $k$. Then $\sum \deg = nk = 2m \Rightarrow m = nk/2$, so **$nk$ must be even** (no 3-regular graph on 5 vertices, etc.).
- $\Delta(G)$ = maximum degree, $\delta(G)$ = minimum degree.

### 4.3 Walks, trails, paths, cycles — *get these exact*

| Term | Vertices may repeat? | Edges may repeat? | Closed version |
|---|---|---|---|
| **Walk** | yes | yes | closed walk (start = end) |
| **Trail** | yes | **no** | **circuit / closed trail** |
| **Path** | **no** | no | **cycle** (only start = end repeats) |

- **Length** of a walk = number of edges in it.
- A **cycle** in a simple graph has length $\ge 3$. (Length 1 needs a self-loop; length 2 needs parallel edges.)
- **Distance** $d(u,v)$ = length of a shortest path between $u$ and $v$ (this is the quantity Topic 10 computes).
- An **acyclic** graph has no cycles.

### 4.4 Connectivity and components

- **Connected (undirected):** there is a path between every pair of vertices.
- **Connected component:** a maximal connected piece. The number of components is a key quantity; GATE often asks how it changes when edges/vertices are deleted.
- **Cut vertex (articulation point):** a vertex whose removal increases the number of components. **Bridge (cut edge):** an edge whose removal does so. *(DFS finds these efficiently — Topic 10.)*
- **Connectivity bounds (state, occasionally used):** vertex-connectivity $\kappa(G) \le$ edge-connectivity $\lambda(G) \le \delta(G)$.

**Directed graphs:**
- **Strongly connected:** for every ordered pair $(u,v)$ there is a directed path $u \to v$ *and* $v \to u$. The maximal such pieces are **strongly connected components (SCCs)**.
- **Weakly connected:** the underlying undirected graph (forget arrow directions) is connected.

### 4.5 Edge-count facts (high-yield)

For a **simple** graph on $n$ vertices:

$$
\boxed{0 \le m \le \binom{n}{2} = \frac{n(n-1)}{2}} \text{ (undirected)},
\qquad
\boxed{0 \le m \le n(n-1)} \text{ (directed, no loops)}
$$

- **Minimum edges to be connected:** $n - 1$ (achieved by a tree).
- **A forest** (acyclic, possibly disconnected) with $n$ vertices and $k$ components has exactly $n - k$ edges.
- **Maximum edges while still *disconnected*** (simple, $n$ vertices): leave one vertex isolated and make the rest complete:
$$\boxed{\binom{n-1}{2} = \frac{(n-1)(n-2)}{2}}$$
  Hence **any simple graph with more than $\binom{n-1}{2}$ edges is connected** — i.e. $\binom{n-1}{2}+1$ edges *force* connectivity. (Classic trap; see §5.)
- **Number of distinct labelled simple graphs** on $n$ vertices: $2^{\binom{n}{2}}$ (each possible edge present or absent).

### 4.6 Special graphs you must recognize

- **Complete graph $K_n$:** every pair joined. $m = \binom{n}{2}$, every vertex has degree $n-1$, it is $(n-1)$-regular.
- **Path graph $P_n$** and **cycle graph $C_n$** ($n$ vertices, $n$ edges, 2-regular).
- **Bipartite graph:** $V$ splits into two sets $X, Y$ with **every edge between $X$ and $Y$** (none inside a set).
  - **Complete bipartite $K_{m,n}$:** all cross-edges present, $|E| = mn$.
  - **Key theorem:** $G$ is **bipartite $\iff$ $G$ has no odd-length cycle $\iff$ $G$ is 2-colourable.**
- **Regular / wheel / star** graphs — recognize by definition.
- **Complement $\overline{G}$:** same vertices, edge present in $\overline{G}$ iff absent in $G$. Then $m(G) + m(\overline G) = \binom{n}{2}$.
- **Tree / forest:** see §4.8.

### 4.7 Subgraphs

- **Subgraph:** $V' \subseteq V$, $E' \subseteq E$ (endpoints inside $V'$).
- **Spanning subgraph:** uses *all* vertices ($V' = V$).
- **Induced subgraph $G[V']$:** take $V'$ and **all** original edges with both endpoints in $V'$.
- **Spanning tree:** a spanning subgraph that is a tree (connects all vertices with no cycle). Every connected graph has at least one.

### 4.8 Trees as special graphs (bridge from Topic 4)

> **Tree:** a connected acyclic undirected graph. A **forest** is an acyclic graph (each component a tree).

**Equivalent characterizations** — for a graph on $n$ vertices, *any two* of the following three imply the third (and imply "tree"):
1. **connected**, 2. **acyclic**, 3. **has exactly $n-1$ edges**.

Other tree facts (all GATE-relevant):
- A tree has a **unique path** between any two vertices.
- A tree with $n \ge 2$ vertices has **at least two leaves**.
- **Adding any one edge** to a tree creates **exactly one** cycle; **removing any one edge** disconnects it into two components.
- **Cayley's formula [lower priority for DA]:** the number of distinct **labelled** trees on $n$ vertices is
$$\boxed{n^{\,n-2}}$$
  which equals the number of **spanning trees of $K_n$**.

### 4.9 Representations — *the DSA heart of this topic*

Let $n=|V|$, $m=|E|$.

**(a) Adjacency matrix $A$** — an $n\times n$ matrix with $A_{ij} = 1$ if edge $i\!\to\! j$ exists (or the weight, if weighted), else $0$.
- **Symmetric** for undirected graphs; not necessarily for directed.
- **Space:** $\Theta(n^2)$ — wasteful for sparse graphs.
- "Is $i$ adjacent to $j$?" in $O(1)$; "list all neighbours of $i$" in $O(n)$.
- **Walk-counting (high-yield):**
$$\boxed{(A^k)_{ij} = \text{number of walks of length } k \text{ from } i \text{ to } j}$$
  *Why:* $(A^2)_{ij}=\sum_t A_{it}A_{tj}$ counts intermediate vertices $t$ adjacent to both — i.e. length-2 walks; induction gives general $k$. Consequences for a simple undirected graph: $(A^2)_{ii} = \deg(i)$, and the **number of triangles** $= \dfrac{1}{6}\,\mathrm{trace}(A^3)$ (each triangle is counted by 3 starting points × 2 directions = 6 closed 3-walks).

**(b) Adjacency list** — for each vertex, a list of its neighbours.
- **Space:** $\Theta(n + m)$ — efficient for sparse graphs (the usual real-world case).
- "List neighbours of $i$" in $O(\deg i)$; "is $i$ adjacent to $j$?" costs $O(\deg i)$.

**(c) Incidence matrix** — $n \times m$, entry marks whether a vertex is an endpoint of an edge. Rarely needed for GATE DA algorithms; know it exists.

> **Rule of thumb:** dense graph or many "is there an edge?" queries → **matrix**; sparse graph or you mostly **scan neighbours** (BFS/DFS) → **list**. Topic 10's $O(V+E)$ traversals assume the **adjacency-list** representation.

### 4.10 Degree sequences & graphical sequences

The **degree sequence** is the list of vertex degrees (usually sorted descending). A sequence is **graphical** if some simple graph realizes it.

- **Necessary:** the sum is even (handshaking), and no term exceeds $n-1$.
- **Havel–Hakimi test (constructive):** sort descending; remove the largest value $d$; subtract $1$ from the next $d$ entries; repeat. The original is graphical **iff** this process reaches all-zeros without ever producing a negative entry. *(Erdős–Gallai gives an inequality-based alternative.)*

### 4.11 Euler vs. Hamiltonian **[lower priority for DA]**

- **Euler trail/circuit** = uses **every edge exactly once** (vertices may repeat).
  - Connected undirected graph has an **Euler circuit $\iff$ every vertex has even degree**.
  - Has an **Euler trail (not circuit) $\iff$ exactly two vertices have odd degree** (they are the endpoints).
  - Directed (ignoring isolated vertices, underlying graph connected): **Euler circuit $\iff \deg^+(v)=\deg^-(v)$ for all $v$**; Euler trail $\iff$ exactly one vertex with $\deg^+-\deg^-=+1$ (start) and one with $-1$ (end), rest equal.
- **Hamiltonian path/cycle** = visits **every vertex exactly once**. **No simple necessary-and-sufficient condition; deciding it is NP-complete.** Only *sufficient* conditions:
  - **Dirac:** if $n \ge 3$ and $\delta(G) \ge n/2$, a Hamiltonian cycle exists.
  - **Ore:** if $\deg(u)+\deg(v) \ge n$ for every non-adjacent pair $u,v$, a Hamiltonian cycle exists.
  - These are sufficient, **not** necessary (a graph can be Hamiltonian without meeting them).

### 4.12 Planarity & coloring **[lower priority for DA]**

- **Planar** = drawable in the plane with no edge crossings. **Euler's formula** for a connected planar graph: $V - E + F = 2$ ($F$ = faces, including the outer one).
- Edge bounds for **simple connected planar** graphs with $V \ge 3$: $E \le 3V - 6$; if also **bipartite/triangle-free**, $E \le 2V - 4$.
- **$K_5$** ($V=5,E=10 > 9$) and **$K_{3,3}$** ($V=6,E=9>8$, bipartite) are the two basic **non-planar** graphs (Kuratowski: a graph is planar iff it contains no subdivision of $K_5$ or $K_{3,3}$).
- **Chromatic number $\chi(G)$** = fewest colours to colour vertices so adjacent ones differ. $\chi(K_n)=n$; $\chi(\text{tree})=2$ (if $\ge 2$ vertices); $\chi(C_n)=2$ if $n$ even, $3$ if $n$ odd; **bipartite $\iff \chi \le 2$**. Bounds: $\chi(G) \ge \omega(G)$ (clique number) and $\chi(G) \le \Delta(G)+1$. **Four-colour theorem:** every planar graph has $\chi \le 4$.

### 4.13 Isomorphism (recognition)

> $G_1 \cong G_2$ if there is a bijection on vertices preserving adjacency.

**Invariants** (must match if isomorphic): $|V|$, $|E|$, **degree sequence**, number of components, cycle lengths, bipartiteness, connectivity.
⚠ **Equal degree sequence is necessary but NOT sufficient** — two graphs can share a degree sequence yet not be isomorphic. (Common trap.)

---

## 5. Worked problems (GATE-style)

### Problem 1 — Handshaking (NAT)
*A simple undirected graph has 21 edges, three vertices of degree 4, and every other vertex of degree 3. How many vertices does it have?*

$\sum \deg = 2m = 42$. Let $n$ be the number of vertices; three contribute $3\times 4 = 12$, the remaining $(n-3)$ contribute $3$ each:
$$12 + 3(n-3) = 42 \Rightarrow 3(n-3)=30 \Rightarrow n-3=10 \Rightarrow \boxed{n=13}.$$

### Problem 2 — Max edges / forced connectivity **(trap, MCQ)**
*Let $G$ be a simple graph on $10$ vertices. (a) What is the maximum number of edges if $G$ is disconnected? (b) What is the smallest $m$ that guarantees $G$ is connected?*

(a) Disconnected-max = isolate one vertex, complete the other $9$:
$$\binom{9}{2} = 36 \text{ edges.}$$
(b) With $36$ edges a graph *can still be* disconnected, so $36$ does **not** guarantee connectivity. With **$37$** edges it is impossible to keep one piece separate, so $G$ must be connected.
$$\boxed{(a)\;36 \qquad (b)\;37}$$
*Trap:* the naive "min edges to connect = $n-1 = 9$" answers a *different* question (that's the minimum a connected graph can have, not the count that **forces** connectivity).

### Problem 3 — Adjacency-matrix walks (NAT)
*For the simple undirected graph on $\{1,2,3,4\}$ with edges $\{1\text-2, 2\text-3, 3\text-4, 4\text-1, 1\text-3\}$, how many walks of length 2 go from vertex 2 to vertex 4?*

$$A=\begin{pmatrix}0&1&1&1\\1&0&1&0\\1&1&0&1\\1&0&1&0\end{pmatrix},\quad
(A^2)_{24}=\sum_t A_{2t}A_{t4}=A_{21}A_{14}+A_{23}A_{34}=1\cdot1+1\cdot1=\boxed{2}.$$
(The two length-2 walks are $2\!\to\!1\!\to\!4$ and $2\!\to\!3\!\to\!4$.) *Check:* $(A^2)_{22}=\deg(2)=2.$

### Problem 4 — Is the sequence graphical? (MSQ-style reasoning)
*Is $(4,4,3,3,3,3)$ a valid degree sequence of a simple graph?*

Sum $=20$ (even ✓), max $=4 \le n-1 = 5$ ✓. Apply **Havel–Hakimi**:
$$(4,4,3,3,3,3)\xrightarrow{\text{drop }4,\ -1\text{ to next }4}(3,2,2,2,3)\to_{\text{sort}}(3,3,2,2,2)$$
$$\xrightarrow{\text{drop }3}(2,1,1,2)\to(2,2,1,1)\xrightarrow{\text{drop }2}(1,0,1)\to(1,1,0)\xrightarrow{\text{drop }1}(0,0).$$
No negative ever appeared and we reached all-zeros, so the sequence **is graphical**.

### Problem 5 — Forest counting (NAT)
*A forest has 62 vertices and 51 edges. How many trees (components) does it contain?*

For a forest, $\text{edges} = n - k$, so $k = n - \text{edges} = 62 - 51 = \boxed{11}$.

### Problem 6 — Euler circuit (MCQ) **[lower priority]**
*Does $K_5$ have an Euler circuit? Does $K_4$?*

$K_5$: every vertex has degree $4$ (even) and the graph is connected $\Rightarrow$ **Euler circuit exists**.
$K_4$: every vertex has degree $3$ (odd) — that's **four** odd-degree vertices ($>2$) $\Rightarrow$ **no Euler trail or circuit**.

### Problem 7 — "$n-1$ edges $\Rightarrow$ tree?" **(trap, True/False)**
*A simple graph has $n$ vertices and exactly $n-1$ edges. Must it be a tree?*

**No.** $n-1$ edges guarantees a tree only **with** connectivity (or acyclicity). A counterexample on $n=4$: edges $\{1\text-2, 2\text-3, 1\text-3\}$ form a triangle on $\{1,2,3\}$ (a cycle) plus an **isolated** vertex $4$ — that's $3 = n-1$ edges but it is **disconnected and contains a cycle**, hence not a tree. *Rule to memorize:* any **two** of {connected, acyclic, exactly $n-1$ edges} give a tree; one alone does not.

---

## 6. Common GATE question types on this topic

- **Handshaking arithmetic:** find $n$, $m$, or a missing degree from $\sum\deg = 2m$; parity ("can a graph have an odd number of odd-degree vertices?" → no).
- **Edge-count bounds:** max edges in simple/disconnected/bipartite graphs; minimum edges to connect; the "$> \binom{n-1}{2} \Rightarrow$ connected" trap.
- **Trees & forests:** edges $= n-k$; spanning-tree existence; Cayley's $n^{n-2}$; "which of these statements forces a tree?".
- **Adjacency-matrix powers:** number of length-$k$ walks $=(A^k)_{ij}$; triangle count $=\tfrac16\mathrm{trace}(A^3)$; symmetry/diagonal facts.
- **Representation cost:** space $\Theta(n^2)$ vs $\Theta(n+m)$; query-time comparisons (which is faster for "scan neighbours" vs "edge exists?").
- **Degree sequences:** "is this sequence graphical?" (Havel–Hakimi / parity / max-degree checks).
- **Connectivity counting:** number of components after deleting given edges/vertices; identifying cut vertices/bridges.
- **Bipartite / odd-cycle / 2-colourability** equivalences.
- **Isomorphism:** decide via invariants; spot the "same degree sequence but not isomorphic" trap.
- **[lower priority]** Euler/Hamiltonian conditions; planarity ($E \le 3V-6$, $K_5/K_{3,3}$); chromatic numbers of standard graphs.

---

## 7. Traps, pitfalls & common mistakes

- **Directed degree sum is $m$, not $2m$.** $\sum\deg^+=\sum\deg^-=m$. Don't blindly apply the handshaking $2m$ to digraphs.
- **Self-loops add 2** to a vertex's degree (one for each endpoint), not 1.
- **Walk ≠ trail ≠ path.** Repetition rules differ. "Shortest *path*" forbids vertex repeats; a generic "walk count" via $A^k$ allows repeats.
- **"$n-1$ edges" alone ≠ tree**, and **"acyclic" alone ≠ tree** (a forest is acyclic but may be disconnected). You need *two* of the three properties.
- **Disconnected-max edges** is $\binom{n-1}{2}$, *not* $\binom{n}{2}-1$. Removing one edge from $K_n$ keeps it connected.
- **Equal degree sequence does not imply isomorphic.** It's only a necessary check.
- **$k$-regular needs $nk$ even.** No 3-regular graph on an odd number of vertices, etc.
- **Bipartite test = no *odd* cycle.** Even cycles are fine; the parity is what matters.
- **Cycle length $\ge 3$ in a simple graph.** A single repeated edge is *not* a cycle there.
- **Adjacency matrix of an undirected graph is symmetric**; forgetting this when counting/transposing loses marks. Also $(A^2)_{ii}=\deg(i)$ only for simple graphs (no loops/parallels).
- **Dirac/Ore are sufficient, not necessary**, and **Hamiltonicity is hard** — don't invent an "if-and-only-if degree condition" for Hamiltonian cycles.
- **Strong vs weak connectivity** (directed): "you can reach everything" must hold **both** directions for *strong* connectivity.

---

## 8. Connection forward

You now have the **objects** (vertices, edges, directed/weighted), the **shapes** (trees, bipartite, complete, components), and the **storage** (adjacency list / matrix) that every graph algorithm needs. **Topic 10 — Graph Traversals & Shortest Path** takes exactly this: BFS and DFS *systematically visit* the vertices using a **queue** and a **stack** (Topic 2) over the **adjacency-list** representation (this topic), running in $\Theta(V+E)$; BFS also yields shortest paths in **unweighted** graphs, and **Dijkstra** extends shortest paths to **weighted** graphs with non-negative weights. The "distance," "connectivity," and "weight" notions defined here become the things those algorithms compute.

---

## 9. One-line cheat-sheet summary

**A graph $(V,E)$ models pairwise relations: $\sum\deg=2m$ (undirected) / $\sum\deg^\pm=m$ (directed); simple-graph edges $\le\binom n2$; trees = connected + acyclic = $n-1$ edges (any two imply the third); store as $\Theta(n^2)$ matrix (with $(A^k)_{ij}=$ #walks of length $k$) or $\Theta(n+m)$ list — the springboard for BFS/DFS and shortest paths.**

---

➡ **Next topic (in order): Topic 10 — Graph Traversals (BFS, DFS) & Shortest Path (Dijkstra).**

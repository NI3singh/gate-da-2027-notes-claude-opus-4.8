# Adversarial Search
### GATE DA — Subject: Artificial Intelligence · Section: Search (Topic 3 of 8 — last of the Search section)

---

## 0. Topic header & syllabus map

- **Topic:** Adversarial search — game trees, **Minimax**, **α-β pruning**, depth-limited search with an evaluation function, and (briefly) chance games via **Expectiminimax**.
- **Subject:** Artificial Intelligence
- **Where it sits:** The **third and final** topic of the *Search* section. It comes **after Informed Search** (single agent, heuristic-guided) and is the **last** search topic — the next prompt begins a new section, **Propositional Logic**, switching from *searching states* to *representing knowledge and reasoning*.

```
Uninformed → Informed (heuristic h(n)) → ADVERSARIAL (this topic: an opponent fights back; h(n) → board-eval)
                                        ───────────────  end of Search  ───────────────►  Logic (next section)
```

The two ideas carried in from Topic 2:

> **(1)** When the game tree is too deep to reach a terminal state, we cut off and apply an **evaluation function `EVAL(s)`** — exactly Topic 2's heuristic, repurposed from *distance-to-goal* to *value-of-position*.
> **(2)** **α-β pruning** is the adversarial twin of A\*'s "don't expand what can't matter (`f > C*`)": don't explore a branch the players would provably never enter.

---

## 1. Why this topic exists (motivation + connection)

**The real problem it solves.** In a **game** — chess, tic-tac-toe, Nim, checkers — you are **not** the only decision-maker. After *your* move, an **opponent** moves to make the outcome as **bad for you as possible**. You can't just compute a path to a goal and walk it, because the opponent will deliberately steer the game off your plan. You need a strategy that is *robust against an intelligent adversary*.

**What the previous topic couldn't do.** Informed search (A\*) assumes a **single agent in a static world it fully controls**: every action on the path is *yours*, so a found path is a found solution. That model is simply wrong for games — half the moves belong to someone working *against* you. Treating the opponent's moves as if you could choose them would produce a plan the opponent will never cooperate with. Adversarial search fixes the model: alternate **MAX** (you, maximizing your payoff) and **MIN** (the opponent, minimizing it), and assume the opponent plays **optimally**. **Minimax** then returns the move with the best *guaranteed* outcome under worst-case opposition.

**The bridge from Topic 2.** A game tree is still a **search tree**; minimax is a **depth-first traversal** of it that backs up values. When the tree is too big (chess has ~`10^40` states), we do exactly what bounded informed search does: **cut off** the search and substitute a **heuristic estimate** — here called `EVAL`. And **α-β pruning** generalizes A\*'s pruning instinct: A\* skipped nodes with `f > C*`; α-β skips branches outside the `[α, β]` window the rational players would ever allow.

**What the next topic builds on this.** All three search topics treat states as **opaque atoms** and find solutions by **enumerating and evaluating** them — the agent never *reasons about facts*. That fails when we must represent *what is true* in the world and **derive new conclusions by inference** rather than by exploring states. **Propositional Logic** (next section) supplies a formal language for facts and an inference machinery (entailment, resolution) — a different paradigm: **knowledge representation + reasoning** instead of state-space search. Search finds answers by *looking*; logic finds them by *deducing*.

---

## 2. Prerequisites

- **Trees and DFS traversal** (minimax and α-β are essentially smart DFS).
- **Informed search (Topic 2):** the heuristic `h(n)` — it becomes `EVAL(s)`; and the "prune what can't matter" idea behind α-β.
- **`b` (branching factor)** and **`m` (max depth)** from Topic 1 — they reappear verbatim in complexity.
- **Basic probability / expected value** — only for the Expectiminimax extension (§4.7).

---

## 3. Core idea — intuition first

Picture tic-tac-toe. It's **your** move (you are **MAX**, you want the highest score, say `+1` = you win, `0` = draw, `−1` = you lose). You think: *"For each move I could make, my opponent (**MIN**) will then reply with whatever is worst for me. So the real value of one of my moves is the worst outcome the opponent can force afterward. I should pick the move whose worst-case is best."*

That's **minimax** in one sentence: **you maximize over your moves; the opponent minimizes over theirs; values bubble up from the leaves.** A leaf is a finished game with a known payoff. A MAX node takes the **max** of its children; a MIN node takes the **min**. The root's value is the score you can **guarantee** against perfect play, and the child achieving it is your **best move**.

Now the optimization. Suppose at the root you've already found a move guaranteeing you **+3**. You start examining a second move and discover the opponent can force it down to **+2**. You can **stop examining that second move immediately** — it's already worse than +3 for you, and looking at the opponent's remaining replies can only make it the same or worse. That early stop is **α-β pruning**: once a branch is provably no better than something you already have, *don't bother exploring the rest of it.* It changes **nothing** about the final answer — it just skips wasted work.

The two bookkeeping numbers:
- **α** = the best score **MAX** can already guarantee somewhere above ("I already have at least α").
- **β** = the best (lowest) score **MIN** can already guarantee above ("the opponent can already hold me to at most β").
- The moment **α ≥ β**, the current branch is irrelevant — **prune**.

---

## 4. Formal treatment

### 4.1 Games as search problems (two-player, zero-sum, perfect information, deterministic)

| Component | Meaning |
|---|---|
| `S₀` | initial state (start of game) |
| `PLAYER(s)` | whose turn — **MAX** or **MIN** |
| `ACTIONS(s)` | legal moves in `s` |
| `RESULT(s, a)` | state after move `a` (transition model) |
| `TERMINAL-TEST(s)` | is the game over? |
| `UTILITY(s)` | payoff at a terminal `s` (to **MAX**) |

**Zero-sum** (a.k.a. constant-sum): one player's gain is the other's loss, `UTILITY_MAX(s) = −UTILITY_MIN(s)` (or they sum to a constant). The **game tree** alternates MAX and MIN plies. *Standard minimax assumes: 2 players, zero-sum, turn-taking, deterministic, perfect information.*

### 4.2 Minimax value

```
MINIMAX(s) =
    UTILITY(s)                                   if TERMINAL-TEST(s)
    max over a∈ACTIONS(s) of MINIMAX(RESULT(s,a))  if PLAYER(s) = MAX
    min over a∈ACTIONS(s) of MINIMAX(RESULT(s,a))  if PLAYER(s) = MIN
```

The **minimax decision** at the root is the action leading to the child with the **maximum** minimax value (for MAX). Computed by depth-first recursion, backing values up from leaves.

### 4.3 Properties of minimax

- **Complete?** **Yes**, if the game tree is **finite**.
- **Optimal?** **Yes, against an optimal opponent.** It guarantees a result **no worse than the minimax value regardless of how the opponent plays** (against a *sub-optimal* opponent you might score *higher* with a different strategy, but minimax never scores *below* the minimax value). ⚠
- **Time:** `O(b^m)` — it visits the whole tree.
- **Space:** `O(b·m)` (depth-first); `O(m)` if successors are generated one at a time.

### 4.4 α-β pruning

Same minimax computation, but carry a window `[α, β]` down the tree and prune branches that fall outside it.

- **α** = best (highest) value found so far on the path for **MAX** (a *lower* bound; starts `−∞`).
- **β** = best (lowest) value found so far on the path for **MIN** (an *upper* bound; starts `+∞`).
- **Cutoff condition: α ≥ β.** Specifically:
  - at a **MAX** node, once its running value `v ≥ β`, **prune** the remaining children (**β-cutoff**);
  - at a **MIN** node, once `v ≤ α`, **prune** the remaining children (**α-cutoff**).

```
function ALPHA-BETA(state):              # called at the root (a MAX node)
    return argmax move of MIN-VALUE(RESULT(state, move), −∞, +∞)

function MAX-VALUE(s, α, β):
    if TERMINAL-TEST(s): return UTILITY(s)
    v ← −∞
    for each a in ACTIONS(s):
        v ← max(v, MIN-VALUE(RESULT(s,a), α, β))
        if v ≥ β: return v               # β-cutoff: prune the rest
        α ← max(α, v)
    return v

function MIN-VALUE(s, α, β):
    if TERMINAL-TEST(s): return UTILITY(s)
    v ← +∞
    for each a in ACTIONS(s):
        v ← min(v, MAX-VALUE(RESULT(s,a), α, β))
        if v ≤ α: return v               # α-cutoff: prune the rest
        β ← min(β, v)
    return v
```

**Crucial fact:** α-β returns the **exact same** minimax value and the **same** optimal move as plain minimax — it is **not** an approximation. It only **examines fewer nodes**. ⚠ (Top misconception.)

### 4.5 Complexity of α-β and the role of move ordering

The amount pruned depends entirely on the **order** children are examined (examine the *best* moves first to prune the most):

| Move ordering | Nodes examined |
|---|---|
| **Best (optimal) ordering** | `O(b^(m/2)) = O((√b)^m)` |
| Random ordering | `O(b^(3m/4))` |
| **Worst ordering** | `O(b^m)` (no pruning — same as minimax) |

**Best-case effect:** the effective branching factor drops from `b` to **`√b`**, so in the same time α-β can search **roughly twice as deep** as minimax. Exact best-case **leaf** count for a uniform tree:

```
leaves examined (best case) = b^⌈m/2⌉ + b^⌊m/2⌋ − 1
                            = 2·b^(m/2) − 1   (m even)
```

**Space:** `O(b·m)`, same as minimax (α-β changes time, not memory).

### 4.6 Depth-limited (cutoff) search + evaluation function — the link to Topic 2

Real game trees are far too deep to reach terminal states. Replace `TERMINAL-TEST` with a **`CUTOFF-TEST`** (e.g., a depth limit) and `UTILITY` with an **`EVAL(s)`** that estimates the position's value:

```
H-MINIMAX(s, d) =
    EVAL(s)                                          if CUTOFF-TEST(s, d)
    max over children of H-MINIMAX(RESULT(s,a), d+1)  if MAX
    min over children of H-MINIMAX(RESULT(s,a), d+1)  if MIN
```

- **`EVAL(s)` is precisely a heuristic** (Topic 2) — "how good is this state" instead of "how far to the goal." Common form: a **weighted linear sum of features**, e.g. chess material `EVAL(s) = 9·(Q−Q′) + 5·(R−R′) + 3·(B+N − B′−N′) + …`.
- **Requirements on `EVAL`:** (1) order **terminal** states the same way `UTILITY` does; (2) be **fast** to compute; (3) for non-terminal states, **correlate with the actual chance of winning**.
- **Horizon effect:** a cutoff can hide an unavoidable bad event *just beyond* the depth limit, making the position look better than it is; **quiescence search** (extend "noisy" positions until they're stable) mitigates it.

### 4.7 Extension — games with chance (Expectiminimax)

For **stochastic** games (dice, e.g. backgammon) the tree has **CHANCE** nodes between MAX and MIN plies. A chance node returns the **expected value** of its children:

```
EXPECTIMINIMAX(s) =
    UTILITY(s)                              if terminal
    max over children                       if MAX
    min over children                       if MIN
    Σ_outcomes P(outcome)·EXPECTIMINIMAX(child)   if CHANCE
```

*Mini-illustration:* a chance node with outcomes of value `8` and `2`, each with probability `0.5`, has value `0.5·8 + 0.5·2 = 5`. *(For GATE DA, know the formula and be able to compute one expected value; minimax + α-β are the heavily-tested core.)*

### 4.8 Comparison table (memorize)

| Criterion | Minimax | α-β pruning |
|---|---|---|
| **Result** | exact minimax value & move | **identical** exact value & move |
| **Complete?** | Yes (finite tree) | Yes |
| **Optimal?** | Yes (vs optimal opponent) | Yes |
| **Time** | `O(b^m)` | `O(b^(m/2))` best · `O(b^m)` worst |
| **Space** | `O(b·m)` | `O(b·m)` |

---

## 5. Worked problems (GATE-style)

**Problems 1 and 2 use the same tree** (root is **MAX**; level-1 nodes **B, C, D** are **MIN**; leaves left-to-right). Branching factor `b = 3`, depth `m = 2`.

```
                 (MAX) root
            /         |         \
        (MIN)B      (MIN)C      (MIN)D
        / | \       / | \       / | \
       3 12  8     2  4  6    14  5  2
```

---

### Problem 1 — Minimax value and optimal move (MCQ/NAT)

Compute the minimax value at the root and MAX's optimal move.

**Solution.** Back values up from the leaves.
- `B = min(3, 12, 8) = 3`
- `C = min(2, 4, 6) = 2`
- `D = min(14, 5, 2) = 2`
- `root = max(B, C, D) = max(3, 2, 2) = `**`3`**

**Minimax value = 3; optimal move = the one leading to B.** Against perfect play, MAX can guarantee at least 3 and no more.

---

### Problem 2 — α-β pruning: which leaves are examined? (NAT)

Run α-β **left-to-right** on the same tree. List pruned leaves and count leaves examined.

**Solution.** (`v` = running value; window `[α, β]` shown as inherited.)

| Step | Node | Action | α, β | Result |
|---|---|---|---|---|
| 1 | B (MIN) | read 3 → 12 → 8 | α=−∞, β=+∞ | `B = min(3,12,8) = 3`; no cutoff |
| 2 | root (MAX) | back up B | — | `α ← max(−∞, 3) = 3` |
| 3 | C (MIN) | read **2** | α=3, β=+∞ | `v = 2`; since `v=2 ≤ α=3` → **α-cutoff** |
| 4 | C | — | — | **prune leaves 4 and 6**; `C ≤ 2` |
| 5 | root | back up C (2) | — | `α` stays 3 (2 doesn't beat 3) |
| 6 | D (MIN) | read 14 → 5 → 2 | α=3, β=+∞ | `D = min(14,5,2) = 2` (last child anyway) |
| 7 | root | back up D | — | `root = max(3, 2, 2) = `**`3`** |

- **Pruned leaves:** the `4` and `6` under **C** (after seeing C's first leaf `2 ≤ α = 3`).
- **Leaves examined:** B(3) + C(1) + D(3) = **7 of 9**.
- **Root value:** **3** — *identical* to Problem 1, confirming α-β doesn't change the answer.

**Teaching point / trap:** this ordering examined **7** leaves, but the *best-case* count for `b=3, m=2` is `3¹+3¹−1 = `**`5`**. **Move ordering matters** — examine stronger moves first and α-β prunes more. ⚠

---

### Problem 3 — Best-case savings of α-β (NAT)

A uniform game tree has branching factor `b = 10` and depth `m = 4`.
**(a)** How many leaf evaluations does plain minimax do?
**(b)** How many does α-β do in the **best case**?
**(c)** Roughly how much deeper can α-β search than minimax in the same time?

**Solution.**
**(a) Minimax:** evaluates **all** leaves `= b^m = 10⁴ = `**`10,000`**.
**(b) α-β best case:** `b^⌈m/2⌉ + b^⌊m/2⌋ − 1 = 10² + 10² − 1 = `**`199`** (≈ 2% of the leaves).
**(c)** Best-case time `O(b^(m/2))` means the **effective branching factor is `√b`**, so for the same node budget α-β reaches **about twice the depth** of minimax. ⚠

---

### Problem 4 — Property statements (MSQ, "trap" style)

Mark TRUE/FALSE.

1. α-β pruning can change the minimax value computed at the root.
2. With optimal move ordering, α-β runs in `O(b^(m/2))` time.
3. Against a **sub-optimal** opponent, the minimax strategy can never score *below* the minimax value.
4. α-β has the same space complexity as minimax, `O(bm)`.
5. α-β always examines the entire leftmost (principal-variation) path.
6. To compute the root value, α-β needs the *exact* value of every pruned subtree.

**Solution.**
1. **FALSE** — α-β returns the **exact same** value/move; it only skips nodes. ⚠
2. **TRUE** — best-case complexity (`√b` effective branching factor).
3. **TRUE** — minimax guarantees ≥ the minimax value against *any* opponent (it may not *maximally exploit* a weak opponent, but it never does worse). ⚠
4. **TRUE** — α-β changes time, not memory.
5. **TRUE** — nothing is known to prune until at least one full path establishes a bound, so the leftmost path is fully explored.
6. **FALSE** — a **bound** suffices for a pruned subtree; its exact value is **irrelevant** (that's *why* it was pruned). ⚠

(TRUE: **2, 3, 4, 5** · FALSE: **1, 6**.)

---

## 6. Common GATE question types on this topic

- **Compute the minimax value** of a given game tree and identify MAX's optimal move.
- **α-β trace:** given a fixed (usually left-to-right) ordering, list the **pruned branches/leaves** and **count nodes (leaves) examined**.
- **Effect of move ordering:** best-case `O(b^(m/2))`, worst-case `O(b^m)`; how many leaves in the best case (`b^⌈m/2⌉+b^⌊m/2⌋−1`); "how much deeper can α-β search."
- **Properties MSQ:** completeness, optimality (vs optimal opponent), time/space — and the trap that α-β is *exact*, not approximate.
- **Cutoff + EVAL:** identify `EVAL` as a heuristic, the requirements on it, and the horizon effect.
- **Expectiminimax:** compute the value of a **chance node** as an expectation.
- **Can you determine the value of a pruned node?** (Answer: only a bound — not in general.)

---

## 7. Traps, pitfalls & common mistakes

- **"α-β changes the answer / is approximate."** It does **not** — same value and move as full minimax, just fewer nodes. The single biggest misconception. ⚠
- **Confusing the two cutoffs.** Prune at a **MIN** node when `v ≤ α` (α-cutoff); at a **MAX** node when `v ≥ β` (β-cutoff). Overall condition: `α ≥ β`. Mixing these up flips the trace.
- **Move ordering ignored.** The *same* tree yields *different* prune counts under different orderings; best-case `O(b^(m/2))` requires examining strong moves first. Read the ordering the question fixes. ⚠
- **`d` vs `m` (again).** Game-tree complexity uses **`m`** (max depth); don't import `d` (solution depth) from Topics 1–2.
- **Value of a pruned subtree.** You only know a **bound**, not the exact value — questions asking for the exact value of a pruned node are usually *unanswerable by design*.
- **Optimality scope.** Minimax is optimal **against an optimal opponent**; it isn't the *score-maximizing* play against a known-weak opponent. State the qualifier.
- **EVAL must respect terminal ordering.** A cutoff evaluation that misorders won/lost positions breaks the search; and beware the **horizon effect** when relying on a fixed depth limit.
- **Zero-sum assumption.** Plain minimax assumes a two-player, zero-sum, deterministic, perfect-information game. Chance ⇒ **Expectiminimax**; hidden information ⇒ beyond plain minimax.
- **Sign of utilities.** MAX maximizes, MIN minimizes the *same* utility (to MAX). Don't accidentally let MIN maximize its own (negated) payoff in your hand-trace — keep one common scale.
- **Pruning the first child.** You can never prune before establishing a bound, so the *first* leaf under any node is always examined.

---

## 8. Connection forward

This completes the **Search** section. Across all three topics, the agent treated states as **opaque atoms** and found solutions by **generating and scoring** them — uninformed (blind), informed (heuristic-guided), and adversarial (against an opponent). What none of them did is **reason about facts**: there was no language to say "if it's raining then the ground is wet" and *deduce* consequences; we only ever explored a state space.

The **next section, Propositional Logic**, changes the paradigm entirely — from *searching* to **knowledge representation and inference**. We introduce a formal language of propositions and connectives, the notions of **truth, models, entailment, and validity**, and machinery (truth tables, inference rules, **resolution**) to derive *new* true statements from known ones. Where search answered "*what sequence of states reaches a goal?*", logic answers "*what must be true given what I already know?*". Search finds solutions by **looking**; logic finds them by **deducing** — and that deductive engine is what the rest of the AI syllabus (predicate logic, then reasoning under uncertainty) builds upon.

---

## 9. One-line cheat-sheet summary

**Adversarial search = minimax over a game tree** (MAX maximizes, MIN minimizes; back values up from leaves; `O(b^m)` time, `O(bm)` space, optimal vs an optimal opponent), sped up by **α-β pruning** (cut when **α ≥ β**; *exact same answer*, best case `O(b^(m/2))` with good move ordering), with **cutoff + EVAL** (= Topic 2's heuristic) for deep trees and **Expectiminimax** (expected value at chance nodes) for games with dice.

---

*Next topic (new section — Logic): **Propositional Logic** — syntax, semantics, entailment, and resolution, shifting from searching states to representing knowledge and reasoning.*

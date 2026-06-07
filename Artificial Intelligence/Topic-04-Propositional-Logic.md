# Topic 4 — Propositional Logic
### GATE DA · Subject: Artificial Intelligence · Section: Logic (1 of 2)

---

## 0. Topic header & syllabus map

- **Topic:** Propositional Logic (also called propositional calculus / sentential logic / Boolean logic).
- **Subject:** Artificial Intelligence.
- **Where it sits:** This is the **first topic of the Logic section**. It comes right after the Search cluster (uninformed → informed → adversarial search) and **leads directly into Predicate Logic / First-Order Logic (FOL)**, which is the next topic. Search taught the agent to *find* solutions by exploring states; Logic teaches the agent to *represent knowledge* and *reason* to provably correct conclusions. Propositional logic is the simplest such language; FOL is its more expressive successor.

---

## 1. Why this topic exists (motivation + connection)

A search agent treats the world as a black box of states and transitions: it can find a path, but it cannot **state a fact and deduce a consequence**. If you tell a search agent "if it rains, the ground is wet" and "it is raining," it has no machinery to *conclude* "the ground is wet" — it can only enumerate states you hand-coded. That is fine for puzzles with a clean state space, but most real reasoning is **declarative**: we want to assert facts and rules and let the agent infer new, guaranteed-true facts on its own.

**What the previous topic (Search) couldn't do that forced us here:** search requires the entire state space to be pre-defined and explored explicitly; it cannot represent general knowledge ("all birds fly," "if A then B") or perform deduction. Propositional logic fixes exactly this gap. It gives us (a) a **formal language** to write down knowledge as true/false statements, and (b) an **inference mechanism** that derives conclusions which are *logically guaranteed* by what we already know — no exploration of an enormous state graph required.

In one sentence: **propositional logic is the simplest formal system for knowledge representation and automated, provably-correct reasoning.**

---

## 2. Prerequisites

- **Boolean basics:** truth values (True/False), and a rough sense of AND/OR/NOT (any discrete-math background).
- **Set theory (light):** membership, subsets — needed for "set of models."
- **The AI agent idea:** a *knowledge-based agent* that stores a **knowledge base (KB)** and answers queries — useful framing, carried over from the AI intro.
- **Search (contrast only):** you only need to remember that search *explores* states, so you can appreciate that logic *reasons* about facts instead.

---

## 3. Core idea — intuition first

Think of a detective. The detective has clues (facts) and general rules ("the murderer must have had a key"). From these, certain conclusions are **forced** — they must be true in every scenario consistent with the clues. Propositional logic formalizes exactly this "forced conclusion" reasoning.

Three ideas do all the work:

1. **A proposition is a statement that is either true or false** — nothing in between. "It is raining" is a proposition. "Close the door!" is not (it is a command, not true/false). We name propositions with symbols like *P, Q, R*.

2. **We glue propositions together with connectives** — *not, and, or, implies, if-and-only-if* — to build complex statements. The whole knowledge base is essentially one giant statement (the AND of everything we believe).

3. **A "world" (model) fixes every symbol to true or false.** A statement that holds in *every* world consistent with our KB is **entailed** — the agent may safely conclude it. Checking all worlds is **model checking**; deriving the conclusion mechanically with rules is **inference**.

**The canonical AI example — the Wumpus World:** an agent in a cave smells a stench near a monster and feels a breeze near a pit. Using rules like "a breeze in square (1,1) means a pit is adjacent," the agent *deduces* which squares are safe without ever stepping on a pit. The agent's brain is a propositional KB, and "is square (2,2) safe?" is answered by entailment. You do not need Wumpus details for GATE, but it shows *why* AI cares about logic: safe, guaranteed decisions from partial knowledge.

A knowledge-based agent supports two operations: **TELL** (add a sentence to the KB) and **ASK** (query whether the KB entails a sentence). Everything below is the machinery behind ASK.

---

## 4. Formal treatment

### 4.1 Syntax — what counts as a legal sentence

The alphabet:

- **Atomic propositions (propositional symbols):** *P, Q, R, …* — each stands for a fact that is true or false.
- **Logical constants:** **True** (⊤) and **False** (⊥).
- **Connectives:**

| Symbol | Name | Read as |
|--------|------|---------|
| ¬ | negation | "not" |
| ∧ | conjunction | "and" |
| ∨ | disjunction | "or" (inclusive) |
| → | implication / conditional | "if … then …" |
| ↔ | biconditional | "if and only if" |

**Well-formed formula (wff) — recursive definition:**

```
1. Every propositional symbol, and ⊤ and ⊥, is a wff (atomic).
2. If α is a wff, then (¬α) is a wff.
3. If α and β are wff, then (α ∧ β), (α ∨ β), (α → β), (α ↔ β) are wff.
4. Nothing else is a wff.
```

**Operator precedence (binding strength), highest first:**

```
¬   >   ∧   >   ∨   >   →   >   ↔
```

So `¬P ∨ Q ∧ R → S` parses as `((¬P) ∨ (Q ∧ R)) → S`.
Convention: **→ is right-associative**, i.e. `P → Q → R` means `P → (Q → R)`.
When in doubt, parenthesize — GATE deliberately writes ambiguous-looking formulas to catch precedence mistakes.

---

### 4.2 Semantics — what sentences mean

A **model** (or **interpretation**, or **truth assignment**) `m` assigns **True** or **False** to *every* propositional symbol. With *n* symbols there are exactly **2ⁿ models** (rows of the truth table).

> **Truth tables for the connectives** (the only thing semantics really is):

| P | Q | ¬P | P∧Q | P∨Q | P→Q | P↔Q | P⊕Q (XOR) |
|---|---|----|-----|-----|-----|-----|-----------|
| T | T | F  | T   | T   | T   | T   | F |
| T | F | F  | F   | T   | **F** | F   | T |
| F | T | T  | F   | T   | **T** | F   | T |
| F | F | T  | F   | F   | **T** | T   | F |

**Burn the implication column into memory.** `P → Q` is **False in exactly one case: P true, Q false.** In particular **a false antecedent makes the implication True** (`F → anything = T`) — this is *vacuous truth* and is the single most tested trap in all of logic.

To evaluate any wff in a model, compute bottom-up using these tables.

---

### 4.3 The four classifications of a formula

For a formula α, look at the set of models in which it is true:

> - **Valid (tautology):** true in **every** model. (e.g. `P ∨ ¬P`)
> - **Satisfiable:** true in **at least one** model. (e.g. `P ∧ Q`)
> - **Unsatisfiable (contradiction):** true in **no** model. (e.g. `P ∧ ¬P`)
> - **Contingent (contingency):** true in some models, false in others — i.e. satisfiable but not valid. (e.g. `P → Q`)

**Key relationships (must know):**

```
α is VALID            ⟺  ¬α is UNSATISFIABLE
α is SATISFIABLE      ⟺  ¬α is NOT VALID
α is UNSATISFIABLE    ⟺  ¬α is VALID
Every valid formula is satisfiable; the converse is false.
```

Counts to remember (n symbols): a **tautology** is true in all **2ⁿ** models; a **contradiction** in **0** models; a contingency in somewhere between 1 and 2ⁿ−1.

---

### 4.4 Entailment, logical equivalence, and the two big theorems

**Entailment** `KB ⊨ α` ("KB entails α", "α is a logical consequence of KB"):
in **every model where KB is true, α is also true.** This is the formal meaning of "the conclusion is forced."

> **Deduction Theorem:**  `KB ⊨ α`  ⟺  `(KB → α)` is **valid**.
>
> **Refutation Theorem:**  `KB ⊨ α`  ⟺  `(KB ∧ ¬α)` is **unsatisfiable**.

These two are the workhorses of every entailment question. The refutation form is what powers **resolution** (Section 4.7): to *prove* a conclusion, assume its negation and show the result is contradictory.

**Logical equivalence** `α ≡ β`: true in exactly the same models.

> `α ≡ β`  ⟺  `α ⊨ β` and `β ⊨ α`  ⟺  `(α ↔ β)` is valid.

**Standard equivalences (the "algebra" of logic) — memorize:**

| Law | Statement |
|-----|-----------|
| Identity | `α ∧ ⊤ ≡ α`,  `α ∨ ⊥ ≡ α` |
| Domination | `α ∨ ⊤ ≡ ⊤`,  `α ∧ ⊥ ≡ ⊥` |
| Idempotence | `α ∧ α ≡ α`,  `α ∨ α ≡ α` |
| Double negation | `¬(¬α) ≡ α` |
| Commutativity | `α ∧ β ≡ β ∧ α`,  `α ∨ β ≡ β ∨ α` |
| Associativity | `(α∧β)∧γ ≡ α∧(β∧γ)`, similarly for ∨ |
| Distributivity | `α∧(β∨γ) ≡ (α∧β)∨(α∧γ)`;  `α∨(β∧γ) ≡ (α∨β)∧(α∨γ)` |
| **De Morgan** | `¬(α∧β) ≡ ¬α∨¬β`;  `¬(α∨β) ≡ ¬α∧¬β` (flip the connective!) |
| Absorption | `α∨(α∧β) ≡ α`;  `α∧(α∨β) ≡ α` |
| Negation | `α∨¬α ≡ ⊤`;  `α∧¬α ≡ ⊥` |
| **Implication elimination** | `α → β ≡ ¬α ∨ β` |
| **Contrapositive** | `α → β ≡ ¬β → ¬α` |
| **Biconditional elimination** | `α ↔ β ≡ (α→β) ∧ (β→α) ≡ (α∧β)∨(¬α∧¬β)` |
| Exportation | `(α∧β)→γ ≡ α→(β→γ)` |

**Conditional and its relatives (huge GATE trap — Section 7):**

```
Conditional :  P → Q
Converse    :  Q → P        (NOT equivalent to the conditional)
Inverse     :  ¬P → ¬Q      (NOT equivalent to the conditional)
Contrapositive: ¬Q → ¬P     (EQUIVALENT to the conditional ✓)

Fact:  Conditional ≡ Contrapositive   and   Converse ≡ Inverse.
```

---

### 4.5 Inference, proofs, and soundness/completeness

An **inference rule** lets you derive a new sentence from existing ones. Writing `KB ⊢ α` means "α is *derivable* from KB by the rules" (syntactic), as opposed to `KB ⊨ α` (semantic). Common rules:

| Rule | From | Infer |
|------|------|-------|
| **Modus Ponens (MP)** | `α`, `α → β` | `β` |
| **Modus Tollens (MT)** | `¬β`, `α → β` | `¬α` |
| **And-elimination** | `α ∧ β` | `α` (also `β`) |
| **And-introduction** | `α`, `β` | `α ∧ β` |
| **Or-introduction** | `α` | `α ∨ β` |
| **Hypothetical syllogism** | `α→β`, `β→γ` | `α→γ` |
| **Disjunctive syllogism** | `α∨β`, `¬α` | `β` |
| **Resolution** (Section 4.7) | `α∨β`, `¬β∨γ` | `α∨γ` |

Two properties tie ⊢ and ⊨ together:

> - **Soundness:** if `KB ⊢ α` then `KB ⊨ α`. (The rules never lie — every derived sentence is truly entailed.)
> - **Completeness:** if `KB ⊨ α` then `KB ⊢ α`. (The rules are strong enough to derive everything that is entailed.)

For **propositional logic**, truth-table enumeration is sound and complete, and **resolution is sound and refutation-complete** (defined below). MP/MT alone are sound but *not* complete for arbitrary KBs.

**Monotonicity (a property of classical logic):** if `KB ⊨ α` then `KB ∧ β ⊨ α` for any β. Adding new facts never retracts an old conclusion. (Real-world "default" reasoning is non-monotonic, but propositional logic is monotonic — occasionally tested as a true/false fact.)

---

### 4.6 Normal forms: CNF and DNF

- **Literal:** an atom or its negation (`P`, `¬P`).
- **Clause:** a disjunction (OR) of literals, e.g. `(P ∨ ¬Q ∨ R)`.
- **Term:** a conjunction (AND) of literals, e.g. `(P ∧ ¬Q ∧ R)`.
- **CNF (Conjunctive Normal Form):** an AND of clauses — `(…∨…) ∧ (…∨…) ∧ …`.
- **DNF (Disjunctive Normal Form):** an OR of terms — `(…∧…) ∨ (…∧…) ∨ …`.

> **Every propositional formula is equivalent to some CNF and to some DNF.** CNF is the input format required by resolution.

**Converting any formula to CNF (algorithm):**

```
1. Eliminate ↔ :   α ↔ β  ⇒  (α→β) ∧ (β→α)
2. Eliminate → :   α → β  ⇒  ¬α ∨ β
3. Push ¬ inward (De Morgan), eliminate double negation, until ¬ sits only on atoms.
4. Distribute ∨ over ∧ :  α∨(β∧γ) ⇒ (α∨β)∧(α∨γ),  to reach an AND of ORs.
```

*Example:* `¬(P → Q)` ⇒ `¬(¬P ∨ Q)` ⇒ `(P ∧ ¬Q)` (already CNF — a single one-literal-per-clause case... actually a conjunction of two unit clauses `P` and `¬Q`).

---

### 4.7 Resolution — the proof method GATE loves

**Resolution rule (propositional):** two clauses with complementary literals combine, dropping that literal:

> `(A ∨ B)` and `(¬B ∨ C)`  ⊢  `(A ∨ C)`     — *B and ¬B "cancel".*
>
> Special cases:  `(B)` and `(¬B)` ⊢ `⊥` (the **empty clause**, written □);  unit resolution `(A∨B)`, `(¬B)` ⊢ `A`.

**Resolution refutation — how to prove `KB ⊨ α`:**

```
1. Convert KB to CNF; collect all its clauses.
2. NEGATE the goal: add the clauses of ¬α to the set.
3. Repeatedly apply the resolution rule to pairs of clauses, adding each resolvent.
4. If you derive the EMPTY CLAUSE (□) ⇒ the set is unsatisfiable ⇒ KB ⊨ α.   ✓
   If no new clauses can be produced and □ never appears ⇒ KB ⊭ α.
```

> **Resolution is sound and *refutation-complete* for propositional logic:** if a clause set is unsatisfiable, resolution is guaranteed to derive the empty clause. (It is *not* "complete" in the generative sense — e.g. it won't directly produce `A∨B` from `A` — which is exactly why we work by refutation: negate the goal and seek a contradiction.)

---

### 4.8 Horn clauses, definite clauses & chaining (efficient inference)

General CNF/SAT reasoning is expensive (Section 4.9). A restricted clause shape makes inference **linear-time**:

- **Definite clause:** a clause with **exactly one positive literal**, e.g. `(¬P ∨ ¬Q ∨ R)`. Equivalently an implication with a single conclusion: `(P ∧ Q) → R`.
- **Horn clause:** **at most one positive literal** (definite clauses, plus *goal clauses* with no positive literal, and facts).
- A **fact** is a definite clause with no negative literals: just `R` (i.e. `⊤ → R`).

For a KB of definite clauses, two efficient, sound-and-complete procedures answer queries:

- **Forward chaining (data-driven):** start from known facts; whenever all premises of a rule are known, fire it to add the conclusion; repeat until the query appears (or nothing more fires).
- **Backward chaining (goal-driven):** start from the query; find rules concluding it; recursively try to prove their premises.

> Both run in **time linear in the size of the KB** for Horn/definite-clause KBs. This is why expert systems and logic programming (Prolog) restrict themselves to Horn clauses. Note: forward/backward chaining are complete **only for Horn KBs**, not arbitrary propositional KBs — for general KBs you need resolution or full model checking.

---

### 4.9 SAT, model checking & counting facts

- **The SAT problem:** "given a propositional formula, is it satisfiable?" SAT is **NP-complete** (Cook–Levin theorem) — the historical first NP-complete problem. Worth knowing as a one-liner; GATE DA does not ask you to *prove* it.
- **Model checking by truth-table enumeration:** check all 2ⁿ rows. Sound and complete, but exponential.
- **DPLL** = enhanced enumeration with three tricks: **unit propagation** (a unit clause forces its literal), **pure-symbol elimination** (a symbol appearing with only one polarity is set to satisfy it), and **early termination**. Still worst-case exponential but vastly faster in practice. (**WalkSAT** is a randomized local-search SAT solver — incomplete but fast on satisfiable instances. Both are "good to recognize" level.)

**Counting facts (frequently a NAT question):**

> - *n* propositional symbols ⇒ **2ⁿ** models (truth-table rows).
> - Number of **distinct Boolean functions** of *n* variables = **2^(2ⁿ)**. (Each of the 2ⁿ rows independently maps to T or F.)
>   - n=1 → 4,  n=2 → 16,  n=3 → 256.
> - Two formulas are logically equivalent ⟺ they define the same Boolean function ⟺ identical final truth-table column.

**Functional completeness:** a set of connectives is **functionally complete** if every Boolean function can be written using only those connectives.

| Connective set | Complete? | Why |
|----------------|-----------|-----|
| {¬, ∧, ∨} | ✔ | the standard basis |
| {¬, ∧} | ✔ | get ∨ via De Morgan |
| {¬, ∨} | ✔ | get ∧ via De Morgan |
| {¬, →} | ✔ | `¬a∨b = a→b`; build ∨, ∧ from these |
| **{NAND}** (↑) alone | ✔ | `¬a=a↑a`, `a∧b=(a↑b)↑(a↑b)`, `a∨b=(a↑a)↑(b↑b)` |
| **{NOR}** (↓) alone | ✔ | dual of NAND |
| {→, ⊥} | ✔ | `¬a = a→⊥` gives negation |
| **{∧, ∨}** (no ¬) | ✘ | only **monotone** functions; cannot build ¬ |
| {→} alone, {↔} alone, {∧} alone, {∨} alone | ✘ | each is too weak |

(NAND and NOR are the *only* single connectives that are functionally complete by themselves — a popular MCQ.)

---

## 5. Worked problems (GATE-style)

### Problem 1 — Classify a formula (MCQ; truth-table method)
**Q.** Classify `α = (P → Q) ↔ (¬P ∨ Q)` as tautology / contingency / contradiction.

**Solution.** Build the table (2 symbols → 4 rows):

| P | Q | P→Q | ¬P | ¬P∨Q | (P→Q) ↔ (¬P∨Q) |
|---|---|-----|----|------|----------------|
| T | T | T   | F  | T    | **T** |
| T | F | F   | F  | F    | **T** |
| F | T | T   | T  | T    | **T** |
| F | F | T   | T  | T    | **T** |

Last column is **all True** ⇒ **α is a tautology (valid).** This is just the *implication-elimination* equivalence (`P→Q ≡ ¬P∨Q`) packaged as a biconditional; an equivalence wrapped in ↔ is always a tautology. ∎

---

### Problem 2 — Entailment, two ways
**Q.** Does `{P → Q, Q → R, P} ⊨ R`?

**Method A — model checking (refutation form).** The premises force values: the third premise gives `P = T`; with `P→Q` true and `P=T`, we must have `Q = T`; with `Q→R` true and `Q=T`, we must have `R = T`. The *only* model satisfying all three premises is `(P,Q,R)=(T,T,T)`, and there `R=T`. Since `R` holds in every model of the KB, **`R` is entailed.** ✓

**Method B — inference (chain of Modus Ponens).**
`P` (given) and `P→Q` ⟹ `Q` (MP). Then `Q` and `Q→R` ⟹ `R` (MP). MP is sound, so `R` is entailed. ✓

Both agree: **yes, `KB ⊨ R`.** ∎

---

### Problem 3 — Prove a conclusion by resolution refutation
**Q.** Prove `{P ∨ Q, P → R, Q → R} ⊨ R` using resolution. (This is "proof by cases" / constructive dilemma.)

**Solution.**
*Step 1 — CNF of KB:* `P→R ≡ ¬P∨R`, `Q→R ≡ ¬Q∨R`. Clauses:
```
C1: (P ∨ Q)
C2: (¬P ∨ R)
C3: (¬Q ∨ R)
```
*Step 2 — negate the goal* `¬R`:
```
C4: (¬R)
```
*Step 3 — resolve:*
```
C2, C4 on R    →  C5: (¬P)          [(¬P∨R) with (¬R)]
C3, C4 on R    →  C6: (¬Q)          [(¬Q∨R) with (¬R)]
C1, C5 on P    →  C7: (Q)           [(P∨Q) with (¬P)]
C7, C6 on Q    →  C8: □  (empty)    [(Q) with (¬Q)]
```
The **empty clause** is derived ⇒ `{KB, ¬R}` is unsatisfiable ⇒ **`KB ⊨ R`.** ✓ ∎

---

### Problem 4 — Count satisfying models (NAT)
**Q.** How many of the 8 truth assignments to `{P,Q,R}` satisfy `(P → Q) ∧ (Q → R)`?

**Solution.** The formula is true unless `P→Q` fails (`P=T,Q=F`) or `Q→R` fails (`Q=T,R=F`). Check all 8:

| P Q R | P→Q | Q→R | formula |
|-------|-----|-----|---------|
| T T T | T | T | **✓** |
| T T F | T | F | ✘ |
| T F T | F | – | ✘ |
| T F F | F | – | ✘ |
| F T T | T | T | **✓** |
| F T F | T | F | ✘ |
| F F T | T | T | **✓** |
| F F F | T | T | **✓** |

Satisfying assignments: `TTT, FTT, FFT, FFF`. **Answer: 4.** ∎

---

### Problem 5 — The implication trap (MCQ)
**Q.** "If it rains (R), the match is cancelled (C)" = `R → C`. Which statement is **logically equivalent**, and which is a **fallacy**?
(i) "If the match is not cancelled, it did not rain." (ii) "If the match is cancelled, it rained." (iii) "If it does not rain, the match is not cancelled."

**Solution.**
- (i) is `¬C → ¬R` = the **contrapositive** ⇒ **equivalent** ✓.
- (ii) is `C → R` = the **converse** ⇒ **not equivalent** (asserting it commits the *affirming-the-consequent* fallacy).
- (iii) is `¬R → ¬C` = the **inverse** ⇒ **not equivalent** (this is *denying the antecedent*; note (ii) ≡ (iii)).

**Trap to feel:** if it does *not* rain (`R=F`), `R→C` is **True no matter what happens to the match** (vacuous truth). So you may **not** conclude `¬C` from `¬R`. Only the contrapositive preserves meaning. ∎

---

### Problem 6 — Boolean-function counting & completeness (NAT + MCQ)
**Q1.** How many distinct Boolean functions of 2 variables are there?
**A1.** `2^(2²) = 2⁴ = ` **16**.

**Q2.** Which of these single connectives is functionally complete: ∧, ∨, →, NAND?
**A2.** **NAND** (also NOR). With NAND alone: `¬a = a↑a`, then `∧` and `∨` follow, giving all 16 functions. None of ∧, ∨, → is complete on its own; in particular `{∧,∨}` (even together) can express only *monotone* functions and can never produce ¬. ∎

---

## 6. Common GATE question types on this topic

- **Classify a formula** as tautology / satisfiable / contradiction / contingency (truth table or smart reasoning).
- **Check logical equivalence** of two formulas (build `α ↔ β`, test validity; or compare truth-table columns).
- **Argument validity / entailment:** "do these premises entail this conclusion?" — via refutation (`KB ∧ ¬α` unsatisfiable) or an inference chain.
- **Resolution proof:** convert to CNF, negate the goal, derive the empty clause.
- **Count models** that satisfy a formula (NAT).
- **Count Boolean functions** `2^(2ⁿ)` and **identify functionally complete connective sets** (NAND/NOR famous).
- **Convert** a formula to CNF / DNF.
- **Identify which inference rule** was applied (MP vs MT vs resolution vs a fallacy).
- **Converse / inverse / contrapositive** identification and equivalence.
- **Horn-clause / forward- or backward-chaining trace** (which atoms become derivable).
- **True/false conceptual facts:** soundness vs completeness, SAT is NP-complete, propositional logic is monotonic, resolution is refutation-complete.

---

## 7. Traps, pitfalls & common mistakes

1. **`F → anything = T` (vacuous truth).** The #1 trap. A false antecedent makes the whole implication true. From `¬P` you can say nothing about `Q` in `P→Q`.
2. **Converse / inverse / contrapositive confusion.** Only the **contrapositive** (`¬Q→¬P`) is equivalent to `P→Q`. Converse and inverse are equivalent to *each other*, not to the original.
3. **Two classic fallacies:** *affirming the consequent* (from `P→Q` and `Q`, wrongly conclude `P`) and *denying the antecedent* (from `P→Q` and `¬P`, wrongly conclude `¬Q`). Both are invalid.
4. **Validity ≠ satisfiability.** "Satisfiable" only needs *one* true model; "valid" needs *all*. A satisfiable formula need not be a tautology. (`P` is satisfiable, not valid.)
5. **De Morgan: you must FLIP the connective.** `¬(P∧Q) = ¬P ∨ ¬Q` (not `¬P ∧ ¬Q`). Forgetting the flip is the most common algebra slip.
6. **Resolution proves UNSAT, so you must NEGATE the goal first.** You derive the **empty clause**, not the goal itself. The empty clause `□` means *contradiction / ⊥*, **not** "no answer."
7. **`KB ⊨ α` is not symmetric.** `KB ⊨ α` does **not** imply `α ⊨ KB`.
8. **Operator precedence.** `¬` binds tightest; `→` is right-associative. `¬P ∧ Q` is `(¬P)∧Q`, not `¬(P∧Q)`.
9. **Model counts of extremes:** a tautology is true in **all 2ⁿ** models, a contradiction in **0** — don't write "1" or "undefined."
10. **`⊨` (semantic) vs `⊢` (syntactic).** They coincide *only* for a sound **and** complete proof system. Don't assume an arbitrary rule set is complete (MP alone is not).
11. **Forward/backward chaining is complete only for Horn KBs.** Do not apply it to a general KB and claim completeness — use resolution there.
12. **Material implication ≠ causation.** `P→Q` is just a truth-table relation; it carries no real-world "because."
13. **Biconditional needs BOTH directions.** `P↔Q` is `(P→Q) ∧ (Q→P)`; proving one direction is not enough.
14. **CNF requires `↔` and `→` eliminated *before* pushing negations.** Skipping a step (e.g. distributing too early) produces wrong clauses.

---

## 8. Connection forward

Propositional logic has a hard ceiling: it can talk about **whole facts** but not about **objects, their properties, and relations**, and it cannot **quantify**. To say "all humans are mortal," propositional logic would need a *separate symbol for every human* — impossible for infinite or unknown domains. Likewise it cannot express "there exists a student who passed."

**Predicate Logic / First-Order Logic (FOL)** — the next topic — fixes exactly this by adding **objects**, **predicates** (relations like `Human(x)`), **functions**, and the **quantifiers ∀ ("for all") and ∃ ("there exists")**. Crucially, the machinery you just learned **carries over and generalizes**: entailment, CNF, and resolution all reappear in FOL, now extended with **unification** to match variables. Propositional logic is the scaffolding; FOL is the full building.

---

## 9. One-line cheat-sheet summary

> **Propositional logic** = facts as True/False symbols joined by `¬ ∧ ∨ → ↔`; **`KB ⊨ α` ⟺ `(KB∧¬α)` is UNSAT**, proved by **resolution to the empty clause** (negate the goal first); **valid** = all 2ⁿ models, **satisfiable** = ≥1, **contradiction** = 0; remember `F→x = T`, contrapositive ≡ original, De Morgan flips the connective, **NAND/NOR are the only single complete connectives**, and *n* symbols give **2ⁿ models** and **2^(2ⁿ) Boolean functions**.

---

*Next topic (logically next in order):* **Topic 5 — Predicate Logic (First-Order Logic).** Ask for it when ready.

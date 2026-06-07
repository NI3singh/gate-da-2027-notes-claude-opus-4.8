# Topic 5 — Predicate Logic (First-Order Logic)
### GATE DA · Subject: Artificial Intelligence · Section: Logic (2 of 2)

---

## 0. Topic header & syllabus map

- **Topic:** Predicate Logic, almost always called **First-Order Logic (FOL)** (also: first-order predicate calculus, FOPC).
- **Subject:** Artificial Intelligence.
- **Where it sits:** This is the **second and final topic of the Logic section**. It comes directly after **Propositional Logic** and generalizes it. After FOL, the syllabus leaves pure logic entirely and **begins a new section — Reasoning under Uncertainty — starting with Conditional Independence Representation (Bayesian networks)**. So FOL is the bridge from *certain, true/false reasoning* to the *probabilistic reasoning* that follows.

---

## 1. Why this topic exists (motivation + connection)

Propositional logic treats every fact as an indivisible black box. `P` might *mean* "Socrates is mortal," but to the logic it is just one bit. This causes two fatal limitations:

1. **No internal structure.** You cannot say that the *same property* (mortality) applies to *different objects*. "Socrates is mortal" and "Plato is mortal" are unrelated symbols `P₁, P₂` with nothing connecting them.
2. **No generalization / quantification.** You cannot write "**all** humans are mortal." You would need a *separate symbol for every human* — impossible for large, infinite, or unknown domains.

The classic consequence: the obviously valid syllogism **"All men are mortal; Socrates is a man; therefore Socrates is mortal"** is **not expressible — let alone provable — in propositional logic.** That is a glaring failure for a knowledge-representation language.

**What the previous topic couldn't do that forced us here:** propositional logic has no notion of *objects*, *relations among objects*, or *quantifiers* ("all," "some"). FOL fixes exactly this by **opening up the atom**: a fact becomes a **predicate applied to objects** (`Mortal(Socrates)`), objects can be named and combined with **functions**, and we can quantify over them with **∀ (for all)** and **∃ (there exists)**. The reasoning machinery you already learned — entailment, CNF, resolution, soundness/completeness — **all carry over**, now extended with **unification** to match variables. In one line: **FOL = propositional logic + objects + relations + functions + quantifiers.**

---

## 2. Prerequisites

- **Propositional logic (everything):** connectives `¬ ∧ ∨ → ↔`, truth tables, validity/satisfiability, entailment (`⊨`), CNF, resolution refutation, soundness/completeness. FOL reuses all of it.
- **Sets and relations:** a domain (set of objects); an *n*-ary relation is a set of *n*-tuples.
- **Functions:** mappings from objects to objects (arity).
- **Quantifier intuition:** the everyday meaning of "all" and "some."

---

## 3. Core idea — intuition first

**The one big leap: stop treating a fact as one bit; give it grammar.**

Propositional logic is like owning a fixed stock of complete, uneditable sentences. FOL hands you a *grammar*:

- **Objects** are the nouns — the things in the world (Socrates, the number 3, John's father).
- **Predicates** are the verbs/adjectives — properties of one object (`Human(x)`) or relations among several (`Loves(x, y)`, `Greater(x, y)`). A predicate evaluates to **true/false**.
- **Functions** are noun-phrases that *name* an object in terms of others (`FatherOf(John)`, `Plus(x, y)`). A function evaluates to an **object**, not a truth value.
- **Quantifiers** are the words "all" and "some": **∀x** ("for every object x…") and **∃x** ("for at least one object x…").

With this grammar a *tiny* vocabulary expresses *infinitely* many facts. The mortality syllogism becomes:

```
∀x (Human(x) → Mortal(x))          "all humans are mortal"
Human(Socrates)                     "Socrates is a human"
∴ Mortal(Socrates)                  follows by instantiating x := Socrates
```

The single rule `∀x (Human(x) → Mortal(x))` silently covers *every* object at once — exactly what propositional logic could never do.

**A knowledge-based agent** still works by **TELL** (add sentences) and **ASK** (query entailment); FOL just makes the sentences vastly more expressive.

---

## 4. Formal treatment

### 4.1 Syntax — the vocabulary of FOL

| Element | Role | Examples |
|---------|------|----------|
| **Constant symbols** | name specific objects | `Socrates`, `0`, `John` |
| **Variables** | range over objects | `x, y, z` |
| **Function symbols** (arity ≥ 1) | map objects → an object | `FatherOf(·)`, `Plus(·,·)` |
| **Predicate symbols** (arity ≥ 0) | relation among objects → True/False | `Human(·)`, `Loves(·,·)`; a 0-ary predicate = a propositional symbol |
| **Connectives** | as in propositional logic | `¬ ∧ ∨ → ↔` |
| **Quantifiers** | quantify over objects | `∀` (universal), `∃` (existential) |
| **Equality** | built-in binary predicate | `=` |

**Terms** (expressions that *denote objects*):
```
a constant, or a variable, or  f(t₁, …, tₙ)  where f is an n-ary function and the tᵢ are terms.
```

**Atomic sentence (atom)** — *evaluates to true/false*:
```
P(t₁, …, tₙ)   (predicate applied to terms),   or   t₁ = t₂   (equality).
```

**Complex sentences:** atoms combined with `¬ ∧ ∨ → ↔`, and quantified forms `∀x α`, `∃x α`.

> **Critical distinction (common trap):** a **function** returns an *object* and appears *inside* atoms (`Mortal(FatherOf(John))`); a **predicate** returns *true/false* and *is* the atom (`Father(John, Bill)`). `FatherOf(John)` is a **term**; `Father(John, Bill)` is a **sentence**. They are not interchangeable.

---

### 4.2 Quantifiers and the all-important pairing rule

- **∀x P(x)** ("for all x"): true iff `P` holds for **every** object in the domain.
- **∃x P(x)** ("there exists x"): true iff `P` holds for **at least one** object.

> **THE pairing rule — memorize, it is the #1 GATE translation trap:**
>
> - **∀ pairs with → :** "All As are Bs" = `∀x (A(x) → B(x))`.
>   *Wrong:* `∀x (A(x) ∧ B(x))` claims **everything** is an A that is also a B.
> - **∃ pairs with ∧ :** "Some A is a B" = `∃x (A(x) ∧ B(x))`.
>   *Wrong:* `∃x (A(x) → B(x))` is **almost always trivially true** (any non-A makes it true) and does not say what you mean.

**Translation reference (high-yield):**

| English | FOL |
|---------|-----|
| All As are Bs | `∀x (A(x) → B(x))` |
| Some A is a B | `∃x (A(x) ∧ B(x))` |
| No A is a B | `∀x (A(x) → ¬B(x))` ≡ `¬∃x (A(x) ∧ B(x))` |
| Not all As are Bs | `¬∀x (A(x) → B(x))` ≡ `∃x (A(x) ∧ ¬B(x))` |
| Only As are Bs | `∀x (B(x) → A(x))`  *(being B requires being A)* |
| Exactly one A | `∃x (A(x) ∧ ∀y (A(y) → y = x))`  — written `∃!x A(x)` |
| At least two As | `∃x ∃y (A(x) ∧ A(y) ∧ x ≠ y)` |

---

### 4.3 Semantics — what a model is in FOL

A **model (interpretation)** consists of:

1. a **non-empty domain** `D` (the set of objects);
2. an assignment giving each **constant** an element of `D`, each **n-ary function symbol** a function `Dⁿ → D`, and each **n-ary predicate symbol** a relation `⊆ Dⁿ`.

Truth of `∀x α` = "α true for every object substituted for x"; `∃x α` = "α true for at least one."

A sentence is **valid** if true in **every** model (every domain and interpretation); **satisfiable** if true in **some**; **unsatisfiable** if in **none**. Entailment `KB ⊨ α` and equivalence `α ≡ β` are defined exactly as in propositional logic, just over FOL models. The refutation theorem still holds: `KB ⊨ α ⟺ (KB ∧ ¬α)` is unsatisfiable.

---

### 4.4 Free vs bound variables; sentences

- The **scope** of a quantifier is the formula it governs.
- A variable occurrence is **bound** if it lies within the scope of a quantifier over that variable; otherwise it is **free**.
- A **sentence (closed formula)** has **no free variables.** Only sentences have a truth value independent of any external variable assignment.

*Example:* in `∀x (P(x, y) → ∃y Q(y))`, the `y` in `P(x, y)` is **free** (no quantifier governs it), while the `y` in `Q(y)` is **bound** by `∃y`. Reusing `y` for two roles is legal but error-prone — hence "standardize variables apart" before mechanical inference (Section 4.8–4.9).

---

### 4.5 Negating quantifiers (quantifier De Morgan)

> ```
> ¬ ∀x P(x)  ≡  ∃x ¬P(x)
> ¬ ∃x P(x)  ≡  ∀x ¬P(x)
> ```
> To negate: **push ¬ inward, FLIP the quantifier, negate the body.** For nested quantifiers, flip each one as the ¬ moves past it.

*Example:* `¬ ∀x ∃y Loves(x, y)` ≡ `∃x ∀y ¬Loves(x, y)` — "there is someone who loves no one."

---

### 4.6 Nested quantifiers — ORDER MATTERS

- **Same type commutes:** `∀x ∀y α ≡ ∀y ∀x α`, and `∃x ∃y α ≡ ∃y ∃x α`.
- **Mixed type does NOT commute:**

> ```
> ∀x ∃y P(x, y)   ≢   ∃y ∀x P(x, y)
> ```
> - `∀x ∃y Loves(x, y)` = "everyone loves **someone** (possibly a different person each)."
> - `∃y ∀x Loves(x, y)` = "there is **one** person whom **everyone** loves (the same person)."
>
> Only one direction is valid: **`∃y ∀x P(x,y) ⊨ ∀x ∃y P(x,y)`** (a universal witness certainly works for each x). The reverse is **not** valid.

This direction-of-entailment question is a GATE staple — see Worked Problem 4.

---

### 4.7 Logical equivalences with quantifiers (distribution)

> **Hold (✔):**
> ```
> ∀x (P(x) ∧ Q(x))  ≡  (∀x P(x)) ∧ (∀x Q(x))      [∀ distributes over ∧]
> ∃x (P(x) ∨ Q(x))  ≡  (∃x P(x)) ∨ (∃x Q(x))      [∃ distributes over ∨]
> ```
> **Do NOT hold (✘) — only one-directional:**
> ```
> ∀x (P(x) ∨ Q(x))  ≢  (∀x P(x)) ∨ (∀x Q(x))      [∀ does NOT distribute over ∨]
> ∃x (P(x) ∧ Q(x))  ≢  (∃x P(x)) ∧ (∃x Q(x))      [∃ does NOT distribute over ∧]
> ```
> *(Valid directions of the failing pair: `(∀xP ∨ ∀xQ) ⊨ ∀x(P∨Q)` and `∃x(P∧Q) ⊨ (∃xP ∧ ∃xQ)`, but not their converses.)*
>
> **Vacuous-variable rule:** if `x` does not occur free in `α`, then `∀x (α ∨ P(x)) ≡ α ∨ ∀x P(x)` and `∃x (α ∧ P(x)) ≡ α ∧ ∃x P(x)` — you may move `α` out of the quantifier's scope.

---

### 4.8 Inference in FOL

**Universal Instantiation (UI).** From `∀x α`, infer `SUBST({x/g}, α)` for **any ground term** `g`. May be applied **repeatedly** with different terms.
```
∀x (King(x) → Person(x))   ⟹   King(John) → Person(John)        (g = John)
```

**Existential Instantiation (EI).** From `∃x α`, infer `SUBST({x/k}, α)` where `k` is a **brand-new constant** (a **Skolem constant**) not appearing anywhere else. Applied **once** per existential.
```
∃x Crown(x) ∧ OnHead(x, John)   ⟹   Crown(C₁) ∧ OnHead(C₁, John)   (C₁ new)
```
*(UI may reuse and repeat; EI must introduce a fresh name. Swapping these is a classic error.)*

**Propositionalization.** Grounding a FOL KB (substitute all ground terms) reduces it to propositional logic — complete in principle (Herbrand) but explodes combinatorially; motivates *lifted* inference.

**Unification.** Find a **substitution** `θ` making two atoms identical: `UNIFY(α, β) = θ` such that `αθ = βθ`. The **Most General Unifier (MGU)** is the least-committed such `θ` (every other unifier is an instance of it). Unification powers all lifted inference.

> **Occurs check:** a variable `x` cannot be unified with a term that **contains** `x` (e.g. `x` vs `f(x)`) — doing so would need an infinite term. Such pairs are **not unifiable**.

**Generalized Modus Ponens (GMP).** The *lifted* form of Modus Ponens, using unification:
```
from  p₁′, …, pₙ′  and  (p₁ ∧ … ∧ pₙ → q),  with θ = MGU making each pᵢθ = pᵢ′θ,   infer  qθ.
```
GMP drives **forward chaining** (data-driven, fire rules as premises match) and **backward chaining** (goal-driven, the basis of **Prolog/Datalog**) for first-order **definite clauses**.

**FOL Resolution.** Convert KB + ¬goal to CNF (Section 4.9), then apply resolution **with unification**. **Sound and refutation-complete** for FOL.

---

### 4.9 Conversion to CNF and Skolemization (the FOL twist)

To run resolution, every sentence must become a set of clauses. The steps extend the propositional CNF procedure with two FOL-specific moves (**standardize apart** and **Skolemize**):

```
1. Eliminate ↔ and → .
2. Move ¬ inward (De Morgan + quantifier negation) until ¬ touches only atoms.
3. Standardize variables apart — rename so every quantifier binds a unique variable.
4. Skolemize — remove existential quantifiers:
     • ∃ NOT inside any ∀  →  replace its variable by a fresh Skolem CONSTANT (e.g. C₁).
     • ∃ inside ∀x₁ … ∀xₙ →  replace its variable by a Skolem FUNCTION f(x₁,…,xₙ)
                              of exactly the enclosing universally-quantified variables.
5. Drop all universal quantifiers (remaining variables are implicitly ∀).
6. Distribute ∨ over ∧  →  CNF (a conjunction of clauses).
```

> **Honesty note (assumption GATE tests):** **Skolemization preserves *satisfiability*, not *logical equivalence*.** The Skolemized form is satisfiable iff the original is — which is all refutation needs — but it is **not** equivalent to the original. Also, a Skolem **function** (not constant) is required whenever the existential is in the scope of a universal, because the witness may depend on those universally-quantified objects.

---

### 4.10 Decidability & meta-facts (one-liners, GATE-theory level)

> - **FOL validity / entailment is *semi-decidable* (recursively enumerable):** if `α` is valid, a sound+complete procedure will eventually confirm it; if `α` is *not* valid, the procedure may run forever. (Satisfiability is *not* even semi-decidable.)
> - **FOL is *undecidable* in general** (Church–Turing): no algorithm decides validity for *all* FOL sentences.
> - **Propositional logic, by contrast, IS decidable** (finite truth tables).
> - **Resolution is sound and refutation-complete for FOL.**
> - **Gödel's completeness theorem:** FOL has a sound and complete proof system (every valid sentence is provable).

You will not be asked to *prove* these; you may be asked to *state* them or pick the correct true/false claim.

---

## 5. Worked problems (GATE-style)

### Problem 1 — Translation, correct pairing (MCQ)
**Q.** Which sentence correctly expresses **"Every dog has a tail"** (Dog(x), HasTail(x))?
- A) `∀x (Dog(x) ∧ HasTail(x))`
- B) `∀x (Dog(x) → HasTail(x))`
- C) `∃x (Dog(x) → HasTail(x))`
- D) `∀x (HasTail(x) → Dog(x))`

**Solution.** **B.** "All As are Bs" pairs **∀ with →**. (A) wrongly claims *every object* is a dog that has a tail. (C) is `∃ with →`, trivially satisfied by any non-dog. (D) is the converse ("everything with a tail is a dog"). ∎

*Companion:* "Some dog is black" = `∃x (Dog(x) ∧ Black(x))`, **not** `∃x (Dog(x) → Black(x))`.

---

### Problem 2 — "No / not all" with negation (MCQ)
**Q.** Express **"Not every student passed"** (Student(x), Passed(x)).

**Solution.**
`¬ ∀x (Student(x) → Passed(x))`
≡ `∃x ¬(Student(x) → Passed(x))`   (flip quantifier)
≡ `∃x (Student(x) ∧ ¬Passed(x))`   (since `¬(P→Q) ≡ P ∧ ¬Q`).
So it means **"there is a student who did not pass."** Note this is *different* from "No student passed" = `∀x (Student(x) → ¬Passed(x))`. ∎

---

### Problem 3 — Negate a nested quantified sentence
**Q.** Negate `∀x (P(x) → ∃y (Q(y) ∧ R(x, y)))` and simplify so ¬ touches only atoms.

**Solution.**
```
¬ ∀x (P(x) → ∃y (Q(y) ∧ R(x,y)))
≡ ∃x ¬ (P(x) → ∃y (Q(y) ∧ R(x,y)))          flip ∀→∃
≡ ∃x ( P(x) ∧ ¬ ∃y (Q(y) ∧ R(x,y)) )         ¬(P→S) = P ∧ ¬S
≡ ∃x ( P(x) ∧ ∀y ¬ (Q(y) ∧ R(x,y)) )         flip ∃→∀
≡ ∃x ( P(x) ∧ ∀y ( ¬Q(y) ∨ ¬R(x,y) ) )       De Morgan
```
English: "there is an x with property P such that for every y, y is not a Q related to x." ∎

---

### Problem 4 — Quantifier order: which entailment is valid? (MSQ + counter-model)
**Q.** Domain = people, `L(x, y)` = "x likes y." For `∀x ∃y L(x,y)` and `∃y ∀x L(x,y)`, which entailment holds, and give a model separating them.

**Solution.** **`∃y ∀x L(x,y) ⊨ ∀x ∃y L(x,y)`** holds (a single universally-liked person `y*` serves as the witness for every `x`). The **reverse fails.**

*Counter-model:* domain `{1, 2}`, with `L = {(1,2), (2,1)}`.
- `∀x ∃y L(x,y)`: x=1 → (1,2) ✓; x=2 → (2,1) ✓. **True.**
- `∃y ∀x L(x,y)`: need one `y` liked by both. y=1 needs `L(1,1)`? no. y=2 needs `L(2,2)`? no. **False.**

So `∀x ∃y L ⊭ ∃y ∀x L`. The two sentences are **not equivalent.** ∎

---

### Problem 5 — Unification / Most General Unifier
**Q.** For each pair, give the MGU or state it fails.
(a) `P(x, f(y))` and `P(a, f(g(z)))`
(b) `P(x, x)` and `P(a, b)`  (a, b distinct constants)
(c) `P(x, f(x))` and `P(y, y)`

**Solution.**
- (a) Unify args: `x` with `a` → `{x/a}`; `f(y)` with `f(g(z))` → `y` with `g(z)` → `{y/g(z)}`. **MGU = `{x/a, y/g(z)}`.**
- (b) `x/a` from first arg, then `x/b` from second → conflict `a ≠ b`. **Not unifiable.**
- (c) `x` with `y` → `{x/y}`; then `f(x)` with `y`, i.e. `f(y)` with `y` → `y` would contain itself. **Occurs-check failure → not unifiable.** ∎

---

### Problem 6 — FOL resolution refutation (the syllogism)
**Q.** KB = { `∀x (Man(x) → Mortal(x))`, `Man(Socrates)` }. Prove `Mortal(Socrates)` by resolution.

**Solution.**
*CNF of KB:* `∀x(Man(x)→Mortal(x))` ⇒ `(¬Man(x) ∨ Mortal(x))`; plus the fact `Man(Socrates)`.
*Negate goal:* add `¬Mortal(Socrates)`.
```
C1: (¬Man(x) ∨ Mortal(x))
C2: Man(Socrates)
C3: ¬Mortal(Socrates)            ← negated goal

C1, C2  with θ = {x/Socrates}  →  C4: Mortal(Socrates)
C4, C3                         →  C5: □  (empty clause)
```
Empty clause derived ⇒ `KB ∧ ¬Mortal(Socrates)` is unsatisfiable ⇒ **`KB ⊨ Mortal(Socrates)`.** ✓ ∎

---

## 6. Common GATE question types on this topic

- **English ↔ FOL translation:** pick the correct rendering (MCQ); get the `∀/→`, `∃/∧` pairing right.
- **"No / not all / only / exactly one"** translations, including with equality.
- **Quantifier negation** — push `¬` in and flip quantifiers.
- **Quantifier order / scope** — which mixed-quantifier entailment direction is valid; same-type commuting.
- **Quantifier distribution** — which of the four distribution laws hold.
- **Validity / satisfiability / entailment** of FOL sentences; give a counter-model.
- **Unification / MGU**; occurs-check; non-unifiable pairs.
- **FOL resolution refutation** and the **CNF/Skolemization steps** (constant vs function).
- **UI vs EI** — which inference is being applied; which kind of constant EI must introduce.
- **Free vs bound variables; is it a sentence.**
- **Conceptual true/false:** FOL undecidable but semi-decidable; propositional decidable; resolution refutation-complete; Skolemization preserves satisfiability.

---

## 7. Traps, pitfalls & common mistakes

1. **∀ with ∧ instead of →** ("all As are Bs" written `∀x(A∧B)` — claims everything is an A&B). The single most common error.
2. **∃ with → instead of ∧** ("some A is B" written `∃x(A→B)` — trivially true via any non-A).
3. **Swapping mixed quantifier order:** `∀x∃y ≠ ∃y∀x`. Only `∃y∀x ⊨ ∀x∃y` holds.
4. **Forgetting to flip the quantifier when negating:** `¬∀x P ≡ ∃x ¬P` (**not** `∀x ¬P`); `¬∃x P ≡ ∀x ¬P`.
5. **Bad distribution:** `∀` does **not** distribute over `∨`; `∃` does **not** distribute over `∧`.
6. **UI vs EI confusion:** UI lets you substitute *any* ground term and may repeat; **EI must use a fresh Skolem constant, once.**
7. **Skolemization:** it preserves **satisfiability, not equivalence**; and you need a Skolem **function** (of the enclosing `∀`-variables), not a constant, when the `∃` is inside a `∀`.
8. **Skipping "standardize apart":** reusing variable names across clauses before resolution causes illegal unifications and false proofs.
9. **Occurs check:** `x` cannot unify with a term containing `x` (e.g. `x` vs `f(x)`).
10. **Empty-domain pitfall:** classical FOL assumes a **non-empty domain**; that is *why* `∀x P(x) → ∃x P(x)` is valid. Don't treat it as valid for an empty universe.
11. **Function vs predicate:** a function returns an **object** (a term, used inside atoms); a predicate returns **true/false** (it *is* the atom). `FatherOf(John)` is a term; `Father(John,Bill)` is a sentence.
12. **Equality is special:** `=` needs its own axioms (reflexive/symmetric/transitive + substitution) unless the system builds it in.
13. **English ambiguity:** "All that glitters is not gold" literally reads `∀x(Glitters(x) → ¬Gold(x))` but *means* `∃x(Glitters(x) ∧ ¬Gold(x))` ("not all…"). Watch scope of negation vs quantifier.
14. **FOL vs propositional decidability:** FOL validity is **undecidable** (only semi-decidable); propositional logic is decidable. Don't conflate them.

---

## 8. Connection forward

This finishes pure logic — and exposes its boundary. Logic delivers **certain, all-or-nothing** conclusions: a sentence is true or false, an entailment holds or it doesn't. But the world is **uncertain and exception-ridden**. "Birds fly" is *usually* true, yet penguins don't; a universal `∀x (Bird(x) → Flies(x))` is *false* the instant one exception exists, and FOL has **no way to say "probably,"** "90% of the time," or "more likely than not." Logic is brittle exactly where real evidence is partial and noisy.

The next section — **Reasoning under Uncertainty**, opening with **Conditional Independence Representation (Bayesian networks)** — replaces hard truth values with **probabilities** and replaces logical entailment with **probabilistic inference**. The deep idea you built here *transfers*: just as quantified rules let a small KB encode a huge set of facts compactly, **conditional independence lets a graph factor a gigantic joint probability distribution compactly.** Structure-for-compactness is the through-line; only the currency changes from *truth* to *probability*.

---

## 9. One-line cheat-sheet summary

> **FOL = propositional logic + objects + predicates + functions + quantifiers (∀, ∃).** Pair **∀ with → and ∃ with ∧**; **`¬∀ = ∃¬`, `¬∃ = ∀¬`**; mixed order matters (`∃y∀x ⊨ ∀x∃y`, not the reverse); **∀ distributes over ∧, ∃ over ∨** (the cross pairs don't). Infer by **UI (any term, repeat) / EI (fresh Skolem constant, once)**, **unification + MGU** (mind the occurs-check), and **resolution after CNF + Skolemization** (preserves satisfiability, not equivalence). FOL validity is **semi-decidable & undecidable**; propositional logic is decidable.

---

*Next: a new section begins —* **Reasoning under Uncertainty, Topic 6: Conditional Independence Representation (Bayesian Networks).** *Start it in a fresh chat when ready; that note will open the probabilistic-reasoning arc and link back to the "structure-for-compactness" idea introduced here.*

# Topic 1 — Programming in Python
### Subject: Programming, Data Structures and Algorithms (GATE DA)

---

## 0. Topic header & syllabus map

- **Topic:** Programming in Python
- **Subject:** Programming, Data Structures and Algorithms
- **Where it sits:** This is the **very first** topic of the subject. The entire rest of the syllabus — stacks, queues, linked lists, trees, hash tables, searching, sorting, divide & conquer, and graphs — is read, written, and reasoned about *in Python* in GATE DA. There is no "previous" topic; everything that follows builds on this. It **leads directly into Stacks & Queues**, which are implemented with the Python `list` / `collections.deque` facilities you learn here.

> GATE DA does **not** test you on writing large programs. It tests whether you can **read a short Python snippet and predict its exact output**, and whether you understand the *semantics* (especially mutability and reference behaviour) precisely enough to not fall for traps. That is the lens for everything below.

---

## 1. Why this topic exists (motivation + connection)

Every data structure and algorithm in this subject has to be *expressed* in some notation, and GATE DA chose Python. Before you can reason about whether a stack push is O(1) or whether quicksort partitions correctly, you must be able to read Python fluently and know **exactly** what each line does — including the unintuitive parts (negative modulo, aliasing, mutable default arguments, `for…else`).

- **What "came before" couldn't do:** there is no earlier topic, so the gap this fills is foundational — without a precise mental model of Python's *object/reference* model, you will mis-predict outputs on *every* later topic (e.g., "why did mutating the list inside the function change the caller's list?").
- **What the next topic builds on this:** Stacks & Queues are nothing but disciplined use of `list.append` / `list.pop` (and `collections.deque`). The list semantics, time complexities, and aliasing rules you nail here are *literally* the next topic's building blocks.

---

## 2. Prerequisites

- Basic programming logic: variables, conditionals, loops, functions (conceptually, in any language).
- School arithmetic, including how floor/ceiling and remainders work for negative numbers.
- Comfort reading nested brackets `[ ]`, `{ }`, `( )`.

That's all. This topic *creates* the foundation; it doesn't depend on earlier GATE topics.

---

## 3. Core idea — intuition first

**Python's master concept: variables are *names bound to objects*, not boxes holding values.**

Think of an object (the integer `5`, a list `[1,2,3]`) as a thing floating in memory, and a variable name as a **sticky label** you attach to it. `a = [1,2,3]` sticks the label `a` on a list object. `b = a` sticks a *second* label `b` on the *same* object — it does **not** copy the list.

This single idea explains 80% of GATE Python traps:

```python
a = [1, 2, 3]
b = a            # b and a label the SAME list
b.append(4)
print(a)         # [1, 2, 3, 4]  <-- a "changed" because it's the same object
```

The second master concept is **mutable vs immutable**:

- **Immutable** objects (int, float, complex, bool, str, tuple, frozenset, range, bytes) *cannot be changed in place*. "Modifying" one actually creates a **new** object and rebinds the label.
- **Mutable** objects (list, dict, set, bytearray) *can* be changed in place, which is why aliasing them is dangerous.

```python
x = 5
y = x
y = y + 1        # int is immutable: this makes a NEW int 6 and rebinds y
print(x, y)      # 5 6   <-- x unaffected
```

Hold these two ideas — *names label objects* and *mutable vs immutable* — and the rest is detail.

---

## 4. Formal treatment

Python has no "theorems" to derive; instead the precise *rules of evaluation* are what GATE tests. Each subsection states the rule, the key edge cases, and the conditions under which it holds.

### 4.1 The object / reference model

- Assignment `name = obj` **binds** a name to an object. Never copies a mutable object.
- `id(x)` returns a unique integer identity of the object (think "memory address").
- `x is y` is **identity** (same object?). `x == y` is **equality** (same value?).

```
RULE: x is y      → True iff x and y are the SAME object (id(x)==id(y))
      x == y      → True iff x and y have equal VALUE (uses __eq__)
```

```python
a = [1, 2]; b = [1, 2]
print(a == b)   # True  (same contents)
print(a is b)   # False (two different list objects)
```

**Small-integer caching (classic `is` trap):** CPython pre-creates and reuses integer objects from **−5 to 256**. So `is` may *accidentally* be `True` for small ints and `False` for large ones. **Never use `is` to compare values** — use `==`.

```python
a = 256; b = 256
print(a is b)   # True  (cached)
a = 257; b = 257
print(a is b)   # False (typically — two new objects)
```

> Condition: the caching range (−5…256) is a CPython implementation detail. GATE expects you to know it exists, and to know that the *correct* tool for value comparison is `==`.

### 4.2 Data types at a glance

| Category | Types | Mutable? |
|---|---|---|
| Numeric | `int` (arbitrary precision), `float`, `complex`, `bool` | Immutable |
| Text | `str` | Immutable |
| Sequence | `list` | **Mutable** |
| Sequence | `tuple`, `range` | Immutable |
| Mapping | `dict` | **Mutable** |
| Set | `set` | **Mutable** |
| Set | `frozenset` | Immutable |
| Binary | `bytes` (immutable), `bytearray` (**mutable**) | mixed |

Key sub-facts:
- `bool` is a **subclass of `int`**: `True == 1`, `False == 0`, `True + True == 2`, `isinstance(True, int) == True`.
- `int` has **no overflow** — `2**100` just works (arbitrary precision).
- `dict` keys and `set` elements must be **hashable** ⇒ effectively **immutable** (so a `list` can't be a dict key, but a `tuple` of immutables can).

### 4.3 Numbers & arithmetic operators

```
+    addition
-    subtraction
*    multiplication
/    TRUE division   → ALWAYS returns float (5/2 == 2.5, 4/2 == 2.0)
//   FLOOR division  → floors toward negative infinity
%    modulo          → result takes the SIGN OF THE DIVISOR
**   exponent        → RIGHT associative
```

**The identity that always holds** (for `b ≠ 0`):

```
a == (a // b) * b + (a % b)
```

This *forces* the negative-number behaviour. Memorise the four sign cases:

```python
 7 //  2 ==  3      7 %  2 ==  1
-7 //  2 == -4     -7 %  2 ==  1     # remainder sign follows divisor (+2 → +1)
 7 // -2 == -4      7 % -2 == -1     # divisor −2 → remainder −1
-7 // -2 ==  3     -7 % -2 == -1
```

> Why `-7 // 2 == -4`? Because `-7/2 = -3.5` and floor(−3.5) = **−4** (floor rounds *down*, i.e., toward −∞, not toward zero). This is the single most common NAT trap.

`divmod(a, b)` returns `(a // b, a % b)` in one shot: `divmod(-7, 2) == (-4, 1)`.

**Exponent associativity & unary minus precedence (two separate traps):**

```python
2 ** 3 ** 2   # == 2 ** (3 ** 2) == 2 ** 9 == 512   (** is right associative)
-2 ** 2       # == -(2 ** 2) == -4   (** binds TIGHTER than unary minus)
(-2) ** 2     # == 4
2 ** -1       # == 0.5  (unary minus on the right operand is fine)
```

**Operator precedence (high → low), the GATE-relevant slice:**

```
**                              (right associative)
unary  +x  -x  ~x
*  /  //  %
+  -
<<  >>
&
^
|
comparisons:  <  <=  >  >=  ==  !=  is  is not  in  not in
not
and
or
conditional expression:  X if C else Y
lambda
assignment / walrus :=
```

### 4.4 Boolean logic & truthiness

**Falsy values (everything else is truthy):**

```
False, None, 0, 0.0, 0j, ''  (empty str), []  (empty list),
()  (empty tuple), {}  (empty dict), set()  (empty set)
```

Note `'0'`, `' '`, `[0]`, `[[]]` are all **truthy** (non-empty).

**`and` / `or` return an *operand*, not necessarily a bool (short-circuit):**

```
x and y  → returns x if x is falsy, else y   (stops at first falsy)
x or  y  → returns x if x is truthy, else y   (stops at first truthy)
not x    → returns a real bool (True/False)
```

```python
print(0 or 'a')      # 'a'   (0 falsy → return second)
print('a' or 0)      # 'a'   (first truthy → return it, never evaluate 0)
print('a' and 'b')   # 'b'   (first truthy → return second)
print('' and 'b')    # ''    (first falsy → return it)
print([] or 0)       # 0     (both falsy → return last)
```

### 4.5 Comparison, identity, membership, chaining

- Comparison: `< <= > >= == !=`
- Identity: `is`, `is not`  (object identity — see 4.1)
- Membership: `in`, `not in`  (works on str, list, tuple, set, dict-keys)

**Chained comparison** `a < b < c` is **not** `(a<b) < c`; it is `(a < b) and (b < c)`, and the middle term `b` is **evaluated only once**.

```python
print(1 < 2 < 3)            # True   == (1<2) and (2<3)
print(3 > 2 == 2)           # True   == (3>2) and (2==2)
print(False == False in [False])   # True  == (False==False) and (False in [False])
```

### 4.6 Strings (immutable)

- Indexing: `s[0]` first, `s[-1]` last. Out-of-range index → `IndexError`.
- **Slicing** `s[start:stop:step]` — `stop` is **exclusive**; out-of-range slices **don't** error (they clamp).

```
defaults: start=0 (or len-1 if step<0), stop=len (or "before 0" if step<0), step=1
s[::-1]  reverses the string
```

```python
s = 'abcdef'              # indices: a0 b1 c2 d3 e4 f5
print(s[1:4])    # 'bcd'
print(s[-2:])    # 'ef'
print(s[::2])    # 'ace'
print(s[::-1])   # 'fedcba'
print(s[4:1:-1]) # 'edc'   (e4 d3 c2, stop at index 1 exclusive)
print(s[10:20])  # ''      (out-of-range slice → empty, NO error)
```

- Concatenation `+`, repetition `*`: `'ab'*3 == 'ababab'`, `'5'+'3' == '53'`, `5*'3' == '333'`, but `'5'+3` → `TypeError`.
- Common methods (all **return new strings**, originals unchanged): `upper`, `lower`, `strip`, `split`, `join`, `replace`, `find` (−1 if absent), `index` (raises if absent), `count`, `startswith`, `endswith`, `isdigit`, `isalpha`.

```python
print('a,b,c'.split(','))    # ['a', 'b', 'c']
print('a b  c'.split())      # ['a', 'b', 'c']  (split on whitespace, no empties)
print('-'.join(['a','b']))   # 'a-b'
print('Hello'.replace('l','L'))  # 'HeLLo'
```

- `ord('A') == 65`, `chr(65) == 'A'`.
- f-strings: `f"{x*2}"`, `f"{3.14159:.2f}" == '3.14'`.

### 4.7 Lists (mutable, ordered, heterogeneous)

- Slicing a list returns a **new** list (a *shallow* copy of that slice). Slice **assignment** edits in place.
- Methods:

```
append(x)   add ONE element at end (the element may itself be a list)
extend(it)  add each element of an iterable at end
insert(i,x) insert before index i
pop([i])    remove & return element at i (default last); IndexError if empty
remove(x)   remove first occurrence by VALUE; ValueError if absent
index(x)    first index of value x
count(x)    how many times x appears
sort()      sort IN PLACE, returns None
reverse()   reverse IN PLACE, returns None
copy()      shallow copy
clear()     empty the list
```

**`append` vs `extend` (very common trap):**

```python
a = [1, 2]; a.append([3, 4]); print(a)   # [1, 2, [3, 4]]   (one nested element)
b = [1, 2]; b.extend([3, 4]); print(b)   # [1, 2, 3, 4]     (elements unpacked)
```

**`sort()` returns `None`** (mutating methods return None), whereas `sorted()` returns a new list:

```python
x = [3, 1, 2].sort()          # x is None!
y = sorted([3, 1, 2])         # y == [1, 2, 3], original untouched
sorted([3,1,2], reverse=True) # [3, 2, 1]
sorted(['bb','a','ccc'], key=len)  # ['a', 'bb', 'ccc']
```

**Aliasing & copying:**

```python
a = [1, 2, 3]
b = a            # alias  (same object)
c = a[:]         # shallow copy (new outer list, but inner objects shared)
d = a.copy()     # shallow copy too
import copy
e = copy.deepcopy(a)   # fully independent copy
```

**Shallow copy is only "one level deep"** — nested mutable objects are still shared:

```python
a = [[1, 2], [3, 4]]
b = a[:]          # shallow copy
b[0].append(99)   # mutates the shared inner list
print(a)          # [[1, 2, 99], [3, 4]]   <-- a changed!
b.append([5,6])   # this only affects b (outer level)
print(a)          # still [[1, 2, 99], [3, 4]]
```

**The `[[...] * n] * m` trap** — `*` on a list of mutables makes **m references to the same inner list**:

```python
m = [[0] * 3] * 3     # 3 references to ONE [0,0,0]
m[0][0] = 1
print(m)              # [[1,0,0],[1,0,0],[1,0,0]]  (all rows change!)
# Correct way (independent rows):
g = [[0]*3 for _ in range(3)]
```

**`+=` vs `= +` on lists (aliasing-sensitive):** `l += [x]` mutates **in place** (like `extend`); `l = l + [x]` creates a **new** list and rebinds.

```python
def f(lst): lst += [9]     # mutates the caller's list
def g(lst): lst = lst + [9]  # rebinds local only; caller unaffected
p = [1]; f(p); print(p)    # [1, 9]
q = [1]; g(q); print(q)    # [1]
```

- `[1,2] + [3] == [1,2,3]`, `[1,2]*2 == [1,2,1,2]`.

### 4.8 Tuples (immutable, ordered)

- Packing / unpacking: `t = 1, 2, 3` packs; `a, b, c = t` unpacks; `a, *rest = [1,2,3,4]` → `a=1, rest=[2,3,4]`.
- **Single-element tuple needs the comma:** `(1,)` is a tuple; `(1)` is just the int `1`.
- A tuple is immutable in its **references**, but if it *holds* a mutable object, that object can still change:

```python
t = ([1, 2], 3)
t[0].append(9)     # OK — mutating the inner list
print(t)           # ([1, 2, 9], 3)
# t[0] = [5]       # ERROR — can't rebind a tuple slot
```

### 4.9 Dictionaries (mutable mapping)

- Keys must be **hashable** (immutable). Values can be anything.
- **Insertion-ordered since Python 3.7** (iteration yields keys in insertion order).
- Access & methods:

```
d[k]            value for k; KeyError if absent
d.get(k, dflt)  value for k, else dflt (NO error)
d.setdefault(k, v)  return d[k]; if absent, set d[k]=v first
d.keys() / d.values() / d.items()   views
d.pop(k) / d.update(other)
```

- Iterating a dict iterates its **keys**: `for k in d:` gives keys; use `d.items()` for pairs.
- **Duplicate keys collapse — and `True`, `1`, `1.0` are the *same* key** (equal hash & value). The key keeps the *first-inserted* form; the value is the *last* assigned:

```python
print({1: 'a', 1: 'b'})              # {1: 'b'}
print({True: 'x', 1: 'y', 1.0: 'z'}) # {True: 'z'}  (one entry!)
```

- Dict comprehension: `{k: v for k, v in pairs}`.

### 4.10 Sets (mutable, unordered, unique)

- Elements must be **hashable**. No duplicates. **No ordering** (don't predict element order).
- Set algebra:

```
|   union              & intersection
-   difference         ^ symmetric difference (in exactly one)
```

```python
print({1,2,3} | {2,3,4})  # {1, 2, 3, 4}
print({1,2,3} & {2,3,4})  # {2, 3}
print({1,2,3} - {2,3,4})  # {1}
print({1,2,3} ^ {2,3,4})  # {1, 4}
print({1, True})          # {1}   (1 == True ⇒ collapse)
```

- Methods: `add`, `remove` (KeyError if absent), `discard` (no error if absent).
- **`frozenset`** is the immutable, hashable set — usable as a dict key or set element.
- Note: `{}` is an **empty dict**, *not* an empty set. Empty set is `set()`.

### 4.11 Control flow

- `if / elif / else`; `while`; `for x in iterable`.
- **`range(start, stop, step)`** — `stop` is **exclusive**; empty if direction wrong.

```python
list(range(2, 10, 3))   # [2, 5, 8]
list(range(5, 0, -1))   # [5, 4, 3, 2, 1]
list(range(0))          # []
list(range(5, 5))       # []
```

- `break` exits the nearest loop; `continue` skips to next iteration; `pass` does nothing (placeholder).
- **`for…else` / `while…else` (signature GATE trap):** the `else` block runs **iff the loop finished *without* hitting `break`**.

```python
for i in range(3):
    if i == 5:
        break
else:
    print('completed')      # prints — no break happened

for i in range(3):
    if i == 1:
        break
else:
    print('completed')      # does NOT print — break happened
```

### 4.12 Functions

- `def f(...): ...`; a function with no `return` (or bare `return`) returns **`None`**.
- Parameter kinds: positional, keyword, **default** (`def f(a, b=10)`), variadic `*args` (tuple of extra positionals), `**kwargs` (dict of extra keywords).
- `lambda args: expr` — a one-expression anonymous function.

**Mutable default argument (the most famous Python trap):** default values are evaluated **once**, at definition time, and *reused* across calls. A mutable default therefore **persists** between calls.

```python
def add(x, acc=[]):     # acc created ONCE
    acc.append(x)
    return acc

print(add(1))   # [1]
print(add(2))   # [1, 2]   <-- same list reused!
print(add(3))   # [1, 2, 3]
```

The safe idiom:

```python
def add(x, acc=None):
    if acc is None:
        acc = []
    acc.append(x)
    return acc
```

**Pass-by-object-reference:** the function receives a *reference* to the same object. *Mutating* it affects the caller; *rebinding* the parameter does not.

```python
def mutate(lst): lst.append(99)   # caller sees change
def rebind(lst): lst = [0]        # caller does NOT see change
a = [1]; mutate(a); print(a)      # [1, 99]
b = [1]; rebind(b); print(b)      # [1]
```

### 4.13 Scope — the LEGB rule

Name lookup order: **L**ocal → **E**nclosing (outer function) → **G**lobal (module) → **B**uilt-in.

- **Reading** a global inside a function is fine.
- **Assigning** a name *anywhere* in a function makes that name **local for the whole function body** — so reading it *before* the assignment raises `UnboundLocalError`.

```python
x = 10
def f():
    print(x)     # UnboundLocalError! x is local (assigned below) but used first
    x = 5
```

- `global x` lets a function rebind a module-level variable; `nonlocal x` rebinds a variable in the nearest enclosing function.

```python
x = 1
def g():
    global x
    x = 2
g(); print(x)    # 2
```

### 4.14 Recursion

A function that calls itself. It must have a **base case** (stops recursion) and a **recursive case** (moves toward the base case).

```python
def fact(n):
    if n == 0:           # base case
        return 1
    return n * fact(n-1) # recursive case
print(fact(5))           # 120
```

- Missing/incorrect base case → `RecursionError` (stack overflow); default recursion limit is ~1000.
- Classic exponential example (trace these in exams):

```python
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)
# fib(0..6) = 0,1,1,2,3,5,8
```

### 4.15 Essential built-in functions

```
len(x)                  number of items
range(...)              lazy integer sequence
enumerate(it, start=0)  yields (index, value) pairs
zip(a, b, ...)          yields tuples; STOPS at the shortest iterable
map(f, it)              apply f lazily (returns an iterator in Py3)
filter(f, it)           keep items where f(item) truthy (iterator)
sorted(it, key=, reverse=)  NEW sorted list
sum / min / max / abs   numeric helpers
round(x, ndigits)       banker's rounding (round half to EVEN)
type(x) / isinstance(x, T)   type checks
reversed(seq)           reverse iterator
all(it) / any(it)       all-true / any-true
```

**`round()` uses round-half-to-even (banker's rounding):**

```python
print(round(0.5))   # 0   (0 is even)
print(round(1.5))   # 2   (2 is even)
print(round(2.5))   # 2   (2 is even)
print(round(3.5))   # 4
print(round(2.675, 2))  # 2.67  (float 2.675 is really 2.67499…)
```

**`all([]) == True`**, **`any([]) == False`** (vacuous truth — easy to forget).

```python
list(map(lambda x: x*2, [1,2,3]))     # [2, 4, 6]
list(filter(lambda x: x%2==0, range(6)))  # [0, 2, 4]
list(zip([1,2,3], ['a','b']))         # [(1,'a'), (2,'b')]  (stops at shortest)
list(enumerate(['p','q'], 1))         # [(1,'p'), (2,'q')]
```

### 4.16 Comprehensions & generator expressions

Compact construction of list/set/dict, with optional filter and/or ternary.

```python
[expr for x in it]                  # list comprehension
[expr for x in it if cond]          # with filter
[a if cond else b for x in it]      # with ternary (note: ternary BEFORE the for)
{x for x in it}                     # set comprehension
{k: v for k, v in pairs}            # dict comprehension
(expr for x in it)                  # GENERATOR expression (lazy, parentheses)
```

```python
print([x*x for x in range(5)])            # [0, 1, 4, 9, 16]
print([x for x in range(10) if x%2==0])   # [0, 2, 4, 6, 8]
print(sum(x*x for x in range(4)))         # 0+1+4+9 = 14  (generator, no list built)
```

**Nested comprehension reading order** = the same order you'd write nested loops, left to right:

```python
print([[i*j for j in range(2)] for i in range(2)])  # [[0,0],[0,1]]
# flatten:
M = [[1,2],[3,4]]
print([x for row in M for x in row])      # [1, 2, 3, 4]
```

**Comprehension variables do NOT leak** in Python 3 (unlike Python 2):

```python
[i for i in range(3)]
print(i)     # NameError, if i wasn't defined elsewhere
```

### 4.17 Exception handling (brief — GATE only lightly touches this)

```python
try:
    risky()
except ValueError:
    ...        # runs if a ValueError occurs
except (TypeError, KeyError):
    ...        # multiple types
else:
    ...        # runs ONLY if no exception was raised in try
finally:
    ...        # ALWAYS runs (cleanup), even on return/break
```

Common exceptions to recognise: `IndexError` (bad sequence index), `KeyError` (missing dict key), `ValueError` (right type, bad value, e.g. `int('x')`), `TypeError` (wrong type, e.g. `'a'+3`), `ZeroDivisionError`, `NameError`/`UnboundLocalError`, `AttributeError`, `StopIteration`.

---

## 5. Worked problems (GATE-style)

> Output-prediction is the dominant format. Each problem shows the full reasoning.

### Problem 1 (MCQ) — mutable default argument
```python
def f(x, s=[]):
    s.append(x)
    return s
print(f(1)); print(f(2)); print(f(3))
```
What is printed?
(A) `[1]` `[2]` `[3]`  (B) `[1]` `[1, 2]` `[1, 2, 3]`  (C) `[1]` `[1,2]` `[3]`  (D) error

**Solution.** The default list `s=[]` is created **once** at definition time and reused on every call that omits `s`. So the *same* list accumulates: call 1 → `[1]`, call 2 → `[1, 2]`, call 3 → `[1, 2, 3]`. **Answer: (B).**

---

### Problem 2 (NAT) — floor division & modulo with negatives
Compute the value of `(-17 // 5) * 5 + (-17 % 5)`.

**Solution.** Use the identity `a == (a//b)*b + (a%b)` — it must hold, so the expression equals `a = -17`.
Check directly: `-17/5 = -3.4`, floor = `-4`, so `-17//5 = -4`. Remainder sign follows the divisor `+5`: `-17 = (-4)(5) + 3`, so `-17 % 5 = 3`. Then `(-4)(5) + 3 = -20 + 3 = -17`. **Answer: −17.**

*(Lesson: the reconstruction identity is a free sanity check on any `//`,`%` NAT.)*

---

### Problem 3 (MCQ) — the `[[...]*n]*m` aliasing trap
```python
grid = [[0]*2]*2
grid[0][1] = 5
print(grid)
```
(A) `[[0,5],[0,0]]`  (B) `[[0,5],[0,5]]`  (C) `[[5,0],[5,0]]`  (D) `[[0,0],[0,0]]`

**Solution.** `[0]*2` builds one list `[0,0]`. Wrapping with `*2` stores **two references to that same inner list**. Setting `grid[0][1]=5` mutates the shared list, so *both* rows reflect it. **Answer: (B).**

---

### Problem 4 (MCQ) — short-circuit `and`/`or` return *operands*
```python
print(0 or '' or [] or 'last')
print(2 and 3 and 0 and 4)
```
**Solution.** `or` returns the first **truthy** operand, else the last. `0`, `''`, `[]` are all falsy ⇒ returns `'last'`.
`and` returns the first **falsy** operand, else the last. `2`, `3` truthy, then `0` is falsy ⇒ returns `0` (and never evaluates `4`).
Output:
```
last
0
```

---

### Problem 5 (MCQ) — `for…else` with `break`
```python
def search(lst, t):
    for x in lst:
        if x == t:
            print("found")
            break
    else:
        print("absent")
search([1,2,3], 2)
search([1,2,3], 9)
```
**Solution.** First call finds `2` → prints `found`, `break` taken ⇒ `else` **skipped**. Second call never breaks (no match) ⇒ loop completes ⇒ `else` runs → `absent`.
Output:
```
found
absent
```

---

### Problem 6 (NAT) — exponent associativity + unary minus
Evaluate `-2 ** 2 ** 3`.

**Solution.** `**` is **right associative** and binds **tighter than unary minus**. So this is `-(2 ** (2 ** 3)) = -(2 ** 8) = -(256) = -256`. **Answer: −256.**

---

### Problem 7 (MSQ) — mutation vs rebinding through a function
```python
def p(a):
    a.append(4)
    a = [9]
    a.append(5)
x = [1, 2, 3]
p(x)
print(x)
```
Which statement(s) are correct?
(I) Output is `[1, 2, 3, 4]`.  (II) Output is `[9, 5]`.  (III) `a.append(4)` affects the caller.  (IV) `a = [9]` affects the caller.

**Solution.** `a` initially references the same list as `x`. `a.append(4)` *mutates* it ⇒ caller's list becomes `[1,2,3,4]` (III true). Then `a = [9]` **rebinds** `a` to a brand-new local list; `x` is untouched (IV false). The later `append(5)` acts on the local `[9]`, irrelevant to `x`. So output is `[1,2,3,4]`. **Correct: (I) and (III).**

---

### Problem 8 (MCQ) — dict key collapse with `True`/`1`/`1.0`
```python
d = {}
d[1] = 'a'
d[True] = 'b'
d[1.0] = 'c'
print(len(d), d[1])
```
(A) `3 a`  (B) `1 c`  (C) `2 b`  (D) `1 a`

**Solution.** `1`, `True`, and `1.0` are **equal in value and hash**, so they are the *same* key. There is one entry; each assignment overwrites the value. Final value is `'c'` and `len(d) == 1`. **Answer: (B).**

---

### Problem 9 (MCQ) — slicing with a negative step
```python
s = "PYTHON"
print(s[4:1:-1])
```
(A) `OHT`  (B) `NOH`  (C) `OHTY`  (D) `THO`

**Solution.** Indices: P0 Y1 T2 H3 O4 N5. Start at 4 (`O`), step `-1`, stop **before** index 1: visit 4(`O`), 3(`H`), 2(`T`), then index 1 is excluded. Result `"OHT"`. **Answer: (A).**

---

### Problem 10 (NAT) — `round()` banker's rounding in a sum
What does `round(0.5) + round(1.5) + round(2.5)` evaluate to?

**Solution.** Round-half-to-even: `round(0.5)=0`, `round(1.5)=2`, `round(2.5)=2`. Sum `= 0 + 2 + 2 = 4`. **Answer: 4.** *(A naïve "round half up" gives 1+2+3 = 6 — the trap.)*

---

### Problem 11 (MCQ) — scope / `UnboundLocalError`
```python
count = 0
def inc():
    count = count + 1
    return count
print(inc())
```
(A) `1`  (B) `0`  (C) `UnboundLocalError`  (D) `None`

**Solution.** Because `count` is **assigned** inside `inc`, it is local to the whole function. The right-hand side `count + 1` reads `count` **before** it has a local value ⇒ `UnboundLocalError`. (Adding `global count` would fix it and make it print `1`.) **Answer: (C).**

---

### Problem 12 (MCQ) — comprehension with filter + flatten
```python
M = [[1,2,3],[4,5,6]]
print([x for row in M for x in row if x % 2 == 0])
```
(A) `[2,4,6]`  (B) `[1,3,5]`  (C) `[[2],[4,6]]`  (D) `[2,6]`

**Solution.** Read like nested loops: outer `for row in M`, inner `for x in row`, keep even `x`. Evens are `2, 4, 6`. **Answer: (A).**

---

## 6. Common GATE question types on this topic

- **Predict exact output** of a short snippet (the dominant format).
- **NAT arithmetic** with `//`, `%`, `**` — especially **negative operands** and **unary-minus-vs-power** precedence.
- **MSQ on semantics:** "which statements are true" about mutability, aliasing, shallow vs deep copy, pass-by-object-reference.
- **Trace control flow:** loops with `break`/`continue` and the `for…else` / `while…else` rule.
- **Function-argument behaviour:** mutable default argument; mutation vs rebinding of a passed list/dict.
- **Scope:** `UnboundLocalError`, effect of `global` / `nonlocal`, LEGB lookup.
- **Data-structure operations:** dict key collapse, set algebra (`| & - ^`), list `append` vs `extend`, slicing (incl. negative step), comprehension reading order.
- **Built-ins:** `round` banker's rounding, `zip` stopping at shortest, `sorted` vs `.sort()`, `all([])`/`any([])`.
- **Recursion tracing:** factorial / Fibonacci-style call counting.

---

## 7. Traps, pitfalls & common mistakes

1. **Floor division rounds toward −∞, not toward 0:** `-7 // 2 == -4` (not −3). Always recheck negatives with `a == (a//b)*b + (a%b)`.
2. **Modulo sign follows the divisor:** `7 % -2 == -1`, `-7 % 2 == 1`.
3. **`**` is right-associative and beats unary minus:** `2**3**2 == 512`; `-2**2 == -4`.
4. **`/` always returns a float:** `4/2 == 2.0` (a `float`), `4//2 == 2` (an `int`).
5. **Aliasing:** `b = a` does **not** copy a list/dict/set; mutating one shows in the other.
6. **Shallow copy is one level deep:** `a[:]`, `.copy()`, `list(a)`, `copy.copy` share *nested* mutables. Use `copy.deepcopy` for full independence.
7. **`[[x]*n]*m` shares inner lists** — use a comprehension `[[x]*n for _ in range(m)]`.
8. **`append` adds one element; `extend` adds many.** `[1].append([2,3]) → [1,[2,3]]`.
9. **Mutating methods return `None`:** `lst.sort()`, `lst.reverse()`, `lst.append(...)` all return `None`; `x = [3,1].sort()` makes `x = None`.
10. **Mutable default argument persists across calls** — use `None` sentinel.
11. **`for…else`/`while…else`:** the `else` runs **only if no `break`** occurred (not "if the loop ran").
12. **`and`/`or` return an operand, not a bool:** `2 or 3 == 2`, `'' or 5 == 5`.
13. **Falsy set:** `0, 0.0, 0j, '', [], (), {}, set(), None, False`. Watch `'0'`, `' '`, `[0]`, `[[]]` — all **truthy**.
14. **`UnboundLocalError`:** assigning a name anywhere in a function makes it local for the *whole* body; reading it earlier fails. Add `global`/`nonlocal` if you mean the outer one.
15. **Dict/set key identity of numbers:** `True == 1 == 1.0` ⇒ they collapse to one key/element; value/last-write wins; first-inserted key form is kept.
16. **`{}` is an empty dict, not a set;** the empty set is `set()`.
17. **Slicing never raises on out-of-range bounds** (`s[5:99]` → `''`), but **indexing does** (`s[99]` → `IndexError`).
18. **Single-element tuple needs a comma:** `(1,)` is a tuple, `(1)` is an int.
19. **`round` is banker's rounding (half to even):** `round(2.5)==2`, `round(0.5)==0`; and float storage gives `round(2.675,2)==2.67`.
20. **`zip` stops at the shortest** iterable; **`map`/`filter` return iterators** in Python 3 (wrap in `list()` to see contents).
21. **`all([]) == True`, `any([]) == False`** (vacuous cases).
22. **Comprehension variables don't leak** in Python 3; don't expect the loop variable afterwards.
23. **Chained comparison** `a < b < c` ≡ `(a<b) and (b<c)`, with `b` evaluated once — not `(a<b) < c`.
24. **`is` vs `==`:** use `==` for values; `is` for identity/`None`. Small-int caching (−5…256) can make `is` *deceptively* True.
25. **`+=` on a list mutates in place** (affects aliases / caller); `l = l + [...]` rebinds and does not.

---

## 8. Connection forward

You now read Python output exactly and know precisely how lists behave under mutation and aliasing — including `append`/`pop` and their O(1) amortised cost. That is the *entire toolbox* for the **next topic, Stacks & Queues**: a stack is just a list with `append` (push) and `pop()` (pop from the end), and a queue is efficiently realised with `collections.deque` (`append`/`popleft`) precisely because list `pop(0)` is O(n). Every aliasing and complexity insight here transfers directly.

---

## 9. One-line cheat-sheet summary

**Variables are name-tags on objects; master *mutable-vs-immutable* + reference/aliasing, and watch the classics — floor division/modulo signs, right-associative `**` (beats unary minus), `for…else`, mutable default args, shallow vs deep copy, banker's `round`, and dict-key collapse of `True/1/1.0`.**

---

### ➡️ Next topic (in order): **Topic 2 — Stacks & Queues** (Chat 1, Python & linear data structures).

---
tags: [coding, python, itertools, stdlib]
type: cheatsheet
source: compiled reference (Python 3 itertools)
last-verified: 2026-08-27
---

# itertools

## Up
- [[Python]]

`itertools` is Python's standard-library module of fast, memory-efficient building blocks for iterators. Everything returns a **lazy iterator** (values produced on demand), so it works on infinite streams and huge datasets without materialising them. Import: `import itertools` (often `from itertools import ...`). Pairs well with [[Threading]]-free streaming pipelines and generator expressions.

```python
import itertools as it
```

> Iterators are **single-use** — once consumed they're exhausted. Wrap in `list(...)` to see/reuse results, but only when the data is finite and fits in memory.

---

## Infinite Iterators

| Function | Produces | Example |
|---|---|---|
| `count(start=0, step=1)` | `start, start+step, …` forever | `count(10, 2)` → 10, 12, 14, … |
| `cycle(iterable)` | repeats the sequence endlessly | `cycle("AB")` → A, B, A, B, … |
| `repeat(obj, times=None)` | `obj` repeated (n times, or forever) | `repeat(5, 3)` → 5, 5, 5 |

```python
from itertools import count, cycle, repeat, islice

for i in islice(count(1), 5):        # cap an infinite iterator
    print(i)                          # 1 2 3 4 5

list(islice(cycle("AB"), 5))          # ['A','B','A','B','A']
list(repeat(0, 4))                    # [0, 0, 0, 0]

# common idioms
list(zip(count(1), "abc"))            # [(1,'a'),(2,'b'),(3,'c')] — like enumerate(start=1)
list(map(pow, range(5), repeat(2)))   # squares: [0,1,4,9,16]
```

---

## Terminating Iterators (finite)

### accumulate — running totals / reductions

```python
from itertools import accumulate
import operator

list(accumulate([1, 2, 3, 4]))                 # [1, 3, 6, 10]  (running sum)
list(accumulate([1, 2, 3, 4], operator.mul))   # [1, 2, 6, 24]  (running product)
list(accumulate([3, 1, 4, 1, 5], max))         # [3, 3, 4, 4, 5] (running max)
list(accumulate([1, 2, 3], initial=100))       # [100, 101, 103, 106]  (3.8+)
```

### chain — concatenate iterables

```python
from itertools import chain

list(chain([1, 2], [3, 4], [5]))               # [1, 2, 3, 4, 5]
list(chain.from_iterable([[1, 2], [3, 4]]))    # [1, 2, 3, 4]  (flatten one level)
```

### compress — filter by a selector mask

```python
from itertools import compress
list(compress("ABCDEF", [1, 0, 1, 0, 1, 1]))   # ['A', 'C', 'E', 'F']
```

### dropwhile / takewhile — trim by a predicate

```python
from itertools import dropwhile, takewhile

list(takewhile(lambda x: x < 5, [1, 3, 6, 1]))  # [1, 3]   (stop at first False)
list(dropwhile(lambda x: x < 5, [1, 3, 6, 1]))  # [6, 1]   (skip while True, then keep rest)
```

### filterfalse — the inverse of filter

```python
from itertools import filterfalse
list(filterfalse(lambda x: x % 2, range(10)))   # [0, 2, 4, 6, 8]  (keeps where predicate is False)
```

### islice — slice an iterator (no indexing needed)

```python
from itertools import islice
list(islice(range(100), 5))          # [0,1,2,3,4]      (stop)
list(islice(range(100), 5, 10))      # [5,6,7,8,9]      (start, stop)
list(islice(range(100), 0, 20, 5))   # [0,5,10,15]      (start, stop, step)
next(islice(gen, 3, 4))              # grab the 4th item of a generator
```

### starmap — map over pre-zipped argument tuples

```python
from itertools import starmap
list(starmap(pow, [(2, 3), (2, 10)]))           # [8, 1024]  (calls pow(2,3), pow(2,10))
```

### tee — split one iterator into n independent ones

```python
from itertools import tee
a, b = tee([1, 2, 3], 2)
list(a); list(b)                     # both yield [1, 2, 3]
# NOTE: don't keep advancing the original after tee(); tee buffers internally
```

### zip_longest — zip padding the short iterables

```python
from itertools import zip_longest
list(zip_longest([1, 2, 3], "ab", fillvalue="?"))
# [(1,'a'), (2,'b'), (3,'?')]
```

### pairwise — consecutive overlapping pairs (3.10+)

```python
from itertools import pairwise
list(pairwise([1, 2, 3, 4]))         # [(1,2), (2,3), (3,4)]
# handy for deltas: [b - a for a, b in pairwise(readings)]
```

---

## groupby — group consecutive equal keys

```python
from itertools import groupby

data = [("a", 1), ("a", 2), ("b", 3), ("a", 4)]
for key, group in groupby(data, key=lambda x: x[0]):
    print(key, list(group))
# a [('a',1),('a',2)]
# b [('b',3)]
# a [('a',4)]        ← note: only groups ADJACENT items
```

**Gotcha:** `groupby` groups only *consecutive* runs — sort first if you want all like keys together:

```python
data.sort(key=lambda x: x[0])
groups = {k: list(g) for k, g in groupby(data, key=lambda x: x[0])}
```

---

## Combinatoric Iterators

```python
from itertools import product, permutations, combinations, combinations_with_replacement

# product — Cartesian product (nested loops flattened)
list(product([1, 2], ["a", "b"]))
# [(1,'a'), (1,'b'), (2,'a'), (2,'b')]
list(product([0, 1], repeat=3))               # all 3-bit combos: (0,0,0)…(1,1,1)

# permutations — order matters, no repeats
list(permutations([1, 2, 3], 2))
# [(1,2),(1,3),(2,1),(2,3),(3,1),(3,2)]

# combinations — order doesn't matter, no repeats
list(combinations([1, 2, 3], 2))
# [(1,2),(1,3),(2,3)]

# combinations_with_replacement — order doesn't matter, repeats allowed
list(combinations_with_replacement([1, 2, 3], 2))
# [(1,1),(1,2),(1,3),(2,2),(2,3),(3,3)]
```

| Function | Order matters? | Repeats? | Count (n items, r) |
|---|---|---|---|
| `product(..., repeat=r)` | yes | yes | nʳ |
| `permutations(n, r)` | yes | no | n! / (n−r)! |
| `combinations(n, r)` | no | no | n! / (r!(n−r)!) |
| `combinations_with_replacement(n, r)` | no | yes | (n+r−1)! / (r!(n−1)!) |

---

## Common Recipes

```python
from itertools import chain, islice, tee, count

# flatten a list of lists (one level)
flat = list(chain.from_iterable(nested))

# chunk/batch an iterable into size-n pieces
def batched(iterable, n):
    it = iter(iterable)
    while (chunk := tuple(islice(it, n))):
        yield chunk
list(batched(range(7), 3))           # [(0,1,2),(3,4,5),(6,)]
# Python 3.12+ ships this as itertools.batched(iterable, n)

# sliding window of width n
def window(seq, n):
    it = iter(seq)
    win = tuple(islice(it, n))
    if len(win) == n: yield win
    for x in it:
        win = win[1:] + (x,)
        yield win

# take first n / nth element
take   = lambda n, it: list(islice(it, n))
nth    = lambda it, n, default=None: next(islice(it, n, None), default)

# round-robin, unique, etc. live in the official "itertools recipes" docs
```

---

## Why Use itertools

- **Lazy & streaming** — processes data one item at a time; works on files, generators, and infinite sequences without loading everything into memory.
- **Fast** — implemented in C; looping in `itertools` beats equivalent hand-written Python loops.
- **Composable** — chain the tools into pipelines (`islice(filterfalse(pred, chain(a, b)), 100)`).
- Reach for it whenever you'd otherwise write nested loops, manual index bookkeeping, or accumulate large intermediate lists you don't actually need.

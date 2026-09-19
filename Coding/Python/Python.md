---
tags: [coding, python]
type: cheatsheet
source: compiled reference (Python 3)
last-verified: 2026-08-27
---

# Python

## Up
- [[Coding]]

## Subtopics
- [[SimpleHTTPServer]] — built-in one-command static file server (http.server)
- [[Requests]] — HTTP client library
- [[BeautifulSoup]] — HTML/XML parsing & web scraping
- [[Threading]] — concurrency with threads, locks & queues
- [[itertools]] — lazy iterator building blocks & combinatorics
- [[Pandas]] — tabular data (DataFrame/Series)
- [[NumPy]] — N-dimensional arrays & vectorised math
- [[Flask]] — lightweight web micro-framework
- [[FastAPI]] — async, type-hint-driven API framework
- [[Scikit-learn]] — classical machine learning
- [[LangChain]] — building LLM applications
- [[asyncio]] — async/await concurrency (event loop, tasks)
- [[pytest]] — testing (fixtures, parametrize, mocking)
- [[SQLAlchemy]] — SQL toolkit & ORM
- [[Pydantic]] — data validation & settings
- [[Celery]] — distributed task queue
- [[Django]] — batteries-included web framework
- [[Packaging]] — venv/pip, pyproject, Poetry & uv

A practical Python 3 cheatsheet — core syntax, built-in data structures, functions, classes, common standard-library modules, and idioms.

---

## Running & Tooling

```bash
python3 script.py                 # run a script
python3 -m module_name            # run a module as a script
python3 -c "print('hi')"          # run an inline command
python3 -m venv .venv             # create a virtual environment
source .venv/bin/activate         # activate it (Linux/macOS)
pip install requests              # install a package
pip install -r requirements.txt   # install from a file
pip freeze > requirements.txt     # pin current deps
python3 -m pip install --upgrade pip
```

Popular tooling: `ruff`/`flake8` (lint), `black` (format), `mypy`/`pyright` (types), `pytest` (tests), `uv`/`poetry`/`pip-tools` (env & deps).

---

## Variables & Types

```python
x = 10                # int
pi = 3.14             # float
name = "Ada"          # str
flag = True           # bool
nothing = None        # NoneType
c = 2 + 3j            # complex

type(x)               # <class 'int'>
isinstance(x, int)    # True
int("42"); float("3.5"); str(42); bool(0)   # conversions
```

Numbers are arbitrary-precision ints and IEEE-754 floats. Use `//` for floor division, `%` modulo, `**` power, `divmod(a, b)` for both.

### f-strings

```python
n, price = 3, 19.5
f"{n} items cost ${price:.2f}"      # '3 items cost $19.50'
f"{n=}"                              # 'n=3'  (debug form)
f"{n:>5}" ; f"{n:05d}" ; f"{255:#x}" # alignment / padding / hex
f"{price:,.2f}"                      # thousands separator
```

---

## Strings

```python
s = "Hello, World"
s.lower(); s.upper(); s.title(); s.strip()
s.replace("l", "L")
s.split(", ")            # ['Hello', 'World']
", ".join(["a", "b"])    # 'a, b'
s.startswith("He"); s.endswith("ld"); "World" in s
s.find("World")          # index or -1
s[0]; s[-1]; s[0:5]; s[::-1]   # index / slice / reverse
len(s)
"  x  ".strip(); "xxabcxx".strip("x")
"abc".zfill(5); "5".rjust(3, "0")
"a,b,,c".split(",")      # ['a','b','','c']
"line1\nline2".splitlines()
```

Strings are immutable. Raw strings `r"\n"` disable escapes; bytes are `b"..."`.

---

## Collections

### List (ordered, mutable)

```python
nums = [3, 1, 2]
nums.append(4); nums.extend([5, 6]); nums.insert(0, 0)
nums.remove(1); nums.pop(); nums.pop(0)
nums.sort(); nums.sort(reverse=True); nums.sort(key=abs)
sorted(nums); nums.reverse()
nums.index(2); nums.count(3)
nums[1:3]; nums[::2]; nums[::-1]     # slicing
[x*2 for x in nums if x > 1]         # comprehension
```

### Tuple (ordered, immutable)

```python
point = (1, 2)
x, y = point            # unpacking
a, *rest = (1, 2, 3, 4) # a=1, rest=[2,3,4]
single = (1,)           # trailing comma required
```

### Dict (key → value)

```python
d = {"a": 1, "b": 2}
d["c"] = 3
d.get("z", 0)                 # default if missing
d.setdefault("d", 4)
d.keys(); d.values(); d.items()
d.pop("a"); d.update({"e": 5})
"b" in d
{k: v*2 for k, v in d.items()}       # dict comprehension
for k, v in d.items(): ...
merged = {**d1, **d2}                 # merge (or d1 | d2)
```

### Set (unique, unordered)

```python
s = {1, 2, 3}
s.add(4); s.discard(2)
s | t   # union      s & t  # intersection
s - t   # difference s ^ t  # symmetric difference
{x for x in range(10) if x % 2 == 0}  # set comprehension
```

---

## Control Flow

```python
if x > 0:
    ...
elif x == 0:
    ...
else:
    ...

# ternary
label = "pos" if x > 0 else "non-pos"

for i in range(5):        # 0..4
    if i == 3: break
    if i == 1: continue
else:                     # runs if no break
    print("done")

while cond:
    ...

# match (Python 3.10+)
match command:
    case "start": ...
    case "stop" | "halt": ...
    case _: ...           # default
```

`range(start, stop, step)`, `enumerate(seq, start=0)`, `zip(a, b)`, `reversed(seq)` are the common loop helpers.

---

## Functions

```python
def greet(name, greeting="Hello", *args, **kwargs):
    return f"{greeting}, {name}"

greet("Ada"); greet("Ada", greeting="Hi")

# type hints
def add(a: int, b: int) -> int:
    return a + b

# lambda
square = lambda x: x ** 2

# unpacking into calls
args = [1, 2]; kw = {"greeting": "Yo"}
greet(*["Ada"], **kw)

# keyword-only / positional-only
def f(a, /, b, *, c): ...   # a positional-only, c keyword-only
```

`*args` collects extra positionals into a tuple; `**kwargs` collects extra keywords into a dict.

### Useful functional built-ins

```python
list(map(str, [1, 2, 3]))
list(filter(lambda x: x > 0, nums))
sum(nums); min(nums); max(nums); any(flags); all(flags)
sorted(items, key=lambda t: t[1], reverse=True)
from functools import reduce, cache, lru_cache, partial
```

---

## Comprehensions & Generators

```python
[x*x for x in range(10)]              # list
{x for x in range(10)}                # set
{x: x*x for x in range(5)}            # dict
(x*x for x in range(10))              # generator (lazy)

def countdown(n):                     # generator function
    while n > 0:
        yield n
        n -= 1

nums = [row[0] for row in matrix for _ in row]   # nested
```

---

## Classes

```python
class Animal:
    kingdom = "Animalia"              # class attribute

    def __init__(self, name):
        self.name = name              # instance attribute

    def speak(self):
        return f"{self.name} makes a sound"

    def __repr__(self):
        return f"Animal({self.name!r})"

class Dog(Animal):                    # inheritance
    def speak(self):
        return f"{self.name} barks"   # override
    # super().__init__(...) to call parent

d = Dog("Rex")
d.speak(); isinstance(d, Animal)
```

### Dataclasses & properties

```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: int
    y: int = 0
    tags: list = field(default_factory=list)

class Circle:
    def __init__(self, r): self._r = r
    @property
    def area(self): return 3.14159 * self._r ** 2

@staticmethod / @classmethod   # decorators for static/class methods
```

Common dunder methods: `__init__`, `__repr__`, `__str__`, `__eq__`, `__len__`, `__iter__`, `__getitem__`, `__enter__`/`__exit__` (context managers), `__call__`.

---

## Exceptions

```python
try:
    risky()
except (ValueError, KeyError) as e:
    print("bad input:", e)
except Exception as e:
    raise RuntimeError("wrapped") from e
else:
    print("no error")            # runs if no exception
finally:
    cleanup()                    # always runs

raise ValueError("message")

class MyError(Exception):        # custom exception
    pass

assert x > 0, "x must be positive"
```

---

## Files & Context Managers

```python
with open("file.txt", "r", encoding="utf-8") as f:
    text = f.read()              # whole file
    # f.readline(); f.readlines(); for line in f: ...

with open("out.txt", "w") as f:  # 'a' append, 'x' create, 'b' binary
    f.write("hi\n")
    f.writelines(["a\n", "b\n"])

from pathlib import Path
p = Path("dir/file.txt")
p.exists(); p.is_file(); p.suffix; p.stem; p.name; p.parent
p.read_text(); p.write_text("x")
Path(".").glob("*.py"); list(Path(".").rglob("*.md"))
p.mkdir(parents=True, exist_ok=True)
```

---

## Standard Library Highlights

```python
import os, sys, re, json, math, random, datetime, itertools, collections

os.environ.get("HOME"); os.getcwd(); os.listdir(".")
sys.argv; sys.exit(1)

# JSON
json.dumps({"a": 1}); json.loads('{"a": 1}')
json.dump(obj, open("f.json","w")); json.load(open("f.json"))

# regex
re.search(r"\d+", s); re.findall(r"\w+", s)
re.sub(r"\s+", " ", s); re.match(r"^abc", s)
m = re.search(r"(\d{4})-(\d{2})", s); m.group(1)

# datetime
from datetime import datetime, timedelta, timezone
now = datetime.now(timezone.utc)
now.strftime("%Y-%m-%d %H:%M"); datetime.fromisoformat("2026-08-27")
now + timedelta(days=7)

# collections
from collections import Counter, defaultdict, deque, namedtuple, OrderedDict
Counter("aabbbc")             # {'b':3,'a':2,'c':1}
dd = defaultdict(list); dd["k"].append(1)
dq = deque([1,2]); dq.appendleft(0); dq.pop()

# itertools
from itertools import chain, product, permutations, combinations, groupby, count, cycle
list(chain([1,2],[3,4])); list(combinations([1,2,3], 2))
```

---

## Idioms & Gotchas

```python
# swap
a, b = b, a

# default arg trap — never use a mutable default
def f(x, acc=None):
    if acc is None: acc = []

# truthiness: [], {}, "", 0, None are falsy
if items: ...            # not len(items) > 0

# enumerate instead of range(len(...))
for i, v in enumerate(seq): ...

# walrus (3.8+)
while (line := input()) != "quit": ...

# ternary chains and any/all
ok = all(x > 0 for x in nums)

# chained comparison
if 0 < x < 10: ...

# is vs ==  — use `is` only for None/singletons
if x is None: ...
```

- Everything is an object; variables are references. Assignment never copies — use `copy.copy`/`copy.deepcopy` when you need a real copy.
- Integer/float division: `7 / 2 == 3.5`, `7 // 2 == 3`.
- Prefer f-strings over `%`/`.format()`; prefer `pathlib` over `os.path`.

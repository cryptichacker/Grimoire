---
tags: [coding, python, numpy, data, math]
type: cheatsheet
source: compiled reference (NumPy)
last-verified: 2026-08-27
---

# NumPy

## Up
- [[Python]]

NumPy is the foundation of scientific Python — the N-dimensional array (`ndarray`) plus fast vectorised math. Underlies [[Pandas]], [[Scikit-learn]], and most numeric libraries. Import: `import numpy as np`.

---

## Creating Arrays

```python
import numpy as np

np.array([1, 2, 3])                 # from a list
np.array([[1, 2], [3, 4]])          # 2-D
np.zeros((3, 4)); np.ones((2, 3)); np.full((2, 2), 7)
np.eye(3)                           # identity
np.arange(0, 10, 2)                 # [0 2 4 6 8]
np.linspace(0, 1, 5)                # 5 evenly spaced 0..1
np.random.rand(2, 3)                # uniform [0,1)
np.random.randn(3)                  # standard normal
np.random.randint(0, 10, size=5)
np.random.seed(42)                  # reproducibility
np.empty((2, 2)); np.zeros_like(a)
```

---

## Array Attributes & dtypes

```python
a.shape        # dimensions, e.g. (3, 4)
a.ndim         # number of axes
a.size         # total elements
a.dtype        # e.g. int64, float64
a.itemsize; a.nbytes

a.astype(np.float32)
np.array([1, 2], dtype=np.float64)
```

---

## Indexing & Slicing

```python
a = np.arange(10)
a[2]; a[-1]; a[2:5]; a[::2]; a[::-1]

m = np.arange(12).reshape(3, 4)
m[1, 2]           # row 1, col 2
m[0]              # first row
m[:, 1]           # second column
m[0:2, 1:3]       # sub-block
m[m > 5]          # boolean mask → 1-D array of matches
m[[0, 2]]         # fancy indexing: rows 0 and 2
m[m > 5] = 0      # masked assignment
np.where(m > 5, m, 0)     # vectorised if/else
```

Slices are **views** (share memory) — use `.copy()` for an independent array.

---

## Reshaping & Combining

```python
a.reshape(3, 4); a.reshape(-1, 2)   # -1 = infer
a.ravel(); a.flatten()              # to 1-D (view / copy)
a.T; np.transpose(a)                # transpose
a[:, np.newaxis]                    # add an axis
np.expand_dims(a, 0); np.squeeze(a)

np.concatenate([a, b], axis=0)
np.vstack([a, b]); np.hstack([a, b])
np.stack([a, b], axis=1)
np.split(a, 3); np.array_split(a, 2)
```

---

## Element-wise Math (vectorised)

```python
a + b; a - b; a * b; a / b; a ** 2   # element-wise
a + 10                               # broadcast scalar
np.sqrt(a); np.exp(a); np.log(a); np.abs(a)
np.sin(a); np.round(a, 2); np.clip(a, 0, 1)
np.floor(a); np.ceil(a)
a @ b; np.dot(a, b); np.matmul(A, B) # matrix / dot product
```

**Broadcasting**: NumPy stretches smaller arrays across larger ones when trailing dimensions match (or are 1), e.g. a `(3,4)` matrix `*` a `(4,)` vector operates row-wise.

---

## Aggregations

```python
a.sum(); a.mean(); a.std(); a.var()
a.min(); a.max(); a.argmin(); a.argmax()
a.cumsum(); a.cumprod()
np.median(a); np.percentile(a, 95)

# along an axis (0 = down columns, 1 = across rows)
m.sum(axis=0)      # column sums
m.mean(axis=1)     # row means
m.max(axis=0)
np.all(a > 0); np.any(a == 5)
np.unique(a, return_counts=True)
```

---

## Comparison & Logic

```python
a > 5; a == b
np.array_equal(a, b)
(a > 2) & (a < 8)              # element-wise, parenthesize
np.logical_and(x, y); np.logical_or(x, y)
np.isnan(a); np.isinf(a)
np.count_nonzero(a > 5)
np.argwhere(a > 5)            # indices where condition holds
```

---

## Linear Algebra & Random

```python
np.linalg.inv(A)              # inverse
np.linalg.det(A)              # determinant
np.linalg.solve(A, b)         # solve Ax = b
vals, vecs = np.linalg.eig(A) # eigen-decomposition
np.linalg.norm(v)             # vector/matrix norm
U, S, Vt = np.linalg.svd(A)

rng = np.random.default_rng(42)   # modern Generator API (preferred)
rng.normal(0, 1, size=(3, 3))
rng.integers(0, 10, size=5)
rng.choice([1, 2, 3], size=10, p=[.5, .3, .2])
rng.shuffle(a)
```

---

## Handling NaN & I/O

```python
np.nanmean(a); np.nansum(a)           # ignore NaNs
a[np.isnan(a)] = 0

np.save("arr.npy", a); np.load("arr.npy")
np.savetxt("a.csv", a, delimiter=","); np.loadtxt("a.csv", delimiter=",")
np.savez("bundle.npz", x=a, y=b)      # multiple arrays
```

---

## Tips

- Prefer vectorised ops over Python loops — they're often 10–100× faster.
- Watch dtype: integer arrays truncate on division unless cast to float.
- `axis=0` collapses rows (per-column result); `axis=1` collapses columns (per-row result).
- Slices are views; mutating a slice mutates the original. `.copy()` when in doubt.
- Use `np.random.default_rng()` (Generator) over the legacy `np.random.*` functions for new code.

---
tags: [coding, python, pandas, data]
type: cheatsheet
source: compiled reference (pandas)
last-verified: 2026-08-27
---

# Pandas

## Up
- [[Python]]

pandas is the standard Python library for tabular data — the `DataFrame` (2-D labelled table) and `Series` (1-D labelled array). Import convention: `import pandas as pd`.

---

## Creating & Loading

```python
import pandas as pd

df = pd.DataFrame({"name": ["Ada", "Bob"], "age": [30, 25]})
s  = pd.Series([1, 2, 3], name="nums")

df = pd.read_csv("file.csv")            # sep=, header=, names=, dtype=, parse_dates=
df = pd.read_csv("f.csv", index_col=0, na_values=["NA", ""])
df = pd.read_excel("f.xlsx", sheet_name="Sheet1")
df = pd.read_json("f.json"); pd.read_parquet("f.parquet")
df = pd.read_sql("SELECT * FROM t", conn)

df.to_csv("out.csv", index=False)
df.to_excel("out.xlsx", index=False); df.to_parquet("out.parquet")
```

---

## Inspecting

```python
df.head(); df.tail(3)
df.shape          # (rows, cols)
df.columns; df.index; df.dtypes
df.info()         # types + non-null counts + memory
df.describe()     # summary stats for numeric cols
df.describe(include="all")
df.sample(5)
df["col"].unique(); df["col"].nunique(); df["col"].value_counts()
df.memory_usage(deep=True)
```

---

## Selecting Data

```python
df["age"]                 # a column (Series)
df[["name", "age"]]       # multiple columns (DataFrame)

# label-based
df.loc[0]                 # row by label
df.loc[0, "age"]
df.loc[df["age"] > 25, ["name"]]
df.loc[0:2, "name":"age"] # inclusive label slice

# position-based
df.iloc[0]                # first row
df.iloc[0:2, 0:2]         # rows/cols by position
df.iloc[-1]               # last row

df.at[0, "age"]; df.iat[0, 1]     # fast scalar access
```

### Boolean filtering

```python
df[df["age"] > 25]
df[(df["age"] > 20) & (df["name"] == "Ada")]   # & | ~, parenthesize
df[df["name"].isin(["Ada", "Bob"])]
df[df["name"].str.startswith("A")]
df.query("age > 25 and name == 'Ada'")
df[df["col"].between(10, 20)]
```

---

## Cleaning & Missing Data

```python
df.isna(); df.isna().sum()          # count NaNs per column
df.dropna(); df.dropna(subset=["age"], how="any")
df.fillna(0); df.fillna({"age": df["age"].mean()})
df.fillna(method="ffill")           # forward fill

df.drop_duplicates(subset=["name"], keep="first")
df.rename(columns={"age": "years"})
df.drop(columns=["col"]); df.drop(index=[0, 1])
df.astype({"age": "int64"})
df["age"] = pd.to_numeric(df["age"], errors="coerce")
df["date"] = pd.to_datetime(df["date"])
df.columns = df.columns.str.lower().str.strip()
df["s"].str.lower().str.replace(" ", "_").str.strip()
df.replace({"yes": True, "no": False})
```

---

## Creating & Transforming Columns

```python
df["total"] = df["a"] + df["b"]
df["flag"] = df["age"] > 30
df["cat"] = df["age"].apply(lambda x: "old" if x > 30 else "young")
df["upper"] = df["name"].str.upper()

# vectorised conditional
import numpy as np
df["band"] = np.where(df["age"] > 30, "senior", "junior")
df["band"] = pd.cut(df["age"], bins=[0, 18, 65, 120],
                    labels=["minor", "adult", "senior"])

df = df.assign(ratio=lambda d: d["a"] / d["b"])   # chainable
df.map(str)                                         # element-wise (whole frame)
df["x"].map({"a": 1, "b": 2})                       # Series value mapping
```

---

## Sorting & Ranking

```python
df.sort_values("age")
df.sort_values(["age", "name"], ascending=[False, True])
df.sort_index()
df["age"].rank(method="dense")
df.nlargest(5, "age"); df.nsmallest(3, "age")
```

---

## GroupBy & Aggregation

```python
df.groupby("dept")["salary"].mean()
df.groupby("dept").agg(avg=("salary", "mean"),
                        n=("salary", "count"),
                        total=("salary", "sum"))
df.groupby(["dept", "role"]).size()
df.groupby("dept")["salary"].transform("mean")   # broadcast back to rows
df.groupby("dept").apply(lambda g: g.head(2))

# pivot
df.pivot_table(index="dept", columns="role",
               values="salary", aggfunc="mean", fill_value=0)
pd.crosstab(df["dept"], df["role"])
```

---

## Combining DataFrames

```python
pd.concat([df1, df2])                 # stack rows
pd.concat([df1, df2], axis=1)         # side by side

pd.merge(left, right, on="id", how="inner")      # inner|left|right|outer
pd.merge(left, right, left_on="lid", right_on="rid")
df1.join(df2, on="key")               # merge on index
```

---

## Dates & Time Series

```python
df["date"] = pd.to_datetime(df["date"])
df["year"] = df["date"].dt.year
df["month"] = df["date"].dt.month; df["dow"] = df["date"].dt.day_name()
df.set_index("date").resample("M")["sales"].sum()   # monthly totals
df.set_index("date").rolling(7).mean()              # 7-period moving avg
pd.date_range("2026-01-01", periods=12, freq="M")
```

---

## Reshaping

```python
df.melt(id_vars=["id"], value_vars=["a", "b"])   # wide → long
df.pivot(index="id", columns="var", values="val") # long → wide
df.stack(); df.unstack()
df.T                                              # transpose
df.explode("list_col")                            # one row per list element
```

---

## Common Idioms & Tips

- Assignment on a filtered slice can trigger `SettingWithCopyWarning` — use `.loc`: `df.loc[df.a > 0, "b"] = 1`.
- Method chaining reads well: `df.dropna().assign(...).query(...).sort_values(...)`.
- Prefer vectorised ops / `np.where` / `.str` accessor over `.apply` for speed.
- `df.pipe(fn)` inserts a custom function into a chain.
- `pd.options.display.max_columns = None` to see all columns.
- Convert big object columns to `category` dtype to save memory.

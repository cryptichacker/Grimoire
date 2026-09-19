---
tags: [coding, sql, kql, kusto, azure, observability]
type: cheatsheet
source: compiled reference (Kusto Query Language)
last-verified: 2026-08-27
---

# KQL

## Up
- [[SQL]]

Kusto Query Language (KQL) is the read-only query language for **Azure Data Explorer**, **Azure Monitor / Log Analytics**, **Microsoft Sentinel**, and **Application Insights**. It's pipeline-based (`|`) and optimised for fast analytics over huge log/telemetry datasets. Unlike SQL, data flows left-to-right through operators.

---

## Core Shape

```kql
TableName
| where TimeGenerated > ago(1h)
| where Level == "Error"
| project TimeGenerated, Computer, Message
| sort by TimeGenerated desc
| take 100
```

Each `|` passes a tabular result to the next operator. Queries are case-sensitive; string comparisons are case-sensitive unless you use the `=~` / `has` family.

---

## Filtering (`where`)

```kql
| where TimeGenerated between (ago(7d) .. now())
| where Level == "Error" and Computer startswith "web"
| where Message contains "timeout"        // case-insensitive substring
| where Message has "error"               // fast term match (indexed)
| where Name in ("a", "b", "c")
| where Name in~ ("A", "B")               // case-insensitive
| where Message matches regex @"\d{3}-\d{4}"
| where isnotempty(UserId)
```

| Operator | Meaning |
|---|---|
| `==` / `!=` | exact (case-sensitive) |
| `=~` / `!~` | exact, case-insensitive |
| `has` / `has_cs` | whole-term match (fast, preferred) |
| `contains` / `startswith` / `endswith` | substring (case-insensitive; add `_cs` for sensitive) |
| `in` / `in~` | membership |
| `matches regex` | regular expression |

Prefer `has` over `contains` — it uses the term index and is much faster.

---

## Selecting & Shaping Columns

```kql
| project Name, Total, Time                 // keep these columns
| project-away Password, Secret             // drop columns
| project-rename User = UserName            // rename
| extend Ratio = Bytes / 1024.0             // add a computed column
| extend Level = tolower(Level)
| extend Day = startofday(TimeGenerated)
```

---

## Aggregation (`summarize`)

```kql
| summarize count() by Computer
| summarize Errors = count(), AvgMs = avg(DurationMs) by bin(TimeGenerated, 1h)
| summarize total = sum(Bytes) by Computer, Level
| summarize dcount(UserId)                  // distinct count
| summarize arg_max(TimeGenerated, *) by Computer   // latest row per group
| summarize percentiles(DurationMs, 50, 95, 99)
| summarize make_list(Name), make_set(Category) by Group
```

Common aggregation functions: `count()`, `countif(pred)`, `sum()`, `avg()`, `min()`, `max()`, `dcount()`, `percentile()`, `stdev()`, `arg_max()/arg_min()`, `make_list()/make_set()`.

### Time bucketing

```kql
| summarize count() by bin(TimeGenerated, 5m)      // 5-minute buckets
| summarize count() by bin(TimeGenerated, 1d), Level
```

---

## Sorting, Limiting, Top

```kql
| sort by TimeGenerated desc                // order (desc default is 'sort')
| order by Count desc                       // synonym
| take 10                                   // arbitrary N rows (fast)
| top 10 by Count desc                      // top N by a column
| top-nested ...                            // hierarchical top-N
```

---

## Joins & Unions

```kql
Errors
| join kind=inner (Requests) on RequestId   // inner | leftouter | rightouter | fullouter | leftanti | leftsemi
| project RequestId, ErrorMsg, Url

union Table1, Table2
| where TimeGenerated > ago(1d)

// lookup (dimension enrichment, like left join)
Events
| lookup kind=leftouter Users on UserId
```

---

## Time Functions & Variables

```kql
ago(1h); ago(7d); now(); startofday(now()); endofmonth(now())
datetime(2026-08-27); todatetime("2026-08-27T10:00Z")
TimeGenerated - ago(1d)                     // datetime math → timespan
format_datetime(TimeGenerated, "yyyy-MM-dd HH:mm")
datetime_diff("minute", end, start)

let threshold = 500;                         // declare a variable/const
let recent = Events | where TimeGenerated > ago(1d);   // a stored subquery
recent | summarize count() by Level;
```

---

## String & Parsing Operators

```kql
| extend Host = tostring(split(Url, "/")[2])
| parse Message with "user=" User " action=" Action    // structured parse
| extend Data = parse_json(RawJson)
| extend City = Data.address.city                       // dynamic navigation
| extend Upper = toupper(Name), Len = strlen(Name)
| mv-expand Tag = Tags                                   // one row per array element
```

---

## Charts & Rendering (Log Analytics / ADX)

```kql
| summarize count() by bin(TimeGenerated, 1h)
| render timechart

| summarize count() by Level
| render piechart          // also: barchart, columnchart, areachart, table
```

---

## Sentinel / Security Example

```kql
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType != "0"                    // failed sign-ins
| summarize Failures = count() by UserPrincipalName, IPAddress
| where Failures > 10
| sort by Failures desc
```

---

## Notes vs SQL

- **Pipeline, not clauses**: read top-to-bottom, left-to-right; order of operators matters for performance (filter early with `where`).
- `take`/`limit` return arbitrary rows (fast); use `top … by` for ordered results.
- `summarize … by` is KQL's `GROUP BY`; `project` is `SELECT`; `where` is `WHERE`.
- Everything is time-series-first — nearly every query starts by filtering `TimeGenerated` with `ago()`.
- KQL is read-only for querying; data ingestion/management uses separate control commands (`.ingest`, `.create table`).
- `let` defines reusable variables, scalars, and subqueries at the top of a query.

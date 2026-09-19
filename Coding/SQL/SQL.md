---
tags: [coding, sql, databases]
type: moc
---

# SQL

## Up
- [[Coding]]

Map-of-content for SQL and query languages — relational engines, their dialects, analytics/log query languages, NoSQL query models, and data-warehouse patterns.

## Subtopics
- [[MySQL]] — open-source relational DB (MySQL/MariaDB dialect)
- [[PostgreSQL]] — advanced open-source relational DB (Postgres dialect)
- [[MSSQL]] — Microsoft SQL Server (engine & administration)
- [[T-SQL]] — Transact-SQL procedural language (SQL Server / Azure SQL)
- [[KQL]] — Kusto Query Language (Azure Data Explorer, Log Analytics, Sentinel)
- [[NoSQL]] — non-relational query models (document, key-value, wide-column, graph)
- [[Datawarehousing SQL]] — analytics SQL: window functions, star schemas, MPP patterns

---

## Standard SQL Refresher (portable core)

```sql
SELECT col1, col2 FROM table
WHERE condition
GROUP BY col1
HAVING agg_condition
ORDER BY col1 DESC
LIMIT 10;                       -- (TOP / FETCH FIRST in some dialects)
```

Logical execution order (not written order): `FROM → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT`.

### Joins at a glance

| Join | Returns |
|---|---|
| `INNER JOIN` | Rows matching in both tables |
| `LEFT JOIN` | All left rows + matches (NULLs where none) |
| `RIGHT JOIN` | All right rows + matches |
| `FULL OUTER JOIN` | All rows from both, matched where possible |
| `CROSS JOIN` | Cartesian product |
| `SELF JOIN` | Table joined to itself |

Dialect differences (identifier quoting, pagination, string funcs, upsert, date handling) are covered in each child note.

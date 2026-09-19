---
tags: [coding, sql, datawarehouse, analytics]
type: cheatsheet
source: compiled reference (data-warehouse / analytics SQL)
last-verified: 2026-08-27
---

# Datawarehousing SQL

## Up
- [[SQL]]

Analytics/OLAP SQL as used in data warehouses (Snowflake, BigQuery, Redshift, Synapse, Databricks SQL). The focus shifts from single-row transactions (OLTP) to large scans, aggregations, window functions, and dimensional models.

---

## OLTP vs OLAP

| | OLTP (operational) | OLAP (warehouse) |
|---|---|---|
| Workload | Many small reads/writes | Few large analytical scans |
| Schema | Normalised (3NF) | Denormalised (star/snowflake) |
| Storage | Row-oriented | **Columnar** (Parquet/ORC) |
| Scaling | Vertical / replicas | **MPP** — distributed across nodes |
| Example queries | "get order 123" | "revenue by region by month" |

---

## Dimensional Modelling

- **Fact table** — measurable events/metrics (sales, clicks), one row per event, foreign keys to dimensions + numeric measures. Large and narrow.
- **Dimension table** — descriptive context (customer, product, date, store). Smaller, wide, textual.
- **Star schema** — one fact table joined directly to denormalised dimensions.
- **Snowflake schema** — dimensions further normalised into sub-tables.
- **Grain** — the level of detail of one fact row (define it first).

```sql
-- classic star-schema query
SELECT d.year, d.month, p.category, SUM(f.amount) AS revenue
FROM fact_sales f
JOIN dim_date    d ON f.date_key    = d.date_key
JOIN dim_product p ON f.product_key = p.product_key
GROUP BY d.year, d.month, p.category
ORDER BY d.year, d.month;
```

### Slowly Changing Dimensions (SCD)

| Type | Behaviour |
|---|---|
| **Type 1** | Overwrite — no history kept |
| **Type 2** | New row per change, with `valid_from`/`valid_to`/`is_current` flags (full history) |
| **Type 3** | Add a "previous value" column (limited history) |

---

## Window Functions (the analytics workhorse)

```sql
SELECT
    region, order_date, amount,
    -- ranking
    ROW_NUMBER() OVER (PARTITION BY region ORDER BY amount DESC) AS rn,
    RANK()       OVER (PARTITION BY region ORDER BY amount DESC) AS rnk,
    DENSE_RANK() OVER (ORDER BY amount DESC)                     AS drnk,
    NTILE(4)     OVER (ORDER BY amount)                          AS quartile,
    -- offsets
    LAG(amount)  OVER (PARTITION BY region ORDER BY order_date) AS prev_amt,
    LEAD(amount) OVER (PARTITION BY region ORDER BY order_date) AS next_amt,
    -- running / moving aggregates
    SUM(amount)  OVER (PARTITION BY region ORDER BY order_date
                       ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total,
    AVG(amount)  OVER (ORDER BY order_date
                       ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)         AS ma_7,
    amount * 100.0 / SUM(amount) OVER (PARTITION BY region)              AS pct_of_region
FROM fact_sales;
```

Frame clauses: `ROWS BETWEEN ... ` (physical rows) vs `RANGE BETWEEN ...` (value range); bounds `UNBOUNDED PRECEDING`, `N PRECEDING`, `CURRENT ROW`, `N FOLLOWING`, `UNBOUNDED FOLLOWING`.

---

## Roll-ups, Cubes & Pivots

```sql
-- subtotals + grand total
SELECT region, product, SUM(amount)
FROM fact_sales
GROUP BY ROLLUP (region, product);

-- all combinations of groupings
SELECT region, product, SUM(amount)
FROM fact_sales
GROUP BY CUBE (region, product);

SELECT region, category, SUM(amount)
FROM fact_sales
GROUP BY GROUPING SETS ((region), (category), ());

-- pivot (dialect-specific; portable form with conditional aggregation)
SELECT region,
    SUM(CASE WHEN year = 2025 THEN amount END) AS y2025,
    SUM(CASE WHEN year = 2026 THEN amount END) AS y2026
FROM fact_sales GROUP BY region;
```

---

## Date Dimension & Time Intelligence

```sql
-- year-over-year growth via LAG on a monthly aggregate
WITH monthly AS (
    SELECT DATE_TRUNC('month', order_date) AS m, SUM(amount) AS revenue
    FROM fact_sales GROUP BY 1
)
SELECT m, revenue,
       revenue - LAG(revenue, 12) OVER (ORDER BY m)          AS yoy_change,
       (revenue - LAG(revenue,12) OVER (ORDER BY m)) * 100.0
         / NULLIF(LAG(revenue,12) OVER (ORDER BY m), 0)       AS yoy_pct
FROM monthly;

DATE_TRUNC('month', ts); EXTRACT(YEAR FROM ts); DATEADD(day, -7, ts);
```

---

## ELT & Warehouse Patterns

```sql
-- staging → transform → mart (ELT: load raw, transform in-warehouse)
CREATE TABLE staging.sales AS SELECT * FROM external_raw;

-- incremental / idempotent MERGE upsert
MERGE INTO dim_customer t
USING staging.customer s ON t.customer_id = s.customer_id
WHEN MATCHED THEN UPDATE SET name = s.name, updated_at = CURRENT_TIMESTAMP
WHEN NOT MATCHED THEN INSERT (customer_id, name) VALUES (s.customer_id, s.name);

-- CTAS to materialise a mart
CREATE TABLE mart.revenue_by_month AS
SELECT DATE_TRUNC('month', order_date) m, region, SUM(amount) revenue
FROM fact_sales GROUP BY 1, 2;
```

- **ETL vs ELT**: modern warehouses favour ELT — load raw, transform with SQL in-warehouse (dbt is the common tool).
- **Materialised views** cache expensive aggregations; refresh on schedule.
- **Partitioning** (by date) + **clustering/sort keys** prune scans and cut cost.

---

## Performance in Columnar/MPP Engines

- Select only needed columns — columnar storage means fewer columns = less I/O.
- Filter on the **partition/cluster key** to prune data (partition elimination).
- Pre-aggregate into marts; avoid `SELECT *` on billion-row facts.
- Watch **data skew** — uneven distribution keys overload single nodes.
- Warehouses bill by scanned bytes (BigQuery) or compute time (Snowflake/Redshift) — narrow, partitioned queries save money.
- Broadcast small dimensions; co-locate large joins on the same distribution key.

---

## Engine Quick Notes

| Engine | Notes |
|---|---|
| **Snowflake** | Separated storage/compute, virtual warehouses, `VARIANT` for semi-structured, zero-copy clones, time travel |
| **BigQuery** | Serverless, standard SQL, bills per bytes scanned, nested/repeated fields, partitioning + clustering |
| **Redshift** | AWS MPP, distribution + sort keys matter a lot, `COPY` from S3 |
| **Synapse / Databricks SQL** | Azure/lakehouse; Databricks uses Spark SQL + Delta Lake |

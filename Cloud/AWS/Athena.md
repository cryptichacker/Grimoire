---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Athena

## Up
- [[AWS]]

**Amazon Athena** — Serverless, interactive query service to analyze data in S3 (and other sources) using standard SQL.

**Category:** Analytics

## Key concepts
- Trino/Presto query engine
- Schemas via the Glue Data Catalog
- Partitions and partition projection
- Formats: Parquet, ORC, CSV, JSON
- Federated queries
- Workgroups and CTAS

## Common CLI
| Command | Description |
|---|---|
| `aws athena start-query-execution` | Run a query |
| `aws athena get-query-results` | Fetch results |
| `aws athena list-databases` | List databases |

## Pricing model
Per TB of data scanned (much cheaper with columnar formats and partitioning).

## Works well with
- S3
- Glue Data Catalog
- QuickSight, Lake Formation

## Use cases
- Ad-hoc S3 analytics
- Log querying
- Data-lake exploration

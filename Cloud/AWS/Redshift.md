---
tags: [cloud, aws, database]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Redshift

## Up
- [[AWS]]

**Amazon Redshift** — Columnar, massively-parallel cloud data warehouse for fast SQL analytics over petabytes, with Spectrum to query S3 directly.

**Category:** Database

## Key concepts
- Leader and compute nodes
- Columnar storage and compression
- Distribution and sort keys
- RA3 managed storage; Serverless option
- Redshift Spectrum queries S3
- Concurrency scaling and materialized views

## Common CLI
| Command | Description |
|---|---|
| `aws redshift create-cluster` | Create a provisioned cluster |
| `aws redshift describe-clusters` | List clusters |
| `aws redshift-data execute-statement` | Run SQL via the Data API |

## Pricing model
Per node-hour (provisioned) or RPU (Serverless) plus managed storage.

## Works well with
- S3, Glue
- QuickSight
- Kinesis, Athena

## Use cases
- Data warehousing
- BI dashboards
- Large-scale reporting

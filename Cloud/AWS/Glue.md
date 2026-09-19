---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Glue

## Up
- [[AWS]]

**AWS Glue** — Serverless data-integration service for cataloging, cleaning and transforming data (ETL/ELT).

**Category:** Analytics

## Key concepts
- Data Catalog (central metadata store)
- Crawlers for schema discovery
- ETL jobs (Spark or Python shell)
- Glue Studio (visual authoring)
- Triggers and workflows
- DataBrew, streaming ETL, job bookmarks

## Common CLI
| Command | Description |
|---|---|
| `aws glue create-database` | Create a catalog database |
| `aws glue create-crawler / start-crawler` | Discover schema |
| `aws glue create-job / start-job-run` | Create and run an ETL job |

## Pricing model
Per DPU-hour for jobs and crawlers plus Data Catalog storage/requests.

## Works well with
- S3, Athena
- Redshift
- Lake Formation, EMR

## Use cases
- ETL pipelines
- Data cataloging
- Data prep for analytics

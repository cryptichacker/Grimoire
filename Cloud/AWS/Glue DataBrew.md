---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Glue DataBrew

## Up
- [[AWS]]

**AWS Glue DataBrew** — A visual data-preparation tool that lets analysts clean and normalize data without writing code.

**Category:** Analytics

## Key concepts
- Projects and datasets
- Recipes (250+ transformations)
- Data profiling
- Recipe jobs (scheduled)
- Data lineage

## Common CLI
| Command | Description |
|---|---|
| `aws databrew create-project` | Create a project |
| `aws databrew create-recipe-job` | Create a prep job |
| `aws databrew start-job-run` | Run a job |

## Pricing model
Per interactive session plus per DataBrew node-hour for jobs.

## Works well with
- S3
- Glue Data Catalog
- Redshift

## Use cases
- No-code data prep
- Data cleaning/normalization
- Profiling datasets

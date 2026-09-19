---
tags: [cloud, aws, application-integration]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# MWAA

## Up
- [[AWS]]

**Amazon Managed Workflows for Apache Airflow** — Managed Apache Airflow to author, schedule and monitor data-pipeline workflows without operating the infrastructure.

**Category:** Application Integration

## Key concepts
- Environments (managed Airflow)
- DAGs stored in S3
- Workers auto-scaling
- Plugins and requirements
- Web UI and CloudWatch logs

## Common CLI
| Command | Description |
|---|---|
| `aws mwaa create-environment` | Create an Airflow environment |
| `aws mwaa list-environments` | List environments |
| `aws mwaa create-web-login-token` | Get a UI login token |

## Pricing model
Per environment-hour by size plus worker auto-scaling and storage.

## Works well with
- S3 (DAGs)
- Glue, EMR, Redshift
- Step Functions

## Use cases
- Data-pipeline orchestration
- Scheduled ETL DAGs
- Airflow migrations

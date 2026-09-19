---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# EMR

## Up
- [[AWS]]

**Amazon EMR** — Managed big-data platform to run Apache Spark, Hadoop, Hive and Presto on scalable clusters (EC2, EKS or serverless).

**Category:** Analytics

## Key concepts
- Clusters (primary/core/task nodes)
- Steps and applications
- EMR on EC2 / EKS / Serverless
- Spot and instance fleets
- EMRFS to S3
- Notebooks and Studio

## Common CLI
| Command | Description |
|---|---|
| `aws emr create-cluster` | Launch a cluster |
| `aws emr add-steps` | Submit work |
| `aws emr list-clusters` | List clusters |
| `aws emr terminate-clusters` | Shut down |

## Pricing model
Per-second EMR fee on top of the underlying EC2/EKS/serverless compute.

## Works well with
- S3
- Glue Data Catalog
- Kinesis, Redshift

## Use cases
- Large-scale ETL
- Spark/Hadoop processing
- ML data preparation

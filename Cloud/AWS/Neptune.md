---
tags: [cloud, aws, database]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Neptune

## Up
- [[AWS]]

**Amazon Neptune** — Fully managed graph database supporting property-graph (Gremlin/openCypher) and RDF (SPARQL) models.

**Category:** Database

## Key concepts
- Graph models and query languages
- Cluster with read replicas
- Fast relationship traversal
- Neptune Serverless
- Bulk loader from S3
- Neptune Analytics for graph analytics

## Common CLI
| Command | Description |
|---|---|
| `aws neptune create-db-cluster` | Create a cluster |
| `aws neptune create-db-instance` | Add an instance |
| `aws neptune describe-db-clusters` | List clusters |

## Pricing model
Per instance-hour (or NCU for Serverless) plus storage and I/O.

## Works well with
- S3 (bulk load)
- VPC, IAM
- Lambda

## Use cases
- Fraud detection
- Social and knowledge graphs
- Recommendation engines

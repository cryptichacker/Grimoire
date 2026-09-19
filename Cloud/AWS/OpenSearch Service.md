---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# OpenSearch Service

## Up
- [[AWS]]

**Amazon OpenSearch Service** — Managed OpenSearch (and legacy Elasticsearch) for search, log analytics and observability, with Dashboards.

**Category:** Analytics

## Key concepts
- Domains and node roles (data/master/UltraWarm/cold)
- Indices and shards
- OpenSearch Dashboards
- Serverless option
- Ingestion pipelines
- Fine-grained access control

## Common CLI
| Command | Description |
|---|---|
| `aws opensearch create-domain` | Create a domain |
| `aws opensearch describe-domain` | Get domain details |
| `aws opensearch list-domain-names` | List domains |

## Pricing model
Per instance-hour and storage (provisioned) or OCU (serverless).

## Works well with
- Kinesis/Firehose
- CloudWatch Logs
- S3, Cognito

## Use cases
- Log and event analytics
- Full-text search
- Observability dashboards

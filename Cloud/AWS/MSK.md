---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# MSK

## Up
- [[AWS]]

**Amazon Managed Streaming for Apache Kafka** — Fully managed Apache Kafka for building real-time streaming data pipelines and applications.

**Category:** Analytics

## Key concepts
- Clusters (provisioned or serverless)
- Brokers and topics
- KRaft/Zookeeper
- MSK Connect
- IAM/SASL/TLS authentication
- Multi-AZ

## Common CLI
| Command | Description |
|---|---|
| `aws kafka create-cluster` | Create a cluster |
| `aws kafka list-clusters` | List clusters |
| `aws kafka describe-cluster` | Get cluster details |

## Pricing model
Per broker-hour and storage (provisioned) or per throughput (serverless).

## Works well with
- Kinesis Analytics/Flink
- Lambda
- S3, Glue Schema Registry

## Use cases
- Event streaming
- Kafka migrations
- Real-time pipelines

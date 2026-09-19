---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Managed Service for Apache Flink

## Up
- [[AWS]]

**Amazon Managed Service for Apache Flink** — Fully managed Apache Flink for transforming and analyzing streaming data in real time (formerly Kinesis Data Analytics).

**Category:** Analytics

## Key concepts
- Flink applications
- SQL, Java, Python and Scala
- Stateful stream processing
- Checkpoints and snapshots
- Studio notebooks

## Common CLI
| Command | Description |
|---|---|
| `aws kinesisanalyticsv2 create-application` | Create a Flink application |
| `aws kinesisanalyticsv2 start-application` | Start it |
| `aws kinesisanalyticsv2 describe-application` | Inspect it |

## Pricing model
Per Kinesis Processing Unit (KPU) used plus running storage.

## Works well with
- Kinesis, MSK
- S3
- OpenSearch, DynamoDB

## Use cases
- Real-time stream analytics
- Streaming ETL
- Event-time aggregations

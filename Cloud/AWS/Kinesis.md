---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Kinesis

## Up
- [[AWS]]

**Amazon Kinesis** — Ingests and processes real-time streaming data at scale across a family of services.

**Category:** Analytics

## Key concepts
- Data Streams (shards, producers/consumers)
- Data Firehose (delivery to S3/Redshift/OpenSearch)
- Managed Service for Apache Flink
- Video Streams
- KCL and enhanced fan-out

## Common CLI
| Command | Description |
|---|---|
| `aws kinesis create-stream` | Create a data stream |
| `aws kinesis put-record` | Write a record |
| `aws kinesis get-records` | Read records |
| `aws firehose create-delivery-stream` | Create a Firehose delivery stream |

## Pricing model
Per shard-hour plus PUT payload units (Streams); per GB ingested (Firehose).

## Works well with
- Lambda
- S3, Redshift
- OpenSearch, Flink

## Use cases
- Real-time analytics
- Log and event ingestion
- Clickstream and IoT data

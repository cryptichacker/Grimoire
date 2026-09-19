---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Data Firehose

## Up
- [[AWS]]

**Amazon Data Firehose** — Fully managed service to reliably load streaming data into data lakes, warehouses and analytics services.

**Category:** Analytics

## Key concepts
- Delivery streams
- Buffering (size/time)
- Data transformation via Lambda
- Format conversion (to Parquet/ORC)
- Dynamic partitioning

## Common CLI
| Command | Description |
|---|---|
| `aws firehose create-delivery-stream` | Create a delivery stream |
| `aws firehose put-record` | Send a record |
| `aws firehose describe-delivery-stream` | Inspect a stream |

## Pricing model
Per GB ingested (plus format conversion and VPC delivery options).

## Works well with
- Kinesis, MSK
- S3, Redshift
- OpenSearch Service

## Use cases
- Streaming ETL delivery
- Log/event loading
- Near-real-time analytics

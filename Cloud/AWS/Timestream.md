---
tags: [cloud, aws, database]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Timestream

## Up
- [[AWS]]

**Amazon Timestream** — Purpose-built, serverless time-series database for IoT and operational data with automatic tiering of recent vs historical data.

**Category:** Database

## Key concepts
- Databases and tables
- Memory store vs magnetic store
- Time-series SQL functions
- Automatic data lifecycle
- Scheduled queries

## Common CLI
| Command | Description |
|---|---|
| `aws timestream-write create-database` | Create a database |
| `aws timestream-write create-table` | Create a table |
| `aws timestream-write write-records` | Ingest records |
| `aws timestream-query query` | Run a query |

## Pricing model
Per data ingested, storage tier and data scanned by queries.

## Works well with
- IoT Core
- Kinesis
- QuickSight, Grafana

## Use cases
- IoT telemetry
- Application and DevOps metrics
- Time-series analytics

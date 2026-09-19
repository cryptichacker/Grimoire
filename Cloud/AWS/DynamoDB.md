---
tags: [cloud, aws, database]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# DynamoDB

## Up
- [[AWS]]

**Amazon DynamoDB** — Serverless NoSQL database delivering single-digit-millisecond latency at any scale with no servers to manage.

**Category:** Database

## Key concepts
- Tables, items and attributes
- Partition key and optional sort key
- Global and local secondary indexes
- On-demand vs provisioned capacity (with auto scaling)
- DynamoDB Streams and TTL
- Global Tables, transactions, DAX in-memory cache

## Common CLI
| Command | Description |
|---|---|
| `aws dynamodb create-table` | Create a table |
| `aws dynamodb put-item / get-item` | Write or read an item |
| `aws dynamodb query / scan` | Read many items |
| `aws dynamodb update-table` | Change capacity or indexes |

## Pricing model
Per read/write capacity (provisioned) or per request (on-demand), plus storage.

## Works well with
- Lambda (Streams)
- API Gateway, AppSync
- Glue

## Use cases
- High-scale web and mobile
- Serverless backends
- Session and state stores

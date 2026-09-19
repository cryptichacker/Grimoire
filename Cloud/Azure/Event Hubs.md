---
tags: [cloud, azure, analytics]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Event Hubs

## Up
- [[Azure]]

**Azure Event Hubs** — A big-data streaming and event-ingestion service that can receive and process millions of events per second.

**Category:** Analytics

## Key concepts
- Namespaces and event hubs
- Partitions and consumer groups
- Capture to Blob/Data Lake
- Kafka-compatible endpoint
- Throughput/processing units
- Schema Registry

## Common CLI
| Command | Description |
|---|---|
| `az eventhubs namespace create` | Create a namespace |
| `az eventhubs eventhub create` | Create an event hub |
| `az eventhubs eventhub consumer-group create` | Create a consumer group |

## Pricing model
Per throughput/processing unit plus ingress events and capture.

## Works well with
- Stream Analytics, Functions
- Data Lake (capture)
- Kafka clients

## Use cases
- Event streaming
- Telemetry/log ingestion
- Real-time pipelines

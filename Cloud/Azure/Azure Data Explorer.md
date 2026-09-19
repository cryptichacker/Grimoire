---
tags: [cloud, azure, analytics]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Data Explorer

## Up
- [[Azure]]

**Azure Data Explorer (ADX)** — A fast, fully managed analytics service for real-time analysis of large volumes of log and telemetry data using KQL.

**Category:** Analytics

## Key concepts
- Clusters and databases
- Kusto Query Language (KQL)
- High-speed ingestion
- Time-series and log analytics
- Materialized views
- Follower databases

## Common CLI
| Command | Description |
|---|---|
| `az kusto cluster create` | Create a cluster |
| `az kusto database create` | Create a database |
| `az kusto data-connection event-hub create` | Connect an Event Hub |

## Pricing model
Per cluster VM-hour (markup) plus storage.

## Works well with
- Event Hubs, IoT Hub
- Data Lake Storage
- Power BI, Grafana

## Use cases
- Log/telemetry analytics
- Time-series analysis
- Interactive exploration

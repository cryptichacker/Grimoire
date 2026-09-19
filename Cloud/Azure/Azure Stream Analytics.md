---
tags: [cloud, azure, analytics]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Stream Analytics

## Up
- [[Azure]]

**Azure Stream Analytics** — A real-time analytics engine that runs SQL-like queries over streaming data from many sources.

**Category:** Analytics

## Key concepts
- Jobs with inputs/outputs
- SQL-based query language
- Windowing functions
- Streaming units (SU)
- Reference data joins
- Built-in ML/anomaly functions

## Common CLI
| Command | Description |
|---|---|
| `az stream-analytics job create` | Create a job |
| `az stream-analytics input create` | Add an input |
| `az stream-analytics job start` | Start the job |

## Pricing model
Per streaming unit-hour.

## Works well with
- Event Hubs, IoT Hub
- Blob / Data Lake, SQL
- Power BI

## Use cases
- Real-time analytics
- IoT telemetry processing
- Fraud/anomaly detection

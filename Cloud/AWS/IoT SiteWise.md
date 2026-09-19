---
tags: [cloud, aws, iot]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# IoT SiteWise

## Up
- [[AWS]]

**AWS IoT SiteWise** — Collects, stores, organizes and monitors industrial equipment data at scale to power operational insights.

**Category:** IoT

## Key concepts
- Asset models and asset hierarchies
- Data ingestion via gateway/OPC-UA
- Transforms and metrics
- Built-in time-series store
- SiteWise Monitor portals

## Common CLI
| Command | Description |
|---|---|
| `aws iotsitewise create-asset-model` | Define an asset model |
| `aws iotsitewise create-asset` | Create an asset |
| `aws iotsitewise batch-put-asset-property-value` | Ingest measurements |

## Pricing model
Per message/ingestion, storage and query, plus Monitor portal usage.

## Works well with
- IoT Core / Greengrass
- Timestream
- QuickSight, Grafana

## Use cases
- Industrial monitoring
- OEE and equipment metrics
- Factory dashboards

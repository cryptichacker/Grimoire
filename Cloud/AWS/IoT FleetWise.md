---
tags: [cloud, aws, iot]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# IoT FleetWise

## Up
- [[AWS]]

**AWS IoT FleetWise** — Makes it easier to collect, transform and transfer vehicle data to the cloud in near-real time for connected-vehicle apps.

**Category:** IoT

## Key concepts
- Signal catalog and vehicle models
- Decoder manifests
- Campaigns (what/when to collect)
- Edge agent
- Data destinations (S3/Timestream)

## Common CLI
| Command | Description |
|---|---|
| `aws iotfleetwise create-signal-catalog` | Create a signal catalog |
| `aws iotfleetwise create-vehicle` | Register a vehicle |
| `aws iotfleetwise create-campaign` | Create a data-collection campaign |

## Pricing model
Per vehicle and per message/data collected.

## Works well with
- IoT Core
- Timestream, S3
- Managed Grafana

## Use cases
- Connected vehicles
- Fleet telemetry
- Predictive maintenance

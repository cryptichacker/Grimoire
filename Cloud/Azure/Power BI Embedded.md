---
tags: [cloud, azure, analytics]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Power BI Embedded

## Up
- [[Azure]]

**Power BI Embedded** — Lets developers embed Power BI reports, dashboards and tiles into their own applications for customers (ISV scenario).

**Category:** Analytics

## Key concepts
- Embedded capacity (A SKUs)
- App-owns-data model
- REST APIs and JavaScript SDK
- Row-level security
- Capacity scaling

## Common CLI
| Command | Description |
|---|---|
| `az powerbi embedded-capacity create` | Create an embedded capacity |
| `az powerbi embedded-capacity list` | List capacities |
| `az powerbi embedded-capacity update` | Scale a capacity |

## Pricing model
Per embedded capacity (A SKU) per hour.

## Works well with
- Power BI service
- Azure SQL / Synapse
- App Service

## Use cases
- Embedded analytics
- ISV/customer-facing reports
- White-label BI

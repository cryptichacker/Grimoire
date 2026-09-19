---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Log Analytics

## Up
- [[Azure]]

**Azure Log Analytics** — A log data store and query engine (KQL) at the heart of Azure Monitor for analyzing telemetry.

**Category:** Management

## Key concepts
- Workspaces
- Kusto Query Language (KQL)
- Tables and data collection rules
- Retention and archive tiers
- Cross-workspace queries
- Basic vs Analytics logs

## Common CLI
| Command | Description |
|---|---|
| `az monitor log-analytics workspace create` | Create a workspace |
| `az monitor log-analytics query` | Run a KQL query |
| `az monitor log-analytics workspace list` | List workspaces |

## Pricing model
Per GB ingested and per GB retained by tier.

## Works well with
- Azure Monitor
- Microsoft Sentinel
- Application Insights

## Use cases
- Log analytics
- KQL queries
- Central telemetry store

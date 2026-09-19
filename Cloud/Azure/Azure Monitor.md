---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Monitor

## Up
- [[Azure]]

**Azure Monitor** — Full-stack observability platform collecting metrics, logs and traces with alerting and dashboards.

**Category:** Management

## Key concepts
- Metrics and Log Analytics logs
- Alerts and action groups
- Dashboards and Workbooks
- Autoscale
- Data collection rules and agents
- Distributed tracing (App Insights)

## Common CLI
| Command | Description |
|---|---|
| `az monitor metrics list` | Query metrics |
| `az monitor alert-rule create` | Create an alert |
| `az monitor action-group create` | Create an action group |
| `az monitor diagnostic-settings create` | Route diagnostics |

## Pricing model
Per GB ingested/retained plus alerts and metric queries.

## Works well with
- Log Analytics, Application Insights
- Every Azure service
- Grafana / Power BI

## Use cases
- Monitoring and alerting
- Autoscale triggers
- Observability

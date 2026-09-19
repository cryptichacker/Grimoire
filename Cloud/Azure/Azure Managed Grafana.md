---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Managed Grafana

## Up
- [[Azure]]

**Azure Managed Grafana** — A fully managed Grafana service for visualizing and analyzing metrics, logs and traces from many sources.

**Category:** Management

## Key concepts
- Workspaces
- Prebuilt Azure Monitor data sources
- Dashboards and alerting
- Entra ID SSO and RBAC
- Plugins

## Common CLI
| Command | Description |
|---|---|
| `az grafana create` | Create a Grafana workspace |
| `az grafana list` | List workspaces |
| `az grafana dashboard list` | List dashboards |

## Pricing model
Per instance (Standard) per month plus active users.

## Works well with
- Azure Managed Prometheus
- Azure Monitor
- Data Explorer

## Use cases
- Observability dashboards
- Cross-source visualization
- Operational alerting

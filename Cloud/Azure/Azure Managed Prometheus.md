---
tags: [cloud, azure, management]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Managed Prometheus

## Up
- [[Azure]]

**Azure Monitor Managed Service for Prometheus** — A fully managed, Prometheus-compatible monitoring backend for collecting and querying container/infrastructure metrics.

**Category:** Management

## Key concepts
- Azure Monitor workspace
- Remote-write ingestion
- PromQL queries
- Recording and alert rules
- Auto-collection from AKS

## Common CLI
| Command | Description |
|---|---|
| `az monitor account create` | Create an Azure Monitor workspace |
| `az monitor account list` | List workspaces |
| `az aks update --enable-azure-monitor-metrics` | Enable Prometheus on AKS |

## Pricing model
Per metric sample ingested and queried.

## Works well with
- AKS
- Azure Managed Grafana
- Azure Monitor

## Use cases
- Kubernetes metrics
- PromQL alerting
- High-scale monitoring

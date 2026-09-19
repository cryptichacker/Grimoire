---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Managed Service for Prometheus

## Up
- [[AWS]]

**Amazon Managed Service for Prometheus** — Prometheus-compatible, serverless monitoring for container and infrastructure metrics at scale.

**Category:** Management

## Key concepts
- Workspaces
- Remote-write ingestion
- PromQL queries
- Recording and alerting rules
- Alert Manager

## Common CLI
| Command | Description |
|---|---|
| `aws amp create-workspace` | Create a workspace |
| `aws amp list-workspaces` | List workspaces |
| `aws amp create-rule-groups-namespace` | Add rules/alerts |

## Pricing model
Per metric sample ingested, stored and queried.

## Works well with
- EKS/ECS
- Managed Grafana
- CloudWatch

## Use cases
- Kubernetes monitoring
- High-scale metrics
- PromQL alerting

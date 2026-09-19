---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Managed Grafana

## Up
- [[AWS]]

**Amazon Managed Grafana** — Fully managed Grafana for querying, visualizing and alerting on metrics, logs and traces from many sources.

**Category:** Management

## Key concepts
- Workspaces
- Data source plugins (CloudWatch, Prometheus, etc.)
- Dashboards and alerting
- SSO via IAM Identity Center
- Fine-grained access

## Common CLI
| Command | Description |
|---|---|
| `aws grafana create-workspace` | Create a workspace |
| `aws grafana list-workspaces` | List workspaces |
| `aws grafana describe-workspace` | Workspace details |

## Pricing model
Per active user (editor/viewer) per month.

## Works well with
- Managed Service for Prometheus
- CloudWatch
- X-Ray, OpenSearch

## Use cases
- Observability dashboards
- Cross-source visualization
- Operational alerting

---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# CloudWatch

## Up
- [[AWS]]

**Amazon CloudWatch** — Observability service that collects metrics, logs and events, with alarms, dashboards and automated actions.

**Category:** Management

## Key concepts
- Metrics and namespaces
- Custom metrics
- Alarms and composite alarms
- Logs and Logs Insights
- Dashboards
- Metric filters, CloudWatch agent, Container/Lambda Insights

## Common CLI
| Command | Description |
|---|---|
| `aws cloudwatch put-metric-data` | Publish a custom metric |
| `aws cloudwatch put-metric-alarm` | Create an alarm |
| `aws logs tail` | Stream log events |
| `aws logs start-query` | Run a Logs Insights query |

## Pricing model
Per metric, alarm and dashboard plus log ingestion/storage and query charges.

## Works well with
- EC2, Lambda, ECS
- RDS
- SNS, Auto Scaling

## Use cases
- Monitoring and alerting
- Log analysis
- Auto Scaling triggers

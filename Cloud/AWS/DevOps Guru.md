---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# DevOps Guru

## Up
- [[AWS]]

**Amazon DevOps Guru** — Uses ML to detect operational issues and anomalies before they impact customers, with actionable insights.

**Category:** Management

## Key concepts
- Reactive and proactive insights
- Anomaly detection on metrics/logs
- Correlated events and root cause
- Recommendations
- Coverage by CloudFormation stack/tag

## Common CLI
| Command | Description |
|---|---|
| `aws devops-guru describe-account-health` | Overall health |
| `aws devops-guru list-insights` | List insights |
| `aws devops-guru describe-insight` | Insight details |

## Pricing model
Per resource-hour analyzed.

## Works well with
- CloudWatch
- X-Ray
- Systems Manager, SNS

## Use cases
- Operational anomaly detection
- Proactive issue alerts
- Faster root-cause analysis

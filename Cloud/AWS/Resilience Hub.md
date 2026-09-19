---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Resilience Hub

## Up
- [[AWS]]

**AWS Resilience Hub** — Assesses and improves the resilience of applications against recovery objectives (RTO/RPO) with recommendations.

**Category:** Management

## Key concepts
- Applications and resiliency policies
- RTO/RPO targets
- Assessments and scores
- Recommendations and SOPs
- Drift detection and alarms

## Common CLI
| Command | Description |
|---|---|
| `aws resiliencehub create-app` | Register an application |
| `aws resiliencehub start-app-assessment` | Run an assessment |
| `aws resiliencehub describe-app` | App details |

## Pricing model
Per assessed application per month.

## Works well with
- CloudFormation
- Elastic Disaster Recovery, Backup
- CloudWatch, Fault Injection Service

## Use cases
- Resilience assessment
- DR readiness
- Meeting RTO/RPO targets

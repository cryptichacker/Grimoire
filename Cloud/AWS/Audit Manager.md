---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Audit Manager

## Up
- [[AWS]]

**AWS Audit Manager** — Continuously collects evidence and maps it to controls to simplify audits against frameworks like SOC 2, PCI and HIPAA.

**Category:** Security

## Key concepts
- Frameworks and controls
- Assessments
- Automated evidence collection
- Control sets
- Assessment reports and delegation

## Common CLI
| Command | Description |
|---|---|
| `aws auditmanager create-assessment` | Start an assessment |
| `aws auditmanager get-assessment` | Get assessment details |
| `aws auditmanager list-assessments` | List assessments |

## Pricing model
Per assessed resource per month.

## Works well with
- Config, Security Hub
- CloudTrail
- Organizations

## Use cases
- Compliance audits
- Evidence collection
- Continuous audit readiness

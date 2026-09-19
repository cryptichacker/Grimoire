---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Security Hub

## Up
- [[AWS]]

**AWS Security Hub** — Aggregates, normalizes and prioritizes security findings across accounts and runs automated security-standard checks.

**Category:** Security

## Key concepts
- Findings in AWS Security Finding Format
- Security standards (CIS, AWS FSBP, PCI)
- Insights
- Cross-account/Region aggregation
- Automated response via EventBridge

## Common CLI
| Command | Description |
|---|---|
| `aws securityhub enable-security-hub` | Enable Security Hub |
| `aws securityhub get-findings` | List findings |
| `aws securityhub batch-import-findings` | Push custom findings |

## Pricing model
Per finding ingested and per automated compliance check.

## Works well with
- GuardDuty, Inspector, Macie
- Config
- EventBridge

## Use cases
- Central security posture
- Compliance scoring
- Finding aggregation

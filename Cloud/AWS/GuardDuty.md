---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# GuardDuty

## Up
- [[AWS]]

**Amazon GuardDuty** — Continuous threat-detection service that analyzes AWS logs with machine learning to flag malicious or unusual activity.

**Category:** Security

## Key concepts
- Detectors per region/account
- Data sources: VPC Flow/DNS/CloudTrail, S3, EKS, malware, RDS
- Findings with severity
- Threat intelligence feeds
- Multi-account via delegated admin
- Automated response with EventBridge

## Common CLI
| Command | Description |
|---|---|
| `aws guardduty create-detector` | Enable GuardDuty |
| `aws guardduty list-findings` | List findings |
| `aws guardduty get-findings` | Get finding details |

## Pricing model
Per volume of logs and events analyzed.

## Works well with
- CloudTrail, VPC
- S3, EKS
- Security Hub, EventBridge

## Use cases
- Threat detection
- Compromised-credential alerts
- Continuous monitoring

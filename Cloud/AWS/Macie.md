---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Macie

## Up
- [[AWS]]

**Amazon Macie** — Uses machine learning to discover, classify and protect sensitive data such as PII stored in Amazon S3.

**Category:** Security

## Key concepts
- Sensitive-data discovery jobs
- Managed and custom data identifiers
- S3 bucket inventory and posture
- Findings
- Multi-account via delegated admin

## Common CLI
| Command | Description |
|---|---|
| `aws macie2 enable-macie` | Enable Macie |
| `aws macie2 create-classification-job` | Scan for sensitive data |
| `aws macie2 get-findings` | Get finding details |

## Pricing model
Per S3 bucket evaluated plus per GB of data inspected.

## Works well with
- S3
- Security Hub, EventBridge
- KMS

## Use cases
- PII discovery
- S3 data-security posture
- Compliance (GDPR/HIPAA)

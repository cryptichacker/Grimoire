---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# IAM Access Analyzer

## Up
- [[AWS]]

**AWS IAM Access Analyzer** — Identifies resources shared with external entities, validates policies and helps you move toward least privilege.

**Category:** Security

## Key concepts
- External access findings
- Unused access findings
- Policy validation and generation
- Custom policy checks (CI/CD)
- Archive rules

## Common CLI
| Command | Description |
|---|---|
| `aws accessanalyzer create-analyzer` | Create an analyzer |
| `aws accessanalyzer list-findings` | List findings |
| `aws accessanalyzer validate-policy` | Validate a policy |

## Pricing model
Free for external-access analysis; charged per resource for unused-access analysis.

## Works well with
- IAM
- Organizations
- CI/CD pipelines

## Use cases
- Detect unintended access
- Least-privilege policies
- Policy validation in CI

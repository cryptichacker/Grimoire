---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Organizations

## Up
- [[AWS]]

**AWS Organizations** — Centrally manage and govern multiple AWS accounts with consolidated billing and policy controls.

**Category:** Management

## Key concepts
- Management account and member accounts
- Organizational Units (OUs)
- Service Control Policies (SCPs)
- Consolidated billing
- Delegated administration
- Programmatic account creation; tag/backup policies

## Common CLI
| Command | Description |
|---|---|
| `aws organizations create-account` | Create a member account |
| `aws organizations list-accounts` | List accounts |
| `aws organizations create-policy` | Create an SCP or other policy |

## Pricing model
Free (you pay only for underlying usage).

## Works well with
- Control Tower
- IAM Identity Center
- Config, GuardDuty, RAM

## Use cases
- Multi-account governance
- Consolidated billing
- Organization-wide guardrails

---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# AWS Health

## Up
- [[AWS]]

**AWS Health** — Provides ongoing visibility into the health of your AWS resources, services and accounts, with personalized alerts and remediation guidance.

**Category:** Management

## Key concepts
- Service Health (public status) vs Personal Health Dashboard (account-specific)
- Event types: issues, scheduled changes, account notifications
- Affected entities
- Organizational view across accounts
- EventBridge integration

## Common CLI
| Command | Description |
|---|---|
| `aws health describe-events` | List health events |
| `aws health describe-event-details` | Get event details |
| `aws health describe-affected-entities` | List affected resources |

## Pricing model
Personal Health Dashboard is free; the Health API requires Business or Enterprise Support.

## Works well with
- EventBridge
- Organizations
- CloudWatch, SNS

## Use cases
- Proactive incident awareness
- Scheduled-change tracking
- Automated health responses

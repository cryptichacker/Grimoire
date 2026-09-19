---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Trusted Advisor

## Up
- [[AWS]]

**AWS Trusted Advisor** — Inspects your AWS environment and recommends improvements across cost, performance, security, fault tolerance and limits.

**Category:** Management

## Key concepts
- Five check categories
- Full checks require Business/Enterprise Support
- Priority (with a TAM)
- Refreshable checks
- Notifications

## Common CLI
| Command | Description |
|---|---|
| `aws support describe-trusted-advisor-checks` | List available checks |
| `aws support describe-trusted-advisor-check-result` | Get check results |

## Pricing model
Core checks free; the full set requires a Business or Enterprise Support plan.

## Works well with
- AWS Support
- CloudWatch events
- Organizations

## Use cases
- Cost optimization
- Security best-practice checks
- Service-limit monitoring

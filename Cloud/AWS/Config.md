---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Config

## Up
- [[AWS]]

**AWS Config** — Records resource configurations over time and evaluates them against compliance rules.

**Category:** Management

## Key concepts
- Configuration items and the recorder
- Config rules (managed and custom)
- Conformance packs
- Remediation via SSM Automation
- Multi-account aggregators
- Configuration timeline/history

## Common CLI
| Command | Description |
|---|---|
| `aws configservice put-config-rule` | Add a rule |
| `aws configservice describe-config-rules` | List rules |
| `aws configservice get-resource-config-history` | View history |

## Pricing model
Per configuration item recorded plus per rule evaluation.

## Works well with
- CloudTrail
- Security Hub
- Systems Manager, Organizations

## Use cases
- Compliance auditing
- Drift and change tracking
- Governance

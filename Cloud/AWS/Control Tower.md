---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Control Tower

## Up
- [[AWS]]

**AWS Control Tower** — Automates setup of a secure, multi-account landing zone with pre-configured guardrails and an account factory.

**Category:** Management

## Key concepts
- Landing zone
- Account Factory for provisioning
- Controls/guardrails (preventive/detective/proactive)
- OUs and baselines
- Dashboard
- Built on Organizations, Config and IAM Identity Center

## Common CLI
| Command | Description |
|---|---|
| `aws controltower list-enabled-controls` | List active controls |
| `aws controltower enable-control` | Enable a control |

## Pricing model
No charge for Control Tower; you pay for the resources it deploys (Config, CloudTrail, etc.).

## Works well with
- Organizations
- Config, CloudTrail
- IAM Identity Center, Service Catalog

## Use cases
- Secure baseline
- Governed account vending
- Enterprise landing zone

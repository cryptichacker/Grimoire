---
tags: [cloud, azure, identity-security]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Policy

## Up
- [[Azure]]

**Azure Policy** — Enforces organizational standards and assesses compliance by evaluating resources against policy rules.

**Category:** Identity & Security

## Key concepts
- Policy definitions and initiatives
- Assignments and scopes
- Effects (deny/audit/deployIfNotExists)
- Compliance dashboard
- Remediation tasks
- Built-in policy sets

## Common CLI
| Command | Description |
|---|---|
| `az policy definition create` | Create a policy |
| `az policy assignment create` | Assign a policy |
| `az policy state list` | Check compliance state |

## Pricing model
Free (guest configuration may add cost).

## Works well with
- Management Groups / Organizations
- Defender for Cloud
- Blueprints / Landing Zones

## Use cases
- Governance guardrails
- Compliance enforcement
- Auto-remediation

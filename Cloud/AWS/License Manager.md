---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# License Manager

## Up
- [[AWS]]

**AWS License Manager** — Centrally manages software licenses from vendors to track usage, enforce limits and reduce compliance risk.

**Category:** Management

## Key concepts
- License configurations
- Rules and usage limits
- Tracking across accounts
- Bring-your-own-license (BYOL)
- Dedicated Host management

## Common CLI
| Command | Description |
|---|---|
| `aws license-manager create-license-configuration` | Create a license config |
| `aws license-manager list-license-configurations` | List configs |
| `aws license-manager get-license-usage` | Check usage |

## Pricing model
Free (pay for any dedicated infrastructure).

## Works well with
- EC2 / Dedicated Hosts
- Organizations
- Systems Manager

## Use cases
- License compliance
- BYOL tracking
- Cost/usage control

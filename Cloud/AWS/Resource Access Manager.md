---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Resource Access Manager

## Up
- [[AWS]]

**AWS Resource Access Manager** — Securely shares AWS resources across accounts and within your organization without duplicating them.

**Category:** Management

## Key concepts
- Resource shares
- Shareable resource types (subnets, Transit Gateway, License Manager, etc.)
- Sharing within an org or with specific accounts
- Principal associations

## Common CLI
| Command | Description |
|---|---|
| `aws ram create-resource-share` | Create a resource share |
| `aws ram associate-resource-share` | Add resources/principals |
| `aws ram list-resource-shares` | List shares |

## Pricing model
Free (pay only for the shared resources).

## Works well with
- VPC (subnets)
- Transit Gateway
- Organizations, License Manager

## Use cases
- Cross-account resource sharing
- Centralized networking
- Multi-account setups

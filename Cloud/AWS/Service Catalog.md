---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Service Catalog

## Up
- [[AWS]]

**AWS Service Catalog** — Lets organizations curate and govern a catalog of approved IT products for users to self-service deploy.

**Category:** Management

## Key concepts
- Portfolios and products
- CloudFormation-backed products
- Constraints (launch/template)
- Provisioned products
- Access via IAM/Identity Center
- Version control

## Common CLI
| Command | Description |
|---|---|
| `aws servicecatalog create-portfolio` | Create a portfolio |
| `aws servicecatalog create-product` | Add a product |
| `aws servicecatalog provision-product` | Launch a product |

## Pricing model
Small per-API-call charge; you pay for the provisioned resources.

## Works well with
- CloudFormation
- Organizations, Control Tower
- IAM

## Use cases
- Self-service provisioning
- Standardized approved stacks
- Governance guardrails

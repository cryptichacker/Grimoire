---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Proton

## Up
- [[AWS]]

**AWS Proton** — A deployment service for platform teams to define standardized infrastructure templates for developers to self-serve.

**Category:** Management

## Key concepts
- Environment and service templates
- Template versions
- Provisioned environments and services
- Self-service for developers
- IaC (CloudFormation/Terraform)

## Common CLI
| Command | Description |
|---|---|
| `aws proton create-environment-template` | Create an environment template |
| `aws proton create-environment` | Provision an environment |
| `aws proton create-service` | Deploy a service |

## Pricing model
Free service; pay for the provisioned resources.

## Works well with
- CloudFormation / Terraform
- CodePipeline
- ECS/EKS, Lambda

## Use cases
- Platform engineering
- Standardized deployments
- Self-service infrastructure

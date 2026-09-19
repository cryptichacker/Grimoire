---
tags: [cloud, aws, compute]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Serverless Application Repository

## Up
- [[AWS]]

**AWS Serverless Application Repository** — A managed repository to find, publish and deploy reusable serverless applications packaged with SAM.

**Category:** Compute

## Key concepts
- Published applications and versions
- Deploy via nested CloudFormation
- Public and private sharing
- SAM-based packaging

## Common CLI
| Command | Description |
|---|---|
| `aws serverlessrepo list-applications` | List available applications |
| `aws serverlessrepo create-application` | Publish an application |
| `aws serverlessrepo create-cloud-formation-template` | Prepare a deploy template |

## Pricing model
Free; pay for the deployed resources.

## Works well with
- Lambda
- CloudFormation / SAM
- API Gateway

## Use cases
- Reusable serverless apps
- Sharing components
- Quick-start deployments

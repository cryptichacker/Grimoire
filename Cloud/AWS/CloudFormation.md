---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# CloudFormation

## Up
- [[AWS]]

**AWS CloudFormation** — Provision and manage AWS infrastructure as code using declarative JSON/YAML templates and stacks.

**Category:** Management

## Key concepts
- Templates
- Stacks and change sets
- Parameters, mappings, outputs
- Resources and intrinsic functions
- Nested stacks
- StackSets (multi-account/region), drift detection, rollback

## Common CLI
| Command | Description |
|---|---|
| `aws cloudformation deploy` | Create or update a stack |
| `aws cloudformation create-stack` | Create a stack |
| `aws cloudformation describe-stacks` | Inspect stacks |
| `aws cloudformation delete-stack` | Tear down a stack |

## Pricing model
No charge for CloudFormation; you pay for the resources the stack creates.

## Works well with
- Nearly all AWS services
- IAM
- SAM and CDK (which synthesize to it)

## Use cases
- Repeatable environments
- Infrastructure as code
- Multi-account rollouts

---
tags: [cloud, aws, developer-tools]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Cloud Development Kit

## Up
- [[AWS]]

**AWS Cloud Development Kit (CDK)** — Defines cloud infrastructure in familiar programming languages and provisions it via CloudFormation.

**Category:** Developer Tools

## Key concepts
- Constructs (L1/L2/L3)
- Stacks and apps
- Synthesize to CloudFormation
- Assets and bootstrapping
- CDK Pipelines

## Common CLI
| Command | Description |
|---|---|
| `cdk init app --language typescript` | Scaffold a project |
| `cdk synth` | Synthesize CloudFormation |
| `cdk deploy` | Deploy the stack |
| `cdk diff / cdk destroy` | Diff or tear down |

## Pricing model
Free (open-source toolkit); pay for the provisioned resources.

## Works well with
- CloudFormation
- CodePipeline
- Construct Hub

## Use cases
- Infrastructure as code in real languages
- Reusable infra constructs
- IaC pipelines

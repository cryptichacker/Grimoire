---
tags: [cloud, aws, developer-tools]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# AWS SAM

## Up
- [[AWS]]

**AWS Serverless Application Model** — An open-source framework and CLI for building serverless applications with a concise template that transforms to CloudFormation.

**Category:** Developer Tools

## Key concepts
- SAM template (shorthand resources)
- sam build/deploy pipeline
- Local testing and debugging
- Guided deploys
- SAM Accelerate (sam sync)

## Common CLI
| Command | Description |
|---|---|
| `sam init` | Scaffold a serverless app |
| `sam build` | Build the app |
| `sam deploy --guided` | Deploy the stack |
| `sam local invoke` | Test a function locally |

## Pricing model
Free framework; pay for the deployed resources.

## Works well with
- CloudFormation
- Lambda, API Gateway
- CodePipeline

## Use cases
- Serverless app development
- Local Lambda testing
- IaC for serverless

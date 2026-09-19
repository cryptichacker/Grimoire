---
tags: [cloud, aws, developer-tools]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# CodePipeline

## Up
- [[AWS]]

**AWS CodePipeline** — Fully managed continuous-delivery service that models, visualizes and automates release pipelines.

**Category:** Developer Tools

## Key concepts
- Pipelines, stages and actions
- Source/build/test/deploy stages
- Manual approvals
- Artifacts passed between stages
- Integrations with CodeBuild/CodeDeploy and partners

## Common CLI
| Command | Description |
|---|---|
| `aws codepipeline create-pipeline` | Create a pipeline |
| `aws codepipeline start-pipeline-execution` | Run the pipeline |
| `aws codepipeline get-pipeline-state` | Check pipeline state |

## Pricing model
Per active pipeline per month.

## Works well with
- CodeBuild, CodeDeploy
- CloudFormation
- GitHub / source repos

## Use cases
- CI/CD automation
- Multi-stage releases
- Approval workflows

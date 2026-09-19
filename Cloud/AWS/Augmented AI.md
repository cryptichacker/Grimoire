---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Augmented AI

## Up
- [[AWS]]

**Amazon Augmented AI (A2I)** — Makes it easy to add human review of machine-learning predictions when confidence is low or spot-checks are needed.

**Category:** Machine Learning

## Key concepts
- Human review workflows (flow definitions)
- Human loops
- Worker task templates
- Workforces (private/vendor/Mechanical Turk)
- Integration with ML services

## Common CLI
| Command | Description |
|---|---|
| `aws sagemaker create-flow-definition` | Define a review workflow |
| `aws sagemaker-a2i-runtime start-human-loop` | Start a human review |
| `aws sagemaker-a2i-runtime list-human-loops` | List review loops |

## Pricing model
Per object reviewed by humans plus workforce cost.

## Works well with
- Textract, Rekognition
- SageMaker
- S3

## Use cases
- Human-in-the-loop review
- Low-confidence prediction checks
- Data labeling QA

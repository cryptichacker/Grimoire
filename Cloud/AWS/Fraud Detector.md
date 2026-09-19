---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Fraud Detector

## Up
- [[AWS]]

**Amazon Fraud Detector** — Managed service that builds, deploys and manages fraud-detection models using your data plus Amazon expertise.

**Category:** Machine Learning

## Key concepts
- Event types and entities
- Models trained on your historical data
- Detectors and rules
- Real-time predictions
- Outcomes and variables

## Common CLI
| Command | Description |
|---|---|
| `aws frauddetector put-detector` | Create a detector |
| `aws frauddetector create-model` | Create a model |
| `aws frauddetector get-event-prediction` | Score an event |

## Pricing model
Per fraud prediction plus model training and hosting.

## Works well with
- S3 (training data)
- Lambda
- SageMaker

## Use cases
- Online payment fraud
- Fake account detection
- Promo/abuse detection

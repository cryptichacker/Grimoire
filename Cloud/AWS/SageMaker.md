---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# SageMaker

## Up
- [[AWS]]

**Amazon SageMaker** — End-to-end platform to build, train, tune and deploy machine-learning models at scale, including generative AI.

**Category:** Machine Learning

## Key concepts
- Studio IDE and notebooks
- Training jobs and built-in algorithms
- Hyperparameter tuning
- Endpoints (real-time/serverless/async) and batch transform
- Pipelines and Model Registry
- Feature Store and JumpStart

## Common CLI
| Command | Description |
|---|---|
| `aws sagemaker create-training-job` | Train a model |
| `aws sagemaker create-model` | Register a model |
| `aws sagemaker create-endpoint` | Deploy an endpoint |
| `aws sagemaker create-notebook-instance` | Start a notebook |

## Pricing model
Per-second compute for notebooks, training and endpoints plus storage.

## Works well with
- S3
- ECR
- Glue, Lambda

## Use cases
- Custom ML models
- Model hosting
- MLOps pipelines

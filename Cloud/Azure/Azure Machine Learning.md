---
tags: [cloud, azure, ai]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Machine Learning

## Up
- [[Azure]]

**Azure Machine Learning** — An enterprise platform to build, train, deploy and manage machine-learning models across the lifecycle (MLOps).

**Category:** AI

## Key concepts
- Workspaces and compute (clusters/instances)
- Jobs, environments and datastores
- Designer and notebooks
- Automated ML
- Managed online/batch endpoints
- Model registry, pipelines and MLflow

## Common CLI
| Command | Description |
|---|---|
| `az ml workspace create` | Create a workspace |
| `az ml job create` | Submit a training job |
| `az ml model create` | Register a model |
| `az ml online-endpoint create` | Deploy an endpoint |

## Pricing model
Per compute-hour for training/inference plus storage.

## Works well with
- Data Lake Storage
- Container Registry
- Azure OpenAI / AI Foundry

## Use cases
- Custom ML models
- MLOps pipelines
- Model hosting

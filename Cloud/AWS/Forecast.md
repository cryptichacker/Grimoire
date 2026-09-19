---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Forecast

## Up
- [[AWS]]

**Amazon Forecast** — Time-series forecasting service that uses machine learning to predict future business outcomes.

**Category:** Machine Learning

## Key concepts
- Datasets and dataset groups
- Predictors (AutoML or algorithms)
- Forecasts and quantiles
- Related time series and item metadata
- What-if analyses

## Common CLI
| Command | Description |
|---|---|
| `aws forecast create-dataset` | Create a dataset |
| `aws forecast create-predictor` | Train a predictor |
| `aws forecast create-forecast` | Generate a forecast |

## Pricing model
Per forecasted data point, training hour and storage.

## Works well with
- S3
- QuickSight
- Lambda

## Use cases
- Demand forecasting
- Inventory planning
- Financial/capacity forecasting

---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Personalize

## Up
- [[AWS]]

**Amazon Personalize** — Creates real-time personalized recommendations using the same ML technology as Amazon.com.

**Category:** Machine Learning

## Key concepts
- Dataset groups (interactions/items/users)
- Recipes and solutions
- Campaigns (real-time) and batch inference
- Event tracking
- Filters and business rules

## Common CLI
| Command | Description |
|---|---|
| `aws personalize create-dataset-group` | Create a dataset group |
| `aws personalize create-solution` | Train a solution |
| `aws personalize-runtime get-recommendations` | Get recommendations |

## Pricing model
Per training hour, ingestion and recommendation requests.

## Works well with
- S3
- Kinesis (events)
- Lambda

## Use cases
- Product recommendations
- Personalized ranking
- Content discovery

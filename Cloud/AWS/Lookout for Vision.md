---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Lookout for Vision

## Up
- [[AWS]]

**Amazon Lookout for Vision** — Uses computer vision to spot defects and anomalies in images for industrial quality inspection.

**Category:** Machine Learning

## Key concepts
- Projects and datasets
- Anomaly-detection models
- Normal vs anomalous training
- Edge deployment (Greengrass)
- Trial detections

## Common CLI
| Command | Description |
|---|---|
| `aws lookoutvision create-project` | Create a project |
| `aws lookoutvision create-model` | Train a model |
| `aws lookoutvision detect-anomalies` | Inspect an image |

## Pricing model
Per training hour and per image inference (cloud or edge).

## Works well with
- S3
- IoT Greengrass
- IoT SiteWise

## Use cases
- Visual quality inspection
- Defect detection
- Manufacturing QA

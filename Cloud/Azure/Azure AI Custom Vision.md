---
tags: [cloud, azure, ai]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure AI Custom Vision

## Up
- [[Azure]]

**Azure AI Custom Vision** — Lets you build, deploy and improve custom image-classification and object-detection models with your own images.

**Category:** AI

## Key concepts
- Projects (classification/object detection)
- Tagged training images
- Iterations and publishing
- Prediction API
- Edge/container export

## Common CLI
| Command | Description |
|---|---|
| `az cognitiveservices account create --kind CustomVision.Training` | Create a training resource |
| `az cognitiveservices account create --kind CustomVision.Prediction` | Create a prediction resource |

## Pricing model
Per training hour and per prediction transaction.

## Works well with
- Blob Storage
- IoT Edge (export)
- AI Vision

## Use cases
- Custom image classification
- Object detection
- Edge vision models

---
tags: [cloud, azure, ai]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure AI Vision

## Up
- [[Azure]]

**Azure AI Vision** — Prebuilt computer-vision APIs for analyzing images and video — objects, text (OCR), faces and spatial analysis.

**Category:** AI

## Key concepts
- Image analysis and tagging
- OCR (Read API)
- Face detection/recognition
- Spatial analysis
- Background removal
- Video retrieval/analysis

## Common CLI
| Command | Description |
|---|---|
| `az cognitiveservices account create --kind ComputerVision` | Create a Vision resource |
| `az cognitiveservices account keys list` | Get keys |

## Pricing model
Per transaction/API call by feature.

## Works well with
- Blob Storage
- Functions
- AI Document Intelligence

## Use cases
- Image analysis and OCR
- Content moderation
- Visual search

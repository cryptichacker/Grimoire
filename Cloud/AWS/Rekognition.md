---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Rekognition

## Up
- [[AWS]]

**Amazon Rekognition** — Adds image and video analysis to applications — objects, faces, text, moderation and more — via API.

**Category:** Machine Learning

## Key concepts
- Label and object detection
- Face detection, comparison and search
- Text in image (OCR)
- Content moderation
- Custom Labels
- Streaming and stored video analysis

## Common CLI
| Command | Description |
|---|---|
| `aws rekognition detect-labels` | Detect objects/scenes |
| `aws rekognition detect-faces` | Analyze faces |
| `aws rekognition start-face-detection` | Start a video analysis job |

## Pricing model
Per image or per minute of video processed.

## Works well with
- S3
- Lambda
- Kinesis Video Streams

## Use cases
- Image and video moderation
- Face-based verification
- Media tagging

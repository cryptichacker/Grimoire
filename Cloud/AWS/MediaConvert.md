---
tags: [cloud, aws, media]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# MediaConvert

## Up
- [[AWS]]

**AWS Elemental MediaConvert** — File-based video transcoding service that converts content into formats for broadcast and multiscreen delivery.

**Category:** Media

## Key concepts
- Transcoding jobs and job templates
- Output groups (HLS/DASH/CMAF/file)
- Input clipping and stitching
- Captions, audio and DRM
- Job queues

## Common CLI
| Command | Description |
|---|---|
| `aws mediaconvert describe-endpoints` | Get your account endpoint |
| `aws mediaconvert create-job` | Submit a transcode job |
| `aws mediaconvert list-jobs` | List jobs |

## Pricing model
Per minute of output video by resolution and features.

## Works well with
- S3
- CloudFront
- MediaPackage

## Use cases
- VOD transcoding
- Adaptive bitrate packaging
- Media format conversion

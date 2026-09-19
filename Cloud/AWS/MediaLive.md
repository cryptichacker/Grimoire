---
tags: [cloud, aws, media]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# MediaLive

## Up
- [[AWS]]

**AWS Elemental MediaLive** — Broadcast-grade live video encoding service that turns live sources into streams for TVs and connected devices.

**Category:** Media

## Key concepts
- Channels and inputs
- Live encoding to ABR outputs
- Input redundancy and failover
- Standard vs single-pipeline
- Statmux (for broadcast)

## Common CLI
| Command | Description |
|---|---|
| `aws medialive create-input` | Create a live input |
| `aws medialive create-channel` | Create a channel |
| `aws medialive start-channel` | Start streaming |

## Pricing model
Per channel-hour by input/output configuration.

## Works well with
- MediaPackage
- MediaConnect
- S3, CloudFront

## Use cases
- Live streaming
- Broadcast to OTT
- Live event delivery

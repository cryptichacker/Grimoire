---
tags: [cloud, aws, media]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Interactive Video Service

## Up
- [[AWS]]

**Amazon Interactive Video Service** — Managed live streaming (IVS) for building low-latency, interactive video experiences into apps quickly.

**Category:** Media

## Key concepts
- Channels and stream keys
- Low-latency and real-time streaming
- Playback via player SDKs
- Timed metadata
- Stage (real-time multi-host)

## Common CLI
| Command | Description |
|---|---|
| `aws ivs create-channel` | Create a channel |
| `aws ivs create-stream-key` | Create a stream key |
| `aws ivs get-stream` | Get live stream status |

## Pricing model
Per hour of input and per hour of output delivered.

## Works well with
- S3 (recording)
- CloudFront
- Lambda / EventBridge

## Use cases
- Live interactive streaming
- Creator platforms
- Live shopping and events

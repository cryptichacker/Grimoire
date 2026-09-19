---
tags: [cloud, aws, media]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# MediaPackage

## Up
- [[AWS]]

**AWS Elemental MediaPackage** — Just-in-time packaging and origination that prepares and protects live and VOD video for delivery.

**Category:** Media

## Key concepts
- Channels and origin endpoints
- Just-in-time packaging (HLS/DASH/CMAF)
- DRM and content protection
- Time-shifted viewing (start-over/catch-up)
- Live-to-VOD

## Common CLI
| Command | Description |
|---|---|
| `aws mediapackagev2 create-channel-group` | Create a channel group |
| `aws mediapackagev2 create-channel` | Create a channel |
| `aws mediapackagev2 create-origin-endpoint` | Create an origin endpoint |

## Pricing model
Per GB of content packaged and delivered.

## Works well with
- MediaLive
- CloudFront
- S3

## Use cases
- Video origination
- Multi-DRM packaging
- Time-shifted TV

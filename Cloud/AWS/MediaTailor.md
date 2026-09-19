---
tags: [cloud, aws, media]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# MediaTailor

## Up
- [[AWS]]

**AWS Elemental MediaTailor** — Personalized ad insertion and channel assembly for streaming video, with server-side ad insertion (SSAI).

**Category:** Media

## Key concepts
- Playback configurations
- Server-side ad insertion (SSAI)
- Channel Assembly (virtual linear channels)
- Ad decision server integration
- Reporting/beaconing

## Common CLI
| Command | Description |
|---|---|
| `aws mediatailor put-playback-configuration` | Create a playback config |
| `aws mediatailor list-playback-configurations` | List configs |
| `aws mediatailor create-channel` | Create a virtual channel |

## Pricing model
Per ad request/transcoded ad and per channel usage.

## Works well with
- MediaPackage
- CloudFront
- Ad servers

## Use cases
- Video monetization
- Server-side ad insertion
- Virtual linear channels

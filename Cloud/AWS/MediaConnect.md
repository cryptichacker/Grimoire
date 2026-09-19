---
tags: [cloud, aws, media]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# MediaConnect

## Up
- [[AWS]]

**AWS Elemental MediaConnect** — A transport service for live video that reliably and securely moves high-value content into, through and out of AWS.

**Category:** Media

## Key concepts
- Flows
- Sources and outputs
- Protocols (Zixi, SRT, RTP, RIST)
- Encryption and redundancy
- Entitlements (share with others)

## Common CLI
| Command | Description |
|---|---|
| `aws mediaconnect create-flow` | Create a flow |
| `aws mediaconnect start-flow` | Start a flow |
| `aws mediaconnect list-flows` | List flows |

## Pricing model
Per flow-hour plus data transfer and outputs.

## Works well with
- MediaLive
- Direct Connect
- S3

## Use cases
- Live video contribution
- Secure video transport
- Broadcast distribution

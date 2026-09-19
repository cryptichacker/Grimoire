---
tags: [cloud, aws, business-applications]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Amazon Connect

## Up
- [[AWS]]

**Amazon Connect** — Cloud contact center that lets you set up omnichannel customer support (voice, chat, tasks) at scale.

**Category:** Business Applications

## Key concepts
- Contact flows
- Omnichannel (voice/chat/task)
- Queues and routing profiles
- Contact Lens analytics
- Agent workspace and Cases

## Common CLI
| Command | Description |
|---|---|
| `aws connect create-instance` | Create a Connect instance |
| `aws connect start-outbound-voice-contact` | Place an outbound call |
| `aws connect list-users` | List agents |

## Pricing model
Per minute of usage plus telephony and optional features.

## Works well with
- Lambda, Lex
- Polly / Transcribe
- QuickSight (Contact Lens)

## Use cases
- Cloud contact centers
- IVR and self-service
- Customer support analytics

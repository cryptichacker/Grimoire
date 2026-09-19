---
tags: [cloud, aws, business-applications]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Pinpoint

## Up
- [[AWS]]

**Amazon Pinpoint** — A multichannel customer-engagement service for targeted campaigns and transactional messages (email, SMS, push, voice).

**Category:** Business Applications

## Key concepts
- Projects and segments
- Campaigns and journeys
- Channels (email/SMS/push/voice)
- Analytics and engagement events
- Templates

## Common CLI
| Command | Description |
|---|---|
| `aws pinpoint create-app` | Create a project |
| `aws pinpoint send-messages` | Send a direct message |
| `aws pinpoint get-campaigns` | List campaigns |

## Pricing model
Per message/endpoint by channel plus targeted events.

## Works well with
- SES
- SNS
- Kinesis (events)

## Use cases
- Marketing campaigns
- Transactional notifications
- Customer journeys

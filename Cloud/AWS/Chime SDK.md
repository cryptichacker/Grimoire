---
tags: [cloud, aws, business-applications]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Chime SDK

## Up
- [[AWS]]

**Amazon Chime SDK** — Developer building blocks to add real-time audio, video, screen sharing and messaging to your own applications.

**Category:** Business Applications

## Key concepts
- Meetings (audio/video)
- Attendees and media pipelines
- Messaging channels
- PSTN audio (SIP media application)
- Voice Connector

## Common CLI
| Command | Description |
|---|---|
| `aws chime-sdk-meetings create-meeting` | Create a meeting |
| `aws chime-sdk-meetings create-attendee` | Add an attendee |
| `aws chime-sdk-messaging create-channel` | Create a messaging channel |

## Pricing model
Per attendee-minute (audio/video) and per message/PSTN usage.

## Works well with
- Lambda
- Kinesis Video Streams
- Transcribe (live)

## Use cases
- In-app video/voice
- Telehealth and collaboration
- Custom communication apps

---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Polly

## Up
- [[AWS]]

**Amazon Polly** — Turns text into lifelike speech using deep-learning voices, including neural and generative options.

**Category:** Machine Learning

## Key concepts
- Standard, neural, long-form and generative voices
- SSML for fine control
- Speech marks
- Lexicons
- Streaming or S3 output
- Newscaster styles

## Common CLI
| Command | Description |
|---|---|
| `aws polly synthesize-speech` | Synthesize audio from text |
| `aws polly describe-voices` | List available voices |

## Pricing model
Per character of text synthesized.

## Works well with
- S3
- Lambda
- Amazon Connect

## Use cases
- Voice for applications
- IVR and telephony
- Accessibility and audio content

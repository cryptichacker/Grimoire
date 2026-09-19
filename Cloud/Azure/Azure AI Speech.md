---
tags: [cloud, azure, ai]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure AI Speech

## Up
- [[Azure]]

**Azure AI Speech** — Speech-to-text, text-to-speech, translation and speaker recognition via API and SDK.

**Category:** AI

## Key concepts
- Speech-to-text (real-time/batch)
- Text-to-speech (neural voices)
- Custom neural voice
- Speech translation
- Speaker recognition
- Pronunciation assessment

## Common CLI
| Command | Description |
|---|---|
| `az cognitiveservices account create --kind SpeechServices` | Create a Speech resource |
| `az cognitiveservices account keys list` | Get keys |

## Pricing model
Per hour of audio or per character synthesized.

## Works well with
- Bot Service
- Communication Services
- Functions

## Use cases
- Voice interfaces
- Transcription/captioning
- Accessibility and audio content

---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Translate

## Up
- [[AWS]]

**Amazon Translate** — Neural machine translation service that localizes text between many languages via API.

**Category:** Machine Learning

## Key concepts
- Real-time and batch translation
- Automatic source-language detection
- Custom terminology
- Active custom translation
- Formality and profanity settings

## Common CLI
| Command | Description |
|---|---|
| `aws translate translate-text` | Translate a string |
| `aws translate start-text-translation-job` | Batch translate documents |
| `aws translate list-languages` | List supported languages |

## Pricing model
Per character of text translated.

## Works well with
- S3
- Comprehend
- Transcribe/Polly

## Use cases
- App and website localization
- Multilingual chat/support
- Document translation

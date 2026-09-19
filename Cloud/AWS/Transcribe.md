---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Transcribe

## Up
- [[AWS]]

**Amazon Transcribe** — Automatic speech recognition that converts audio and video to text, in batch or real-time streaming.

**Category:** Machine Learning

## Key concepts
- Batch and streaming transcription
- Speaker diarization
- Custom vocabulary and language models
- Automatic language identification
- Content redaction
- Call Analytics and Medical

## Common CLI
| Command | Description |
|---|---|
| `aws transcribe start-transcription-job` | Start a batch job |
| `aws transcribe get-transcription-job` | Get the result |
| `aws transcribe list-transcription-jobs` | List jobs |

## Pricing model
Per second of audio transcribed.

## Works well with
- S3
- Lambda
- Comprehend, Amazon Connect

## Use cases
- Meeting and call transcripts
- Media subtitling
- Voice analytics

---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Comprehend

## Up
- [[AWS]]

**Amazon Comprehend** — Natural-language-processing service that extracts insights such as sentiment, entities, key phrases and language from text.

**Category:** Machine Learning

## Key concepts
- Sentiment and entity detection
- Key-phrase and language detection
- PII detection and redaction
- Topic modeling
- Custom classification and entities
- Comprehend Medical

## Common CLI
| Command | Description |
|---|---|
| `aws comprehend detect-sentiment` | Detect sentiment |
| `aws comprehend detect-entities` | Detect entities |
| `aws comprehend detect-key-phrases` | Extract key phrases |

## Pricing model
Per unit of text (100 characters) analyzed.

## Works well with
- S3
- Lambda
- Kinesis

## Use cases
- Customer-feedback analysis
- PII detection
- Document classification

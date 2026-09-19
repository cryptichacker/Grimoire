---
tags: [cloud, azure, ai]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure AI Language

## Up
- [[Azure]]

**Azure AI Language** — A consolidated natural-language-processing service for understanding and analyzing text.

**Category:** AI

## Key concepts
- Sentiment and opinion mining
- Named-entity recognition and PII detection
- Key-phrase extraction
- Summarization
- Conversational Language Understanding (CLU)
- Custom question answering

## Common CLI
| Command | Description |
|---|---|
| `az cognitiveservices account create --kind TextAnalytics` | Create a Language resource |
| `az cognitiveservices account keys list` | Get keys |

## Pricing model
Per text record (1,000 characters) analyzed.

## Works well with
- Bot Service
- AI Search
- Functions / Logic Apps

## Use cases
- Text analytics
- Intent/entity understanding
- Question answering

---
tags: [cloud, azure, ai]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure AI Content Safety

## Up
- [[Azure]]

**Azure AI Content Safety** — Detects and moderates harmful or unsafe content (text and images) for AI-generated and user-generated content.

**Category:** AI

## Key concepts
- Text and image moderation
- Severity categories (hate, sexual, violence, self-harm)
- Prompt Shields (jailbreak detection)
- Groundedness detection
- Custom categories and blocklists

## Common CLI
| Command | Description |
|---|---|
| `az cognitiveservices account create --kind ContentSafety` | Create a Content Safety resource |
| `az cognitiveservices account keys list` | Get keys |

## Pricing model
Per API transaction (text/image).

## Works well with
- Azure OpenAI / AI Foundry
- App Service / Functions
- Communication Services

## Use cases
- Content moderation
- LLM guardrails
- Trust and safety

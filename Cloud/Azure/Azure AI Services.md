---
tags: [cloud, azure, ai]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure AI Services

## Up
- [[Azure]]

**Azure AI Services** — A family of prebuilt AI APIs (formerly Cognitive Services) for vision, speech, language and decision tasks.

**Category:** AI

## Key concepts
- Multi-service and single-service resources
- Vision, Speech, Language, Document Intelligence
- Content Safety
- API keys and Entra auth
- Containers for on-prem
- Responsible AI

## Common CLI
| Command | Description |
|---|---|
| `az cognitiveservices account create` | Create an AI Services resource |
| `az cognitiveservices account keys list` | Get API keys |
| `az cognitiveservices account list` | List resources |

## Pricing model
Per transaction/API call by service and tier.

## Works well with
- App Service, Functions
- Azure OpenAI
- Logic Apps

## Use cases
- Vision and OCR
- Speech and translation
- Text analytics/content safety

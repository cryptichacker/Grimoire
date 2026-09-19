---
tags: [cloud, azure, ai]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure AI Translator

## Up
- [[Azure]]

**Azure AI Translator** — A neural machine translation service supporting real-time and document translation across many languages.

**Category:** AI

## Key concepts
- Real-time text translation
- Document translation (preserves formatting)
- Custom Translator models
- Language detection
- Transliteration
- Dictionary lookup

## Common CLI
| Command | Description |
|---|---|
| `az cognitiveservices account create --kind TextTranslation` | Create a Translator resource |
| `az cognitiveservices account keys list` | Get keys |

## Pricing model
Per character translated (document translation per page/character).

## Works well with
- AI Language
- Blob Storage (documents)
- App Service / Functions

## Use cases
- App/website localization
- Document translation
- Multilingual support

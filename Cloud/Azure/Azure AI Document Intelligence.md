---
tags: [cloud, azure, ai]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure AI Document Intelligence

## Up
- [[Azure]]

**Azure AI Document Intelligence** — Extracts text, key-value pairs, tables and structure from documents using prebuilt and custom models (formerly Form Recognizer).

**Category:** AI

## Key concepts
- Prebuilt models (invoice, receipt, ID, layout)
- Custom extraction/classification models
- Layout and OCR
- Query fields
- Confidence scores
- Batch analysis

## Common CLI
| Command | Description |
|---|---|
| `az cognitiveservices account create --kind FormRecognizer` | Create a resource |
| `az cognitiveservices account keys list` | Get keys |

## Pricing model
Per page analyzed by model type.

## Works well with
- Blob Storage
- AI Search
- Logic Apps / Functions

## Use cases
- Document processing
- Invoice/receipt extraction
- Forms automation

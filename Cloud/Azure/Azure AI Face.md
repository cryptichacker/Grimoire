---
tags: [cloud, azure, ai]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure AI Face

## Up
- [[Azure]]

**Azure AI Face** — Detects, analyzes and recognizes human faces in images (access is limited/gated for responsible use).

**Category:** AI

## Key concepts
- Face detection and attributes
- Face verification and identification
- Liveness detection
- Face grouping
- Limited-access/responsible AI gating

## Common CLI
| Command | Description |
|---|---|
| `az cognitiveservices account create --kind Face` | Create a Face resource |
| `az cognitiveservices account keys list` | Get keys |

## Pricing model
Per transaction/API call.

## Works well with
- Blob Storage
- AI Vision
- Entra Verified ID

## Use cases
- Identity verification
- Access control
- Face-based experiences

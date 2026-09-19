---
tags: [cloud, azure, ai]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure OpenAI Service

## Up
- [[Azure]]

**Azure OpenAI Service** — Provides REST access to advanced OpenAI and other foundation models with Azure security, compliance and networking.

**Category:** AI

## Key concepts
- Model deployments (GPT, embeddings, image, audio)
- Chat completions and responses API
- On Your Data (RAG)
- Content filtering
- Provisioned throughput units (PTU) vs pay-as-you-go
- Private networking

## Common CLI
| Command | Description |
|---|---|
| `az cognitiveservices account create --kind OpenAI` | Create an Azure OpenAI resource |
| `az cognitiveservices account deployment create` | Deploy a model |
| `az cognitiveservices account deployment list` | List deployments |

## Pricing model
Per 1,000 tokens (input/output) or provisioned throughput units.

## Works well with
- Azure AI Search (RAG)
- Key Vault, Managed Identity
- Container Apps / Functions

## Use cases
- Chatbots and assistants
- RAG applications
- Content generation and summarization

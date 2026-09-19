---
tags: [cloud, azure, ai]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure AI Foundry

## Up
- [[Azure]]

**Azure AI Foundry** — A unified platform (formerly Azure AI Studio) to build, evaluate, deploy and manage generative-AI apps and agents.

**Category:** AI

## Key concepts
- Hubs and projects
- Model catalog (OpenAI, Meta, Mistral, etc.)
- Prompt flow
- Evaluations and tracing
- Content safety and guardrails
- Agents and RAG

## Common CLI
| Command | Description |
|---|---|
| `az ml workspace create --kind hub` | Create an AI Foundry hub |
| `Portal: AI Foundry project` | Build and deploy in the portal |
| `az ml online-endpoint create` | Deploy a model endpoint |

## Pricing model
Per model/token usage plus underlying compute and storage.

## Works well with
- Azure OpenAI
- AI Search (RAG)
- Machine Learning

## Use cases
- Generative-AI apps
- Agent development
- Model evaluation and deployment

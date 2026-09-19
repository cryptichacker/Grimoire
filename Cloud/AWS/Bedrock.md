---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Bedrock

## Up
- [[AWS]]

**Amazon Bedrock** — Fully managed service to build generative-AI apps using foundation models from Amazon and third parties via one API.

**Category:** Machine Learning

## Key concepts
- Foundation models (Anthropic, Amazon, Meta and more)
- Invoke and streaming APIs
- Knowledge Bases (RAG)
- Agents
- Guardrails
- Fine-tuning and provisioned throughput

## Common CLI
| Command | Description |
|---|---|
| `aws bedrock list-foundation-models` | List available models |
| `aws bedrock-runtime invoke-model` | Call a model |
| `aws bedrock-agent create-agent` | Create an agent |

## Pricing model
Per input/output token (on-demand) or provisioned throughput.

## Works well with
- Lambda
- Knowledge Bases (OpenSearch/S3)
- Step Functions

## Use cases
- Chatbots and assistants
- RAG applications
- Text generation and summarization

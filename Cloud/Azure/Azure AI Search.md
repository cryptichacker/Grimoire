---
tags: [cloud, azure, ai]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure AI Search

## Up
- [[Azure]]

**Azure AI Search** — A managed search-as-a-service (formerly Cognitive Search) with vector, keyword and semantic retrieval for apps and RAG.

**Category:** AI

## Key concepts
- Indexes and indexers
- Vector, keyword and hybrid search
- Semantic ranking
- Skillsets (AI enrichment)
- Integrated vectorization
- Scoring profiles

## Common CLI
| Command | Description |
|---|---|
| `az search service create` | Create a search service |
| `az search admin-key show` | Get the admin key |
| `az search query-key create` | Create a query key |

## Pricing model
Per search unit (replicas x partitions) per hour.

## Works well with
- Azure OpenAI (RAG)
- Blob / Data Lake
- AI Document Intelligence

## Use cases
- Application search
- Retrieval-augmented generation
- Knowledge mining

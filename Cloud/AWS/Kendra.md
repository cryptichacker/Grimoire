---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Kendra

## Up
- [[AWS]]

**Amazon Kendra** — Intelligent, ML-powered enterprise search that returns precise answers from across your content and data sources.

**Category:** Machine Learning

## Key concepts
- Indexes
- Data source connectors
- Natural-language queries
- FAQs and document ranking
- Relevance tuning
- Access-control filtering

## Common CLI
| Command | Description |
|---|---|
| `aws kendra create-index` | Create an index |
| `aws kendra query` | Search the index |
| `aws kendra batch-put-document` | Add documents |

## Pricing model
Per index-hour (Developer/Enterprise editions) plus connector scans.

## Works well with
- S3, RDS
- SharePoint/Confluence connectors
- Lex

## Use cases
- Enterprise search
- Support knowledge bases
- RAG retrieval

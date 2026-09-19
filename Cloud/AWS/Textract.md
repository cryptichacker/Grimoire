---
tags: [cloud, aws, machine-learning]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Textract

## Up
- [[AWS]]

**Amazon Textract** — Automatically extracts text, forms and tables from scanned documents and images beyond simple OCR.

**Category:** Machine Learning

## Key concepts
- Text, line and word detection
- Forms (key-value) and tables
- Queries
- Analyze Expense and Analyze ID
- Sync vs async for multipage docs
- Human review via A2I

## Common CLI
| Command | Description |
|---|---|
| `aws textract detect-document-text` | Plain OCR |
| `aws textract analyze-document` | Extract forms/tables |
| `aws textract start-document-analysis` | Async multipage job |

## Pricing model
Per page processed (more for forms, tables and queries).

## Works well with
- S3
- Lambda
- Comprehend, A2I

## Use cases
- Document processing
- Invoice and receipt extraction
- ID verification

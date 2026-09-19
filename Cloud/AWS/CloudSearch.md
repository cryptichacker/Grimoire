---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# CloudSearch

## Up
- [[AWS]]

**Amazon CloudSearch** — A managed search service to set up, manage and scale a search solution for your website or application.

**Category:** Analytics

## Key concepts
- Search domains
- Index fields and analysis schemes
- Automatic scaling
- Faceting, highlighting, suggestions
- Search API

## Common CLI
| Command | Description |
|---|---|
| `aws cloudsearch create-domain` | Create a search domain |
| `aws cloudsearch index-documents` | Rebuild the index |
| `aws cloudsearchdomain search` | Run a search query |

## Pricing model
Per search instance-hour plus document batch uploads and storage.

## Works well with
- S3
- Lambda
- DynamoDB

## Use cases
- Site/app search
- Faceted product search
- Auto-complete suggestions

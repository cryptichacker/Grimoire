---
tags: [cloud, aws, application-integration]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# B2B Data Interchange

## Up
- [[AWS]]

**AWS B2B Data Interchange** — Automates the transformation of EDI documents into common data formats to modernize business-partner integrations.

**Category:** Application Integration

## Key concepts
- Profiles and trading partners
- Transformers (EDI to JSON/XML)
- Partnerships
- X12 EDI support
- Event-driven processing

## Common CLI
| Command | Description |
|---|---|
| `aws b2bi create-profile` | Create a profile |
| `aws b2bi create-transformer` | Create a transformer |
| `aws b2bi create-partnership` | Set up a partnership |

## Pricing model
Per transaction/document processed plus storage.

## Works well with
- S3
- Transfer Family
- EventBridge, Lambda

## Use cases
- EDI processing
- Supply-chain integration
- Partner onboarding

---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Data Exchange

## Up
- [[AWS]]

**AWS Data Exchange** — A marketplace to find, subscribe to and use third-party data sets in the cloud, delivered via S3, APIs or Redshift.

**Category:** Analytics

## Key concepts
- Data products and revisions
- Subscriptions and entitlements
- Delivery via S3/API/Redshift/Lakehouse
- Provider publishing
- Automated updates

## Common CLI
| Command | Description |
|---|---|
| `aws dataexchange list-data-sets` | List entitled data sets |
| `aws dataexchange create-job` | Export/import a revision |
| `aws dataexchange get-revision` | Get revision details |

## Pricing model
Subscription fees set by data providers plus any API/export costs.

## Works well with
- S3
- Redshift
- Lake Formation

## Use cases
- Third-party data ingestion
- Data monetization
- Enriching analytics

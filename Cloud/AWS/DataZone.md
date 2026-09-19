---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# DataZone

## Up
- [[AWS]]

**Amazon DataZone** — A data-management service to catalog, discover, share and govern data across the organization with a business catalog.

**Category:** Analytics

## Key concepts
- Domains and projects
- Business data catalog with glossary
- Data assets and subscriptions
- Governed publishing/subscribing
- Producer/consumer projects

## Common CLI
| Command | Description |
|---|---|
| `aws datazone create-domain` | Create a domain |
| `aws datazone create-project` | Create a project |
| `aws datazone create-data-source` | Add a data source |

## Pricing model
Per user plus metadata/API usage; underlying data services billed separately.

## Works well with
- Glue Data Catalog
- Redshift, Athena
- Lake Formation

## Use cases
- Data cataloging
- Self-service data sharing
- Data governance

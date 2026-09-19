---
tags: [cloud, aws, application-integration]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# AppFlow

## Up
- [[AWS]]

**Amazon AppFlow** — No-code service to securely transfer data between SaaS applications and AWS services on a schedule or event.

**Category:** Application Integration

## Key concepts
- Flows
- Connectors (Salesforce, Slack and more)
- Source/destination field mapping
- Filters and transformations
- Triggers (on-demand/scheduled/event)
- PrivateLink transfer

## Common CLI
| Command | Description |
|---|---|
| `aws appflow create-flow` | Create a flow |
| `aws appflow start-flow` | Run a flow |
| `aws appflow describe-flow` | Get flow details |

## Pricing model
Per flow run plus per GB of data processed.

## Works well with
- S3, Redshift
- Salesforce and other SaaS
- EventBridge

## Use cases
- SaaS data ingestion
- App-to-app sync
- Analytics data collection

---
tags: [cloud, aws, analytics]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Lake Formation

## Up
- [[AWS]]

**AWS Lake Formation** — Simplifies building, securing and governing data lakes on S3 with centralized, fine-grained permissions.

**Category:** Analytics

## Key concepts
- Data-lake registration
- Blueprints and workflows
- Fine-grained (table/column/row) permissions
- LF-Tags for access control
- Governed tables
- Cross-account sharing

## Common CLI
| Command | Description |
|---|---|
| `aws lakeformation register-resource` | Register S3 data |
| `aws lakeformation grant-permissions` | Grant access |
| `aws lakeformation list-permissions` | List permissions |

## Pricing model
No extra charge; pay for underlying Glue/S3/Athena usage.

## Works well with
- Glue
- Athena
- Redshift Spectrum, EMR

## Use cases
- Data-lake governance
- Centralized data access control
- Secure data sharing

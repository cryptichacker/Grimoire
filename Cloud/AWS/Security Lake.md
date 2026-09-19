---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Security Lake

## Up
- [[AWS]]

**Amazon Security Lake** — Automatically centralizes security data from AWS, SaaS and on-prem into a purpose-built data lake in OCSF format.

**Category:** Security

## Key concepts
- Centralized security data lake on S3
- Open Cybersecurity Schema Framework (OCSF)
- Sources and custom sources
- Subscribers (query/data access)
- Multi-account/Region rollup

## Common CLI
| Command | Description |
|---|---|
| `aws securitylake create-data-lake` | Create the security data lake |
| `aws securitylake create-subscriber` | Add a subscriber |
| `aws securitylake list-log-sources` | List log sources |

## Pricing model
No service fee; pay for underlying S3, Glue and query usage.

## Works well with
- S3, Glue, Athena
- Security Hub, GuardDuty
- OpenSearch / partner SIEMs

## Use cases
- Central security data lake
- SIEM data feeds
- Threat analytics

---
tags: [cloud, aws, management]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# CloudTrail

## Up
- [[AWS]]

**AWS CloudTrail** — Records account activity and API calls for governance, compliance and security auditing.

**Category:** Management

## Key concepts
- Management vs data events
- Trails (multi-region and organization)
- Event history (last 90 days)
- Log file integrity validation
- CloudTrail Lake for SQL queries
- Delivery to S3 and CloudWatch Logs

## Common CLI
| Command | Description |
|---|---|
| `aws cloudtrail create-trail` | Create a trail |
| `aws cloudtrail lookup-events` | Search recent events |
| `aws cloudtrail start-logging` | Enable a trail |

## Pricing model
First management-event trail is free; extra trails, data events and Lake cost.

## Works well with
- S3, CloudWatch Logs
- EventBridge
- Security Hub, Athena

## Use cases
- Audit trails
- Security forensics
- Compliance

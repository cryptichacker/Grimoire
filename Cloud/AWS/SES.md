---
tags: [cloud, aws, business-applications]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# SES

## Up
- [[AWS]]

**Amazon Simple Email Service** — Scalable, cost-effective email service for transactional, marketing and bulk email sending and receiving.

**Category:** Business Applications

## Key concepts
- Verified identities (domains/emails)
- Configuration sets and dedicated IPs
- DKIM/SPF/DMARC
- Sending and reputation dashboards
- Inbound email receiving

## Common CLI
| Command | Description |
|---|---|
| `aws ses verify-email-identity` | Verify a sender identity |
| `aws sesv2 send-email` | Send an email |
| `aws sesv2 create-configuration-set` | Create a configuration set |

## Pricing model
Per 1,000 emails sent plus data and dedicated-IP options.

## Works well with
- Lambda, SNS
- S3 (inbound)
- EventBridge

## Use cases
- Transactional email
- Marketing campaigns
- Inbound email processing

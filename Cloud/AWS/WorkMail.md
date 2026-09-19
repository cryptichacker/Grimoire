---
tags: [cloud, aws, business-applications]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# WorkMail

## Up
- [[AWS]]

**Amazon WorkMail** — Managed, secure business email and calendaring with support for existing desktop and mobile clients.

**Category:** Business Applications

## Key concepts
- Organizations and users/groups
- IMAP and Exchange ActiveSync support
- Calendar and contacts
- Encryption with KMS
- Optional Active Directory integration

## Common CLI
| Command | Description |
|---|---|
| `aws workmail create-organization` | Create an organization |
| `aws workmail create-user` | Create a user |
| `aws workmail register-to-work-mail` | Enable a mailbox |

## Pricing model
Per user mailbox per month.

## Works well with
- Directory Service
- KMS
- SES

## Use cases
- Business email hosting
- Calendaring
- Outlook/mobile mail

---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# WAF

## Up
- [[AWS]]

**AWS WAF** — Web application firewall that inspects HTTP(S) requests and blocks common exploits and unwanted traffic.

**Category:** Security

## Key concepts
- Web ACLs
- Rules and rule groups
- Managed rule groups (AWS and Marketplace)
- Rate-based rules
- Conditions on IPs, headers, SQLi/XSS
- Logging and metrics

## Common CLI
| Command | Description |
|---|---|
| `aws wafv2 create-web-acl` | Create a web ACL |
| `aws wafv2 associate-web-acl` | Attach it to a resource |
| `aws wafv2 update-web-acl` | Edit rules |

## Pricing model
Per web ACL per month plus per rule and per million requests.

## Works well with
- CloudFront, ALB
- API Gateway, AppSync
- Shield

## Use cases
- Block SQLi/XSS
- Rate limiting
- Bot and geo filtering

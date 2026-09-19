---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# PrivateLink

## Up
- [[AWS]]

**AWS PrivateLink** — Provides private connectivity between VPCs, AWS services and on-prem without traversing the public internet, via interface endpoints.

**Category:** Networking

## Key concepts
- Interface VPC endpoints (ENI)
- Endpoint services fronted by an NLB
- Gateway endpoints for S3/DynamoDB
- Private DNS integration
- Cross-account and cross-Region access

## Common CLI
| Command | Description |
|---|---|
| `aws ec2 create-vpc-endpoint` | Create an endpoint |
| `aws ec2 create-vpc-endpoint-service-configuration` | Publish an endpoint service |
| `aws ec2 describe-vpc-endpoints` | List endpoints |

## Pricing model
Per endpoint-hour plus per-GB data processed.

## Works well with
- VPC
- Elastic Load Balancing (NLB)
- Route 53

## Use cases
- Private access to AWS services
- SaaS provider connectivity
- Avoiding internet egress

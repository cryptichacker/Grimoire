---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Global Accelerator

## Up
- [[AWS]]

**AWS Global Accelerator** — Uses the AWS global network and anycast static IPs to improve availability and performance for global users.

**Category:** Networking

## Key concepts
- Static anycast IP addresses
- Edge network entry points
- Endpoint groups per Region
- Traffic dials and weights
- Health checks and fast failover
- TCP and UDP

## Common CLI
| Command | Description |
|---|---|
| `aws globalaccelerator create-accelerator` | Create an accelerator |
| `aws globalaccelerator create-listener` | Add a listener |
| `aws globalaccelerator create-endpoint-group` | Add endpoints |

## Pricing model
Fixed hourly accelerator fee plus per-GB premium data transfer.

## Works well with
- Elastic Load Balancing
- EC2 / Elastic IPs
- Shield

## Use cases
- Global low-latency access
- Multi-Region failover
- Gaming and real-time apps

---
tags: [cloud, aws, networking]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Route 53 Application Recovery Controller

## Up
- [[AWS]]

**Amazon Route 53 Application Recovery Controller** — Continuously verifies recovery readiness and provides highly reliable routing controls for multi-Region failover.

**Category:** Networking

## Key concepts
- Readiness checks
- Routing controls and control panels
- Safety rules
- Highly available cluster of endpoints

## Common CLI
| Command | Description |
|---|---|
| `aws route53-recovery-readiness create-recovery-group` | Create a recovery group |
| `aws route53-recovery-cluster get-routing-control-state` | Read a routing control |
| `aws route53-recovery-cluster update-routing-control-state` | Flip a routing control |

## Pricing model
Per readiness check and per cluster/routing control.

## Works well with
- Route 53
- Elastic Load Balancing
- CloudWatch

## Use cases
- Multi-Region failover
- DR readiness verification
- Reliable traffic switching

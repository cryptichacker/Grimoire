---
tags: [cloud, aws, security]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Network Firewall

## Up
- [[AWS]]

**AWS Network Firewall** — Managed, stateful network firewall and intrusion prevention for your VPCs.

**Category:** Security

## Key concepts
- Firewalls and firewall policies
- Stateless and stateful rule groups
- Suricata-compatible rules
- Domain and IP filtering
- Logging to S3/CloudWatch/Firehose

## Common CLI
| Command | Description |
|---|---|
| `aws network-firewall create-firewall` | Create a firewall |
| `aws network-firewall create-firewall-policy` | Create a policy |
| `aws network-firewall create-rule-group` | Create rules |

## Pricing model
Per firewall endpoint-hour plus per-GB traffic processed.

## Works well with
- VPC
- Transit Gateway
- Firewall Manager

## Use cases
- VPC traffic inspection
- Egress filtering
- IPS/IDS

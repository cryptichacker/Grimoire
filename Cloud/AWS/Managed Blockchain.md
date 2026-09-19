---
tags: [cloud, aws, blockchain]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Managed Blockchain

## Up
- [[AWS]]

**Amazon Managed Blockchain** — Creates and manages scalable blockchain networks and provides node access to public blockchains.

**Category:** Blockchain

## Key concepts
- Networks, members and peer nodes
- Hyperledger Fabric framework
- Accessor tokens for public chains (Bitcoin/Ethereum)
- Ordering service
- Governance/voting

## Common CLI
| Command | Description |
|---|---|
| `aws managedblockchain create-network` | Create a network |
| `aws managedblockchain create-member` | Add a member |
| `aws managedblockchain create-node` | Create a peer node |

## Pricing model
Per network membership and node-hour plus storage/requests (or per-request for public chains).

## Works well with
- VPC
- QLDB (ledger alternative)
- CloudWatch

## Use cases
- Consortium blockchains
- Public-chain node access
- Supply-chain/traceability apps

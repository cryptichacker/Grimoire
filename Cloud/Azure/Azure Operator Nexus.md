---
tags: [cloud, azure, industry]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Operator Nexus

## Up
- [[Azure]]

**Azure Operator Nexus** — A hybrid, carrier-grade platform for telecom operators to deploy and manage network functions on-prem and in Azure.

**Category:** Industry

## Key concepts
- Operator Nexus clusters (near/far edge)
- Carrier-grade Kubernetes/VM hosting
- Network function (VNF/CNF) deployment
- Managed via Azure/Arc
- Bare-metal automation

## Common CLI
| Command | Description |
|---|---|
| `az networkcloud cluster create` | Create a Nexus cluster |
| `az networkcloud baremetalmachine list` | List bare-metal machines |
| `az networkcloud virtualmachine create` | Create a VM |

## Pricing model
By infrastructure/consumption for the operator platform.

## Works well with
- Azure Arc
- Virtual Network
- Operator Insights

## Use cases
- Telco network functions
- 5G/edge deployments
- Carrier-grade hosting

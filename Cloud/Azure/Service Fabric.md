---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Service Fabric

## Up
- [[Azure]]

**Azure Service Fabric** — A distributed-systems platform for packaging, deploying and managing scalable microservices and containers.

**Category:** Compute

## Key concepts
- Clusters and nodes
- Stateless and stateful services
- Reliable Collections and actors
- Application and service manifests
- Rolling upgrades and health model
- Runs on Azure, on-prem or other clouds

## Common CLI
| Command | Description |
|---|---|
| `az sf cluster create` | Create a cluster |
| `az sf application create` | Deploy an application |
| `az sf cluster list` | List clusters |

## Pricing model
Free orchestration; pay for the underlying VM/scale-set compute.

## Works well with
- VM Scale Sets
- Key Vault
- Azure Monitor

## Use cases
- Stateful microservices
- Container orchestration
- Low-latency distributed apps

---
tags: [cloud, azure, compute]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Container Registry

## Up
- [[Azure]]

**Azure Container Registry (ACR)** — Managed private Docker/OCI registry for storing and building container images and artifacts.

**Category:** Compute

## Key concepts
- Registries and repositories
- Tags and manifests
- ACR Tasks (cloud builds)
- Geo-replication
- Content trust and scanning
- Tiers (Basic/Standard/Premium)

## Common CLI
| Command | Description |
|---|---|
| `az acr create` | Create a registry |
| `az acr login` | Authenticate Docker |
| `az acr build` | Build an image in the cloud |
| `az acr repository list` | List repositories |

## Pricing model
Per registry per day by tier plus storage and builds.

## Works well with
- AKS, Container Apps/Instances
- Microsoft Defender
- GitHub / Azure DevOps

## Use cases
- Store application images
- Cloud image builds
- Image scanning

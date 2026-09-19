---
tags: [cloud, azure, developer-tools]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Artifacts

## Up
- [[Azure]]

**Azure Artifacts** — A package-management service (part of Azure DevOps) for hosting and sharing NuGet, npm, Maven, Python and Universal packages.

**Category:** Developer Tools

## Key concepts
- Feeds (project/organization)
- Upstream sources
- Views (release channels)
- Package versions and retention
- Universal Packages

## Common CLI
| Command | Description |
|---|---|
| `az artifacts universal publish` | Publish a universal package |
| `az artifacts universal download` | Download a package |
| `az devops artifacts (feeds via REST/UI)` | Manage feeds |

## Pricing model
Free up to a storage limit, then per GB stored.

## Works well with
- Azure Pipelines
- npm/pip/NuGet/Maven clients
- Azure DevOps

## Use cases
- Private package hosting
- Dependency proxying
- Build artifact sharing

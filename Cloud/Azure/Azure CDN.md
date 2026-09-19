---
tags: [cloud, azure, networking]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure CDN

## Up
- [[Azure]]

**Azure Content Delivery Network** — Caches static content at global edge locations to deliver it with low latency and high availability.

**Category:** Networking

## Key concepts
- Profiles and endpoints
- Origin and origin groups
- Caching rules and purge
- Compression and rules engine
- Custom domains and TLS

## Common CLI
| Command | Description |
|---|---|
| `az cdn profile create` | Create a CDN profile |
| `az cdn endpoint create` | Create an endpoint |
| `az cdn endpoint purge` | Purge cached content |

## Pricing model
Per GB delivered and per request by pricing tier.

## Works well with
- Blob Storage (static sites)
- App Service
- Front Door (superset)

## Use cases
- Static content delivery
- Website acceleration
- Media distribution

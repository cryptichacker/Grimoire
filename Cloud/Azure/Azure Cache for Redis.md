---
tags: [cloud, azure, database]
type: service
source: Azure service reference (compiled)
last-verified: 2026-08-25
---

# Azure Cache for Redis

## Up
- [[Azure]]

**Azure Cache for Redis** — Fully managed, in-memory Redis-compatible cache for microsecond latency and high throughput.

**Category:** Database

## Key concepts
- Tiers (Basic/Standard/Premium/Enterprise)
- Clustering and sharding
- Replication and zone redundancy
- Persistence (Premium)
- Redis modules (Enterprise)
- Geo-replication

## Common CLI
| Command | Description |
|---|---|
| `az redis create` | Create a cache |
| `az redis list` | List caches |
| `az redis show` | Show details |
| `az redis update` | Update configuration |

## Pricing model
Per cache instance-hour by tier and size.

## Works well with
- App Service, AKS
- Azure SQL / Cosmos DB
- Functions

## Use cases
- Application caching
- Session stores
- Leaderboards/rate limiting

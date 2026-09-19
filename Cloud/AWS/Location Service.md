---
tags: [cloud, aws, front-end-web-mobile]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Location Service

## Up
- [[AWS]]

**Amazon Location Service** — Adds maps, places, routing, geofencing and asset tracking to applications using trusted data providers.

**Category:** Front-End Web & Mobile

## Key concepts
- Maps
- Place indexes (search/geocoding)
- Route calculators
- Geofence collections
- Trackers

## Common CLI
| Command | Description |
|---|---|
| `aws location create-map` | Create a map resource |
| `aws location search-place-index-for-text` | Geocode/search a place |
| `aws location calculate-route` | Calculate a route |

## Pricing model
Per request by resource type (maps, search, routes, tracking).

## Works well with
- Lambda
- EventBridge (geofence events)
- IoT / DynamoDB

## Use cases
- Maps in apps
- Delivery/asset tracking
- Geofencing alerts

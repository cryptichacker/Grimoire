---
tags: [cloud, aws, migration-transfer]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# Migration Hub

## Up
- [[AWS]]

**AWS Migration Hub** — Central place to discover on-prem resources, plan migrations and track progress across AWS migration tools.

**Category:** Migration & Transfer

## Key concepts
- Discovery and application grouping
- Migration tracking dashboard
- Strategy recommendations
- Refactor Spaces for incremental refactoring
- Home Region for migration data

## Common CLI
| Command | Description |
|---|---|
| `aws migrationhub list-migration-tasks` | List tracked migration tasks |
| `aws migrationhub create-progress-update-stream` | Create a progress stream |

## Pricing model
Free (pay for the underlying migration services used).

## Works well with
- Application Migration Service
- Database Migration Service
- Application Discovery Service

## Use cases
- Migration planning
- Portfolio discovery
- Progress tracking

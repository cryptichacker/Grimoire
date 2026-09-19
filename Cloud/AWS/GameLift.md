---
tags: [cloud, aws, game-tech]
type: service
source: AWS service reference (compiled)
last-verified: 2026-08-25
---

# GameLift

## Up
- [[AWS]]

**Amazon GameLift** — Dedicated server hosting and matchmaking for session-based multiplayer games at scale.

**Category:** Game Tech

## Key concepts
- Fleets (managed EC2/containers)
- Game sessions and player sessions
- FlexMatch matchmaking
- Queues and scaling
- Spot fleets and Anywhere (own hardware)

## Common CLI
| Command | Description |
|---|---|
| `aws gamelift create-fleet` | Create a hosting fleet |
| `aws gamelift create-game-session` | Start a game session |
| `aws gamelift describe-fleets` | List fleets |

## Pricing model
Per instance-hour of hosting plus matchmaking usage.

## Works well with
- EC2
- FlexMatch
- CloudWatch

## Use cases
- Multiplayer game servers
- Matchmaking
- Session-based games

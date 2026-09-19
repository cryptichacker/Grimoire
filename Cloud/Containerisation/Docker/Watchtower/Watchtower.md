---
tags: [cloud, containerisation, docker, watchtower]
type: cheatsheet
source: Watchtower (containrrr/watchtower) — compiled reference
last-verified: 2026-08-25
---

# Watchtower

## Up
- [[Docker]]

Watchtower watches your running containers and automatically pulls newer images and recreates the containers with the same options. Runs itself as a container with access to the Docker socket.

## Run
```bash
docker run -d --name watchtower \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower
```
Watch only specific containers by naming them as arguments:
```bash
docker run -d --name watchtower \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower app1 app2
```

## Options
Each flag has an equivalent environment variable.

| Flag | Env var | Description |
|---|---|---|
| `--interval N` | `WATCHTOWER_POLL_INTERVAL` | Poll every N seconds (default 86400 = 24h) |
| `--schedule "CRON"` | `WATCHTOWER_SCHEDULE` | 6-field cron schedule (mutually exclusive with interval) |
| `--cleanup` | `WATCHTOWER_CLEANUP` | Remove old images after updating |
| `--run-once` | `WATCHTOWER_RUN_ONCE` | Update once and exit (use `--rm`) |
| `--monitor-only` | `WATCHTOWER_MONITOR_ONLY` | Check/notify but don't update |
| `--no-restart` | `WATCHTOWER_NO_RESTART` | Update the image but don't restart the container |
| `--rolling-restart` | `WATCHTOWER_ROLLING_RESTART` | Restart one container at a time |
| `--label-enable` | `WATCHTOWER_LABEL_ENABLE` | Only update containers with the enable label set true |
| `--include-stopped` | `WATCHTOWER_INCLUDE_STOPPED` | Also consider stopped containers |
| `--scope NAME` | `WATCHTOWER_SCOPE` | Only manage containers with a matching scope label (multiple instances) |
| `--debug` / `--trace` | `WATCHTOWER_DEBUG` | Verbose logging |
| `--http-api-update` | `WATCHTOWER_HTTP_API_UPDATE` | Trigger updates via HTTP instead of polling |

## Labels (per container)
| Label | Effect |
|---|---|
| `com.centurylinklabs.watchtower.enable=false` | Exclude this container from updates |
| `com.centurylinklabs.watchtower.enable=true` | Include it when running with `--label-enable` |
| `com.centurylinklabs.watchtower.monitor-only=true` | Notify but never update this container |
| `com.centurylinklabs.watchtower.lifecycle.pre-update` | Command to run inside the container before updating |
| `com.centurylinklabs.watchtower.lifecycle.post-update` | Command to run after updating |

## Notifications
Set via shoutrrr URLs — e.g. Slack, email, Discord, Telegram, Gotify:
```bash
docker run -d --name watchtower \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e WATCHTOWER_NOTIFICATIONS=shoutrrr \
  -e WATCHTOWER_NOTIFICATION_URL="slack://token@channel" \
  containrrr/watchtower --cleanup --schedule "0 0 4 * * *"
```

## Common recipes
| Goal | Command |
|---|---|
| Update once, then exit | `docker run --rm -v /var/run/docker.sock:/var/run/docker.sock containrrr/watchtower --run-once` |
| Poll every 5 min + clean old images | `... containrrr/watchtower --interval 300 --cleanup` |
| Only opt-in containers | `... containrrr/watchtower --label-enable` |
| Dry-run / alert only | `... containrrr/watchtower --monitor-only` |

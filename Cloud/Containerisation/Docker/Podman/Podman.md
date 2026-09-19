---
tags: [cloud, containerisation, docker, podman]
type: cheatsheet
source: compiled reference (Podman)
last-verified: 2026-09-16
---

# Podman

## Up
- [[Docker]]

Podman is a **daemonless, rootless-friendly** container engine that's a near drop-in for Docker (same OCI images, same CLI verbs). No central root daemon → better security and clean systemd integration. Also natively understands **pods** (Kubernetes-style groups of containers).

---

## Docker → Podman

```bash
alias docker=podman            # most commands just work
podman run -d -p 8080:80 nginx
podman ps ; podman images ; podman pull alpine ; podman build -t app .
podman exec -it <c> sh ; podman logs -f <c> ; podman stop <c>
```

Key differences vs Docker:
- **No daemon** — each command is its own process (fork/exec via conmon). Nothing to run/attack as root in the background.
- **Rootless by default** — containers run as your user via user namespaces; UID 0 in-container maps to your unprivileged UID on the host.
- **Pods** — group containers sharing a network namespace, like a K8s pod.
- Docker socket compatibility via `podman system service` (emulates the Docker API for tools that expect it).

---

## Pods & Kubernetes bridge

```bash
podman pod create --name web -p 8080:80
podman run -d --pod web nginx
podman run -d --pod web redis
podman pod ps ; podman pod stop web

# generate/consume Kubernetes YAML
podman kube generate web -f web.yaml        # pod → K8s manifest
podman kube play web.yaml                    # run a K8s manifest locally
```

---

## systemd integration (Quadlet)

```bash
# modern way: a .container unit file managed by systemd (Quadlet)
# /etc/containers/systemd/app.container  →  systemctl start app.service
podman generate systemd --new --name app > ~/.config/systemd/user/app.service   # legacy generator
systemctl --user enable --now app
```
Great for running containers as managed services on a host (no daemon, auto-restart, boot start).

---

## Compose & Building
- **podman-compose** or Docker Compose pointed at the Podman socket runs `compose.yaml` files.
- Builds use **Buildah** under the hood (`podman build`), or call `buildah` directly for scriptable, daemonless image builds — see [[BuildKit & buildx]] for the Docker-side equivalent.

```bash
podman compose up -d          # (with the compose provider installed)
podman build -t app:1.0 .
podman push app:1.0 registry.example.com/app:1.0
```

---

## Podman vs Docker

| | Podman | Docker |
|---|---|---|
| Daemon | **None** (daemonless) | Central `dockerd` (root) |
| Rootless | Default, first-class | Supported, extra setup |
| Pods | Native | No (compose/K8s only) |
| Build | Buildah | BuildKit |
| systemd | Quadlet / generate | Weaker |
| CLI | Docker-compatible | — |

---

## Tips
- **Rootless is the big win** — a container breakout lands as your unprivileged user, not host root (mitigates escape risk from [[Container Attack Concepts]]).
- Rootless has caveats: ports <1024 need config, some storage/network features differ; use `podman unshare` for UID-mapped file ops.
- Default on RHEL/Fedora; install on Ubuntu/macOS (`podman machine` runs a Linux VM like Docker Desktop).
- `podman kube generate/play` is a handy local↔Kubernetes workflow. See [[Kubernetes]].

---
tags: [cloud, containerisation, docker, buildkit, buildx]
type: cheatsheet
source: compiled reference (BuildKit / buildx)
last-verified: 2026-09-16
---

# BuildKit & buildx

## Up
- [[Docker]]

**BuildKit** is Docker's modern build engine (default since Docker 23): parallel stages, smart caching, build secrets/SSH, and **multi-platform** images. **buildx** is the CLI front-end that drives BuildKit. Pairs with [[Dockerfile Best Practices]].

---

## Basics

```bash
docker buildx version
docker build -t app:1.0 .                  # already uses BuildKit by default
DOCKER_BUILDKIT=1 docker build .           # force on older Docker
docker buildx build -t app:1.0 .           # explicit buildx
docker buildx build --progress=plain .     # full build logs
```

---

## Dockerfile features BuildKit unlocks

```dockerfile
# syntax=docker/dockerfile:1        # opt into latest frontend features (put at top)

# cache mounts — persist package caches across builds (huge speedup)
RUN --mount=type=cache,target=/root/.cache/pip pip install -r requirements.txt
RUN --mount=type=cache,target=/var/cache/apt apt-get update && apt-get install -y curl

# build secrets — never baked into a layer
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc npm ci
# SSH agent forwarding for private repos
RUN --mount=type=ssh git clone git@github.com:org/private.git

# bind mount source without COPY
RUN --mount=type=bind,source=.,target=/src make -C /src
```
```bash
docker buildx build --secret id=npmrc,src=$HOME/.npmrc --ssh default -t app .
```

---

## Multi-platform images

```bash
# one manifest serving amd64 + arm64
docker buildx create --name multi --use            # a builder that can emulate
docker buildx build --platform linux/amd64,linux/arm64 -t org/app:1.0 --push .
```
`--push` is required for multi-arch (the multi-platform manifest can't load into the local daemon). QEMU provides cross-arch emulation.

---

## Cache export/import (fast CI)

```bash
# registry-backed cache — reuse layers across CI runners
docker buildx build \
  --cache-to   type=registry,ref=org/app:buildcache,mode=max \
  --cache-from type=registry,ref=org/app:buildcache \
  -t org/app:1.0 --push .
# other backends: type=gha (GitHub Actions), type=local,dest=./cache, type=inline
```
`mode=max` caches all stages (incl. intermediate) — best hit rate.

---

## Output types & Bake

```bash
docker buildx build -o type=local,dest=./out .     # export filesystem, not an image
docker buildx build -o type=tar,dest=app.tar .

# Bake: build many targets from a file (docker-bake.hcl / compose)
docker buildx bake                                  # build all targets
docker buildx bake app --set app.platform=linux/arm64
```

---

## Tips
- Add **`# syntax=docker/dockerfile:1`** to opt into the newest features without upgrading Docker.
- Use **`--mount=type=cache`** for package managers (pip/npm/apt/go) — the single biggest build-time win.
- Keep credentials out of layers with **`--mount=type=secret`/`type=ssh`** (never `COPY` a token then delete it — it stays in history; see [[Container Attack Concepts]]).
- Registry/GHA **cache export** makes CI builds reuse layers across ephemeral runners.
- Order Dockerfile steps by change frequency for cache hits — details in [[Dockerfile Best Practices]].

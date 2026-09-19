---
tags: [cloud, containerisation, docker, dockerfile, best-practices]
type: cheatsheet
source: compiled reference (Dockerfile best practices)
last-verified: 2026-09-16
---

# Dockerfile Best Practices

## Up
- [[Docker]]

How to write Dockerfiles that build fast, ship small, and stay secure. Builds on [[BuildKit & buildx]]; the security angle ties into [[Container Attack Concepts]] and image scanning ([[Trivy]]).

---

## Layer caching & order
Each instruction is a cached layer; a change busts that layer **and all below it**. Order from **least- to most-frequently changed**:

```dockerfile
# ✅ deps first (change rarely) → then source (changes often)
COPY package.json package-lock.json ./
RUN npm ci                     # cached until deps change
COPY . .                       # only this re-runs on code edits
RUN npm run build
```
```dockerfile
# ❌ this rebuilds deps on every code change
COPY . .
RUN npm ci
```

- Combine related `RUN`s with `&&` and **clean up in the same layer** (`apt-get ... && rm -rf /var/lib/apt/lists/*`), or files stay in history.
- Add a **`.dockerignore`** (`.git`, `node_modules`, secrets, build artifacts) — shrinks context, speeds builds, avoids leaking files.

---

## Multi-stage builds (small, clean images)

```dockerfile
# build stage — full toolchain
FROM golang:1.23 AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /app ./cmd/server

# runtime stage — only the binary
FROM gcr.io/distroless/static:nonroot
COPY --from=build /app /app
USER nonroot:nonroot
ENTRYPOINT ["/app"]
```
Ship only what runs — no compilers, package managers, or source in the final image.

---

## Small & pinned base images
- Prefer **slim / alpine / distroless** over full OS images. Distroless/scratch = tiny attack surface (no shell/package manager).
- **Pin** versions (`python:3.12-slim`, ideally by digest `@sha256:…`) — never rely on `latest` for reproducibility.
- Fewer packages = fewer CVEs (scan with [[Trivy]]).

---

## Security
```dockerfile
RUN adduser -D -u 10001 app
USER app                       # never run as root
```
- **Run as non-root** (`USER`); set `readOnlyRootFilesystem` at runtime.
- **No secrets in the image** — not via `ENV`, `ARG`, or `COPY`-then-delete (it persists in a layer). Use BuildKit `--mount=type=secret` or runtime env/secrets.
- Drop capabilities, avoid `--privileged`, don't expose the Docker socket. See [[Container Attack Concepts]].
- Use **`COPY`** over `ADD` (ADD auto-extracts/fetches URLs — surprising). Prefer `exec` form `CMD ["app"]` (signal handling), and set `WORKDIR` (not `cd`).

---

## Correctness & signals
```dockerfile
FROM node:20-slim
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
EXPOSE 3000
HEALTHCHECK --interval=30s CMD wget -qO- http://localhost:3000/health || exit 1
USER node
CMD ["node", "server.js"]      # exec form → PID 1 gets signals (clean shutdown)
```
- Use exec-form `CMD`/`ENTRYPOINT` so the process is PID 1 and receives `SIGTERM` (or add `tini`/`--init`).
- One concern per container; log to **stdout/stderr**.

---

## Checklist
- [ ] `.dockerignore` present
- [ ] Deps copied before source (cache-friendly)
- [ ] Multi-stage → minimal runtime image
- [ ] Pinned, slim/distroless base
- [ ] Non-root `USER`, no secrets in layers
- [ ] exec-form CMD, HEALTHCHECK, EXPOSE
- [ ] `hadolint Dockerfile` (lint) + [[Trivy]] image scan in CI

---

## Tips
- Lint with **hadolint**; scan built images with **[[Trivy]]** and gate CI on Critical/High.
- Rebuild regularly to pick up base-image security patches (pin by digest, bump deliberately).
- Combine with [[BuildKit & buildx]] cache mounts + multi-platform for fast, portable builds.

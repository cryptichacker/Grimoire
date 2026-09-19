---
tags: [cloud, containerisation, docker, harbor, registry]
type: cheatsheet
source: compiled reference (Harbor)
last-verified: 2026-09-16
---

# Harbor

## Up
- [[Docker]]

Harbor is an open-source (CNCF graduated) **private container registry** with the enterprise features Docker Hub/Distribution lack: RBAC + projects, vulnerability scanning ([[Trivy]] built in), image signing, replication, retention, and quotas. Stores OCI images and Helm/OCI artifacts.

---

## Core Concepts

| Concept | Meaning |
|---|---|
| **Project** | Namespace for repositories (public or private); the unit of RBAC/quota/policy |
| **Repository** | An image name within a project (`project/app`) |
| **Robot account** | Non-human token for CI/CD to push/pull |
| **Replication** | Sync images to/from Docker Hub, ECR/GCR/ACR, or other Harbors |
| **Retention / Immutability / Quota** | Auto-prune old tags; lock tags; cap project storage |
| **Scanner** | Trivy (default) — scan on push, block by severity |
| **Signing** | Cosign/Notation signatures + policy enforcement |

---

## Use it

```bash
# login & push (behind TLS)
docker login harbor.example.com
docker tag app:1.0 harbor.example.com/team/app:1.0
docker push harbor.example.com/team/app:1.0
docker pull harbor.example.com/team/app:1.0

# CI uses a robot account
docker login harbor.example.com -u 'robot$team+ci' -p "$ROBOT_TOKEN"
```

Kubernetes pulls via an image-pull secret:
```bash
kubectl create secret docker-registry harbor-cred \
  --docker-server=harbor.example.com --docker-username='robot$team+ci' --docker-password="$TOKEN"
# reference it in the pod spec's imagePullSecrets
```

---

## Security features
- **Vulnerability scanning:** Trivy scans images on push; view CVEs per tag; **prevent pull** of images above a severity threshold (project setting).
- **Content trust / signing:** require **cosign** signatures; a deployment policy (with [[Kyverno & OPA Gatekeeper]] `verifyImages`) can enforce "only signed, scanned images from Harbor."
- **RBAC:** project roles (guest/developer/maintainer/admin), OIDC/LDAP SSO, robot accounts scoped per project.
- **Immutable tags** stop overwrite of released versions; **CVE allowlists** manage accepted risk.

---

## Replication & retention
```text
Replication rule: Harbor ⇄ Docker Hub / ECR / another Harbor
  - push-based or pull-based, filtered by repo/tag, scheduled or on-event
Retention policy: keep last N tags / tags newer than X days per repo
Proxy cache project: pull-through cache of an upstream registry (dockerhub) → faster, rate-limit-proof
```

---

## Install (quick)
```bash
# Helm onto Kubernetes (production)
helm repo add harbor https://helm.goharbor.io
helm install harbor harbor/harbor -n harbor --create-namespace \
  --set expose.type=ingress --set externalURL=https://harbor.example.com
# or the offline/online installer (docker-compose) for a VM
```

---

## Tips
- Front Harbor with TLS ([[cert-manager]]/[[Ingress Controllers]]) — registries must be HTTPS (or configured insecure, which you shouldn't).
- Turn on **scan-on-push + block-by-severity** and **signature enforcement** to make the registry a supply-chain gate (pairs with [[Container & K8s Security]]).
- Use **robot accounts** (not personal creds) in CI; scope them per project, rotate tokens.
- A **proxy-cache project** for Docker Hub avoids rate limits and speeds pulls.
- Set **retention + quotas** early — registries balloon fast.

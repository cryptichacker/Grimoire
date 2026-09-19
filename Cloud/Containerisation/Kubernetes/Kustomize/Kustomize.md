---
tags: [cloud, containerisation, kubernetes, kustomize]
type: cheatsheet
source: compiled reference (Kustomize)
last-verified: 2026-09-16
---

# Kustomize

## Up
- [[Kubernetes]]

Kustomize is Kubernetes' built-in, **template-free** configuration manager: you keep plain YAML and layer **overlays** (patches) on top of a **base** for each environment. No templating language — it's declarative merging. Built into `kubectl` (`-k`). Compare with [[Helm]] (templated packages).

---

## The Model: base + overlays

```text
app/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   └── service.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml     # patches base for dev
    └── prod/
        └── kustomization.yaml     # patches base for prod
```

```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
commonLabels: { app: web }
```

```yaml
# overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: prod
resources:
  - ../../base
namePrefix: prod-
replicas:
  - { name: web, count: 5 }
images:
  - { name: web, newTag: "1.25.0" }
patches:
  - path: resources-patch.yaml           # strategic-merge or JSON6902
    target: { kind: Deployment, name: web }
configMapGenerator:
  - name: app-config
    literals: [LOG_LEVEL=info]
```

---

## Build & Apply

```bash
kubectl kustomize overlays/prod            # render to stdout (dry preview)
kubectl apply -k overlays/prod             # build + apply
kustomize build overlays/prod | kubectl apply -f -   # standalone CLI
kubectl delete -k overlays/prod
```

---

## Key Features

| Field | Does |
|---|---|
| `resources` | Files/dirs/bases to include |
| `namespace` / `namePrefix` / `nameSuffix` | Scope & rename all objects |
| `commonLabels` / `commonAnnotations` | Stamp on everything |
| `images` | Override image name/tag/digest |
| `replicas` | Override replica counts |
| `patches` | Strategic-merge or **JSON6902** patches, targeted by GVK/name |
| `configMapGenerator` / `secretGenerator` | Generate CM/Secrets (+ content hash suffix → rolling updates) |
| `components` | Reusable mixin overlays (opt-in features) |
| `vars` / `replacements` | Cross-resource value injection |

**Generator hash suffix:** ConfigMap/Secret names get a content hash → changing config auto-triggers a rolling restart of consumers.

---

## Patches

```yaml
# strategic merge (partial object)
patches:
  - patch: |-
      apiVersion: apps/v1
      kind: Deployment
      metadata: { name: web }
      spec: { template: { spec: { containers: [{ name: web, resources: { limits: { memory: 512Mi }}}]}}}
    target: { kind: Deployment, name: web }

# JSON 6902 (precise op)
  - target: { kind: Deployment, name: web }
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/image
        value: web:2.0
```

---

## Kustomize vs Helm

| | Kustomize | [[Helm]] |
|---|---|---|
| Approach | Overlay/patch plain YAML | Go-templated packages |
| Templating | None (declarative merge) | Yes (values, logic) |
| Packaging/versioning | No package concept | Charts, repos, releases |
| Built into kubectl | ✅ (`-k`) | ❌ (separate) |
| Best for | Env variants of your own manifests | Distributable, parameterised apps |

Common combo: **Helm to install third-party charts + Kustomize to post-render/patch** (`helm template ... | kustomize`), or Kustomize for your own apps, Helm for vendors. Both are first-class in [[ArgoCD]]/GitOps.

---

## Tips
- Keep the **base environment-agnostic**; put all env differences in overlays.
- Use `configMapGenerator` (not hand-written CMs) so config changes roll pods automatically.
- `kubectl kustomize` (or `kustomize build`) to **preview** before applying — no surprises.
- Prefer **strategic-merge** patches for readability; **JSON6902** for precise list edits.
- Pin the standalone `kustomize` version in CI; the kubectl-embedded one can lag features.

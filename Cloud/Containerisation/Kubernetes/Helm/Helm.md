---
tags: [cloud, containerisation, kubernetes, helm]
type: cheatsheet
source: Helm (helm.sh) — compiled reference
last-verified: 2026-08-27
---

# Helm

## Up
- [[Kubernetes]]

Helm is the package manager for Kubernetes. It bundles a set of manifests into a versioned, parameterisable unit called a **chart**, renders them with values you supply, and installs the result as a tracked **release** you can upgrade, roll back, and uninstall as one object.

---

## Core Concepts

| Term | Meaning |
|---|---|
| **Chart** | A package of pre-configured Kubernetes resources (templates + default values + metadata). |
| **Repository** | A place charts are stored and shared (an HTTP index, or an OCI registry). |
| **Release** | An installed instance of a chart in a cluster. Each `helm install` creates one, with its own name and revision history. |
| **Values** | The configuration passed into a chart's templates. Defaults live in `values.yaml`; you override them at install/upgrade time. |
| **Revision** | An incrementing version number for a release; every install/upgrade/rollback bumps it, enabling rollback. |
| **Chart dependency (subchart)** | Another chart your chart depends on, declared in `Chart.yaml` and pulled into `charts/`. |

Helm v3 (current) is **Tiller-less** — it talks to the Kubernetes API directly using your kubeconfig, so release state is stored as Secrets in the release's namespace. Anything you can `kubectl` you can `helm`.

---

## Install Helm

| Method | Command |
|---|---|
| Homebrew (macOS/Linux) | `brew install helm` |
| Script | `curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 \| bash` |
| Chocolatey (Windows) | `choco install kubernetes-helm` |
| Scoop (Windows) | `scoop install helm` |
| Apt (Debian/Ubuntu) | `curl https://baltocdn.com/helm/signing.asc \| sudo apt-key add -` then add the repo and `sudo apt install helm` |

```bash
helm version              # confirm install + client version
helm env                  # show Helm's environment (paths, registry config)
```

---

## Repository Management

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami   # add a repo
helm repo add stable https://charts.helm.sh/stable
helm repo list                                             # list configured repos
helm repo update                                           # refresh local cache of all repos
helm repo remove bitnami                                   # remove a repo
```

### Finding charts

```bash
helm search repo nginx              # search added repos
helm search repo bitnami/ --versions  # list all versions of a repo's charts
helm search hub wordpress           # search the Artifact Hub (public charts)
```

---

## Installing & Managing Releases

```bash
# Install: helm install <release-name> <chart>
helm install my-nginx bitnami/nginx
helm install my-nginx bitnami/nginx -n web --create-namespace

# Generate a release name automatically
helm install bitnami/nginx --generate-name

# Install a specific chart version
helm install my-nginx bitnami/nginx --version 15.4.4

# Install from a local chart directory or packaged .tgz
helm install my-app ./mychart
helm install my-app ./mychart-0.1.0.tgz
```

### Passing values

```bash
helm install my-app ./mychart -f values-prod.yaml          # a values file
helm install my-app ./mychart -f base.yaml -f overrides.yaml  # merged, last wins
helm install my-app ./mychart --set image.tag=1.25 --set replicaCount=3
helm install my-app ./mychart --set-string version=1.0     # force string
helm install my-app ./mychart --set-file config=./app.conf # value from a file
```

### Inspect / list / status

```bash
helm list                       # releases in current namespace
helm list -A                    # all namespaces
helm list -a                    # include failed/pending/uninstalling
helm status my-nginx            # release status + notes
helm get values my-nginx        # user-supplied values
helm get values my-nginx -a     # computed values (defaults + overrides)
helm get manifest my-nginx      # rendered manifests actually applied
helm get notes my-nginx         # the NOTES.txt output
helm get all my-nginx           # everything about the release
```

### Upgrade

```bash
helm upgrade my-nginx bitnami/nginx --set replicaCount=5
helm upgrade my-nginx bitnami/nginx -f values-prod.yaml

# Install if not present, else upgrade (idempotent — great for CI/CD)
helm upgrade --install my-nginx bitnami/nginx -n web --create-namespace

# Reuse previously-set values and layer new ones on top
helm upgrade my-nginx bitnami/nginx --reuse-values --set image.tag=1.26

# Wait until resources are Ready (with a timeout), roll back automatically on failure
helm upgrade --install my-nginx bitnami/nginx --wait --timeout 5m --atomic
```

### Rollback & history

```bash
helm history my-nginx           # list all revisions
helm rollback my-nginx          # roll back to the previous revision
helm rollback my-nginx 3        # roll back to revision 3
helm rollback my-nginx 3 --wait
```

### Uninstall

```bash
helm uninstall my-nginx
helm uninstall my-nginx -n web
helm uninstall my-nginx --keep-history   # retain revision history (status: uninstalled)
```

---

## Dry-Run, Diff & Templating

```bash
# Render templates locally without touching the cluster
helm template my-app ./mychart
helm template my-app ./mychart -f values-prod.yaml --set replicaCount=2

# Simulate an install/upgrade against the API server (validates + shows manifests)
helm install my-app ./mychart --dry-run --debug
helm upgrade my-app ./mychart --dry-run

# See what an upgrade WOULD change (requires the helm-diff plugin)
helm plugin install https://github.com/databus23/helm-diff
helm diff upgrade my-nginx bitnami/nginx -f values-prod.yaml
```

`helm template` renders offline (no cluster contact) — useful for GitOps and for piping into `kubectl apply`. `--dry-run` contacts the API server, so it can catch schema/validation errors that pure templating cannot.

---

## Authoring a Chart

```bash
helm create mychart      # scaffold a new chart with sensible defaults
helm lint ./mychart      # check the chart for issues/best-practice violations
helm package ./mychart   # build a versioned .tgz for distribution
helm show chart ./mychart      # print Chart.yaml
helm show values ./mychart     # print default values.yaml
helm show readme bitnami/nginx # print a chart's README
```

### Chart directory layout

```text
mychart/
├── Chart.yaml          # chart metadata: name, version, appVersion, dependencies
├── values.yaml         # default configuration values
├── values.schema.json  # optional JSON schema validating values
├── charts/             # subcharts (dependencies) vendored here
├── crds/               # CustomResourceDefinitions (installed before templates)
├── templates/          # the manifest templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── _helpers.tpl    # reusable named template partials
│   ├── NOTES.txt       # post-install usage message
│   └── tests/          # `helm test` hooks
│       └── test-connection.yaml
└── .helmignore         # files to exclude when packaging
```

### Chart.yaml essentials

```yaml
apiVersion: v2               # v2 = Helm 3
name: mychart
description: A Helm chart for my app
type: application            # or "library"
version: 0.1.0               # the CHART version (bump on any chart change)
appVersion: "1.25.0"         # the version of the app being deployed
dependencies:
  - name: postgresql
    version: "13.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled   # toggle the subchart via values
```

---

## Templating Language (Go templates + Sprig)

Templates in `templates/` are rendered with Go's `text/template` plus the [Sprig](https://masterminds.github.io/sprig/) function library and Helm's own additions.

### Built-in objects

| Object | Contains |
|---|---|
| `.Values` | Values from `values.yaml` and `--set`/`-f` overrides |
| `.Release` | `.Name`, `.Namespace`, `.Revision`, `.IsInstall`, `.IsUpgrade` |
| `.Chart` | Contents of `Chart.yaml` (e.g. `.Chart.Name`, `.Chart.AppVersion`) |
| `.Capabilities` | Cluster/API capabilities (e.g. `.Capabilities.KubeVersion`) |
| `.Files` | Access to non-template files in the chart (e.g. `.Files.Get`) |
| `.Template` | `.Name` and `.BasePath` of the current template |

### Common patterns

```yaml
# values interpolation
metadata:
  name: {{ .Release.Name }}-web
spec:
  replicas: {{ .Values.replicaCount }}
  image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"

# defaults, quoting, pipelines
image: {{ .Values.image.tag | default .Chart.AppVersion | quote }}

# conditionals
{{- if .Values.ingress.enabled }}
# ... ingress manifest ...
{{- end }}

# loops
{{- range .Values.env }}
- name: {{ .name }}
  value: {{ .value | quote }}
{{- end }}

# a named template (define in _helpers.tpl, use anywhere)
{{- define "mychart.labels" -}}
app.kubernetes.io/name: {{ .Chart.Name }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end -}}
labels:
  {{- include "mychart.labels" . | nindent 4 }}

# blocks: with (scope), toYaml (render a map), required (fail fast)
resources:
  {{- toYaml .Values.resources | nindent 4 }}
password: {{ required "a password is required!" .Values.password }}
```

- `{{-` / `-}}` trim whitespace on the left/right to keep YAML valid.
- `indent n` / `nindent n` add leading spaces (nindent also adds a newline first).
- `include` is preferred over `template` because its output can be piped.
- `tpl` renders a string from values as a template at runtime.

---

## Hooks

Annotate a resource to run it at a lifecycle point (e.g. a migration Job before upgrade):

```yaml
metadata:
  annotations:
    "helm.sh/hook": pre-upgrade,pre-install
    "helm.sh/hook-weight": "-5"                    # lower runs first
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
```

| Hook | Fires |
|---|---|
| `pre-install` / `post-install` | Before/after templates are installed |
| `pre-upgrade` / `post-upgrade` | Before/after an upgrade |
| `pre-rollback` / `post-rollback` | Before/after a rollback |
| `pre-delete` / `post-delete` | Before/after an uninstall |
| `test` | Run by `helm test` |

---

## Testing a Release

Define test pods under `templates/tests/` (annotated `helm.sh/hook: test`), then:

```bash
helm test my-nginx           # run the chart's tests against the live release
helm test my-nginx --logs    # show pod logs from the tests
```

---

## OCI Registries

Helm 3.8+ treats OCI registries (ECR, GHCR, Docker Hub, ACR, GAR) as first-class chart stores:

```bash
helm registry login registry-1.docker.io -u USER          # authenticate
helm push mychart-0.1.0.tgz oci://registry-1.docker.io/myorg   # push a packaged chart
helm install my-app oci://registry-1.docker.io/myorg/mychart --version 0.1.0
helm pull oci://registry-1.docker.io/myorg/mychart --version 0.1.0
helm registry logout registry-1.docker.io
```

---

## Dependencies (Subcharts)

Declare dependencies in `Chart.yaml`, then:

```bash
helm dependency update ./mychart    # fetch deps into charts/ and write Chart.lock
helm dependency build ./mychart     # rebuild charts/ from an existing Chart.lock
helm dependency list ./mychart      # show declared deps and their status
```

Override a subchart's values from the parent by nesting under the subchart name:

```yaml
# parent values.yaml
postgresql:
  enabled: true
  auth:
    database: myapp
```

---

## Plugins

```bash
helm plugin install https://github.com/databus23/helm-diff   # preview upgrades
helm plugin install https://github.com/jkroepke/helm-secrets  # encrypted values (SOPS)
helm plugin list
helm plugin update diff
helm plugin uninstall diff
```

Popular plugins: **helm-diff** (upgrade previews), **helm-secrets** (SOPS/age-encrypted values), **helm-git** (charts from git), **helm-unittest** (unit tests for charts).

---

## Global Flags (handy on most commands)

| Flag | Effect |
|---|---|
| `-n, --namespace` | Target namespace |
| `--create-namespace` | Create the namespace if it doesn't exist (install/upgrade) |
| `--kube-context` | Use a specific kubeconfig context |
| `--dry-run` | Simulate without applying |
| `--debug` | Verbose output (rendered manifests, errors) |
| `-o json\|yaml` | Machine-readable output (list, status, get) |
| `--wait` | Block until resources are Ready |
| `--timeout` | Time to wait (default 5m0s) |
| `--atomic` | Roll back automatically if install/upgrade fails |

---

## Typical Workflows

**Deploy an off-the-shelf app**

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install redis bitnami/redis -n data --create-namespace \
  --set architecture=standalone --set auth.password=$REDIS_PW
```

**Idempotent CI/CD deploy of your own chart**

```bash
helm upgrade --install my-app ./charts/my-app \
  -n prod --create-namespace \
  -f charts/my-app/values-prod.yaml \
  --set image.tag=$GIT_SHA \
  --wait --atomic --timeout 5m
```

**Safely change a running release**

```bash
helm diff upgrade my-app ./charts/my-app -f values-prod.yaml   # preview
helm upgrade my-app ./charts/my-app -f values-prod.yaml        # apply
helm history my-app                                            # check revision
helm rollback my-app                                           # undo if needed
```

---

## Quick Troubleshooting

| Symptom | Try |
|---|---|
| Upgrade stuck / release "pending-upgrade" | `helm rollback <name>`; if wedged, `helm history` then rollback to last good revision |
| Want to know what actually got applied | `helm get manifest <name>` |
| Values not taking effect | check precedence with `helm get values <name> -a`; remember `--set` beats `-f`, later `-f` beats earlier |
| Template errors | `helm template ... --debug` or `helm install --dry-run --debug` |
| "cannot re-use a name that is still in use" | release exists — use `helm upgrade --install` |
| Failed install left resources behind | `helm uninstall <name>`; use `--atomic` next time |

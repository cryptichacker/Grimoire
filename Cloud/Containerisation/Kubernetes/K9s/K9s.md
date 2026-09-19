---
tags: [cloud, containerisation, kubernetes, k9s]
type: cheatsheet
source: K9s (derailed/k9s) — compiled reference
last-verified: 2026-08-25
---

# K9s

## Up
- [[Kubernetes]]

A terminal UI to observe and manage Kubernetes clusters — navigate resources, stream logs, shell into pods, edit and delete objects, all with vim-like keys and live updates.

## Install
| Method | Command |
|---|---|
| Homebrew | `brew install k9s` |
| Webi | `curl -sS https://webi.sh/k9s \| sh` |
| Scoop (Windows) | `scoop install k9s` |
| Go | `go install github.com/derailed/k9s@latest` |

## Launch
| Command | Description |
|---|---|
| `k9s` | Start against the current kube-context |
| `k9s -n <namespace>` | Start scoped to a namespace |
| `k9s -A` | Start across all namespaces |
| `k9s --context <ctx>` | Use a specific context |
| `k9s -c <view>` | Start on a view, e.g. `k9s -c pod` |
| `k9s --readonly` | Disable all mutating commands |
| `k9s info` | Show config/log/screendump paths |

## Command mode (`:`)
Type `:` then a resource alias to jump to that view.

| Command | View |
|---|---|
| `:pod` (`:po`) | Pods |
| `:deploy` (`:dp`) | Deployments |
| `:svc` | Services |
| `:ns` | Namespaces |
| `:node` (`:no`) | Nodes |
| `:cm` / `:secret` | ConfigMaps / Secrets |
| `:ing` / `:sts` / `:ds` | Ingress / StatefulSets / DaemonSets |
| `:job` / `:cj` | Jobs / CronJobs |
| `:ctx` | Switch context (`:ctx <name>`) |
| `:xray <type>` | Xray dependency tree (e.g. `:xray deploy`) |
| `:pulse` (`:pu`) | Cluster pulse dashboard |
| `:popeye` | Cluster sanitizer report |
| `:q` | Quit |

## Keys
| Key | Action |
|---|---|
| `:` | Command mode (jump to a resource) |
| `/` | Filter the current list |
| `?` | Help / full keybinding list |
| `esc` | Back / clear filter |
| `enter` | Drill into the selected resource |
| `d` | Describe |
| `y` | View YAML |
| `e` | Edit (opens `$EDITOR`) |
| `l` | View logs · `p` previous logs |
| `s` | Shell into the pod/container |
| `ctrl-d` | Delete (with confirmation) |
| `ctrl-k` | Kill (no grace period) |
| `0` | All namespaces · `1`–`9` favourite namespaces |
| `Shift-<col>` | Sort by that column |

## Config
Config, skins and aliases live under `~/.config/k9s/` (`config.yaml`, `skins/`, `aliases.yaml`). Run `k9s info` to see the exact paths. Skins let you recolour the UI; `aliases.yaml` maps custom shortcuts to resources.

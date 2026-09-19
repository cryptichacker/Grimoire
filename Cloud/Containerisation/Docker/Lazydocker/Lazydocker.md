---
tags: [cloud, containerisation, docker, lazydocker]
type: cheatsheet
source: lazydocker (jesseduffield/lazydocker) — compiled reference
last-verified: 2026-08-25
---

# Lazydocker

## Up
- [[Docker]]

A terminal UI for Docker and Docker Compose — browse containers, images, volumes and compose services, and view logs/stats/config without memorising `docker` flags.

## Install
| Method | Command |
|---|---|
| Homebrew | `brew install lazydocker` |
| Go | `go install github.com/jesseduffield/lazydocker@latest` |
| Linux script | `curl https://raw.githubusercontent.com/jesseduffield/lazydocker/master/scripts/install_update_linux.sh \| bash` |
| Docker | `docker run --rm -it -v /var/run/docker.sock:/var/run/docker.sock -v ~/.config/lazydocker:/.config/jesseduffield/lazydocker lazyteam/lazydocker` |

## Launch
| Command | Description |
|---|---|
| `lazydocker` | Start the UI (uses the current Docker context) |
| `lazydocker --help` | Show CLI flags |
| `lazydocker -f docker-compose.yml` | Point at a specific compose file |

## Layout
Left column holds the panels — **Project**, **Services** (compose), **Containers**, **Images**, **Volumes**, and **Custom Commands**. The large **main panel** on the right shows the selected item's logs, stats (CPU/memory graphs), config, or top output.

## Keys
| Key | Action |
|---|---|
| `[` / `]` | Previous / next panel (tab) |
| `↑`/`↓` or `j`/`k` | Move within a panel |
| `enter` | Focus the main panel (scroll logs/output) |
| `/` | Filter the current list |
| `+` / `_` | Enlarge / shrink the main panel |
| `x` | Open the **action menu** for the selected item |
| `d` | Remove / prune (opens a confirm menu) |
| `esc` / `q` | Back / quit |

> Select any container/image/volume and press **`x`** to see all available actions (restart, stop, pause, remove, attach, exec shell, view logs, bulk commands). Press `x` on the Images/Volumes panels for prune options.

## Config
User config lives at `~/.config/lazydocker/config.yml` — customise keybindings, log settings, graph stats, and add **custom commands** per container/service.

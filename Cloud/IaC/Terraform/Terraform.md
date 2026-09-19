---
tags: [cloud, iac, terraform]
type: cheatsheet
source: Terraform CLI reference (compiled)
last-verified: 2026-08-25
---

# Terraform

## Up
- [[IaC]]

Command cheatsheet for the Terraform CLI — the core provision workflow, state, workspaces, modules, and troubleshooting.

## Core workflow
| Command | Description |
|---|---|
| `terraform init` | Initialize a working directory (download providers, set up backend) |
| `terraform init -upgrade` | Re-init and upgrade providers/modules to newest allowed versions |
| `terraform plan` | Show an execution plan (what will change) without applying |
| `terraform plan -out=tfplan` | Save the plan to a file for a guaranteed apply |
| `terraform apply` | Create/update infrastructure to match the config |
| `terraform apply tfplan` | Apply a previously saved plan (no re-prompt) |
| `terraform apply -auto-approve` | Apply without the interactive confirmation |
| `terraform destroy` | Destroy all resources managed by the config |
| `terraform destroy -auto-approve` | Destroy without confirmation |

## Targeting & variables
| Command | Description |
|---|---|
| `terraform apply -target=aws_instance.web` | Apply only the given resource (and its deps) |
| `terraform plan -var="region=us-east-1"` | Pass a single input variable |
| `terraform apply -var-file="prod.tfvars"` | Load variables from a file |
| `terraform apply -replace=aws_instance.web` | Force-recreate a resource (replaces old `taint`) |
| `terraform plan -refresh-only` | Detect drift without proposing config changes |

## Formatting & validation
| Command | Description |
|---|---|
| `terraform fmt` | Rewrite config files to canonical style |
| `terraform fmt -recursive` | Format files in all subdirectories |
| `terraform fmt -check` | Check formatting without writing (CI-friendly) |
| `terraform validate` | Check config for syntactic/semantic validity |
| `terraform version` | Show Terraform and provider versions |

## State management
| Command | Description |
|---|---|
| `terraform state list` | List resources tracked in state |
| `terraform state show aws_instance.web` | Show a resource's attributes in state |
| `terraform state mv SRC DEST` | Rename/move a resource in state |
| `terraform state rm aws_instance.web` | Remove a resource from state (stops managing it) |
| `terraform state pull` | Output the raw remote state |
| `terraform state push FILE` | Overwrite remote state (dangerous) |
| `terraform refresh` | Update state to match real infrastructure (deprecated → use `apply -refresh-only`) |
| `terraform force-unlock LOCK_ID` | Remove a stuck state lock |

## Import
| Command | Description |
|---|---|
| `terraform import aws_instance.web i-1234567890` | Bring an existing resource under Terraform management |
| `import { to = ..., id = ... }` block + `terraform plan` | Declarative import (config-driven, Terraform 1.5+) |

## Inspecting output
| Command | Description |
|---|---|
| `terraform output` | Show all output values |
| `terraform output instance_ip` | Show a single output value |
| `terraform output -json` | Output values as JSON (for scripts) |
| `terraform show` | Human-readable view of state or a plan file |
| `terraform show -json tfplan` | Machine-readable plan (for policy checks) |
| `terraform graph` | Emit a Graphviz DOT dependency graph |

## Workspaces
| Command | Description |
|---|---|
| `terraform workspace list` | List workspaces (state isolation within one backend) |
| `terraform workspace new dev` | Create a workspace |
| `terraform workspace select dev` | Switch to a workspace |
| `terraform workspace show` | Show the current workspace |
| `terraform workspace delete dev` | Delete a workspace |

## Providers & modules
| Command | Description |
|---|---|
| `terraform providers` | Show providers required by the config |
| `terraform providers lock` | Update the dependency lock file (`.terraform.lock.hcl`) |
| `terraform get -update` | Download/update modules referenced in config |
| `terraform init -reconfigure` | Reconfigure the backend, ignoring saved settings |
| `terraform init -migrate-state` | Migrate state when changing backends |

## Console & misc
| Command | Description |
|---|---|
| `terraform console` | Interactive REPL to evaluate expressions/functions |
| `terraform taint aws_instance.web` | Mark a resource for recreation (deprecated → use `-replace`) |
| `terraform login` | Obtain credentials for Terraform Cloud/Enterprise |
| `terraform logout` | Remove stored Terraform Cloud credentials |
| `TF_LOG=DEBUG terraform apply` | Enable debug logging via env var (`TRACE/DEBUG/INFO/WARN/ERROR`) |

## Handy flags
| Flag | Description |
|---|---|
| `-auto-approve` | Skip the interactive approval prompt |
| `-no-color` | Disable colored output (CI logs) |
| `-input=false` | Fail instead of prompting for missing input |
| `-lock=false` | Disable state locking (use with caution) |
| `-parallelism=n` | Limit concurrent resource operations (default 10) |

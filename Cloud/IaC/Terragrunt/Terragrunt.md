---
tags: [cloud, iac, terragrunt, terraform]
type: cheatsheet
source: compiled reference (Terragrunt)
last-verified: 2026-09-16
---

# Terragrunt

## Up
- [[IaC]]

Terragrunt is a thin wrapper around [[Terraform]] (and OpenTofu) that keeps large multi-environment setups **DRY**: it generates backend/provider config, wires **dependencies** between modules, and can plan/apply many modules at once. You still write Terraform modules — Terragrunt orchestrates them.

---

## Why it exists (problems it solves)
- **Repeated backend/provider blocks** across every env → generate them once.
- **Duplicated variables** across dev/stage/prod → inherit from parent config.
- **Module ordering** (VPC before EKS before apps) → declare `dependencies`.
- **Apply many stacks** together → `run-all`.

---

## Layout

```text
live/
├── terragrunt.hcl                 # root: remote_state + provider generation
├── dev/
│   ├── env.hcl                    # env-level vars
│   ├── vpc/terragrunt.hcl
│   └── eks/terragrunt.hcl         # depends on vpc
└── prod/
    └── ...
modules/                           # plain Terraform modules
├── vpc/
└── eks/
```

---

## Root config (generate backend + provider once)

```hcl
# live/terragrunt.hcl
remote_state {
  backend = "s3"
  generate = { path = "backend.tf", if_exists = "overwrite" }
  config = {
    bucket         = "my-tf-state"
    key            = "${path_relative_to_include()}/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "tf-locks"
  }
}

generate "provider" {
  path      = "provider.tf"
  if_exists = "overwrite"
  contents  = <<EOF
provider "aws" { region = "us-east-1" }
EOF
}
```

## Child unit (uses a module + inherits root + dependency)

```hcl
# live/dev/eks/terragrunt.hcl
include "root" { path = find_in_parent_folders() }

terraform { source = "../../../modules//eks" }

dependency "vpc" {
  config_path = "../vpc"
  mock_outputs = { vpc_id = "vpc-fake", subnet_ids = ["subnet-fake"] }
}

inputs = {
  vpc_id     = dependency.vpc.outputs.vpc_id
  subnet_ids = dependency.vpc.outputs.subnet_ids
  cluster_name = "dev"
}
```

---

## Commands

```bash
terragrunt plan                    # wraps `terraform plan` (auto-init, backend, deps)
terragrunt apply
terragrunt output
terragrunt run-all plan            # every module under this dir, in dependency order
terragrunt run-all apply
terragrunt run-all destroy         # reverse order
terragrunt destroy
```

`run-all` builds a DAG from `dependency` blocks and executes modules in the right order (mock_outputs let `plan` work before deps exist).

---

## Handy helpers
- `find_in_parent_folders()` — locate the root config.
- `path_relative_to_include()` — unique state key per unit.
- `read_terragrunt_config(...)` + `env.hcl`/`region.hcl` — layered variable inheritance.
- `before_hook`/`after_hook`, `errors { retry }` — lifecycle hooks and retries.

---

## Tips
- Terragrunt **doesn't replace** Terraform — you still author modules; it removes boilerplate and orchestrates.
- Use **`generate`** for backend/provider so each env is configured automatically and consistently.
- **`dependency` + `run-all`** is the killer feature for multi-module infra; add `mock_outputs` so plans work standalone.
- Keep a clean `live/` (thin per-env `terragrunt.hcl`) vs `modules/` (reusable TF) split.
- Works with OpenTofu too; pin the Terraform/Tofu binary version. For PR-driven applies see [[Atlantis]].

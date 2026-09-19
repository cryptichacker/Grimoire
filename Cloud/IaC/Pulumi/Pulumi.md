---
tags: [cloud, iac, pulumi]
type: cheatsheet
source: compiled reference (Pulumi)
last-verified: 2026-09-16
---

# Pulumi

## Up
- [[IaC]]

Pulumi is Infrastructure-as-Code using **real programming languages** (TypeScript/JavaScript, Python, Go, C#/.NET, Java, YAML) instead of a DSL. Same declarative/desired-state model as [[Terraform]] (providers, state, plan/apply), but with loops, functions, classes, and package managers from your language.

---

## Concepts

| Term | Meaning |
|---|---|
| **Project** | A program (`Pulumi.yaml` + code) that defines infra |
| **Stack** | An isolated instance/config of a project (dev/staging/prod) |
| **Resource** | A cloud object (`new aws.s3.Bucket(...)`) |
| **Input / Output** | Values flow as `Output<T>` (async, dependency-tracked) — use `.apply()` / `pulumi.interpolate` |
| **State backend** | Pulumi Cloud (default), or self-managed (S3/GCS/Azure/local) |
| **Config / Secrets** | Per-stack config; secrets encrypted in state |

---

## Workflow

```bash
pulumi new aws-typescript          # scaffold a project
pulumi stack init dev              # create a stack
pulumi config set aws:region us-east-1
pulumi config set --secret dbPassword s3cr3t     # encrypted
pulumi preview                     # like terraform plan
pulumi up                          # apply
pulumi stack output bucketName
pulumi destroy
pulumi stack select prod
```

---

## Example (TypeScript)

```typescript
import * as aws from "@pulumi/aws";
import * as pulumi from "@pulumi/pulumi";

const cfg = new pulumi.Config();

// real language features: loop to create N buckets
const buckets = ["logs", "assets", "backups"].map(name =>
  new aws.s3.Bucket(name, { acl: "private", tags: { env: "dev" } })
);

const web = new aws.ec2.Instance("web", {
  ami: "ami-123", instanceType: cfg.get("size") ?? "t3.micro",
});

// Outputs are async — interpolate/apply, don't use directly as strings
export const url = pulumi.interpolate`http://${web.publicDns}`;
```

Python/Go/C# equivalents use the same resource model in idiomatic syntax.

---

## Pulumi vs Terraform

| | Pulumi | [[Terraform]] |
|---|---|---|
| Language | TS/Python/Go/C#/Java/YAML | HCL (DSL) |
| Logic | Native loops/functions/classes | HCL constructs (`for_each`, `count`) |
| State | Pulumi Cloud or self-managed | Terraform state (local/remote) |
| Testing | Real unit tests in your language | terratest / plan checks |
| Ecosystem | Uses TF providers under the hood too | Huge provider registry |
| Best for | Teams who want real code + tests | HCL familiarity, largest ecosystem |

You can even **import/convert** Terraform (`pulumi convert --from terraform`) and Pulumi can consume Terraform providers.

---

## Testing & CI
```bash
# unit tests with mocks (jest/pytest/go test) — Pulumi's edge over HCL
pulumi up --yes                    # in CI, non-interactive
pulumi preview --diff
```

---

## Tips
- **`Output<T>` is async** — never string-concatenate raw; use `pulumi.interpolate`/`.apply()` and export via `export`/`pulumi.export`.
- One **stack per environment**; keep environment differences in stack config, not code branches.
- Store state in **Pulumi Cloud** (free tier) or a self-managed backend (S3+DynamoDB-style locking) — don't use local state for teams.
- Mark sensitive values with **`--secret`** so they're encrypted in state.
- Pick Pulumi when you want **real tests and language power**; pick [[Terraform]] for maximum ecosystem/HCL familiarity.

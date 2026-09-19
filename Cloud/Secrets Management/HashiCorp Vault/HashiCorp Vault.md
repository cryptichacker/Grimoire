---
tags: [cloud, secrets-management, vault, hashicorp]
type: cheatsheet
source: personal Notion — "💠 HashiCorp Vault" (The Complete Vault Reference Guide)
last-verified: 2026-08-25
---

# HashiCorp Vault

## Up
- [[Secrets Management]]

> Comprehensive guide to secrets management with HashiCorp Vault — imported from my Notion.

## Introduction to Secrets Management

### The Problem: Secret Sprawl
Modern apps and infrastructure deal with countless secrets.

**Common types of secrets:**

- API keys and tokens
- Database credentials (usernames, passwords)
- Encryption keys
- SSH keys
- Cloud provider access keys (AWS, Azure, GCP)
- TLS/SSL certificates
- Service account credentials
- OAuth tokens
- Private keys and certificates

### Traditional approaches and their problems

**1. Hardcoded secrets**
```python
# BAD PRACTICE - Never do this!
db_password = "MySecretPassword123"
api_key = "sk_live_abc123def456"
aws_secret = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```
Problems: secrets visible in source, accidental commits to VCS, no rotation/expiration, hard to audit, shared across teams/envs, leak into logs.

**2. Environment variables**
```bash
export DATABASE_PASSWORD="secretpass"
export API_KEY="abc123"
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
```
Problems: still static/long-lived, can leak via `ps aux`, visible in logs, no central management/rotation, hard to track usage.

**3. Configuration files**
```yaml
# config.yml
database:
  host: db.example.com
  username: admin
  password: super_secret_password
api:
  key: sk_live_abc123def456
```
Problems: accidentally shared/committed, no encryption at rest by default, VCS risk, no access auditing, same creds everywhere.

### What modern secrets management should provide
1. **Centralized storage** — single source of truth
2. **Encryption** — at rest and in transit (TLS)
3. **Dynamic secrets** — on-demand credentials with automatic expiration
4. **Access control** — fine-grained, identity/role based
5. **Audit logging** — who accessed what and when
6. **Secret rotation** — automated renewal/retirement
7. **High availability** — no single point of failure
8. **API-first** — programmatic access
9. **Secret versioning** — track changes and rollback
10. **Revocation** — instantly invalidate compromised credentials

## What is HashiCorp Vault?
**HashiCorp Vault** is an open-source secrets management tool that provides a unified interface to any secret while enforcing tight access control and recording a detailed audit log.

### Key features
- **🔐 Secrets management** — store/retrieve static secrets, generate dynamic secrets, versioning, automatic rotation.
- **🔑 Encryption as a service** — encrypt/decrypt data without storing it, generate signatures, produce random bytes.
- **🎫 Dynamic credentials** — temporary DB creds, cloud access keys, short-lived certs, automatic revocation on expiry.
- **👤 Identity-based access** — many auth methods (LDAP, AWS, Kubernetes…), token authorization, fine-grained policies, MFA.
- **📊 Audit & compliance** — detailed, tamper-proof audit logs; compliance-ready reporting.

### Vault vs. other solutions
| Feature | HashiCorp Vault | AWS Secrets Manager | Azure Key Vault | Environment Variables |
|---|---|---|---|---|
| **Cloud agnostic** | ✅ Yes | ❌ AWS only | ❌ Azure only | ✅ Yes |
| **Dynamic secrets** | ✅ Yes | ❌ Limited | ❌ No | ❌ No |
| **Encryption as a service** | ✅ Yes | ❌ No | ✅ Limited | ❌ No |
| **Fine-grained policies** | ✅ Yes | ⚠️ Basic | ⚠️ Basic | ❌ No |
| **Audit logging** | ✅ Comprehensive | ✅ Yes | ✅ Yes | ❌ No |
| **Secret versioning** | ✅ Yes | ✅ Yes | ✅ Yes | ❌ No |
| **On-premises support** | ✅ Yes | ❌ No | ❌ No | ✅ Yes |
| **Open source** | ✅ Yes | ❌ No | ❌ No | ✅ N/A |

## Core Concepts

**1. Secrets** — anything you want to tightly control access to (passwords, API keys, certs, encryption keys, DB creds, cloud tokens).

**2. Secrets engines** — components that store, generate, or encrypt data (plugins you enable and configure): static KV stores, dynamic-secret generators, and encryption engines.

**3. Authentication methods** — verify the identity of users/apps:
- **Token** — default, tokens created by Vault
- **Username & password** — simple user/pass
- **LDAP/Active Directory** — enterprise directory integration
- **AWS** — authenticate using AWS IAM roles
- **Kubernetes** — service-account based
- **GitHub** — via personal access tokens
- **AppRole** — designed for machines and applications
- **TLS certificates** — certificate-based auth

**4. Policies** — define allowed actions on paths, written in HCL:
```hcl
# Example policy
path "secret/data/myapp/*" {
  capabilities = ["read", "list"]
}

path "database/creds/readonly" {
  capabilities = ["read"]
}
```

**5. Tokens** — the core auth mechanism; every request must include one. Properties: TTL, renewable, policies, accessor (a reference for management).

**6. Leases** — metadata returned with dynamic secrets: duration, renewable, lease ID (for revocation).

**7. Paths** — everything is path-based, like a filesystem:
```text
secret/           # Secrets engine mount
├── data/
│   ├── myapp/
│   │   ├── config     # Secret at secret/data/myapp/config
│   │   └── database   # Secret at secret/data/myapp/database
```

**8. Seal/Unseal** — *sealed* = encrypted and inaccessible; *unsealed* = has the encryption key and can read/write. Unsealing needs a threshold of unseal keys (default 3 of 5, via Shamir's Secret Sharing).

## Vault Architecture

### High-level layers
**1. External access layer (HTTP API)** — RESTful, TLS-encrypted; CLI and UI are built on top.

**2. Internal components layer:**
- **Storage backend** — where encrypted data lives (Consul, Raft, etc.)
- **Barrier** — encryption layer that encrypts/decrypts all data
- **Secrets engines** — store or generate secrets
- **Auth methods** — verify identity and issue tokens
- **Audit devices** — log all requests/responses
- **Policy store**, **Token store**, **Expiration manager**, **Core** (routes requests, enforces ACLs)

**3. Storage layer** — physical backend outside the barrier; all data encrypted before storage, never plaintext.

```text
┌─────────────────────────────────────────────────────┐
│                   HTTP API (TLS)                     │
└─────────────────────────────────────────────────────┘
                         │
┌─────────────────────────────────────────────────────┐
│                     Barrier                          │
│              (Encryption/Decryption)                 │
└─────────────────────────────────────────────────────┘
                         │
┌─────────────────────────────────────────────────────┐
│                   Vault Core                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐            │
│  │ Secrets  │  │   Auth   │  │ Policies │            │
│  │ Engines  │  │ Methods  │  │          │            │
│  └──────────┘  └──────────┘  └──────────┘            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐            │
│  │  Token   │  │  Audit   │  │  Lease   │            │
│  │  Store   │  │ Devices  │  │  Manager │            │
│  └──────────┘  └──────────┘  └──────────┘            │
└─────────────────────────────────────────────────────┘
                         │
┌─────────────────────────────────────────────────────┐
│              Storage Backend                         │
│         (Consul, Raft, etcd, etc.)                   │
│           (Encrypted at Rest)                        │
└─────────────────────────────────────────────────────┘
```

### Seal/Unseal process
**Sealed state:** Vault starts sealed — it knows where data is but can't decrypt it; only status checks and unsealing are allowed. The encryption key is itself encrypted by the "unseal key".

**Unseal (Shamir's Secret Sharing):** at init Vault generates an encryption key, splits it into 5 shares (configurable), and needs 3 (threshold) to reconstruct. Each key holder submits their share; once the threshold is met, Vault unseals.

**Auto-unseal (Enterprise/Cloud):** use a cloud KMS (AWS KMS, Azure Key Vault, GCP KMS) so Vault unseals automatically on startup — convenient but requires trusting the cloud provider.

### Request flow
```text
1. Client Request → 2. HTTP API (with Token) → 3. Authentication Validation
→ 4. Policy Check (Authorization) → 5. Route to Secrets Engine
→ 6. Generate/Retrieve Secret → 7. Attach Lease (if dynamic)
→ 8. Log to Audit Devices → 9. Return Encrypted Response → 10. Client Receives Secret
```

### Storage backends
- **Integrated Storage (Raft)** — *recommended*: built-in consensus, no external deps, HA, snapshots.
- **Consul** — HashiCorp's service mesh; HA but more complex.
- **Others** — etcd, DynamoDB, PostgreSQL, MySQL, File (dev only).

## Installation and Setup

**Method 1: Binary download**
```bash
# Download Vault (Linux example)
wget https://releases.hashicorp.com/vault/1.20.0/vault_1.20.0_linux_amd64.zip
unzip vault_1.20.0_linux_amd64.zip
sudo mv vault /usr/local/bin/
vault version
```

**Method 2: Package manager**
```bash
# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/vault

# Ubuntu/Debian
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vault
```

**Method 3: Docker**
```bash
docker run --cap-add=IPC_LOCK -d --name=dev-vault -p 8200:8200 hashicorp/vault
```

### Dev mode (⚠️ never in production)
```bash
vault server -dev
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='<root-token-from-output>'
```

### Production initialization
**Config file (config.hcl):**
```hcl
storage "raft" {
  path    = "/opt/vault/data"
  node_id = "node1"
}

listener "tcp" {
  address       = "0.0.0.0:8200"
  tls_disable   = "false"
  tls_cert_file = "/opt/vault/tls/vault-cert.pem"
  tls_key_file  = "/opt/vault/tls/vault-key.pem"
}

api_addr = "https://vault.example.com:8200"
ui = true
```
**Initialize, unseal, log in:**
```bash
vault operator init -key-shares=5 -key-threshold=3
vault operator unseal <key-1>
vault operator unseal <key-2>
vault operator unseal <key-3>
vault login <initial-root-token>
```

## Secrets Engines

### Key-Value (KV) v2
```bash
vault secrets enable -path=secret kv-v2
vault kv put secret/myapp/config username="admin" password="pass123"
vault kv get secret/myapp/config
vault kv list secret/myapp/
```

### Database engine (dynamic creds)
```bash
vault secrets enable database

vault write database/config/mydb \
    plugin_name=postgresql-database-plugin \
    connection_url="postgresql://{{username}}:{{password}}@postgres:5432/db" \
    username="vault" \
    password="vaultpass"

vault write database/roles/readonly \
    db_name=mydb \
    creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
    default_ttl="1h"

vault read database/creds/readonly
```

## Authentication Methods

### AppRole (recommended for CI/CD)
```bash
vault auth enable approle

vault write auth/approle/role/jenkins \
    token_policies="jenkins-policy" \
    token_ttl=20m \
    token_max_ttl=1h

vault read auth/approle/role/jenkins/role-id
vault write -f auth/approle/role/jenkins/secret-id

vault write auth/approle/login \
    role_id="<role-id>" \
    secret_id="<secret-id>"
```

## Policies and Access Control
```hcl
# myapp-policy.hcl
path "secret/data/myapp/*" {
  capabilities = ["read", "list"]
}
path "database/creds/readonly" {
  capabilities = ["read"]
}
path "secret/data/admin/*" {
  capabilities = ["deny"]
}
```
**Capabilities:** `create`, `read`, `update`, `delete`, `list`, `sudo` (root-protected paths), `deny` (explicit deny).

**Create and assign:**
```bash
vault policy write myapp-policy myapp-policy.hcl
vault policy list
vault policy read myapp-policy
vault token create -policy=myapp-policy
vault write auth/userpass/users/john policies="myapp-policy,default"
```

## Using Vault — Basic Operations

### CLI
```bash
vault kv put secret/myapp/db password="secretdb123"
vault kv get secret/myapp/db
vault kv get -field=password secret/myapp/db
vault kv delete secret/myapp/db
vault kv list secret/myapp/
vault status
vault token lookup
```

### API
```bash
# Write
curl \
    --header "X-Vault-Token:$VAULT_TOKEN" \
    --request POST \
    --data '{"data": {"password": "secretdb123"}}' \
    https://vault.example.com/v1/secret/data/myapp/db

# Read
curl \
    --header "X-Vault-Token:$VAULT_TOKEN" \
    https://vault.example.com/v1/secret/data/myapp/db
```

### Python (hvac)
```python
import hvac, os

client = hvac.Client(
    url='https://vault.example.com',
    token=os.environ['VAULT_TOKEN']
)

client.secrets.kv.v2.create_or_update_secret(
    path='myapp/db',
    secret=dict(password='secretdb123'),
)

secret = client.secrets.kv.v2.read_secret_version(path='myapp/db')
password = secret['data']['data']['password']

secrets_list = client.secrets.kv.v2.list_secrets(path='myapp')
print(secrets_list['data']['keys'])
```

### Go
```go
package main

import (
    "fmt"
    "log"
    vault "github.com/hashicorp/vault/api"
)

func main() {
    config := vault.DefaultConfig()
    config.Address = "https://vault.example.com"

    client, err := vault.NewClient(config)
    if err != nil { log.Fatal(err) }
    client.SetToken("your-vault-token")

    data := map[string]interface{}{
        "data": map[string]interface{}{"password": "secretdb123"},
    }
    _, err = client.Logical().Write("secret/data/myapp/db", data)
    if err != nil { log.Fatal(err) }

    secret, err := client.Logical().Read("secret/data/myapp/db")
    if err != nil { log.Fatal(err) }
    password := secret.Data["data"].(map[string]interface{})["password"]
    fmt.Printf("Password: %v\n", password)
}
```

## CI/CD Pipeline Integration

### Jenkins
1. Install the **HashiCorp Vault** plugin (Manage Jenkins → Plugins).
2. Configure Vault (URL + AppRole credential: Role ID + Secret ID).
3. Policy:
```hcl
# jenkins-policy.hcl
path "secret/data/jenkins/*" { capabilities = ["read", "list"] }
path "database/creds/deploy" { capabilities = ["read"] }
```
```bash
vault policy write jenkins jenkins-policy.hcl
vault write auth/approle/role/jenkins \
    token_policies="jenkins-policy" token_ttl=20m token_max_ttl=1h secret_id_ttl=24h
```
4. Jenkinsfile:
```groovy
pipeline {
    agent any
    environment { VAULT_ADDR = 'https://vault.example.com' }
    stages {
        stage('Get Secrets') {
            steps {
                withVault([
                    vaultSecrets: [[
                        path: 'secret/jenkins/myapp',
                        secretValues: [
                            [envVar: 'DB_PASSWORD', vaultKey: 'db_password'],
                            [envVar: 'API_KEY', vaultKey: 'api_key']
                        ]
                    ]]
                ]) {
                    sh 'echo "Using DB Password: ${DB_PASSWORD}"'
                }
            }
        }
        stage('Deploy') { steps { sh './deploy.sh' } }
    }
}
```

### GitHub Actions
Set repo secrets `VAULT_ADDR`, `VAULT_ROLE_ID`, `VAULT_SECRET_ID`, then:
```yaml
# .github/workflows/deploy.yml
name: Deploy Application
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Import Secrets from Vault
        uses: hashicorp/vault-action@v2
        with:
          url: ${{ secrets.VAULT_ADDR }}
          method: approle
          roleId: ${{ secrets.VAULT_ROLE_ID }}
          secretId: ${{ secrets.VAULT_SECRET_ID }}
          secrets: |
            secret/data/github/myapp db_password | DB_PASSWORD ;
            secret/data/github/myapp api_key | API_KEY
      - name: Deploy Application
        run: ./deploy.sh
        env:
          DB_PASSWORD: ${{ env.DB_PASSWORD }}
          API_KEY: ${{ env.API_KEY }}
```

### GitLab CI
```yaml
# .gitlab-ci.yml
variables:
  VAULT_ADDR: "https://vault.example.com"
stages:
  - deploy
deploy:
  stage: deploy
  image: vault:latest
  script:
    - export VAULT_TOKEN=$(vault write -field=token auth/approle/login role_id=$VAULT_ROLE_ID secret_id=$VAULT_SECRET_ID)
    - export DB_PASSWORD=$(vault kv get -field=db_password secret/gitlab/myapp)
    - export API_KEY=$(vault kv get -field=api_key secret/gitlab/myapp)
    - ./deploy.sh
  only:
    - main
```

### Kubernetes (Agent Injector)
```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault --set "injector.enabled=true"

# Enable Kubernetes auth
vault auth enable kubernetes
vault write auth/kubernetes/config \
    kubernetes_host="https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT"
vault write auth/kubernetes/role/myapp \
    bound_service_account_names=myapp \
    bound_service_account_namespaces=default \
    policies=myapp-policy \
    ttl=24h
```
Pod annotations to inject secrets:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "myapp"
    vault.hashicorp.com/agent-inject-secret-database: "secret/data/myapp/db"
    vault.hashicorp.com/agent-inject-template-database: |
      {{- with secret "secret/data/myapp/db" -}}
      export DB_PASSWORD="{{ .Data.data.password }}"
      export DB_USERNAME="{{ .Data.data.username }}"
      {{- end }}
spec:
  serviceAccountName: myapp
  containers:
    - name: app
      image: myapp:latest
      command: ["/bin/sh"]
      args: ["-c", "source /vault/secrets/database && ./start.sh"]
```

## Production Deployment

### High availability (5-node Raft cluster)
```hcl
# node1.hcl
storage "raft" {
  path    = "/opt/vault/data"
  node_id = "node1"
  retry_join { leader_api_addr = "https://node2:8200" }
  retry_join { leader_api_addr = "https://node3:8200" }
}

listener "tcp" {
  address       = "0.0.0.0:8200"
  tls_cert_file = "/opt/vault/tls/cert.pem"
  tls_key_file  = "/opt/vault/tls/key.pem"
}

api_addr     = "https://node1.vault.example.com:8200"
cluster_addr = "https://node1.vault.example.com:8201"
ui = true
```
**Load balancer:** health check `GET /v1/sys/health`; expect 200 (active) or 429/473 (standby); sticky sessions not required.

### Backup and recovery (Raft snapshots)
```bash
vault operator raft snapshot save backup.snap
vault operator raft snapshot restore backup.snap
# Automate with cron
0 2 * * * vault operator raft snapshot save /backups/vault-$(date +\%Y\%m\%d).snap
```

### Monitoring
```bash
curl https://vault.example.com/v1/sys/health
curl https://vault.example.com/v1/sys/metrics
```
**Health response codes:** `200` active · `429` standby (unsealed) · `472` DR secondary active · `473` performance standby · `501` not initialized · `503` sealed.

**Key metrics:** `vault.core.unsealed`, `vault.runtime.alloc_bytes`, `vault.barrier.put`, `vault.token.creation`, `vault.raft.leader`, `vault.audit.log_request`.

## Best Practices

### Security
1. **Never use the root token in production** — create an admin policy/token instead, then revoke root:
```bash
vault policy write admin admin-policy.hcl
vault token create -policy=admin -ttl=1h
vault token revoke <root-token>
```
2. **Enable audit logging:** `vault audit enable file file_path=/var/log/vault_audit.log`
3. **Use short-lived tokens:** `vault token create -ttl=1h -policy=myapp` (use `-period=24h` for renewable long-running processes).
4. **Rotate credentials:** `vault write -f database/rotate-root/my-database`, `vault write -f transit/keys/my-key/rotate`.
5. **Use AppRole for applications** — never static tokens or user/pass.
6. **Least privilege** — grant only the minimum in policies.
7. **Enable MFA for sensitive operations:**
```bash
vault write sys/mfa/method/totp/my_totp \
    issuer=Vault period=30 key_size=30 algorithm=SHA256
```

### Operational
1. Automate unsealing with cloud KMS in production.
2. Automate Raft snapshots daily.
3. Monitor key metrics; alert on sealed state and failed auth.
4. Regularly test snapshot restore (DR).
5. Separate clusters for dev/staging/prod.
6. Version-control policies in Git, apply via CI/CD.
7. Document emergency runbooks (unseal, failover).

## Troubleshooting

**Vault is sealed**
```bash
vault status
vault operator unseal <key-1>
vault operator unseal <key-2>
vault operator unseal <key-3>
```
**Permission denied**
```bash
vault token lookup
vault policy read myapp-policy
vault token capabilities secret/data/myapp/config
```
**Connection refused**
```bash
systemctl status vault
cat /etc/vault/config.hcl
sudo ufw status
sudo iptables -L
```
**High memory usage**
```bash
vault read sys/metrics
vault token revoke -mode=orphan -accessor <accessor>
systemctl restart vault   # requires unseal afterward
```

## Glossary
- **AppRole** — auth method for machines/applications.
- **Audit Device** — logs all Vault requests/responses.
- **Auth Method** — authenticates users/machines and issues tokens.
- **Barrier** — encryption layer between core and storage backend.
- **Lease** — metadata on dynamic secrets defining TTL and renewal.
- **Mount** — path where a secrets engine or auth method is enabled.
- **Policy** — HCL document defining allowed operations on paths.
- **Seal/Unseal** — encrypted (sealed) vs operational (unsealed) state.
- **Secrets Engine** — component that stores, generates, or encrypts data.
- **Shamir's Secret Sharing** — splits the unseal key into shares.
- **Token** — auth credential for API requests.
- **TTL** — Time To Live; how long a secret/token is valid.

## Additional Resources
- Docs: https://developer.hashicorp.com/vault
- Tutorials: https://developer.hashicorp.com/vault/tutorials
- API: https://www.vaultproject.io/api-docs
- Community: https://discuss.hashicorp.com/c/vault · https://github.com/hashicorp/vault
- Certification: HashiCorp Certified: Vault Associate

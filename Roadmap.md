---
tags: [meta, roadmap, todo]
type: note
last-updated: 2026-08-30
---

# Roadmap — Nodes To Add

Running to-do list of tools/concepts to add to the knowledge base. Check off as notes are created.

---

## Cloud & DevOps

### New categories (no node yet)
- [ ] **CI/CD** (root/category) — GitHub Actions, GitLab CI, Jenkins, CircleCI, Tekton, Argo Workflows
- [ ] **GitOps** — ArgoCD, Flux
- [ ] **Observability stack** (expand Monitoring) — Grafana, Loki, Tempo, Jaeger, OpenTelemetry, ELK/EFK
- [ ] **DevSecOps / Scanning** — Trivy, Grype, Checkov, tfsec, KICS, OWASP ZAP, Snyk, gitleaks, trufflehog, Sigstore/cosign + SBOM
- [ ] **Messaging & Streaming** — Kafka, RabbitMQ, NATS

### Kubernetes (add under existing node)
- [x] Kustomize ✅
- [x] Ingress controllers (NGINX / Traefik) ✅
- [x] cert-manager ✅
- [x] Kyverno / OPA Gatekeeper (policy) ✅
- [x] Falco (runtime security) ✅ (see Hacking/Container & K8s Security; cross-linked from Kubernetes)
- [x] KEDA / HPA / VPA (autoscaling) → `Kubernetes/Autoscaling` (HPA, VPA, KEDA) ✅ (2026-09-02)
- [x] CNI (Calico / Cilium) ✅
- [x] Velero (backup/restore) ✅
- [x] Operators & CRDs ✅ (2026-09-16)

### Docker (add under existing node)
- [x] Podman ✅
- [x] BuildKit / buildx ✅
- [x] Dockerfile best practices ✅
- [x] Harbor (registry) ✅ (2026-09-16)

### IaC (add under existing node)
- [x] Pulumi ✅
- [x] Terragrunt ✅
- [x] Packer ✅
- [x] Crossplane ✅
- [x] AWS CDK / CloudFormation ✅
- [x] Atlantis ✅ (2026-09-16)

### Secrets Management (add under existing node)
- [ ] External Secrets Operator
- [ ] Sealed Secrets
- [ ] SOPS

---

## Coding

### New languages (new nodes under Coding)
- [ ] **Go** (highest priority — cloud-native language)
- [ ] Rust
- [ ] JavaScript / TypeScript
- [ ] PowerShell
- [ ] Git
- [ ] Regex
- [ ] YAML / JSON

### Python (add under existing node)
- [x] asyncio ✅
- [x] pytest ✅
- [x] SQLAlchemy ✅
- [x] Pydantic ✅
- [x] Celery ✅
- [x] Django ✅
- [x] Packaging (Poetry / uv) ✅ (2026-09-03)

---

## Hacking

> Categories to flesh out — see discussion. Currently only **Web** (PortSwigger + OWASP) exists.

- [x] **Reconnaissance** (active) — Nmap, Masscan ✅
- [x] **OSINT** (passive) — Amass, Sublist3r, theHarvester, Shodan, recon-ng, Google Dorking ✅ (split out 2026-08-30)
- [ ] **Network / Infrastructure** — Metasploit, Nessus, OpenVAS, Responder, CrackMapExec, Impacket
- [x] **Wireless** (expanded to all RF, not just Wi-Fi) — RF Fundamentals, SDR, Wi-Fi, Bluetooth/BLE, RFID/NFC, Zigbee/Z-Wave, Cellular, GPS/GNSS, Sub-GHz/ISM ✅ (added 2026-09-01)
- [x] **Password Cracking** — Password Attack Types, Hashcat, John the Ripper, Hydra, Ophcrack (rainbow tables), Wordlists/SecLists ✅ (added 2026-08-30)
- [ ] **Exploitation frameworks** — Metasploit, Cobalt Strike, Sliver, Havoc
- [x] **Active Directory** — full category under Hacking (Fundamentals, Enum, Kerberos, Delegation, ACL/GPO, NTLM Relay, ADCS, Lateral, Trusts, Defense) ✅ (2026-09-03)
- [ ] **Privilege Escalation** — LinPEAS/WinPEAS, GTFOBins, LOLBAS
- [x] **Cloud Pentesting** — Cloud Attack Concepts, Pacu, ScoutSuite, Prowler, kube-hunter, Peirates ✅ (added 2026-08-30)
- [x] **Container / K8s Security** — Container Attack Concepts, Trivy, kube-bench, Falco (kube-hunter already under Cloud Pentesting) ✅ (added 2026-09-01)
- [ ] **Binary / Reverse engineering** — Ghidra, IDA, radare2, x64dbg, gdb/pwndbg
- [ ] **Mobile** — MobSF, Frida, objection, apktool
- [ ] **Forensics / DFIR** — Volatility, Autopsy, Wireshark, YARA
- [ ] **Malware / Evasion** — obfuscation, packers, C2 concepts (defensive study)
- [ ] **Web (expand)** — Burp Suite, ffuf, sqlmap, nuclei, Nikto, wpscan
- [x] **Methodology / Frameworks** — Cyber Kill Chain, MITRE ATT&CK, PTES, OSSTMM, CVSS ✅ (added 2026-08-30)

---

## AI

> New root node created 2026-09-01 (`AI/`). Building out incrementally.

### Theory / Fundamentals
- [x] ML basics — supervised/unsupervised/RL, train/val/test, over/underfitting, loss & gradient descent ✅ (2026-09-17)
- [x] Deep learning — neural nets, backprop, activations, CNNs, RNNs/LSTMs ✅ (2026-09-17)
- [x] Transformers & attention — self-attention, positional encoding, encoder/decoder ✅ (2026-09-17)
- [x] LLM fundamentals — tokenization, embeddings, pretraining, context window, decoding ✅ (2026-09-17)
- [x] Training & alignment — RLHF, DPO, instruction tuning, distillation, quantization, LoRA/PEFT ✅ (2026-09-17)
- [x] Prompting & inference — prompt engineering, few-shot, chain-of-thought, temperature/top-p ✅ (2026-09-17)
- [x] RAG — embeddings, chunking, vector search, retrieval pipelines ✅ (2026-09-17)
- [x] Agents — tool use, ReAct, planning, memory, MCP, A2A ✅ (2026-09-17)
- [x] Evaluation — benchmarks (MMLU, HumanEval, …), hallucination, guardrails ✅ (2026-09-17)
- [x] Multimodal — vision, audio (Whisper), diffusion / image generation ✅ (2026-09-17)
- [x] Ethics & safety — bias, alignment, jailbreaks, privacy (cross-link Hacking → Web LLM attacks) ✅ (2026-09-17)

### Famous Models
- [ ] OpenAI — GPT family (GPT-4o, o-series, …)
- [ ] Anthropic — Claude family
- [ ] Google — Gemini
- [ ] Meta — Llama (open weights)
- [ ] Mistral / Mixtral
- [ ] DeepSeek
- [ ] xAI — Grok
- [ ] Alibaba — Qwen
- [ ] Image — Stable Diffusion / FLUX
- [ ] Speech — Whisper
- [ ] Embedding models — BERT, sentence-transformers

### Tools / Frameworks
- [ ] Local runtimes — Ollama, LM Studio, llama.cpp, vLLM
- [ ] Frameworks — LangChain (have under Coding/Python), LlamaIndex, Haystack, DSPy
- [ ] Agent frameworks — LangGraph, CrewAI, AutoGen, Semantic Kernel
- [ ] Vector DBs — Chroma, Qdrant, Weaviate, Pinecone, FAISS, pgvector
- [ ] Gateways / routers — OpenRouter, LiteLLM
- [ ] Fine-tuning — Hugging Face Transformers/PEFT, Axolotl, Unsloth
- [ ] LLMOps — LangSmith, Weights & Biases, MLflow
- [ ] Dev tools — Claude Code, Cursor, Copilot, Aider, Cline

### Tools → Misc (done)
- [x] **OmniRoute** — self-hosted multi-provider AI gateway/router → `AI/Tools/Misc/Omniroute` ✅ (2026-09-01)

### Skills (agent skills / plugins)

> Node: `AI/Skills/`. Popular, **vault-relevant** skills — mapped to what we already have (Cloud/DevOps, Hacking, Coding, docs). Always scan a third-party skill with **[[nvidia-skillspector]]** before installing.

Already added as nodes:
- [x] **superpowers** — agentic dev methodology (TDD, plan-before-code) → `AI/Skills/superpowers` ✅ (2026-09-16)
- [x] **ui-ux-pro-max** — design-system intelligence for UI/UX → `AI/Skills/ui-ux-pro-max` ✅ (2026-09-16)
- [x] **andrej-karpathy-skills** — CLAUDE.md coding guidelines (tight diffs, explicit assumptions) → `AI/Skills/andrej-karpathy-skills` ✅ (2026-09-16)
- [x] **nvidia/skillspector** — security scanner for agent skills (supply-chain / prompt-injection) → `AI/Skills/nvidia-skillspector` ✅ (2026-09-16)
- [x] **caveman** — token-compression skill/proxy (terse "caveman-speak", cost savings) → `AI/Skills/caveman` ✅ (2026-09-16)
- [x] **graphify** — codebase/docs → queryable knowledge graph (tree-sitter AST, no vectors) → `AI/Skills/graphify` ✅ (2026-09-16)

To explore — official / high-value:
- [x] **anthropics/skills** (official) — document skills **docx / pptx / xlsx / pdf** (+ `mcp-builder`, spec & template) → `AI/Skills/anthropics-skills` ✅ (2026-09-16)
- [ ] **mcp-builder** (from anthropics/skills) — scaffolds new **MCP servers**; pairs with the MCP/agents work under [[AI]]
- [ ] **hesreallyhim/awesome-claude-code** / **travisvn/awesome-claude-skills** — curated directories to discover + vet more skills

To explore — Cloud / DevOps (map to [[Kubernetes]], [[Docker]], [[IaC]]):
- [ ] **wshobson/agents** — large multi-harness marketplace of subagents/skills incl. **Kubernetes, Terraform, Docker, CI/CD, cloud-architecture** reviewers
- [ ] Terraform / IaC review skill — plan-diff review, drift & security checks (cross-link [[Terraform]], [[Atlantis]])
- [ ] Kubernetes manifest / Helm review skill — lint + security posture (cross-link [[Helm]], [[Container & K8s Security]])

To explore — Hacking / security (map to [[Hacking]]):
- [ ] Security code-review / SAST skill — secrets, injection, authz flaws in a repo
- [ ] Recon / OSINT helper skill — structured target enumeration write-ups (cross-link [[Reconnaissance]], [[OSINT]])
- [ ] Bug-bounty report-writing skill — templated PoC → impact → remediation (cross-link [[Bug Bounty]])

To explore — Coding / general (map to [[Coding]]):
- [ ] **davila7/claude-code-templates** (aitmpl) — CLI + big catalog of agents/skills/commands/MCPs; good source for Python/SQL/Git helpers
- [ ] Git / commit-message / PR-review skill — conventional commits, changelogs, review passes
- [ ] Test-generation skill — unit/integration tests (composes with [[superpowers]] TDD)

---

## Notes
- Keep third-party copyrighted material as original write-ups + original diagrams (per existing convention).
- Each new leaf note: `## Up - [[Parent]]`; each new category: MOC with `## Subtopics`.

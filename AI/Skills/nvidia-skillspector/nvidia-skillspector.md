---
tags: [ai, skills, claude, security, nvidia, supply-chain]
type: cheatsheet
source: web research — github.com/NVIDIA/SkillSpector
last-verified: 2026-09-16
---

# nvidia/skillspector

## Up
- [[Skills]]

**SkillSpector** (by **NVIDIA**, Apache-2.0) is a **security scanner for AI agent skills** — you point it at a skill *before* you install it and it flags prompt injection, data exfiltration, supply-chain, and other malicious/vulnerable patterns. It's the "check what you're about to `/plugin install`" tool: NVIDIA's own research found **~26% of public skills contain vulnerabilities and ~5% show likely malicious intent**, so scanning third-party skills (like the ones in this [[Skills]] node) before enabling them is worth the 10 seconds.

> Purely **static** analysis — it never executes the skill. Optional LLM pass adds semantic judgement.

---

## What it detects — 69 patterns across 17 categories

| Category | Examples |
|---|---|
| **Prompt Injection** | instruction overrides, hidden directives, exfiltration commands |
| **Data Exfiltration** | external transmission, env-var harvesting, filesystem enumeration |
| **Supply Chain** | unpinned deps, vulnerable/typosquatted packages, obfuscated code |
| **Excessive Agency** | unrestricted tool access, autonomous decisions |
| **Behavioral (AST)** | `exec`/`eval`, subprocess spawns, dynamic imports |
| **Taint Tracking** | credential → network flows, exfiltration chains |
| **Anti-Refusal** | refusal suppression, disclaimer removal |
| **MCP Security** | least-privilege violations, tool poisoning, hidden instructions |
| **+ more** | privilege escalation, system-prompt leakage, memory poisoning, rogue agents, YARA sigs |

Also does **live CVE lookups** via OSV.dev (offline fallback).

---

## Install

```bash
# recommended (uv)
uv tool install git+https://github.com/NVIDIA/skillspector.git

# with MCP-server support
uv tool install 'skillspector[mcp] @ git+https://github.com/NVIDIA/skillspector.git'

# docker
docker run --rm -v "$PWD:/scan" skillspector scan ./my-skill/ --no-llm
```
Requires **Python 3.12+**.

---

## Scan anything

```bash
skillspector scan ./my-skill/                 # a directory
skillspector scan ./SKILL.md                  # a single file
skillspector scan https://github.com/u/skill  # a git repo (scan BEFORE cloning/installing)
skillspector scan ./my-skill.zip              # a zip

skillspector scan ./my-skill/ --no-llm        # fast static-only pass
```

### Reports / CI
```bash
skillspector scan ./s/ --format json     --output report.json
skillspector scan ./s/ --format markdown --output report.md
skillspector scan ./s/ --format sarif    --output report.sarif   # GitHub code-scanning

# suppress known false positives
skillspector baseline ./s/ -o .skillspector-baseline.yaml
skillspector scan ./s/ --baseline .skillspector-baseline.yaml
```

**Exit codes:** `0` safe/caution · `1` do-not-install · `2` error — so it gates a pipeline cleanly.

---

## Risk score (0–100)

| Score | Severity | Verdict |
|---|---|---|
| 0–20 | LOW | ✅ SAFE |
| 21–50 | MEDIUM | ⚠️ CAUTION |
| 51–80 | HIGH | ⛔ DO NOT INSTALL |
| 81–100 | CRITICAL | ⛔ DO NOT INSTALL |

---

## Optional LLM analysis

Static is default; add semantic evaluation by setting a provider:

```bash
export SKILLSPECTOR_PROVIDER=anthropic; export ANTHROPIC_API_KEY=sk-ant-...
# or openai / bedrock / local ollama (OPENAI_BASE_URL=http://localhost:11434/v1)
# or the local Claude CLI, no key needed:
export SKILLSPECTOR_PROVIDER=claude_cli
skillspector scan ./my-skill/
```

---

## Run it as an MCP server / in CI

```bash
skillspector mcp                                   # stdio (local agents)
skillspector mcp --transport http --port 8000      # remote callers
```
Exposes `scan_skill(target, use_llm=true, output_format="json")` for **runtime install-gating**. Also drops into a **GitHub Action** to scan skill PRs automatically (SARIF → code-scanning tab).

Python API:
```python
from skillspector import graph
r = graph.invoke({"input_path": "/path/to/skill", "output_format": "json", "use_llm": True})
print(r["risk_score"], r["risk_recommendation"])
```

---

## When to use
- **Before installing any third-party skill/plugin** — including [[superpowers]], [[ui-ux-pro-max]], [[andrej-karpathy-skills]] or anything off a random marketplace.
- Gating skill installs in a team via **CI (SARIF)** or the **MCP runtime hook**.
- Auditing MCP servers for tool-poisoning / hidden-instruction risks.

## Notes
- Ties this [[Skills]] node to the security work in [[Hacking]] — it's essentially supply-chain / prompt-injection scanning for the agent-skill ecosystem.
- Static-only by default = safe to run on untrusted skills (no execution). Turn on LLM only when you want deeper semantic checks.

## Source
- github.com/NVIDIA/SkillSpector · docs.nvidia.com/skills/scanning-agent-skills

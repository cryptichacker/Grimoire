---
tags: [ai, tools, gateway, llm, omniroute]
type: cheatsheet
source: compiled reference (OmniRoute) — web research 2026-09-01
last-verified: 2026-09-01
---

# Omniroute

## Up
- [[Misc]]

OmniRoute is an **MIT-licensed, self-hosted AI gateway/router**: you run it on your own machine and it exposes a single **OpenAI-compatible endpoint** that fans out to hundreds of LLM providers. It adds automatic failover, cost/quota-aware routing, prompt compression, and keeps your API keys **local**. Think "local OpenRouter/LiteLLM" — a drop-in proxy for coding agents and apps.

---

## What It Is / Why Use It

- **One API, many providers** — ~230+ catalogued providers (OpenAI, Anthropic, Google, xAI, DeepSeek, Mistral, Meta, Groq, NVIDIA, …), including many with **free tiers**, behind a single URL.
- **Local-first & private** — runs on `localhost`, credentials encrypted at rest (AES-256-GCM); keys never leave your machine.
- **Resilient** — quota-aware **automatic fallback** with circuit breakers + exponential backoff so an agent keeps working when one provider rate-limits.
- **Cheaper** — multiple routing strategies + **prompt compression** to cut token spend.
- **Drop-in** — OpenAI-compatible `/v1`, so tools like Claude Code, Cursor, Cline, Continue, Aider work unchanged.

---

## Install & Run

```bash
# npm (global) — simplest
npm install -g omniroute
omniroute                     # starts the gateway + web dashboard

# Docker
docker run -p 20128:20128 -v omniroute-data:/app/data \
  diegosouzapw/omniroute:latest

# also available via pnpm, Arch AUR, Nix, an Electron desktop app, and Android (Termux)
```

- Server listens on **`http://localhost:20128`**; the OpenAI endpoint is **`http://localhost:20128/v1`**.
- A **web dashboard** manages providers, keys, routing, and generates the gateway API key.

---

## Point a Client At It (OpenAI-compatible)

```bash
# any OpenAI SDK / tool:
export OPENAI_BASE_URL="http://localhost:20128/v1"
export OPENAI_API_KEY="<key-from-omniroute-dashboard>"

curl http://localhost:20128/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"auto","messages":[{"role":"user","content":"hi"}]}'
```

- **Model field:** use a specific model, `auto` (zero-config best-pick), or a custom **combo chain**.
- Works as the backend for **Claude Code, Cursor, Codex, Cline, Aider, Continue** and 16+ coding tools by just setting base URL + key.

---

## Routing Strategies

Many strategies (priority, round-robin, **cost-optimized**, auto-scoring, **fusion**, pipeline, and more). Auto modes build virtual, real-time-scored model chains:

| Auto mode | Picks for |
|---|---|
| `auto` | Best overall, zero-config |
| `auto/coding` | Coding-tuned models |
| `auto/fast` | Lowest latency |
| `auto/cheap` | Lowest cost |
| `auto/offline` | Local/offline models |

**Combos** chain models in sequence — when one hits a quota/limit, requests fail over to the next automatically.

---

## Token Compression

- Multi-engine pipeline (RTK, Caveman, **LLMLingua-2**) compresses outbound prompts.
- Reported **~15–95% token savings** on tool-heavy/repetitive payloads, while preserving code integrity (targets prose/repetition, not your source files).

---

## Agent Integration (MCP / A2A)

- Ships **MCP** tools so agents can reconfigure routing/compression/providers autonomously (`claude mcp add-server omniroute`).
- Supports **A2A** (agent-to-agent) control patterns.

---

## OmniRoute vs Alternatives

| | **OmniRoute** | **OpenRouter** | **LiteLLM** |
|---|---|---|---|
| Model | Self-hosted gateway (local) | Cloud service | Python SDK / proxy |
| Keys | Stay **local** (encrypted) | Held by the service | Your infra |
| Standout | Compression + many routing modes + free-tier aggregation | Huge hosted catalog, simple | Cost tracking, budgets, rate-limits, admin UI |
| API | OpenAI-compatible `/v1` | OpenAI-compatible | OpenAI-compatible |

- Choose **OmniRoute** for local/private, cost-cutting, resilient multi-provider access.
- **OpenRouter** for zero-ops hosted access; **LiteLLM** for a library/proxy with budgets & governance.

---

## Notes & Cautions

- It's a **proxy for your own provider keys** — you still need accounts/keys for paid providers; free tiers are aggregated but rate-limited.
- Verify provider **ToS** for gateway/aggregation use; keep the dashboard/API key protected (it fronts all your provider keys).
- Fast-moving project (fork lineage from 9router/CLIProxyAPI) — pin versions and re-check features/flags against current docs.

## Related
- [[LangChain]] — can target OmniRoute's OpenAI endpoint as its LLM backend
- [[Tools]] · [[AI]] — parent nodes
- Compare with **OpenRouter** / **LiteLLM** (to be added — see [[Roadmap]])

## References
- ai-tldr.dev/tools/omniroute · explainx.ai OmniRoute guide · omniroute.online (see chat for links)

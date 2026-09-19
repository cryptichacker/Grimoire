---
tags: [ai, skills, claude, knowledge-graph, ast, codebase]
type: cheatsheet
source: web research — github.com/Graphify-Labs/graphify
last-verified: 2026-09-16
---

# graphify

## Up
- [[Skills]]

**graphify** (Graphify-Labs) is a `/graphify` skill that turns **any folder** — code, docs, SQL schemas, configs, PDFs, images/video — into a **queryable knowledge graph**. Code is parsed **deterministically with tree-sitter (~37–40 languages, no LLM, nothing leaves your machine)**; docs/PDFs/media use your agent's model for semantic extraction. **No vector store** — it builds a real graph you *traverse*, and **every edge is tagged** so you know what was read vs. inferred. Handy for understanding a new codebase (or even mapping this vault) instead of grepping blindly.

> Dual-licensed **Apache-2.0 / MIT**. Works in Claude Code, Cursor, Codex, Gemini CLI, Copilot CLI, Aider + 15 others.

---

## How it works
- **Local AST parsing** — tree-sitter across ~37 grammars (Python, TS, Go, Rust, Java, C/C++…), deterministic, offline.
- **Semantic extraction** — Markdown/HTML/YAML/rST docs, SQL schemas, Terraform/MCP configs, package manifests, PDFs, images, audio/video (transcription) via your model.
- **Explained edges** — every connection carries a confidence tag: **`EXTRACTED`** (explicit source ref), **`INFERRED`** (derived), or **`AMBIGUOUS`**.
- **No embeddings** — you traverse an actual graph, not a similarity search.

---

## Inputs it handles

| Kind | Examples |
|---|---|
| **Code** | 37 tree-sitter grammars — Python, TypeScript, Go, Rust, Java, C/C++, … |
| **Docs** | Markdown, HTML, YAML, reStructuredText |
| **Data / infra** | SQL schemas, **Terraform** configs, **MCP** configs, package manifests |
| **Media** | PDFs, images, video/audio (transcribed) |

---

## Output → `graphify-out/`
1. **`graph.html`** — interactive force-directed viz, clickable nodes, community detection
2. **`GRAPH_REPORT.md`** — "god nodes", surprising connections, suggested questions
3. **`graph.json`** — full queryable graph

---

## Install (Claude Code)

```bash
uv tool install graphifyy
graphify install                 # global
graphify install --project       # project-scoped
```

Then in the agent:
```text
/graphify .                                   # build graph for current folder
/graphify query "what connects auth to the database?"
/graphify path "UserService" "DatabasePool"   # how are two nodes linked
/graphify explain "RateLimiter"               # what a node touches
```

---

## When to use
- **Onboarding to an unfamiliar codebase** — see structure + hotspots before diving in.
- Tracing **cross-layer links** (app code ↔ SQL schema ↔ [[Terraform]]/infra ↔ MCP config) in one graph.
- Mapping a **doc/PDF corpus** (or this knowledge base) into something you can query by relationship.

## Notes
- Deterministic + offline for code = trustworthy, no data exfil for the AST layer (the semantic pass on docs/media does use a model).
- Multiple community forks exist (`sharkkyyy10/graphify-`, `collabsoft/ai_graphify`, `aiminnovations/claude-graphify`…) — Graphify-Labs is the upstream; **[[nvidia-skillspector]]** a fork before installing.
- Complements a vault: pairs with the AST/query idea behind good docs — visual map + queryable edges instead of flat notes.

## Source
- github.com/Graphify-Labs/graphify

---
tags: [ai, skills, claude]
type: moc
---

# Skills

## Up
- [[AI]]

**Agent Skills** — packaged instructions/workflows that extend a coding agent (Claude Code, Cursor, Codex, etc.). A skill is typically a folder with a `SKILL.md` (or a `CLAUDE.md`) plus optional scripts/resources; the agent loads it when the task matches, changing how it plans, codes, designs, or reviews. Distributed as **plugins** via marketplaces (`/plugin marketplace add …` → `/plugin install …`) or dropped into a repo.

## Subtopics
- [[superpowers]] — agentic software-development methodology + composable skills (obra / Prime Radiant)
- [[ui-ux-pro-max]] — design-intelligence skill for professional UI/UX across many stacks
- [[andrej-karpathy-skills]] — a CLAUDE.md of coding guidelines derived from Karpathy's LLM-pitfall observations
- [[nvidia-skillspector]] — NVIDIA's **security scanner** for agent skills: scan any skill/MCP for prompt-injection, exfiltration & supply-chain risks *before* you install it
- [[anthropics-skills]] — Anthropic's **official** skills repo: document skills (docx/pptx/xlsx/pdf), `mcp-builder`, the Agent-Skills spec & template
- [[caveman]] — viral **token-compression** skill/proxy — terse "caveman-speak" to cut output/cost
- [[graphify]] — turn any code/docs folder into a **queryable knowledge graph** (local AST, no vector store, explained edges)

## Related
- [[Tools]] · [[Omniroute]] — AI tooling under [[AI]]
- Claude Agent Skills docs · plugin marketplaces (Cowork/Claude Code)

---

## How skills are installed (Claude Code)

```bash
# from a marketplace
/plugin marketplace add <owner>/<marketplace-repo>
/plugin install <skill>@<marketplace>

# per-project (drop-in)
# add a .claude/skills/<name>/SKILL.md  (or a CLAUDE.md at repo root)
```

- **Plugin/marketplace** — reusable across projects, versioned, shareable.
- **Per-project `CLAUDE.md` / `.claude/skills/`** — repo-local guidance committed with the code.
- A skill's **frontmatter `description`** decides when the agent auto-activates it; keep it specific.

> These three are third-party, community/independent projects — verify the source repo and review a skill's contents before installing (skills run with your agent's permissions).

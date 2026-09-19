---
tags: [ai, skills, claude, karpathy]
type: cheatsheet
source: web research — github (andrej-karpathy-skills, multiple forks)
last-verified: 2026-09-16
---

# andrej-karpathy-skills

## Up
- [[Skills]]

**andrej-karpathy-skills** is a set of **coding-agent guidelines** (a `CLAUDE.md`, also portable to `AGENTS.md`/Cursor rules) derived from **Andrej Karpathy's** observations of common LLM-coding pitfalls — models that make unfounded assumptions, overcomplicate solutions, and edit more than asked. It's guidance/behaviour tuning, not a tool.

---

## The four principles it encodes

1. **Think Before Coding** — surface ambiguities and **state assumptions explicitly** instead of silently picking an interpretation.
2. **Simplicity First** — write the **minimum viable code**; no speculative features, premature abstractions, or "flexibility" nobody asked for.
3. **Surgical Changes** — modify **only what's necessary**, match existing style, and don't "improve" adjacent code.
4. **Goal-Driven Execution** — turn the task into **verifiable success criteria** and loop until validated (independent verification).

The net effect: fewer hallucinated assumptions, less over-engineering, tighter diffs, and self-checking against a clear definition of done.

---

## Install

```bash
# Claude Code plugin (via a marketplace fork)
/plugin marketplace add forrestchang/andrej-karpathy-skills
/plugin install <plugin>@<marketplace>

# per-project (drop-in): fetch the CLAUDE.md into your repo
curl -o CLAUDE.md https://raw.githubusercontent.com/<owner>/andrej-karpathy-skills/main/CLAUDE.md
```

Also ships Cursor integration (a committed rule file) and works with any `AGENTS.md`/Agent-Skills-compatible agent (Codex, Gemini, Aider, Copilot, OpenCode…).

---

## When to use
- You want the agent to **stop over-engineering** and make **minimal, on-target changes**.
- Codebases where **tight diffs and explicit assumptions** matter (reviews, production repos).
- A lightweight, universal baseline you can commit per-project.

## Notes
- It's essentially a **CLAUDE.md behaviour file** — light, fast, and easy to read/tweak; no runtime.
- **Many community forks exist** (multica-ai, swarmclawai, LearnPrompt, skydr1ft, forrestchang…) with slightly different packaging/marketplaces and inflated star claims — pick one, read its `CLAUDE.md`, and adapt to your repo rather than trusting the branding.
- Composes well with heavier workflow skills like [[superpowers]] (methodology) — this one just keeps edits disciplined.

## Source
- github.com/forrestchang/andrej-karpathy-skills (and community forks: multica-ai, swarmclawai, LearnPrompt, skydr1ft)

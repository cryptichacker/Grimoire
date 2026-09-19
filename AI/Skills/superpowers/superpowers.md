---
tags: [ai, skills, claude, superpowers]
type: cheatsheet
source: web research — github.com/obra/superpowers
last-verified: 2026-09-16
---

# superpowers

## Up
- [[Skills]]

**Superpowers** (by Jesse Vincent / "obra" and Prime Radiant) is an **agentic software-development methodology** for coding agents, built on a set of **composable skills** plus bootstrap instructions. Instead of one-shot prompts, it drives structured, multi-phase development — design → plan → TDD implementation → debugging → review — and auto-triggers the right skill for each phase.

---

## What it provides

A library of composable skills grouped by phase:

| Area | Skills / behaviour |
|---|---|
| **Testing & Quality** | Test-Driven Development on a strict **RED → GREEN → REFACTOR** cycle |
| **Debugging** | Systematic **root-cause analysis** with explicit verification steps |
| **Collaboration** | Design refinement, detailed planning, code-review workflows, parallel/branched development |
| **Meta** | Skill creation, and an intro/onboarding skill |

Guiding principles: **Test-Driven Development · Systematic over ad-hoc · Complexity reduction · Evidence over claims.** The agent can run multi-hour cycles with minimal deviation from the approved plan.

---

## Install (Claude Code)

```bash
# official plugin marketplace
/plugin install superpowers@claude-plugins-official

# or the project's own marketplace
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

Related repos: **obra/superpowers** (framework), **obra/superpowers-skills** (community-editable skills), **obra/superpowers-marketplace** (plugin marketplace), **obra/superpowers-lab** (experimental).

---

## When to use
- You want the agent to follow a **disciplined engineering process** (spec → plan → tests-first → implement → verify) rather than jumping straight to code.
- Larger features where **planning, TDD, and systematic debugging** pay off.
- Teams standardising *how* their agents build software.

---

## Notes
- It's opinionated (TDD-first, plan-before-code) — great for engineering-grade work, heavier than you need for quick one-offs.
- Composability means you can adopt individual skills (e.g. just the debugging or code-review skill).
- Review the skills before enabling; they change the agent's default workflow significantly.

## Source
- github.com/obra/superpowers · github.com/obra/superpowers-skills · github.com/obra/superpowers-marketplace

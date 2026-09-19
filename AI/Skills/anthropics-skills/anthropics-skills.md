---
tags: [ai, skills, claude, anthropic, official, docx, pptx, xlsx, pdf, mcp]
type: cheatsheet
source: web research — github.com/anthropics/skills
last-verified: 2026-09-16
---

# anthropics/skills

## Up
- [[Skills]]

**anthropics/skills** is Anthropic's **official, public Agent-Skills repo** — the reference collection of skills plus the **Agent Skills spec** and a **skill template**. This is where the **document skills** that power Claude's file output live (`docx`, `pptx`, `xlsx`, `pdf`), alongside example creative/dev/enterprise skills and builders like `mcp-builder`. If you want one trustworthy, first-party source of skills for this vault, start here.

> ~169k★. Skills are **Apache-2.0**; the four **document skills are source-available** (not OSS) since they power Claude's own doc features.

---

## What's inside

| Folder | Contents |
|---|---|
| `skills/` | The actual skills, grouped: **Document**, Creative & Design, Development & Technical, Enterprise & Communication |
| `spec/` | The **Agent Skills specification** (how a `SKILL.md` is defined/loaded) |
| `template/` | Starter `SKILL.md` for authoring your own |
| `.claude-plugin/` | Plugin manifest so the repo installs as a marketplace |

**Highlight skills:**
- **Document skills** — `docx`, `pptx`, `xlsx`, `pdf`: build/edit real Office & PDF files. These are what turn vault notes into shippable **reports, decks, spreadsheets** → directly useful for [[Coding]] & docs work.
- **mcp-builder** — scaffolds a new **MCP server** (pairs with the MCP/agents track under [[AI]]).
- **artifacts / canvas-design** and various creative/enterprise examples.

---

## Install (Claude Code)

```bash
# add the official marketplace
/plugin marketplace add anthropics/skills

# install the sets you want
/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
```

Also available on **claude.ai** (upload/enable in the UI; some prebuilt on paid plans) and via the **Skills API** (`docs.claude.com/en/api/skills-guide`).

---

## Skill anatomy (the spec)

Every skill is a folder with a **`SKILL.md`** (optional scripts/resources beside it):

```markdown
---
name: my-skill-name
description: What it does AND when to use it — this line decides auto-activation
---

# My Skill Name
[instructions Claude follows when the skill is active]

## Examples
## Guidelines
```

- Required frontmatter: **`name`** (lowercase-hyphens) + **`description`**.
- The **`description`** is what makes the agent auto-trigger the skill — keep it specific.
- Drop-in alternative to a marketplace: put it at **`.claude/skills/<name>/SKILL.md`** in a repo.

---

## When to use
- You want **first-party, trustworthy** skills (vs. random third-party repos).
- Producing **Office/PDF deliverables** from notes/data (docx/pptx/xlsx/pdf).
- **Authoring your own** skill — use `spec/` + `template/` as the canonical reference.
- Building an **MCP server** with `mcp-builder`.

## Notes
- Even though it's official, still worth a quick **[[nvidia-skillspector]]** scan when pulling community forks or unfamiliar example skills.
- The document skills being *source-available* means: fine to use, read the license before redistributing.

## Source
- github.com/anthropics/skills · spec + template + `SKILL.md` format

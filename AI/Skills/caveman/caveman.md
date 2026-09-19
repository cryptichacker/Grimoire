---
tags: [ai, skills, claude, tokens, cost-optimization]
type: cheatsheet
source: web research — github.com/JuliusBrussee/caveman
last-verified: 2026-09-16
---

# caveman

## Up
- [[Skills]]

**caveman** (JuliusBrussee) is a viral **token-compression** system for coding agents — *"why use many token when few token do trick."* It makes the agent answer in terse **"caveman-speak"** (drop articles, contractions, filler) to cut output tokens, and ships an optional local **proxy** that compresses input/output before it hits the provider. Purely a **cost/efficiency** tool, not a workflow skill.

> Two products: the **skill** (MIT) = talk terse; the **proxy/engine** (BSL-1.1 → Apache-2.0 later) = compress payloads. Savings are mostly **`inferred`**, not verified billing.

---

## What it saves

| Mode | Claim | How |
|---|---|---|
| **Skill** | ~**65%** output tokens (representative tasks, *inferred*) | Sparse style — no articles/contractions/verbose explanation |
| **Proxy** | ~**33%** provider-reported **input** tokens (CC benchmark) | Per-type compression: JSON 70–90%, logs 85–95%, code 40–70%, recovery cache for originals |
| **Pixel mode** | varies | Renders dense text → PNG for vision-token efficiency |

**Style example:**
`"The reason your React component re-renders is likely because you create a new object reference each render cycle"` → `"New object ref each render. Inline object prop = new ref = re-render."`

---

## Install (Claude Code)

```bash
# full ecosystem (skill + proxy)
curl -fsSL https://raw.githubusercontent.com/JuliusBrussee/caveman/v2.2.0/install.sh | bash

# skill only
npx skills add JuliusBrussee/caveman

# proxy (npm)
npm install -g @caveman-ai/cli
caveman setup --install
caveman claude
```

Then in Claude Code:
```text
/caveman                 # toggle on
/caveman lite|full|ultra # intensity
```

---

## Caveats / tradeoffs
- The **skill itself adds ~1–1.5k input tokens/turn**, so whole-session savings are **below** the headline output %.
- On **already-terse** workloads net cost can go **negative** (you pay more).
- **Pixel mode** only wins on dense, long-line content; sparse code is *"not profitable."*
- All local numbers are **`inferred`** — only the Caveman Cloud backend reports `verified`.
- Terser output can lose nuance — fine for grinding tasks, **not** for careful reviews/explanations.

## When to use
- High-volume, **repetitive coding** where output verbosity (and cost) is the bottleneck.
- Piping big **JSON / logs** through an agent (proxy compression shines there).
- **Not** for careful design docs, security write-ups, or anything where clarity > tokens.

## Notes
- Fun + genuinely useful, but measure on **your** workload before trusting the %.
- Scan with **[[nvidia-skillspector]]** before installing — the `curl | bash` installer and a local proxy touch your env; know what you're running.
- Composes with disciplined-edit skills like [[andrej-karpathy-skills]] (tight output on both sides).

## Source
- github.com/JuliusBrussee/caveman (skill MIT · engine/proxy BSL-1.1)

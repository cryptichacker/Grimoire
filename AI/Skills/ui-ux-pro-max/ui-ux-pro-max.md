---
tags: [ai, skills, claude, ui-ux, design]
type: cheatsheet
source: web research — github.com/nextlevelbuilder/ui-ux-pro-max-skill
last-verified: 2026-09-16
---

# ui-ux-pro-max

## Up
- [[Skills]]

**UI UX Pro Max** (by nextlevelbuilder) is a **design-intelligence** skill: it analyses your project requirements and generates a complete, tailored **design system** (styles, colors, fonts, UX rules, chart choices) *before* code, so the agent builds professional UI/UX across many stacks. A reasoning-rules + searchable-library engine rather than a static template.

---

## What it provides

| Asset | Count / examples |
|---|---|
| **UI styles** | 79 searchable (≈50 active) — Glassmorphism, Claymorphism, Brutalism, … |
| **Reasoning rules** | 192 industry-specific (SaaS, healthcare, fintech, …) |
| **Color palettes** | 192, aligned to product type |
| **Font pairings** | 74 (Google Fonts) |
| **UX guidelines** | 119 — accessibility, text layout, anti-patterns |
| **Charts** | 25 types for dashboards/analytics |

**Stacks targeted (~22):** React, Next.js, Vue, Nuxt, Angular, Laravel, SwiftUI, Jetpack Compose, Flutter, React Native, HTML+Tailwind, plus desktop (JavaFX, WPF, WinUI 3).

---

## Install

```bash
# CLI (recommended by the project)
npm install -g ui-ux-pro-max-cli
uipro init --ai claude

# or as a Claude Code plugin
/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill
/plugin install ui-ux-pro-max@ui-ux-pro-max-skill
```

## Usage
Just ask naturally — e.g. *"Build a landing page for my SaaS product."* The skill activates, picks a style + palette + fonts + UX rules for your domain/stack, and produces design recommendations before implementation.

---

## When to use
- You want **coherent, professional design decisions** (style, color, type, layout, accessibility) instead of generic default UI.
- Front-end/app work where **design system consistency** matters across screens.
- Rapid prototyping that still respects UX best practices and anti-patterns.

## Notes
- It's a design-*decision* engine; pair it with your framework of choice for the actual build.
- Overlaps conceptually with Claude's own design/artifact tooling — useful when you want an opinionated, catalog-driven design system.

## Source
- github.com/nextlevelbuilder/ui-ux-pro-max-skill

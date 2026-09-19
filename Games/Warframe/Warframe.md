---
tags: [games, warframe]
type: guide
source: personal Notion — "🥷 Warframe" (Warframe Damage Calculation Guide)
last-verified: 2026-08-25
---

# Warframe

## Up
- [[Games]]

*Warframe Damage Calculation Guide — transcribed from my Notion.*

## Table of Contents
1. Core Damage Formula
2. Damage Types & Resistances
3. Damage Calculation Breakdown
4. Worked Examples
5. Modding Basics
6. Advanced Damage Optimization
7. Armor Strip
8. Reaching "Damage Cap"
9. Notes & Tips

---

## Core Damage Formula
Damage is a multiplicative chain. The "everything in one line" version:
```text
Final Hit = Base Damage
          × (1 + Σ Base-Damage Mods)         ← Serration, Hornet Strike, Pressure Point
          × (1 + Σ Elemental/Physical Mods)   ← applied to the relevant damage portions
          × Multishot                         ← extra instances, not a flat multiplier
          × Critical Multiplier
          × (1 + Faction Bonus)               ← Bane mods, Roar; applied at the moment of hit
          × Damage-Type Modifier              ← ×1.5 Vulnerable / ×0.5 Resistant
          × Armor Damage Reduction            ← (1 − 0.9·√(Armor/2700))
          × Body-Part Multiplier              ← headshots, weak points
          × Combo / Stealth (melee only)
```
The arsenal "total damage" number in the UI only covers the first three lines (base, elemental, multishot). Everything from crit onward happens live against a specific target — which is why real damage swings with headshots, status, and armor.

### Critical Multiplier
The **average** damage multiplier from critting:
```text
Avg Crit Multiplier = 1 + Critical Chance × (Critical Damage − 1)
```
Example: 50% CC, 2.0× CD → `1 + 0.5 × (2.0 − 1) = 1.5×` average.

**Crit tiers** (above 100% chance):
```text
Crit-tier multiplier = 1 + tier × (Critical Damage − 1)
```
- Yellow (tier 1, normal crit): `CD`
- Orange (tier 2, CC > 100%): `2·CD − 1`
- Red (tier 3, CC > 200%): `3·CD − 2`

With a 2.0× weapon: yellow = 2.0×, orange = 3.0×, red = 4.0×. At 150% CC you get a **guaranteed orange** plus a **50% chance** to roll the next tier, and the game averages between them.

### Elemental Combination Order
Elements combine in mod-slot order (left→right, top→bottom). Two mods of the same element add together first, then combine.

| Combination | Result |
|---|---|
| Heat + Cold | **Blast** |
| Heat + Toxin | **Gas** |
| Heat + Electricity | **Radiation** |
| Cold + Toxin | **Viral** |
| Cold + Electricity | **Magnetic** |
| Toxin + Electricity | **Corrosive** |

---

## Damage Types & Resistances

### The three defense layers
| Layer | Behavior |
|---|---|
| **Shields** | Absorbed first. Ignored by Toxin (bypasses to Health on most enemies). Not reduced by Armor. Shows in blue. |
| **Armor** | Reduces incoming Health damage via the √ formula. Strippable. |
| **Health** | The actual kill bar underneath. |

### Physical (IPS) damage
- **Impact** — strong vs Grineer (Vulnerable). Also staggers.
- **Puncture** — strong vs Corpus (Vulnerable). Reduces enemy damage output on proc.
- **Slash** — causes Bleed: a true-damage DoT that **bypasses armor entirely**, dealing **35% of base damage per tick** over ~6 seconds. The backbone of armored-target killing.

### Base elements
- **Heat** — strips up to **50% armor** over a few ticks; panics enemies; DoT.
- **Cold** — slows; boosts your own crit damage against the target while applied; pairs into Viral.
- **Toxin** — bypasses shields; strong DoT.
- **Electricity** — chains/stuns; AoE DoT.

### Combined elements
- **Viral** — multiplies damage *to Health*. 1st proc `+100%` (×2), then `+25%` per stack up to 10 → **+325% = ×4.25**. The single most important multiplier for killing health bars.
- **Corrosive** — strips armor: **26% on the first proc, +6% per additional stack, capped at 80%** (10 stacks; each lasts 8s). Most Grineer are also Vulnerable to it.
- **Magnetic** — multiplies damage to Shields (great vs Corpus).
- **Gas** — AoE cloud DoT, capped at 10 stacks. Best as a status *primer* for Condition Overload.
- **Radiation** — Confusion (enemies attack each other); good crowd control and strong vs some bosses.
- **Blast** — reduces enemy accuracy and adds AoE/stagger utility.

### Faction quick-reference (Vulnerable = ×1.5, Resistant = ×0.5)
Resistances are per-faction and readable in the Codex. Rules of thumb:
- **Grineer:** lean Impact + Viral + Heat/Slash. Corrosive still useful for the strip + bonus.
- **Corpus:** lean Puncture + Magnetic (shields), then Toxin/Viral for health.
- **Infested:** Heat and Gas excel; Slash is reliable.
- **Narmer / Archon bosses:** Vulnerable to **Slash and Toxin**, Resistant to **Magnetic**.

You build for *who you're shooting*, not for an armor subtype — vulnerabilities live on the faction.

---

## Damage Calculation Breakdown
1. **Base & quantization.** The game divides each damage type's value into 1/16ths of total base damage and rounds — so mixed-IPS weapons gain or lose a little when modded. It's why arsenal numbers occasionally look off by one.
2. **Base-damage mods** (Serration etc.) are summed, then applied: `Base × (1 + ΣmodPct)`.
3. **Elemental/physical mods** apply to their portions, are also quantized, and added to the pool.
4. **Multishot** spawns additional independent instances. 180% multishot = 1 guaranteed pellet + an 80% chance of a second; each instance crits and procs status on its own.
5. **Critical hits** use the tier math above.
6. **Status effects.** Per-instance status chance equals the modded status %. Each element's *share* of total damage determines which status it's likely to roll. Viral multiplies health damage; Slash bleeds bypass armor.
7. **Faction bonus** is applied multiplicatively *at the moment of hit* (it needs the target's faction). It's also folded into status-DoT damage — effectively twice in the DoT calc.
8. **Enemy defenses.** Shields first (Toxin skips them); then Armor reduces Health damage by `0.9·√(Armor/2700)`; then the faction damage-type modifier (`×1.5`/`×0.5`).

### The armor formula in detail
```text
Damage Reduction = 90% × √(Net Armor / 2700)
Effective Damage Multiplier = 1 − Damage Reduction
```

| Net Armor | Damage Reduction | You deal |
|---|---|---|
| 300 | ~30% | 70% |
| 1,200 | ~60% | 40% |
| 2,700 | 90% | 10% |

Each damage type always deals a **minimum of 1** against armor, and enemy armor scaling is capped — which is why **partial** armor strip (e.g. Heat's flat 50%) is often enough rather than needing a full strip.

---

## Worked Examples

### Example A — a single rifle shot, step by step
Weapon: 100 base damage, 25% crit chance, 2.0× crit damage, on a Grineer with 1,200 armor (neutral type for simplicity).
- Serration (+165%): `100 × (1 + 1.65) = 265`
- Two 90% elementals merged to Corrosive add to the pool → assume total modded hit **≈ 742**.
- Multishot 90% (Split Chamber): average instances `1.9` → `742 × 1.9 ≈ 1,410` per trigger pull.
- Average crit: `1 + 0.25 × (2.0 − 1) = 1.25×` → `1,410 × 1.25 ≈ 1,763`.
- Armor (1,200): DR `= 0.9 × √(1200/2700) = 0.9 × 0.667 = 60%` → `× 0.40` → **≈ 705 to health**.
- Add a Primed Bane (×1.55 vs Grineer): `705 × 1.55 ≈ 1,093`.

Armor ate more than half the hit; one faction mod added ~55% back. Strip the armor and that 705 becomes ~1,763 before the faction mod even applies.

### Example B — why Viral + Slash beats raw damage
- **Pure base damage:** big yellow numbers, but the armor √-curve and the kill bar shrink your effective output hard at level.
- **Viral + Slash:** 10 Viral stacks = **×4.25 to health**, and Slash Bleed is **true damage that ignores armor entirely**. The bleed ticks (35% of a large modded base, multiplied by Viral) often out-damage your direct hits and don't care about the armor formula. The meta favors *multipliers and armor-bypass* over flat damage.

### Example C — faction mod stacking (multiplicative)
Serration + Primed Bane of Grineer + Roar, 100-damage weapon, vs Grineer:
`100 × (1 + 1.65) × (1 + 0.5 + 0.55) = 100 × 2.65 × 2.05 ≈ 543`.
Faction/ability bonuses stack *additively with each other* but *multiplicatively with your damage mods* — which is why adding a faction mod to an already-modded gun feels so strong.

---

## Modding Basics

### Primary / Secondary
- **Base Damage:** Serration +165% (rifle) · Hornet Strike +220% (pistol) · Point Blank +90% (shotgun, Primed +165%)
- **Multishot:** Split Chamber +90% · Barrel Diffusion +120% (secondary) · Hell's Chamber +120% (shotgun)
- **Crit Chance:** Point Strike +150% · Pistol Gambit +120%
- **Crit Damage:** Vital Sense +120% · Target Cracker +60% (Primed +110%)
- **Elementals:** 60% (dual-stat, adds +60% status) or 90% (pure)
- **Fire Rate:** Speed Trigger, Lethal Torrent (also +multishot)

### Melee
- **Base Damage:** Pressure Point +120% · Primed Pressure Point +165% · Sacrificial Pressure +110%
- **Crit Chance:** True Steel +120% · **Blood Rush** (scales crit with combo counter)
- **Crit Damage:** Organ Shatter +90%
- **Attack Speed:** Fury / Primed Fury
- **Combo Duration:** Drifting Contact, Body Count
- **Condition Overload:** +80% melee damage *per unique status type* on the target (up to 16 status types can stack).

### Standard build template
1. Base-damage mod (almost always)
2. Multishot (guns) / attack speed (melee)
3. Crit chance + crit damage **if** base crit is ~15%+ (20%+ to really lean in)
4. A two-mod element (usually Viral) — often dual-stat 60/60s for status
5. Utility: faction mod, fire rate, Hunter Munitions, Galvanized mods

---

## Advanced Damage Optimization

### Spread your multipliers
Damage is multiplicative *across categories* but additive *within* one. `100% base + 100% elemental` beats `200% base` because the bonuses live in different multiplied buckets. Balance base / elemental / crit / multishot rather than stacking one.

### Viral + Slash / Viral + Heat meta
The endgame staple for armored content:
- Viral for the ×4.25 health multiplier.
- Slash Bleed (true damage) **or** Heat (DoT + 50% armor strip) to handle armor.
- **Hunter Munitions** forces Slash procs on crits — pair it with high-crit weapons so most hits bleed.

### Critical scaling
- 20%+ base crit chance to make a crit build worthwhile.
- Stack CC × CD; push past 100% for guaranteed orange crits, past 200% for reds.
- **Blood Rush** (melee) scales crit chance with the combo counter for exponential melee crits.
- Arcanes like Arcane Avenger add flat crit chance (+45%) to tip you over a tier breakpoint.

### Galvanized mods (Primary/Secondary)
Stack-on-kill mods that replace your base damage/utility slot:
- **Galvanized Chamber** (rifle multishot): +80% multishot; on kill +30% multishot per stack, up to **5 stacks → +230%** total.
- **Galvanized Aptitude** (rifle status/damage): +80% status chance; on kill, **+40% direct-hit damage per status type** on the target, up to 2 stacks. The bonus applies on the hit *after* the one that procs the status ("gun Condition Overload").
- **Galvanized Scope** (rifle crit *chance*): on headshot, +120% crit chance while aiming; on headshot kill, +40% crit chance per stack, up to 5 stacks.
- Shotgun/secondary equivalents: Galvanized Savvy / Hell / Shot / Crosshairs / Diffusion, etc.

### Faction mods
- Regular Bane mods: +30% (×1.30). Primed: +55% (×1.55).
- Factions: Bane of Grineer / Corpus / Infested / Orokin / Murmur (melee also has Sentient).
- Applied multiplicatively at the hit, and folded into status-DoT damage too.

---

## Armor Strip
With the √ armor curve and the enemy-armor scaling cap, you rarely need a *full* strip. Taking an enemy from 90% DR down to ~50% DR roughly **quintuples** your effective damage, and partial strips reach that fast.

Options, fastest/most reliable first:
- **Heat procs:** flat **50%** strip, no stacking required, ramps over a few ticks. The easiest "always on" partial strip.
- **Corrosive procs:** **26% + 6%/stack to 80%** cap (10 stacks). Strong but needs status-heavy weapons; Emerald Archon Shards or Corrosive Projection push past the cap.
- **Heat + Corrosive together** (multiplicative): an active Heat proc + 10 Corrosive stacks ≈ `(1−0.5) × (1−0.80) = 10%` armor remaining — effectively a near-full strip.
- **Warframe abilities:** full strip from Mag (Pillage), Hildryn (Pillage), Nyx, Nezha, Frost (subsume), etc.; partials from Heat/Corrosive-based abilities.

Full strip = your damage skips the entire armor multiplier, a multi-fold increase on high-armor targets.

---

## Reaching "Damage Cap"
The hard ceiling is the **32-bit signed integer limit, 2,147,483,647**. In practice it's academic — visual damage numbers abbreviate large values, and only extreme stacked setups approach it.

### What a maximization stack looks like
```text
Final = Base × Mods × Crit × Faction × Warframe Buffs × Combo × Stealth × (armor stripped)
```
Layer, roughly multiplicatively:
- ~300% modded weapon damage (mods + Riven)
- Galvanized mods at max stacks
- Crit build pushed to red-crit tier (3–4× and up)
- Viral ×4.25 (+ armor fully stripped so nothing is lost to DR)
- A buff frame: Rhino **Roar**, Mirage **Eclipse**, Chroma **Vex Armor**, Void/Xata's Whisper
- Melee only: combo multiplier (up to 12×), Heavy Attack and stealth (8×) / finisher multipliers

### Practical "very high but consistent" setup
1. **Viral + Heat** (or Viral + Slash via Hunter Munitions) — ×4.25 plus armor handled.
2. **Crit weapon at 100%+ crit chance** for guaranteed orange/red crits.
3. **One buff frame** (Roar / Eclipse / Vex Armor) — easy 2–4× on top.
4. **Galvanized mods** at max stacks (requires kills to ramp).
5. **An arcane** (Primary/Secondary Merciless +360% at max, or Deadhead on headshot kills).
6. **A faction mod** (Primed Bane, +55%, always on).

### Max-damage checklist
- [ ] 20%+ base crit weapon
- [ ] Full crit build (chance + damage), aiming for orange/red tier
- [ ] Viral + an armor-handling element (Slash via Hunter Munitions, or Heat)
- [ ] Galvanized mods equipped (and a way to keep stacks)
- [ ] Arcane Merciless / Deadhead
- [ ] Primed faction mod for the enemy you're fighting
- [ ] Riven with multistat damage/crit/multishot
- [ ] A damage-buff frame (Rhino / Chroma / Mirage)
- [ ] An armor-strip source (Heat proc minimum) for armored factions
- [ ] Practice headshots for the body-part multiplier and headshot arcanes

---

## Notes & Tips
- **Combo counter (melee):** build and hold combo for Blood Rush crit scaling and Heavy Attack burst.
- **Weak-point multipliers:** headshots are typically 2×–4× depending on weapon, applied late in the chain — they multiply everything before them.
- **Status stacking → Condition Overload:** more unique status types on a target = more +80% melee stacks. Gas, dual-stat 60/60 mods, and primers feed this.
- **Visual cap vs real damage:** large numbers are abbreviated on screen; the underlying value keeps scaling.
- **Don't over-build for the content:** one-shotting level-40s is wasted modding. Build for the level you actually run.
- **Test in the Simulacrum:** spawn level-9999 enemies, and use the Codex to read each faction's Vulnerabilities/Resistances.

> **Balance is key.** A hyper-specialized stack melts one faction and fumbles the next. Read the faction's resistance table, bring the right element and the right strip, and let the multipliers do the work.

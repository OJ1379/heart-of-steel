# Heart of Steel (H.O.S)

## What This Is

A complete 2D fighting game delivered as a single self-contained HTML5 file — no external dependencies, no server required, opens directly in any modern browser. Features 5 playable political caricature fighters, 4 destructible-wall stages, and full arcade-style gameplay inspired by Street Fighter II (SNES/Genesis era) with 32-bit pixel art rendered entirely in canvas code.

## Core Value

A fully playable, fun fighting game that runs from a single HTML file — controls must feel responsive, characters must feel distinct, and every fight must have personality.

## Requirements

### Validated

(None yet — ship to validate)

### Active

**Engine & Rendering**
- [ ] Single self-contained HTML5 file — all JS, CSS, canvas art inline, zero external dependencies
- [ ] 32-bit pixel art aesthetic rendered in code (canvas pixel art, no image files)
- [ ] 60fps game loop with responsive keyboard input
- [ ] Canvas-based renderer for all sprites, backgrounds, and UI

**Roster — 5 Fighters**
- [ ] Donald Trump — brash brawler, wide power swings, "Art of the Deal" ultimate (golden wall barrels across screen)
- [ ] Joe Biden — tanky grappler with reach, "Ice Cream Avalanche" ultimate (wave of ice cream freezes opponent)
- [ ] Barack Obama — balanced technical fighter, "Drone Commander" ultimate (targeting reticle → missile volley)
- [ ] George W. Bush — unorthodox scrappy timing, "Now Watch This Drive" ultimate (flaming golf ball ricochets)
- [ ] Bill Clinton — evasive counter-fighter, "Executive Charm" ultimate (sax solo stuns → combo finisher)
- [ ] Each fighter: unique speed/power/reach stats, idle/walk/jump/crouch/fast-attack/power-attack/block/hit-stun/KO/ultimate animations

**Stages — 4 with Wall Breaks**
- [ ] The White House (Oval Office → Rose Garden transition)
- [ ] Greek Island (cliffside → beach tier transition)
- [ ] Mar-a-Lago Club (ballroom → patio/pool transition)
- [ ] Golf Course (fairway → sand bunker transition)
- [ ] Wall break triggered by power attack near boundary; cinematic slow-mo zoom on impact; bonus damage

**Combat Mechanics**
- [ ] Fast Attack (tap J/Z): quick jab/kick, chainable 3–4 hit combos
- [ ] Power Attack (hold J/Z): heavy hit with wind-up, wall-break capable, launches for juggle
- [ ] Block (hold K/X): damage reduction, guard meter drains — guard break on depletion
- [ ] Juggle system: power attacks launch airborne; 2–4 hit air combos with gravity/diminishing returns
- [ ] Ultimate Ability (L/C): meter fills from dealing/taking damage; screen flash + unique character animation
- [ ] Movement: walk forward/back (WASD or arrows), jump (up), crouch (down)

**AI Opponent**
- [ ] 3 difficulty tiers: Easy / Medium / Hard (selectable from menu)
- [ ] AI blocks, combos, moves, and uses ultimate when meter full
- [ ] Harder tiers react faster and combo more reliably

**Game Flow**
- [ ] Title screen with pixel art logo and "PRESS START"
- [ ] Character select screen (player picks, CPU picks randomly)
- [ ] Stage select screen (4 stages)
- [ ] Best-of-3 rounds per match; round splash screens ("ROUND 1", "FIGHT!", "K.O.")
- [ ] Victory screen with winner pose animation
- [ ] Health bars, round counter, timer, ultimate meter — styled like 90s arcade cabinet UI

### Out of Scope

- Multiplayer/networking — single player vs CPU only (complexity vs. playability tradeoff)
- External assets (images, audio files, fonts) — must be fully self-contained
- Mobile/touch controls — keyboard-only per spec
- Save states / persistent progress — session-only
- More than 5 fighters or 4 stages in v1 — scope management

## Context

- Target: modern desktop browsers (Chrome, Firefox, Edge, Safari) running from local filesystem (`file://`)
- All art must be procedural/canvas-drawn — no base64 images, no external sprites
- Pixel art style: chunky silhouettes, limited palette, 32-bit SNES/Genesis era feel
- Each fighter's ultimate is a political caricature of a real public figure — keep it satirical, not malicious
- The entire game output is one `.html` file the user can double-click

## Constraints

- **Format**: Single `.html` file — no build step, no dependencies, no CDN links
- **Rendering**: Canvas API only for game graphics — CSS only for outer shell/UI chrome
- **Input**: Keyboard only (WASD/arrows + J/Z/K/X/L/C)
- **Compatibility**: Must run from `file://` without CORS issues (no fetch, no workers that require origin)
- **Assets**: Zero external assets — all art generated procedurally in canvas code

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Single HTML file | Core requirement — portability and shareability | — Pending |
| Canvas pixel art (no images) | Zero external deps requirement | — Pending |
| 5 fighters matching spec | Sufficient roster variety; manageable scope | — Pending |
| Best-of-3 round structure | Standard fighting game convention | — Pending |
| Keyboard-only controls | Spec requirement; simplifies input handling | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd:transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-03-28 after initialization*

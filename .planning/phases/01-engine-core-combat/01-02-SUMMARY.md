---
phase: 01-engine-core-combat
plan: 02
subsystem: fighter-system-combat
tags: [fighter, state-machine, combat, sprites, hitbox, hit-stop, combo, cpu-ai, hud]
dependency_graph:
  requires: [01-01]
  provides: [fighter-definitions, combat-system, trump-sprites, cpu-ai, hud-rendering]
  affects: [index.html]
tech_stack:
  added: []
  patterns:
    - Hierarchical FSM with enter/update/exit per state
    - Frame-data-driven attack system (startup/active/recovery)
    - AABB hitbox/hurtbox collision gated on active phase
    - Hit stop freeze pattern (global hitStopFramesRemaining)
    - Offscreen canvas sprite cache (initSpriteCache) with drawImage blitting
    - Data-driven fighter definition objects (TRUMP, PLACEHOLDER_CPU)
    - Per-fighter charge detection for fast vs power attack discrimination
key_files:
  created: []
  modified:
    - path: index.html
      role: Complete playable combat — fighters, sprites, state machines, combat system, HUD
decisions:
  - "Per-fighter _chargeFrames tracks attack hold independently of the global attackHeldFrames (which remains for title-screen/general input tracking)"
  - "Trump sprite uses parametric drawTrumpBase function with armState/legState/offsetY/crouchFactor to generate all 11 animation states efficiently"
  - "CPU sprite uses drawCPUBase with gray palette (#666666/#888888/#444444) — same silhouette as Trump, clearly distinct"
  - "Combo chain uses input.attackBuffered (6-frame buffer) during fastAttack recovery phase — feels responsive without being too easy"
  - "Power attack triggers on keyup after >= 12 frames held (CHARGE_THRESHOLD) — prevents fast attack delay per Pitfall 6 in RESEARCH.md"
  - "CPU blocks only when player is actively attacking with 0.003/tick probability (~20% coverage per D-04)"
  - "flashFrames added to fighter runtime for 2-frame white hit flash — uses source-atop compositing on the sprite"
  - "resolveFighterOverlap prevents fighters from walking through each other (minDist = sum of hurtbox half-widths - 4)"
metrics:
  duration: ~65 min
  completed: "2026-03-28"
  tasks_completed: 2
  files_modified: 1
---

# Phase 01 Plan 02: Fighter System and Combat Mechanics Summary

**One-liner:** Frame-data-driven Trump fighter with hierarchical FSM, AABB hit detection, hit stop, combo chains, CPU AI, and 90s-style HUD — fully playable combat in a single HTML file.

## What Was Built

### Task 1: Fighter Definitions, Sprite Drawing, and State Machine

Added the complete fighter system to `index.html`:

**Fighter definitions:**
- `TRUMP` object with `name`, `spriteWidth/Height`, `stats` (walkSpeed: 1.8, jumpForce: -7.5, gravity: 0.45, health: 1000, power: 1.3, reach: 1.15, hurtboxWidth: 36, hurtboxHeight: 48, crouchHurtboxHeight: 30), `attacks.fast` (startup: 4, active: 4, recovery: 8, damage: 60, maxChain: 4), `attacks.power` (startup: 12, active: 5, recovery: 18, damage: 150), and `sprites` with 11 states x 4 frames each
- `PLACEHOLDER_CPU` with walkSpeed: 0.9 (50% of Trump), power: 1.0, reach: 1.0, gray palette (#666666/#888888/#444444)

**Sprite drawing system:**
- `drawTrumpBase(ctx, offsetY, crouchFactor, armState, legState)` — parametric sprite generator using fillRect only. Trump palette: hair #FFD700/#DAA520, skin #FFB366/#E8944D, suit #1A1A4E/#0D0D2B, tie #CC0000/#990000
- `drawCPUBase(ctx, ...)` — gray palette recolor of same silhouette
- `makeTrumpFrames/makeCPUFrames` — closure factories that bind params and return draw functions for sprite cache

**Fighter runtime factory:** `createFighter(def, x, facing)` returns runtime object with all required fields: `def`, `x`, `y`, `vx`, `vy`, `facing`, `state`, `prevState`, `stateFrame`, `currentFrame`, `attackPhase`, `currentAttackType`, `comboCount`, `health`, `isBlocking`, `hitThisAttack`, `hitstunRemaining`, `knockedDown`, `flashFrames`, `_chargeFrames`, `_prevAttack`

**Hierarchical fighter state machine:** `FIGHTER_STATES` object with 11 state handlers (idle, walkForward, walkBack, jumpRise, jumpFall, crouch, fastAttack, powerAttack, block, hitstun, knockdown). Each handler has `enter`, `update`, `exit`. `transitionFighterState` manages enter/exit lifecycle.

**`updateFighter(fighter, input, opponent)`** applies superstate logic: hitstun check, state dispatch, auto-facing, airborne gravity.

**CPU AI:** `getCPUInput(cpu, player)` moves toward player when dist > 60, blocks at 0.003/tick probability when player is attacking.

**`resolveFighterOverlap(f1, f2)`** prevents hurtbox overlap with equal push-back.

**Fight scene** wired up: `fightScene.enter()` creates fighters, calls `initSpriteCache([TRUMP, PLACEHOLDER_CPU])`, resets round state.

### Task 2: Hit Detection, Damage, Hit Stop, and Combo Chain System

**`checkHit(attacker, defender)`:** Returns false if `attackPhase !== 'active'` or `hitThisAttack` is true. Calculates hitbox in world space using `attacker.def.stats.reach` multiplier and facing direction. Calculates defender hurtbox (crouch-aware using `crouchHurtboxHeight`). Returns `aabbOverlap(hitbox, hurtbox)`.

**`resolveHit(attacker, defender)`:** Sets `hitThisAttack = true`. Calculates `baseDamage = Math.floor(atk.damage * attacker.def.stats.power)`. Applies 80% block reduction (`* 0.2`). Applies `applyHitStop(5)` for power hits, `applyHitStop(3)` for normal hits. Triggers defender `flashFrames = 2` for white hit flash. Transitions defender to hitstun (with knockback) or knockdown if health <= 0. Block gives smaller pushback (3px vs 8px).

**Hit stop system:** Global `hitStopFramesRemaining`. In fight scene update: decrements when > 0 and skips fighter state machine updates entirely (input still read for responsiveness).

**Combo chain:** During `fastAttack` recovery, `input.attackBuffered` is checked against `comboCount < maxChain`. Chains return to `fastAttack` state (enter re-increments comboCount).

**KO detection:** After collision checks, `player.health <= 0` or `cpu.health <= 0` sets `matchOver = true` and records `winner`. Timer expiry also ends the round (higher health wins).

**HUD:** `renderHUD(ctx, player, cpu, roundTimer)` draws health bars with green/yellow/red color thresholds, player names, and 99-second countdown timer.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 2 - Missing Functionality] Per-fighter charge detection**
- **Found during:** Task 1 implementation
- **Issue:** The plan's state machine used `updateChargeDetection` (global function referencing global `attackHeldFrames`/`prevAttack`). This only works for one player. CPU AI would interfere.
- **Fix:** Added `_chargeFrames` and `_prevAttack` fields to the fighter runtime object. Created `updateFighterCharge(fighter, input)` that reads/writes fighter-local state. The global `updateChargeDetection` is retained for backward compatibility but not used in combat.

**2. [Rule 2 - Missing Functionality] Hit flash visual feedback**
- **Found during:** Task 2 implementation
- **Issue:** Plan mentioned "brief white flash" as optional polish. Added as Rule 2 because visual hit feedback is critical for combat feel (otherwise hits feel invisible).
- **Fix:** Added `flashFrames` field to fighter runtime. `resolveHit` sets `flashFrames = 2`. `renderFighter` uses `source-atop` compositing to overlay white for 2 frames.
- **Files modified:** index.html

**3. [Rule 2 - Missing Functionality] Round timer and HUD**
- **Found during:** Task 2 (KO detection required a timer for draws)
- **Issue:** Plan specified KO detection and victory transition but the HUD/timer integration was partially in scope (FLW-04 requirement). Added full HUD rendering and 99-second round timer to complete the functional gameplay loop.
- **Fix:** `renderHUD` draws P1/P2 health bars with color transitions, names, and countdown timer. Timer expiry resolves to winner-by-health.

## Known Stubs

None. All data is wired to live fighter state. Health bars read `fighter.health`, sprites read from sprite cache populated at fight scene enter. No placeholder values flow to UI.

## Acceptance Criteria Verification

- TRUMP object with all required stats: PASS
- TRUMP.attacks.fast: startup 4, active 4, recovery 8, damage 60, maxChain 4: PASS
- TRUMP.attacks.power: startup 12, active 5, recovery 18, damage 150: PASS
- PLACEHOLDER_CPU with walkSpeed 0.9 (~50% of Trump's 1.8): PASS
- createFighter returns runtime object with all required fields: PASS
- FIGHTER_STATES with 11 handlers (idle through knockdown): PASS
- Each state has enter/update/exit: PASS
- Engine uses fighter.def.stats.* (never hardcoded "TRUMP" in gameplay): PASS
- getCPUInput implements D-04 behavior: PASS
- hitStopFramesRemaining + skip logic: PASS
- Trump sprite colors #FFD700, #1A1A4E, #FFB366, #CC0000: PASS
- renderFighter uses drawImage + Math.floor: PASS
- initSpriteCache called in fight scene enter: PASS
- checkHit returns false when attackPhase !== 'active': PASS
- resolveHit with 0.2 block multiplier: PASS
- applyHitStop(3) for normal, applyHitStop(5) for power: PASS
- comboCount incremented on chain, checked against maxChain: PASS
- hitThisAttack prevents multi-hit: PASS
- KO detection on health <= 0: PASS
- Hurtbox height changes in crouch state: PASS

## Self-Check

**File exists:** C:/Users/OJC02/index.html — FOUND
**JS syntax valid:** Confirmed via `new Function()` parse — VALID
**File size:** 61 KB, 1966 lines
**All verification checks:** PASS (25/25 Task 1 checks, 14/14 Task 2 checks, 15/15 specific checks)

## Self-Check: PASSED

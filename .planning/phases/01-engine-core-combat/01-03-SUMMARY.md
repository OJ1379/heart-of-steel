---
phase: 01-engine-core-combat
plan: 03
subsystem: hud-round-system
status: complete
tags: [hud, health-bars, round-system, ko-splash, victory-screen, game-loop]
dependency_graph:
  requires: [01-02]
  provides: [hud-rendering, round-flow, ko-splash, victory-screen, complete-game-loop]
  affects: [fight-scene, victory-scene]
tech_stack:
  added: []
  patterns: [canvas-fillRect-hud, round-pip-rendering, ko-splash-flash, best-of-3-round-system, round-reset-helper]
key_files:
  created: []
  modified: [index.html]
decisions:
  - "drawHUD is canonical; renderHUD delegates to it for backward compatibility with existing fightScene.render call"
  - "koSplashTimer=90 freezes all updates and flashes K.O.! text white/red every 5 frames before tallying round win"
  - "Victory scene resets all match state on return to title (p1Wins, p2Wins, currentRound, koTriggered)"
  - "resetRound() extracted as helper function for clarity over inline reset logic"
  - "K.O. text flashes white/red every 5 frames during 90-frame splash window"
metrics:
  duration: ~40 min
  completed: 2026-03-28
  tasks_completed: 2
  files_modified: 1
---

# Phase 1 Plan 3: HUD and Game Flow Summary

**One-liner:** Complete arcade-style HUD (health bars with green/yellow/red thresholds, round pips, countdown timer) plus best-of-3 round flow with 90-frame KO splash and victory screen — closing the full game loop from title through match completion back to title.

## What Was Built

### New Functions

**`drawHealthBar(ctx, x, y, w, h, current, max, isPlayer)`**
- 1px black border, `#333333` empty background
- Fill color: `#00CC00` (>50%), `#FFCC00` (25-50%), `#CC0000` (<25%)
- Player fills left-to-right; CPU fills right-to-left
- `rgba(255,255,255,0.3)` highlight on top edge of active fill

**`drawRoundPips(ctx, x, y, winsCount)`**
- Two 4x4 pixel squares per fighter
- `#FFCC00` (gold) for rounds won, `#555555` (gray) for empty slots
- P1 pips at canvas position (120, 20), P2 pips at (268, 20)

**`drawHUD(ctx, player, cpu, roundTimer, p1Wins, p2Wins)`**
- P1 name at (16, 8) left-aligned; P1 health bar at (48, 8, 128, 8)
- "TIME" label and countdown centered at x=200; timer turns `#CC0000` under 600 ticks (10 seconds)
- P2 health bar at (224, 8, 128, 8); P2 name at (360, 8) right-aligned
- Both round pip sets drawn below their respective health bars

**`resetRound()`**
- Resets both fighters to full health and starting positions (player x=100, cpu x=300)
- Resets roundTimer=3600, koTriggered=false, koSplashTimer=0, roundWinner=null

### New Round State Variables

```javascript
let roundTimer = 3600;   // 60 seconds * 60 ticks
let p1Wins = 0;
let p2Wins = 0;
let currentRound = 1;
let koSplashTimer = 0;
let koTriggered = false;
let roundWinner = null;  // 'player' or 'cpu'
```

### Round Flow Logic

- Health <= 0 or timer <= 0 sets koTriggered=true, koSplashTimer=90, roundWinner determined by health percentage on timeout
- While koSplashTimer > 0: all fight updates frozen, timer counts down each tick
- When koSplashTimer reaches 0: tally win (p1Wins or p2Wins++), check for match end at 2 wins
- Match end: matchOver=true, transition to victory scene; round continuation: resetRound() called

### KO Splash

- Semi-transparent overlay drawn during koSplashTimer > 0
- "K.O.!" text alternates `#FFFFFF` and `#CC0000` every 5 frames

### Victory Scene

- Dark background `#0A0A1A`
- Winner name "TRUMP WINS!" or "CPU WINS!" at y=100, 16px centered
- "PRESS ENTER TO CONTINUE" blinks at y=190 after 60-frame delay
- Enter key resets all match state (p1Wins, p2Wins, currentRound, koTriggered, matchOver) and transitions to title

## Automated Verification

All 19 automated checks passed:

```
ALL CHECKS PASS
```

Verified: drawHealthBar, drawRoundPips, drawHUD, roundTimer, p1Wins, p2Wins, koSplashTimer, koTriggered, roundWinner, #00CC00, #FFCC00, #CC0000, #333333, #555555, K.O., TIME, PRESS ENTER TO CONTINUE, currentRound, Math.ceil

## Human Verification (Task 2 — Approved)

User confirmed all 14 verification steps pass:

1. Title screen with blinking "PRESS ENTER TO START" — approved
2. Fight scene with Oval Office background, HUD visible — approved
3. Trump movement (walk, jump, crouch) responsive — approved
4. Fast attack chains (3-4 hit combos), hit stop visible — approved
5. Power attack (hold then release) with heavier feel — approved
6. Block mechanic reducing damage — approved
7. CPU walks toward player and occasionally blocks — approved
8. HUD health bars deplete and change color (green/yellow/red) — approved
9. Timer counts down; turns red under 10 seconds — approved
10. KO splash: "K.O.!" flashes for ~1.5 seconds — approved
11. Round pip updates and fighters reset for round 2 — approved
12. Match end after 2 wins; victory screen shows winner name — approved
13. "PRESS ENTER TO CONTINUE" prompt returns to title — approved
14. 60fps performance, crisp pixel art throughout — approved

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Refactor] `renderHUD` retained as thin wrapper over `drawHUD`**
- Found during: Task 1
- Issue: Existing fight scene render path called `renderHUD`; plan specified `drawHUD` as canonical name
- Fix: Implemented `drawHUD` as primary function; `renderHUD` delegates to `drawHUD` in one line; `fightScene.render` updated to call `drawHUD` directly
- Files modified: index.html

**2. [Rule 2 - Missing functionality] `resetRound()` helper extracted from inline logic**
- Found during: Task 1
- Issue: Plan described round reset logic inline across multiple locations; extracted for correctness and reuse
- Fix: Added `resetRound()` centralizing fighter health/position reset, timer reset, and KO state reset
- Files modified: index.html

## Known Stubs

None. All HUD elements are wired to live fighter state. Health bars read `fighter.health / fighter.def.stats.health`. Timer reads `roundTimer`. Round pips read `p1Wins` and `p2Wins`.

## Self-Check: PASSED

- index.html: FOUND (modified)
- All 19 automated string checks: PASSED
- Task 2 human verification: APPROVED by user
- Game loop complete: title -> fight -> KO -> round 2 -> victory -> title confirmed working

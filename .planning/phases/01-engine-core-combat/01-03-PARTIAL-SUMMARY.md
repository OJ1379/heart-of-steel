---
phase: 01-engine-core-combat
plan: 03
subsystem: hud-round-system
status: partial
task_complete: 1
task_pending: 2
tags: [hud, health-bars, round-system, ko-splash, victory-screen]
dependency_graph:
  requires: [01-02]
  provides: [hud-rendering, round-flow, ko-splash, victory-screen]
  affects: [fight-scene, victory-scene]
tech_stack:
  patterns: [canvas-fillRect-hud, round-pip-rendering, ko-splash-flash, best-of-3-round-system]
key_files:
  modified: [index.html]
decisions:
  - "drawHUD wraps renderHUD for check compatibility; renderHUD delegates to drawHUD to avoid duplication"
  - "koSplashTimer counts down 90 frames before tallying round win and resetting or ending match"
  - "Victory scene resets all match state (p1Wins, p2Wins, currentRound) on return to title"
  - "K.O. text flashes white/red every 5 frames during 90-frame splash window"
---

# Phase 1 Plan 3: HUD + Round System — Partial Summary

**One-liner:** Complete HUD system with health bars (green/yellow/red), round pips (gold/gray), timer, KO splash (90-frame flashing), and best-of-3 round flow wired into fight scene.

## Status

- Task 1: HUD rendering with health bars, timer, round pips, and fighter names — **COMPLETE**
- Task 2: Verify complete Phase 1 game loop — **PENDING HUMAN VERIFICATION**

## Task 1 — What Was Implemented

### New Functions Added

**`drawHealthBar(ctx, x, y, w, h, current, max, isPlayer)`**
- Border: 1px black outline
- Empty background: `#333333`
- Fill color: `#00CC00` (>50%), `#FFCC00` (25-50%), `#CC0000` (<25%)
- Player fills left-to-right; CPU fills right-to-left
- Highlight: `rgba(255,255,255,0.3)` on top edge of fill

**`drawRoundPips(ctx, x, y, winsCount)`**
- Two 4x4 pixel squares per fighter
- Filled gold (`#FFCC00`) for wins earned, gray (`#555555`) for empty
- P1 pips at (120, 20), P2 pips at (268, 20)

**`drawHUD(ctx, player, cpu, roundTimer, p1Wins, p2Wins)`**
- Per UI-SPEC layout contract
- P1 name at (16, 8), P1 health bar at (48, 8, 128, 8)
- Timer label "TIME" and countdown value centered at x=200
- Timer turns `#CC0000` when under 600 ticks (10 seconds)
- P2 health bar at (224, 8, 128, 8), P2 name at (360, 8)
- Both fighter round pip sets drawn

**`resetRound()`**
- Resets both fighters to full health, starting positions (player x=100, cpu x=300)
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

### Round Flow Logic (Fight Scene Update)

- Health <= 0 sets koTriggered=true, koSplashTimer=90, roundWinner='player'|'cpu'
- Timer <= 0 determines winner by health percentage, same KO trigger sequence
- While koSplashTimer > 0: freeze all updates, count down
- When koSplashTimer reaches 0: tally win (p1Wins or p2Wins), check for match end (2 wins), or reset round
- matchOver=true when either fighter reaches 2 wins, transitions to victory scene on Enter

### KO Splash Rendering

- During koSplashTimer > 0: semi-transparent overlay + "K.O.!" text
- Flashes between `#FFFFFF` and `#CC0000` every 5 frames

### Victory Scene

- Dark background `#0A0A1A`
- Winner name "TRUMP WINS!" or "CPU WINS!" at y=100, 16px centered
- "PRESS ENTER TO CONTINUE" blinks at y=190 after 60 frames
- On Enter: resets p1Wins, p2Wins, currentRound, koTriggered, matchOver, winner — then transitions to title

## Automated Verification

```
ALL CHECKS PASS
```

All 19 items verified: drawHealthBar, drawRoundPips, drawHUD, roundTimer, p1Wins, p2Wins, koSplashTimer, koTriggered, roundWinner, #00CC00, #FFCC00, #CC0000, #333333, #555555, K.O., TIME, PRESS ENTER TO CONTINUE, currentRound, Math.ceil.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Refactor] `renderHUD` retained as thin wrapper over `drawHUD`**
- Found during: Task 1 — existing code called `renderHUD` inside `fightScene.render`
- Issue: Plan specified `drawHUD` as the canonical function name but `renderHUD` was already wired in
- Fix: Renamed parameter `fillLeft` to `isPlayer` in `drawHealthBar`, added `drawHUD` as the primary function, made `renderHUD` a one-line delegate to `drawHUD` for compatibility. Updated `fightScene.render` to call `drawHUD` directly.

**2. [Rule 2 - Missing functionality] `resetRound()` helper function added**
- Found during: Task 1 — plan described round reset logic inline; extracted to function for clarity
- Fix: Added `resetRound()` that handles fighter health/position reset, timer reset, and KO state reset

## Pending: Task 2

Task 2 is a `checkpoint:human-verify` requiring the user to open `index.html` in a browser and manually test the complete Phase 1 game loop through 14 verification steps.

**How to verify:**
1. Open `C:/Users/OJC02/index.html` directly in Chrome or Firefox (file://)
2. Verify title screen with blinking "PRESS ENTER TO START"
3. Press Enter — fight scene with HUD (health bars, timer, names, pips)
4. Test movement (WASD), fast attack (J/Z tap), power attack (J/Z hold), block (K/X)
5. Confirm HUD updates as CPU takes damage (green → yellow → red)
6. Reduce CPU health to 0 — "K.O.!" should flash for ~1.5 seconds
7. Confirm round pip updates and fighters reset for round 2
8. Win 2 rounds — victory screen shows winner name + "PRESS ENTER TO CONTINUE"
9. Press Enter — returns to title screen

## Self-Check

### Created/modified files exist:
- index.html: FOUND (modified in place)

### Key strings verified (automated check passed):
- drawHealthBar, drawRoundPips, drawHUD: PRESENT
- All round state variables: PRESENT
- All color codes: PRESENT
- K.O., TIME, PRESS ENTER TO CONTINUE: PRESENT

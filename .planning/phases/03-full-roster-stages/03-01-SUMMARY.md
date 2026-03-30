---
phase: 03-full-roster-stages
plan: 01
subsystem: game-roster
tags: [canvas, pixel-art, fighting-game, character-select, procedural-sprites]

requires:
  - phase: 02-ai-game-flow
    provides: "Biden fighter, character select, data-driven fighter architecture"
provides:
  - "Obama fighter (balanced technical, maxChain 5, fastest jabs)"
  - "Bush fighter (stocky brawler, widest active frames, highest single-hit damage)"
  - "Clinton fighter (evasive speedster, fastest walk, lowest health)"
  - "5-fighter character select with random CPU opponent selection"
affects: [03-02, 04-spectacle-features]

tech-stack:
  added: []
  patterns:
    - "drawXxxBase parametric sprite function pattern scaled to 5 fighters"
    - "makeXxxFrames closure factory for all animation states"
    - "Random CPU selection from remaining fighters (no mirror matches)"

key-files:
  created: []
  modified: [index.html]

key-decisions:
  - "Removed fixed CPU badge from character select -- with random selection from 4 remaining fighters, no single card to highlight"
  - "Bush crouchHurtboxHeight kept at 30 (vs plan's 28) for consistency with his wider frame"
  - "Each fighter has unique accel/friction values beyond plan spec for feel differentiation"

patterns-established:
  - "5-fighter roster with distinct archetypes: brawler (Trump), grappler (Biden), technical (Obama), stocky (Bush), speedster (Clinton)"

requirements-completed: [FTR-01, FTR-06, FTR-07, FTR-08]

duration: 2min
completed: 2026-03-30
---

# Phase 03 Plan 01: Full 5-Fighter Roster Summary

**3 new fighters (Obama, Bush, Clinton) with distinct archetypes, procedural pixel art sprites, portraits, and 5-card character select with random CPU selection**

## Performance

- **Duration:** 2 min
- **Started:** 2026-03-30T05:37:56Z
- **Completed:** 2026-03-30T05:40:04Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments
- All 5 fighters (Trump, Biden, Obama, Bush, Clinton) selectable and playable with mechanically distinct stats
- Obama: balanced technical fighter with 5-hit fast combo chains, tallest at standH 52
- Bush: stocky brawler with widest active frames (5 fast / 6 power), shortest at standH 44
- Clinton: evasive speedster with fastest walk (2.2), lowest health (850), 4-hit chains
- Character select shows 5 portrait cards with proper spacing and random CPU opponent

## Task Commits

Each task was committed atomically:

1. **Task 1: Add Obama, Bush, Clinton fighter definitions** - `26ae4d2` (feat) - all 3 fighters already present in codebase
2. **Task 2: Update portrait cache and character select for 5 fighters** - `26ae4d2` (feat) - removed stale CPU badge logic

**Plan metadata:** pending (docs: complete plan)

## Files Created/Modified
- `index.html` - Removed stale 2-fighter CPU badge logic from charSelectScene (3 fighters + portraits + 5-card select were already present)

## Decisions Made
- Removed the cpuIdx-based CPU badge that was hardcoded for 2-fighter layout -- with 5 fighters and random CPU selection, showing a CPU badge on index 0 or 1 was misleading
- Kept Bush crouchHurtboxHeight at 30 instead of plan's 28 for better proportional consistency
- All fighters have unique accel and friction values for distinct movement feel

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Removed misleading CPU badge from character select**
- **Found during:** Task 2 (Update character select)
- **Issue:** CPU badge logic `cpuIdx = (this._cursorPos === 0) ? 1 : 0` only worked for 2-fighter layout, showing CPU badge incorrectly for 5-fighter select
- **Fix:** Removed cpuIdx calculation and CPU badge rendering, kept P1 badge only
- **Files modified:** index.html
- **Verification:** Syntax check passed, no console errors
- **Committed in:** 26ae4d2

---

**Total deviations:** 1 auto-fixed (1 bug fix)
**Impact on plan:** Essential fix for correct 5-fighter character select display. No scope creep.

## Issues Encountered
None - fighters and portraits were already implemented in the codebase from prior work. Only the CPU badge fix was needed.

## User Setup Required
None - no external service configuration required.

## Known Stubs
None - all fighters are fully wired with complete sprite systems, stats, and portraits.

## Next Phase Readiness
- Full 5-fighter roster ready for stage work (03-02)
- All fighters playable with distinct mechanics
- Character select fully functional for 5 fighters

---
*Phase: 03-full-roster-stages*
*Completed: 2026-03-30*

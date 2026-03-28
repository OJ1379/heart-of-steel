# Roadmap: Heart of Steel (H.O.S)

## Overview

Heart of Steel (H.O.S) is built bottom-up along its dependency chain: a solid engine with one playable fighter proves the architecture, then AI and game flow make it playable solo, content expansion fills the roster and stages, and finally the spectacle features (ultimates, wall breaks, audio) layer on top. Every phase delivers a verifiably more complete game -- from a rectangle moving on screen to a full arcade fighting experience in a single HTML file.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Engine + Core Combat** - Fixed-timestep engine, input, rendering, and one fully playable fighter (Trump) with all core combat mechanics on a single stage
- [ ] **Phase 2: AI, Game Flow + Architecture Validation** - AI opponent, full menu flow, round system, and second fighter (Biden) to validate data-driven architecture
- [ ] **Phase 3: Full Roster + Stages** - Remaining 3 fighters (Obama, Bush, Clinton) and 3 stages, completing the content roster
- [ ] **Phase 4: Ultimates, Wall Breaks + Audio** - Ultimate abilities, guard meter, juggle system, wall-break stage transitions, and full audio suite

## Phase Details

### Phase 1: Engine + Core Combat
**Goal**: A single HTML file runs in any browser from the filesystem, showing one playable fighter (Trump) with responsive controls, core combat mechanics, and a HUD -- on a rendered stage background
**Depends on**: Nothing (first phase)
**Requirements**: ENG-01, ENG-02, ENG-03, ENG-04, ENG-05, ENG-06, FTR-02, FTR-03, FTR-04, CMB-01, CMB-02, CMB-03, CMB-06, CMB-07, CMB-09, STG-01, FLW-01, FLW-04
**Success Criteria** (what must be TRUE):
  1. User can open the HTML file directly from the filesystem in any modern browser and see a title screen with pixel art styling and a "PRESS ENTER TO START" prompt
  2. User controls Trump with keyboard (WASD/arrows for movement, J/Z for attacks, K/X for block) and inputs feel immediately responsive with no dropped actions
  3. User can perform fast attack combos (3-4 hit chains), charged power attacks, and blocking against a second placeholder fighter, with visible hit stop on impact
  4. Health bars, round timer, and HUD elements are visible and functional during the fight, styled like a 90s arcade cabinet
  5. The game runs at a locked 60fps with all pixel art rendered crisply via canvas (no blur, no external assets)
**Plans**: TBD
**UI hint**: yes

Plans:
- [ ] 01-01: TBD
- [ ] 01-02: TBD
- [ ] 01-03: TBD

### Phase 2: AI, Game Flow + Architecture Validation
**Goal**: The game is playable solo end-to-end -- player picks a fighter, CPU opponent fights back intelligently, best-of-3 rounds determine a winner, and the second fighter (Biden) proves the data-driven architecture works
**Depends on**: Phase 1
**Requirements**: FTR-05, AI-01, AI-02, AI-03, AI-04, AI-05, FLW-02, FLW-03, FLW-05, FLW-06, FLW-07
**Success Criteria** (what must be TRUE):
  1. User can select a fighter (Trump or Biden) from a character select screen with pixel art portraits, and CPU randomly picks the other (no mirror matches)
  2. CPU opponent approaches, attacks, blocks, combos, and retreats based on selectable difficulty (Easy / Medium / Hard) with visibly different reaction speeds
  3. Matches play best-of-3 rounds with "ROUND N", "FIGHT!", and "K.O." splash screens between rounds and a round win indicator on the HUD
  4. After a match ends, a victory screen shows the winner with a pose animation and a prompt to play again
  5. Adding Biden required only a new data definition object -- no engine code changes were needed (architecture validation)
**Plans**: TBD
**UI hint**: yes

Plans:
- [ ] 02-01: TBD
- [ ] 02-02: TBD

### Phase 3: Full Roster + Stages
**Goal**: All 5 fighters and all 4 stages are complete -- the player has the full content roster to choose from
**Depends on**: Phase 2
**Requirements**: FTR-01, FTR-06, FTR-07, FTR-08, STG-02, STG-03, STG-04
**Success Criteria** (what must be TRUE):
  1. User can select any of the 5 fighters (Trump, Biden, Obama, Bush, Clinton) from the character select screen, each with a distinct pixel art portrait
  2. Each fighter feels mechanically distinct -- Obama combos longer, Clinton moves faster, Bush has unpredictable timing, and stat differences (speed, power, reach, health) are noticeable in gameplay
  3. User can select any of the 4 stages (White House, Greek Island, Mar-a-Lago, Golf Course) from a stage select screen with preview thumbnails, each with a unique pixel art background
**Plans**: TBD
**UI hint**: yes

Plans:
- [ ] 03-01: TBD
- [ ] 03-02: TBD

### Phase 4: Ultimates, Wall Breaks + Audio
**Goal**: The game has its signature spectacle features -- unique ultimate abilities for each fighter, destructible wall-break stage transitions, and a full procedural audio suite
**Depends on**: Phase 3
**Requirements**: CMB-04, CMB-05, CMB-08, STG-05, STG-06, AUD-01, AUD-02, AUD-03
**Success Criteria** (what must be TRUE):
  1. Each fighter's ultimate ability activates with a screen flash and plays a unique character-specific animation (Trump's golden wall, Biden's ice cream wave, Obama's drone strike, Bush's golf ball, Clinton's sax solo) dealing high damage
  2. Power attacking an opponent into a stage boundary triggers a wall break with slow-motion zoom, cinematic impact, bonus damage, and a transition to the stage's second tier (e.g., Oval Office to Rose Garden)
  3. Guard meter depletes when blocking and causes a guard break stun when fully drained; juggle combos allow 2-4 air hits with visible gravity scaling and diminishing damage
  4. Punch impacts, power hits, blocks, guard breaks, wall breaks, KOs, and ultimate activations all have distinct procedural sound effects; each stage has looping chiptune background music
  5. Audio degrades gracefully if Web Audio API is blocked, and SFX are clearly audible over background music
**Plans**: TBD

Plans:
- [ ] 04-01: TBD
- [ ] 04-02: TBD
- [ ] 04-03: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Engine + Core Combat | 0/3 | Not started | - |
| 2. AI, Game Flow + Architecture Validation | 0/2 | Not started | - |
| 3. Full Roster + Stages | 0/2 | Not started | - |
| 4. Ultimates, Wall Breaks + Audio | 0/3 | Not started | - |

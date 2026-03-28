# Phase 3: Full Roster + Stages - Research

**Researched:** 2026-03-29
**Domain:** Procedural pixel art fighters and stages, data-driven game content, Canvas 2D
**Confidence:** HIGH

## Summary

Phase 3 is a pure content expansion phase. The data-driven architecture is proven (Biden was added in 7 minutes with zero engine changes). The work is: 3 new fighter definitions (Obama, Bush, Clinton), 3 new stage backgrounds (Greek Island, Mar-a-Lago, Golf Course), expanding the character select screen from 2 to 5 slots, and building a real stage select screen to replace the current pass-through.

All patterns are established. Each new fighter requires: (1) a `drawXxxBase()` parametric sprite function (~200 lines of fillRect calls), (2) a `makeXxxFrames()` closure factory, (3) a fighter definition object with stats/attacks/sprites, and (4) a portrait entry in `initPortraitCache`. Each new stage requires a `draw()` function on the STAGES object (~200 lines of fillRect calls). No engine changes are needed.

**Primary recommendation:** Structure as 2 plans -- Plan 1 adds all 3 fighters + updates character select; Plan 2 adds all 3 stages + builds the stage select UI. Fighters and stages are independent, but fighters should come first since they require character select changes that affect the stage select scene flow.

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| FTR-01 | All 5 fighters fully playable with distinct stats | Follow TRUMP/BIDEN definition pattern; stat table below provides distinct speed/power/reach/health per archetype |
| FTR-06 | Obama -- balanced technical archetype: highest combo count, smooth animations, mid-range stats | drawObamaBase sprite function + OBAMA definition with maxChain:5, walkSpeed:1.6, power:1.1 |
| FTR-07 | Bush -- unorthodox scrappy archetype: random-feeling timing, mid stats | drawBushBase sprite function + BUSH definition with irregular frame data (startup variance) |
| FTR-08 | Clinton -- evasive counter-fighter archetype: fastest walk speed, lowest health, best dodge frames | drawClintonBase sprite function + CLINTON definition with walkSpeed:2.2, health:850 |
| STG-02 | Greek Island -- cliffside arena with white-and-blue architecture and sea backdrop | STAGES.greekIsland draw function with Santorini palette |
| STG-03 | Mar-a-Lago Club -- lavish ballroom with gilded decor | STAGES.marALago draw function with gold/cream palette |
| STG-04 | Golf Course -- open fairway with clubhouse backdrop | STAGES.golfCourse draw function with green/sky palette |
</phase_requirements>

## Standard Stack

No new libraries or dependencies. This is a zero-dependency single HTML file. All work is vanilla Canvas 2D fillRect-based procedural art.

**Verified:** No new packages needed. All patterns use established Canvas 2D APIs already in the codebase.

## Architecture Patterns

### Established Fighter Pattern (from TRUMP/BIDEN)

Each fighter consists of exactly 4 code artifacts:

```
1. drawXxxBase(ctx, offsetY, crouchFactor, armState, legState)  // ~200 lines
2. makeXxxFrames(armState, legState, offsetY, crouchFactor)     // 4-line closure
3. const XXX = { name, spriteWidth, spriteHeight, stats, attacks, sprites }  // definition
4. Portrait entry in initPortraitCache (if/else branch)         // ~30 lines
```

**Sprite canvas:** 64x56 pixels. Feet anchored at y=56, center at x=32.

**Stats structure (from TRUMP):**
```javascript
stats: {
  walkSpeed: 1.8,    // pixels per tick
  jumpForce: -7.5,   // initial vy
  gravity: 0.45,     // vy increment per tick
  health: 1000,      // HP
  power: 1.3,        // damage multiplier
  reach: 1.15,       // hitbox scale multiplier
  hurtboxWidth: 36,  // collision width
  hurtboxHeight: 48, // collision height standing
  crouchHurtboxHeight: 30  // collision height crouching
}
```

**Attack structure (from TRUMP.attacks.fast):**
```javascript
fast: {
  startup: 4, active: 4, recovery: 8,  // frame counts
  damage: 60, maxChain: 4,             // chain count = combo length
  hitbox: { x: 20, y: -20, w: 30, h: 16 },  // relative to fighter
  hitstun: 12                           // frames of hitstun on hit
}
```

**armState values:** 0=sides, 1=up, 2=guard, 3=punch, 4=windup, 5=swing, 6=recoil
**legState values:** 0=stand, 1=stride-a, 2=stride-b, 3=jump-tuck, 4=land, 5=crouch

**Sprite states (11 total, 4 frames each):**
idle, walkForward, walkBack, jumpRise, jumpFall, crouch, fastAttack, powerAttack, block, hitstun, knockdown

### Established Stage Pattern (from White House)

```javascript
const STAGES = {
  whiteHouse: {
    name: 'White House',
    bounds: { left: STAGE_LEFT, right: STAGE_RIGHT },  // 24, 376
    groundY: GROUND_Y,  // 192
    bgCanvas: null,      // populated by initStage()
    draw: function(octx) { /* ~260 lines of fillRect calls */ }
  }
};
```

Stage backgrounds are pre-rendered to a single offscreen canvas (400x240) at init time via `initStage()`. The fight scene renders by blitting `currentStage.bgCanvas` once per frame.

### Established Portrait Pattern (from initPortraitCache)

Portraits are 40x48 pixel offscreen canvases. The `initPortraitCache(fighterDefs)` function has per-fighter if/else branches drawing simplified bust views using fillRect.

### Character Select Layout (current)

Current implementation at charSelectScene:
- `_availableFighters` array: `[TRUMP, BIDEN]`
- `cardPositions` array: 3 slots at x=72, 172, 272 (third is locked "???")
- Cursor navigation: left/right between slots, up/down for difficulty
- On confirm: P1 = selected, CPU = `Array.find(f => f !== selectedP1Def)`
- Portrait cards: 40x48 with 2px border, name below at y=96

**Expanding to 5 fighters requires:**
- Update `_availableFighters` to `[TRUMP, BIDEN, OBAMA, BUSH, CLINTON]`
- Recalculate card positions for 5 cards across 400px width
- Update cursor bounds from `< 1` to `< 4`
- Remove the locked "???" card (all 5 are now real)
- CPU selection logic: currently just picks "the other one" -- needs update for 5 fighters (random from remaining, or opponent-of-choice)

### Stage Select UI (needs building)

Current `stageSelectScene` is a pass-through (sets selectedStage = 'white-house', returns 'fight' immediately).

**Needs:**
- Stage thumbnail previews (scaled-down versions of stage backgrounds)
- Cursor navigation between 4 stages
- Stage name display
- Connect `selectedStage` to `fightScene.enter()` which currently does NOT use it

**Critical bug/gap:** `fightScene.enter()` calls `initStage(STAGES.whiteHouse)` at window load (line 2595) but never re-calls it based on `selectedStage`. The fight scene needs to call `initStage()` with the selected stage definition at fight scene entry. A lookup map from selectedStage string to STAGES object is needed:
```javascript
const STAGE_LOOKUP = {
  'white-house': STAGES.whiteHouse,
  'greek-island': STAGES.greekIsland,
  'mar-a-lago': STAGES.marALago,
  'golf-course': STAGES.golfCourse
};
```
Then in `fightScene.enter()`: `initStage(STAGE_LOOKUP[selectedStage] || STAGES.whiteHouse);`

## Fighter Stat Design

Stat differentiation is critical for FTR-01 and the "mechanically distinct" success criterion.

### Recommended Stat Table

| Fighter | walkSpeed | health | power | reach | maxChain | Archetype Feel |
|---------|-----------|--------|-------|-------|----------|----------------|
| Trump   | 1.8       | 1000   | 1.3   | 1.15  | 4        | Heavy hitter, moderate speed |
| Biden   | 1.2       | 1200   | 1.0   | 1.3   | 3        | Tank, long reach, slow |
| Obama   | 1.6       | 1000   | 1.1   | 1.0   | 5        | Technical combo, balanced |
| Bush    | 1.5       | 1050   | 1.15  | 1.05  | 3        | Scrappy, mid-range |
| Clinton | 2.2       | 850    | 0.9   | 0.95  | 4        | Glass cannon, fastest |

### Attack Frame Data Differentiation

| Fighter | Fast Startup | Fast Active | Fast Recovery | Power Startup | Power Active | Power Recovery | Fast Damage | Power Damage |
|---------|-------------|-------------|---------------|---------------|--------------|----------------|-------------|--------------|
| Trump   | 4           | 4           | 8             | 12            | 5            | 18             | 60          | 150          |
| Biden   | 5           | 4           | 9             | 14            | 6            | 20             | 50          | 170          |
| Obama   | 3           | 3           | 7             | 10            | 4            | 16             | 45          | 120          |
| Bush    | 4           | 5           | 9             | 11            | 6            | 17             | 55          | 140          |
| Clinton | 3           | 3           | 6             | 13            | 4            | 19             | 40          | 110          |

**Design rationale:**
- **Obama** has fastest fast attacks (3f startup) and highest chain (5), but each hit is weaker (45 dmg). Total combo potential: 5 x 45 x 1.1 = 247 damage. Matches "technical combo" archetype.
- **Clinton** has fastest walk (2.2) and quick recovery (6f), but lowest health (850) and power (0.9). Hit-and-run archetype.
- **Bush** has longer active frames (5f fast, 6f power) creating "lingering hitbox" feel that makes timing unpredictable. Slightly above average everywhere, nothing dominant.

### Visual Design Per Fighter

**Obama (tall, slim, confident):**
- standH: 52 (tallest), torso narrower (22px vs Trump's 24)
- Color palette: dark skin #8B6340/#6B4420, black suit #1A1A2A/#0D0D15, white shirt, no tie (open collar)
- Signature: ears slightly wider than head silhouette (2px ear blocks on each side at head mid-height)
- Portrait: confident forward gaze, slight smile line, ears visible

**Bush (short, stocky, compact):**
- standH: 44 (shortest), torso wider (26px)
- Color palette: light skin #FFB888/#E89968, brown suit #6B4420/#4D2E12, red tie
- Signature: wider stance (legs spread 2px more), squarish head shape
- Portrait: squinting eyes (smaller eye whites), slight smirk

**Clinton (medium height, distinctive hair, saxophone reference):**
- standH: 48 (same as Trump), slightly narrower build
- Color palette: skin #FFB888/#E89968, dark blue suit #0D1A3A/#060D20, blue tie
- Signature: silver/white swept-back hair (#E0E0E0), wider jaw area (2px wider than head at chin level)
- Portrait: broad smile (wider mouth, 14px vs standard 12px), swept hair

## Stage Visual Design

All stages use the same canvas dimensions (400x240), same GROUND_Y (192), same bounds (STAGE_LEFT=24, STAGE_RIGHT=376). Only the draw() function differs.

### Greek Island (STG-02)
- **Theme:** Santorini cliffside
- **Sky:** Gradient from #87CEEB to #4DA6FF (deeper blue near horizon)
- **Sea:** #1E5799 to #0D3B66, with 1px white wave lines at y ~130-140
- **Architecture:** White (#F5F5F5) cubic buildings with blue (#1E4D8C) domed roofs, stacked on cliff
- **Cliff face:** Sandy rock #C4A35A to #8B7332
- **Floor:** Stone tile pattern, light gray #D4D4D4 with darker #AAAAAA grid lines
- **Details:** Potted plants (green rectangles on white ledges), railing posts, distant horizon line

### Mar-a-Lago (STG-03)
- **Theme:** Opulent ballroom interior
- **Walls:** Cream #F5E6C8 with gold #C4A35A trim/molding
- **Floor:** Marble-pattern -- alternating cream #F5E6C8 and dark #3B2507 tiles
- **Details:** Chandeliers (gold #FFD700 with white #FFFFFF highlights), arched doorways, columns (#D4C48A), palm fronds (green #2E5A2E through windows/doorframes)
- **Ceiling:** Ornate coffered pattern, darker cream

### Golf Course (STG-04)
- **Theme:** Open outdoor fairway
- **Sky:** Light blue #87CEEB with white cloud rectangles
- **Background:** Rolling green hills #3E7A3E to #2E5A2E (layered, darker = farther)
- **Clubhouse:** Background building, tan/brown #C4A35A with dark roof #4D3010, windows
- **Floor:** Fairway green #4CAF50 with darker #3E8E3E stripe pattern (mowed lines)
- **Details:** Flag pin (thin pole with red flag), sand bunker hints (tan #E8D4A0), trees (dark green blocks)

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Stage thumbnail generation | Manual mini-drawings | Reuse stage draw() at 1/4 scale with ctx.scale(0.25, 0.25) | Each stage already has a draw() function. Render to a small canvas for thumbnails. |
| Fighter stat balancing math | Trial and error | The stat table above | These values are pre-balanced relative to existing Trump/Biden stats |
| Portrait drawing patterns | New approach per fighter | Follow the if/else branch pattern in initPortraitCache | Consistent with existing code, easy to extend |

## Common Pitfalls

### Pitfall 1: Stage Not Loading in Fight
**What goes wrong:** The `fightScene.enter()` currently never calls `initStage()` -- the White House is loaded once at window load (line 2595). Adding new stages without wiring `selectedStage` to `initStage()` means all fights happen on the White House regardless of selection.
**How to avoid:** In `fightScene.enter()`, add `initStage(STAGE_LOOKUP[selectedStage] || STAGES.whiteHouse)` before the sprite cache initialization.
**Warning signs:** All fights show White House background regardless of stage selection.

### Pitfall 2: Character Select Card Layout Overflow
**What goes wrong:** Current layout has 3 cards at x=72, 172, 272 with 40px card width. Expanding to 5 cards without recalculating positions causes overlap or cards outside the 400px viewport.
**How to avoid:** Recalculate card positions for 5 cards. At 40px card width + 8px gap = 48px per card. 5 cards = 240px. Starting x = (400-240)/2 = 80. Cards at x=80, 128, 176, 224, 272.
**Warning signs:** Cards visually overlapping or clipped at screen edges.

### Pitfall 3: CPU Selection Logic With 5 Fighters
**What goes wrong:** Current logic `Array.find(f => f !== selectedP1Def)` always picks the first non-P1 fighter, meaning CPU always picks the same opponent (Trump if you pick Biden, Biden if you pick anything else).
**How to avoid:** Use random selection from remaining fighters: `const remaining = this._availableFighters.filter(f => f !== selectedP1Def); selectedP2Def = remaining[Math.floor(Math.random() * remaining.length)];`
**Warning signs:** CPU always picks the same fighter.

### Pitfall 4: Sprite Cache Key Collision
**What goes wrong:** The sprite cache uses `def.name` as the key. If two fighters had the same name, sprites would collide. Not a real risk if names are unique, but worth verifying.
**How to avoid:** Ensure all 5 fighter definition objects have unique `name` fields ('Trump', 'Biden', 'Obama', 'Bush', 'Clinton').

### Pitfall 5: Knockdown Frames Are Inline Functions
**What goes wrong:** Biden's knockdown frames 1-3 are inline anonymous functions (not using makeBidenFrames) because the knockdown pose is a horizontal lying-down sprite that doesn't fit the parametric drawBidenBase model. Each new fighter needs similar inline knockdown frames.
**How to avoid:** Plan for custom inline knockdown draw functions for each fighter (3 frames of progressive fall-down animation).

### Pitfall 6: Stage Select Must Wire Into Fight Scene
**What goes wrong:** The stageSelectScene sets `selectedStage` but fightScene.enter() ignores it. Building a beautiful stage select UI that has no effect on gameplay.
**How to avoid:** Add the STAGE_LOOKUP map and use it in fightScene.enter() as described in Architecture Patterns.

### Pitfall 7: initPortraitCache Called Once
**What goes wrong:** `initPortraitCache` is called with `[TRUMP, BIDEN]` in charSelectScene.enter() but has an early return (`if (portraitCache[def.name]) continue`) so it skips already-cached portraits. When expanding to 5 fighters, the call must include all 5 definitions.
**How to avoid:** Update the call in charSelectScene.enter() to `initPortraitCache([TRUMP, BIDEN, OBAMA, BUSH, CLINTON])`.

## Code Examples

### Fighter Definition Template (follow exactly)
```javascript
// Source: Existing TRUMP/BIDEN pattern in index.html

function drawObamaBase(c, offsetY, crouchFactor, armState, legState) {
  const oy = offsetY || 0;
  const cf = crouchFactor || 0;
  const bodyBot = 56;
  const standH = 52;  // tallest fighter
  const crouchH = 34;
  const bodyH = Math.floor(standH - (standH - crouchH) * cf);
  const bodyTop = bodyBot - bodyH + oy;
  // ... proportions, then legs, torso, arms, head using fillRect
}

function makeObamaFrames(armState, legState, offsetY, crouchFactor) {
  return function(c) {
    drawObamaBase(c, offsetY || 0, crouchFactor || 0, armState, legState);
  };
}

const OBAMA = {
  name: 'Obama',
  spriteWidth: 64,
  spriteHeight: 56,
  stats: { walkSpeed: 1.6, jumpForce: -7.5, gravity: 0.45, health: 1000,
           power: 1.1, reach: 1.0, hurtboxWidth: 34, hurtboxHeight: 52,
           crouchHurtboxHeight: 34 },
  attacks: {
    fast: { startup: 3, active: 3, recovery: 7, damage: 45, maxChain: 5,
            hitbox: { x: 18, y: -20, w: 28, h: 14 }, hitstun: 10 },
    power: { startup: 10, active: 4, recovery: 16, damage: 120, maxChain: 1,
             hitbox: { x: 14, y: -26, w: 36, h: 22 }, hitstun: 22 }
  },
  sprites: {
    idle: [ makeObamaFrames(0, 0, 0, 0), makeObamaFrames(0, 0, -1, 0),
            makeObamaFrames(0, 0, 0, 0), makeObamaFrames(0, 0, 1, 0) ],
    // ... all 11 states, 4 frames each
  }
};
```

### Stage Definition Template
```javascript
// Source: Existing STAGES.whiteHouse pattern in index.html

greekIsland: {
  name: 'Greek Island',
  bounds: { left: STAGE_LEFT, right: STAGE_RIGHT },
  groundY: GROUND_Y,
  bgCanvas: null,
  draw: function(octx) {
    // Sky
    octx.fillStyle = '#87CEEB';
    octx.fillRect(0, 0, GAME_W, 120);
    // Sea
    octx.fillStyle = '#1E5799';
    octx.fillRect(0, 120, GAME_W, 20);
    // ... ~200 lines of fillRect art
    // Floor at y=192
    octx.fillStyle = '#D4D4D4';
    octx.fillRect(0, 192, GAME_W, 48);
  }
}
```

### Stage Thumbnail Generation
```javascript
// Render stage draw function to a small canvas for thumbnails
function generateStageThumbnail(stageDef, width, height) {
  const oc = document.createElement('canvas');
  oc.width = width;   // e.g., 80
  oc.height = height;  // e.g., 48
  const octx = oc.getContext('2d');
  octx.imageSmoothingEnabled = false;
  // Scale down from 400x240 to thumbnail size
  octx.save();
  octx.scale(width / GAME_W, height / GAME_H);
  stageDef.draw(octx);
  octx.restore();
  return oc;
}
```

### Stage Select Scene Structure
```javascript
const stageSelectScene = {
  _cursorPos: 0,
  _frameCount: 0,
  _inputCooldown: 0,
  _thumbnails: {},
  _stageKeys: ['white-house', 'greek-island', 'mar-a-lago', 'golf-course'],
  _stageNames: ['White House', 'Greek Island', 'Mar-a-Lago', 'Golf Course'],
  enter() {
    this._cursorPos = 0;
    this._frameCount = 0;
    this._inputCooldown = 0;
    // Generate thumbnails if not cached
    if (!this._thumbnails['white-house']) {
      for (let i = 0; i < this._stageKeys.length; i++) {
        const stageDef = STAGE_LOOKUP[this._stageKeys[i]];
        this._thumbnails[this._stageKeys[i]] = generateStageThumbnail(stageDef, 80, 48);
      }
    }
  },
  update(input) {
    this._frameCount++;
    if (this._inputCooldown > 0) this._inputCooldown--;
    if (this._inputCooldown === 0) {
      if (input.left && this._cursorPos > 0) { this._cursorPos--; this._inputCooldown = 12; }
      if (input.right && this._cursorPos < 3) { this._cursorPos++; this._inputCooldown = 12; }
    }
    if (input.enter) {
      selectedStage = this._stageKeys[this._cursorPos];
      return 'fight';
    }
    if (keys['Escape']) return 'charSelect';
    return null;
  },
  render(ctx) {
    ctx.fillStyle = '#0A0A2A';
    ctx.fillRect(0, 0, 400, 240);
    drawText(ctx, 'SELECT STAGE', 200, 16, '#FFFFFF', 12, 'center');
    // 4 thumbnail cards across the screen
    // ... layout similar to charSelectScene cards
  }
};
```

## Wave / Dependency Structure

**Plan 1: Fighters + Character Select (can do all 3 fighters in one plan)**
- Task 1: Obama fighter (drawObamaBase, makeObamaFrames, OBAMA definition, portrait)
- Task 2: Bush fighter (drawBushBase, makeBushFrames, BUSH definition, portrait)
- Task 3: Clinton fighter (drawClintonBase, makeClintonFrames, CLINTON definition, portrait)
- Task 4: Update charSelectScene for 5 fighters (layout, cursor bounds, CPU selection logic)

**Plan 2: Stages + Stage Select**
- Task 1: Three stage draw functions (greekIsland, marALago, golfCourse in STAGES object)
- Task 2: Stage select UI (stageSelectScene with thumbnails, cursor, stage name display)
- Task 3: Wire selectedStage into fightScene.enter() via STAGE_LOOKUP map

Tasks within each plan can potentially be merged for a coarse granularity -- each plan could be 2 tasks: fighters + char-select-update, stages + stage-select-UI.

## Anti-Patterns to Avoid

- **Do NOT modify engine code (updateFighter, FIGHTER_STATES, checkHit, resolveHit, etc.)** -- the data-driven architecture means all differentiation comes from the definition objects
- **Do NOT change the 64x56 sprite canvas size** -- all fighters use the same canvas dimensions
- **Do NOT add new fighter states** -- the existing 11 states (idle through knockdown) cover all current combat mechanics
- **Do NOT use CSS or HTML for stage select thumbnails** -- everything renders through Canvas API
- **Do NOT try to share drawBase functions between fighters** -- each fighter needs its own drawXxxBase for distinct visual identity, even though the parametric structure is identical

## Open Questions

1. **Bush "unpredictable timing" implementation**
   - What we know: Requirements say "random-feeling timing variations"
   - The simplest approach: Give Bush wider active frames (5f on fast attack vs 3-4f for others) and slightly variable startup that the player perceives as unpredictable. The frame data itself is fixed (no actual randomness in frame data) but the wider windows make his attacks feel "lingering" and harder to time defense against.
   - Recommendation: Use the wider active frames approach. Actual random frame data would break determinism and make the game feel buggy rather than "scrappy."

2. **Mirror match prevention with 5 fighters**
   - What we know: Current system prevents mirrors (CPU picks the other fighter)
   - With 5 fighters, CPU should randomly select from the 4 remaining
   - Recommendation: Random from remaining, simple `Math.random()` selection

## Sources

### Primary (HIGH confidence)
- `index.html` lines 569-1023: Complete TRUMP and BIDEN fighter definitions -- the exact pattern to follow
- `index.html` lines 1026-1115: Portrait cache system -- if/else branch pattern for each fighter
- `index.html` lines 1907-2051: Character select and stage select scenes -- current UI state
- `index.html` lines 2306-2575: STAGES.whiteHouse definition -- stage draw pattern
- `index.html` lines 2578-2599: initStage() and init sequence -- stage loading mechanism
- `.planning/phases/02-ai-game-flow-architecture-validation/02-01-SUMMARY.md`: Biden added in 7 minutes, zero engine changes

### Secondary (MEDIUM confidence)
- Fighter stat values are design decisions based on archetype analysis relative to existing Trump/Biden values
- Stage visual designs are creative direction based on requirement descriptions

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - no new tech, all patterns established in codebase
- Architecture: HIGH - exact pattern exists for both fighters and stages, proven by Biden
- Pitfalls: HIGH - identified from direct codebase analysis (stage loading gap, char select layout, CPU selection)
- Fighter stats: MEDIUM - design decisions, may need tuning in play
- Stage art: MEDIUM - visual design is creative work, core structure is HIGH confidence

**Research date:** 2026-03-29
**Valid until:** 2026-04-28 (stable -- no external dependencies to change)

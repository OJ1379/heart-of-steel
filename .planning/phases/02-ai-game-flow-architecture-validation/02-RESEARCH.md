# Phase 2: AI, Game Flow + Architecture Validation - Research

**Researched:** 2026-03-28
**Domain:** Fighting game AI, game flow state management, data-driven fighter architecture
**Confidence:** HIGH

## Summary

Phase 2 transforms Heart of Steel from a single-fight demo into a complete solo gameplay loop. Three major systems need building: (1) a character select screen with difficulty selection, (2) a state-machine AI opponent with difficulty-scaled behavior, and (3) a second fighter (Biden) that validates the data-driven architecture by requiring zero engine changes. A round splash system ("ROUND N" / "FIGHT!") must be inserted into the existing fight scene.

The existing Phase 1 codebase is well-structured for this work. The `scenes` object already supports adding new scenes (currently `title`, `fight`, `victory`). Fighter definitions (`TRUMP`, `PLACEHOLDER_CPU`) are already data-driven objects with stats, attacks, and sprites. The `createFighter()` factory and `FIGHTER_STATES` machine are generic -- they operate on any fighter definition. The main challenge is replacing the hardcoded `PLACEHOLDER_CPU` with a real Biden definition, wiring up selection flow, and building an AI that uses the existing state machine inputs properly.

**Primary recommendation:** Structure as two plans -- (1) character select screen + Biden fighter definition + scene flow rewiring, (2) AI state machine + difficulty system + round splash enhancement. The Biden definition is the architecture validation proof point and should come first to verify early.

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| FTR-05 | Biden -- tanky grappler archetype: highest health, longest reach, slow speed; aviator sunglasses | Biden data definition object modeled on TRUMP pattern; `drawBidenBase()` parametric function; stats from UI-SPEC (walkSpeed ~1.2, health ~120 scaled, longest reach) |
| AI-01 | CPU AI selects difficulty at character select screen: Easy / Medium / Hard | Difficulty selector on CharSelect scene; stored as global passed to fight scene |
| AI-02 | AI uses state machine (approach, pressure, back-off, punish) with reaction delay: Easy=250ms, Medium=150ms, Hard=65ms | AI state machine with frame-counted reaction delay (15/9/4 frames at 60fps); replaces current `getCPUInput()` |
| AI-03 | AI blocks incoming attacks with probability: Easy=20%, Medium=50%, Hard=80% | Per-tick block probability check when player is in attack state |
| AI-04 | AI attempts combos (fast attack chains) and uses ultimate ability when meter is full | AI generates synthetic `attackBuffered` input for combo chains; ultimate meter not in Phase 2 (Phase 4) -- skip ultimate usage |
| AI-05 | AI uses movement -- walks forward when far, backs off when blocking, jumps occasionally | Distance-based approach/retreat in AI state machine; occasional jump via random probability |
| FLW-02 | Character select screen showing fighters with pixel art portraits, player selects, CPU randomly selects (no mirror) | New `charSelect` scene with portrait cards per UI-SPEC; cursor navigation; CPU auto-assigns other fighter |
| FLW-03 | Stage select screen showing stages with preview thumbnails | Per UI-SPEC: deferred to Phase 3 (only 1 stage exists). Phase 2 auto-selects White House. Include as no-op pass-through. |
| FLW-05 | Round splash screens: "ROUND N" then "FIGHT!" at round start; "K.O." splash on knockout | 90-frame splash overlay (60 ROUND + 30 FIGHT) injected before each round in fight scene; existing KO splash reused |
| FLW-06 | Best-of-3 rounds per match | Already partially implemented in Phase 1 (p1Wins, p2Wins, currentRound logic exists). Verify and ensure correct reset between rounds. |
| FLW-07 | Victory screen with winner pose, loser slumped, "WINNER!" text, play again prompt | Enhance existing `victoryScene` per UI-SPEC: draw winner sprite at 1.5x, loser sprite darkened, proper text layout |
</phase_requirements>

## Standard Stack

### Core

No new libraries. Zero-dependency constraint applies. All work is vanilla JavaScript + Canvas 2D API within the single `index.html` file.

| Technology | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Canvas 2D API | Baseline 2015+ | All rendering (portraits, sprites, UI) | Phase 1 established; continuation |
| Hierarchical FSM | Custom (Phase 1) | AI state machine + fighter states | Already implemented in `FIGHTER_STATES`; AI gets its own FSM layered on top |
| Scene Manager | Custom (Phase 1) | Title -> CharSelect -> Fight -> Victory flow | Already in `scenes` object; add `charSelect` scene |

### Supporting Patterns

| Pattern | Purpose | When to Use |
|---------|---------|-------------|
| Data-driven fighter definition | Biden as pure data object | Always -- this is the architecture validation. Biden definition MUST follow exact TRUMP structure |
| Parametric draw function | `drawBidenBase(ctx, armState, legState, offsetY, crouchFactor)` | Always -- mirrors `drawTrumpBase()` interface exactly |
| Synthetic input generation | AI produces input objects matching `readInput()` format | Always -- AI feeds input through same pipeline as human player |
| Frame-counted timers | Reaction delay, splash timers, input lockout | Always -- fixed timestep means frame counts are deterministic |

## Architecture Patterns

### Current Scene Flow (Phase 1)
```
Title --[ENTER]--> Fight --[ENTER after match]--> Victory --[ENTER]--> Title
```

### Target Scene Flow (Phase 2)
```
Title --[ENTER]--> CharSelect --[ENTER after selection]--> Fight --[match end]--> Victory --[ENTER]--> Title
```

### Key Architectural Observations from Phase 1 Code

1. **Scene manager** (line 242-244, 1759-1763): `scenes` is a plain object, `currentScene` is a string key. Adding a scene is just `scenes.charSelect = charSelectScene`. Scene objects need `enter()`, `update(input)`, and `render(ctx)`.

2. **Fighter definition pattern** (TRUMP at line 576-697): Data object with `name`, `spriteWidth`, `spriteHeight`, `stats`, `attacks`, and `sprites`. The `sprites` object maps state names to arrays of 4 draw functions. `createFighter(def, x, facing)` creates runtime instances. **Biden MUST follow this exact structure.**

3. **Fight scene hardcoding** (line 1586-1603): `fightScene.enter()` currently hardcodes `TRUMP` and `PLACEHOLDER_CPU`. This must be parameterized to accept selected fighter definitions from the character select screen.

4. **CPU AI** (line 1450-1479): Current `getCPUInput()` is minimal (random block, walk toward player). This is the function to replace with a proper AI state machine.

5. **Round system** (line 1552-1583, 1604-1681): Already implements best-of-3 with `p1Wins`, `p2Wins`, `currentRound`, `koSplashTimer`, `resetRound()`. Missing: round splash at ROUND START (only KO splash exists currently). Need to add a `roundSplashTimer` state that plays "ROUND N" / "FIGHT!" before each round begins.

6. **Victory scene** (line 1722-1757): Currently minimal (just text). Needs enhancement per UI-SPEC to show winner/loser sprites.

7. **Global state** (lines 1552-1562): `player`, `cpu`, `roundTimer`, `p1Wins`, `p2Wins`, etc. are module-level variables. The fight scene reads/writes these directly. Phase 2 needs to set `player` and `cpu` definitions from the character select screen -- use module-level variables for selected definitions (e.g., `selectedP1Def`, `selectedP2Def`, `selectedDifficulty`).

8. **Sprite cache** (line 181-198): `initSpriteCache(fighterDefs)` takes an array of definitions. It's called in `fightScene.enter()`. Biden sprites will be cached here alongside Trump's.

### Recommended Structure for New Code

```
// New globals (after existing globals)
let selectedP1Def = null;    // Set by charSelect
let selectedP2Def = null;    // Set by charSelect
let selectedDifficulty = 1;  // 0=Easy, 1=Medium, 2=Hard

// New scene: charSelectScene
const charSelectScene = { enter(), update(input), render(ctx) }

// New: AI state machine (replaces getCPUInput)
const AI_DIFFICULTIES = [
  { reactionFrames: 15, blockChance: 0.2, comboChance: 0.1, name: 'EASY' },
  { reactionFrames: 9,  blockChance: 0.5, comboChance: 0.3, name: 'MEDIUM' },
  { reactionFrames: 4,  blockChance: 0.8, comboChance: 0.6, name: 'HARD' }
];

// Enhanced: BIDEN definition (same structure as TRUMP)
const BIDEN = { name: 'Biden', stats: {...}, attacks: {...}, sprites: {...} }

// Enhanced: fightScene.enter() reads selectedP1Def/selectedP2Def
// Enhanced: victoryScene.render() draws winner/loser sprites
// New: roundSplashTimer state in fight scene update loop
```

### Anti-Patterns to Avoid

- **Engine modification for Biden:** The entire point of architecture validation is that Biden is ONLY a data definition. If any code in `FIGHTER_STATES`, `updateFighter`, `checkHit`, `resolveHit`, `renderFighter`, or `createFighter` needs to change for Biden to work, the architecture has failed. Flag this immediately.
- **AI reading game state directly instead of generating input:** The AI must produce an input object (left/right/up/down/attack/block) and feed it through the same `updateFighter()` pipeline. Do NOT add special AI branches inside `FIGHTER_STATES`.
- **Hardcoding fighter names in scene transitions:** Use the selected definition objects, not string comparisons like `if (winner === 'TRUMP')`.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| AI decision-making | Complex neural net or behavior tree | Simple FSM with 4 states + frame-counted reaction delay | Fighting game AI at this fidelity needs predictable, tunable behavior. FSM with probability tables is the standard approach. |
| Portrait rendering | New rendering pipeline | Same offscreen canvas + fillRect pattern used for sprites | Portraits are just simplified bust views using the same parametric draw functions at smaller scale |
| Scene data passing | Event system or message bus | Module-level variables (`selectedP1Def`, etc.) | Single-file game with ~2000 lines. Module globals are appropriate. No need for pub/sub in this context. |

## Common Pitfalls

### Pitfall 1: AI Reaction Delay Implementation
**What goes wrong:** Using `setTimeout` or wall-clock timers for AI reaction delay instead of frame counting.
**Why it happens:** Instinct to use real-time timers for "millisecond" delays (Easy=250ms, Medium=150ms).
**How to avoid:** Convert ms to frames at 60fps. 250ms = 15 frames, 150ms = 9 frames, 65ms = 4 frames (already specified in UI-SPEC). Use a frame counter that increments each tick, only allow AI to act when counter exceeds threshold after the triggering event.
**Warning signs:** AI behavior changes at different frame rates; AI acts during hit stop.

### Pitfall 2: Charge Detection for AI
**What goes wrong:** AI tries to use the charge detection system (hold attack for 12+ frames) to trigger power attacks, but the system relies on keyup events which AI synthetic input doesn't naturally produce.
**Why it happens:** The `updateFighterCharge()` function (line 1281-1301) detects power attacks by tracking hold duration and release. AI needs to simulate this.
**How to avoid:** AI must simulate press-and-hold by setting `attack: true` for N frames then `attack: false` for one frame. For fast attacks: set `attack: true` for 1 frame then `attack: false` for 1 frame. For power attacks: set `attack: true` for 12+ frames then `attack: false`. Build this into the AI state machine's "attack" behavior.
**Warning signs:** AI never uses power attacks; AI attack input is always treated as fast attack.

### Pitfall 3: Round Splash Blocking Input But Not Timer
**What goes wrong:** Round splash overlay shows "ROUND N" / "FIGHT!" but the round timer ticks down during the splash, eating 90 frames (1.5 seconds) of fight time.
**Why it happens:** Forgetting to freeze the round timer during splash display.
**How to avoid:** Add a `roundSplashTimer` counter. When > 0, block ALL game updates (fighter movement, timer countdown, hit detection). Only decrement the splash timer itself.
**Warning signs:** Rounds feel short; timer shows 58 instead of 60 at fight start.

### Pitfall 4: Biden Sprite Canvas Size Mismatch
**What goes wrong:** Biden sprite is specified as 2px taller (50px vs 48px) in UI-SPEC, but `spriteWidth`/`spriteHeight` in the definition must match the offscreen canvas size used for rendering.
**How to avoid:** Biden's `spriteHeight` can be 56 (same canvas size as Trump -- canvas is already 56px tall with sprite occupying up to 48px of it). The 2px taller stance is achieved by drawing Biden's body 2px taller WITHIN the same 64x56 canvas. The `renderFighter()` function anchors at bottom-center, so a taller body just extends upward.
**Warning signs:** Biden sprite is clipped at top; Biden's feet float above ground.

### Pitfall 5: Mirror Match Prevention Logic
**What goes wrong:** CPU can select the same fighter as player, or selection logic allows both to pick locked slots.
**Why it happens:** Off-by-one in available fighter array, or forgetting to exclude player's pick from CPU pool.
**How to avoid:** When player confirms selection, CPU picks from `availableFighters.filter(f => f !== selectedP1Def)`. With only 2 fighters in Phase 2, CPU always gets the other one. Simple.
**Warning signs:** Both fighters are Trump; both fighters are Biden.

### Pitfall 6: Victory Screen Sprite Rendering Without Fight Context
**What goes wrong:** Victory scene tries to render winner/loser sprites but `player` and `cpu` objects have been garbage collected or their state is corrupted after fight scene exit.
**Why it happens:** Scene transitions clear fight state, but victory scene needs sprite references.
**How to avoid:** In `victoryScene.enter()`, capture references to the winner and loser fighter definitions and their final states. Store as scene-local properties. Use `spriteCache` (which persists) to render the sprites.
**Warning signs:** Victory screen shows blank space where sprites should be; crash on scene transition.

## Code Examples

### Biden Fighter Definition Structure (template)
```javascript
// Source: Modeled on TRUMP definition at line 576-697 of index.html
const BIDEN = {
  name: 'Biden',
  spriteWidth: 64,
  spriteHeight: 56,
  stats: {
    walkSpeed: 1.2,       // slowest (Trump is 1.8)
    jumpForce: -7.5,
    gravity: 0.45,
    health: 1200,         // highest (Trump is 1000)
    power: 1.0,           // medium (Trump is 1.3)
    reach: 1.3,           // longest (Trump is 1.15)
    hurtboxWidth: 40,     // wider (Trump is 36) -- conveys bulk
    hurtboxHeight: 50,    // taller (Trump is 48)
    crouchHurtboxHeight: 32
  },
  attacks: {
    fast: {
      startup: 5, active: 4, recovery: 9,   // slightly slower than Trump
      damage: 50, maxChain: 3,               // fewer chain hits (grappler)
      hitbox: { x: 22, y: -22, w: 34, h: 18 }, // longer reach
      hitstun: 14
    },
    power: {
      startup: 14, active: 6, recovery: 20,  // slower wind-up, longer active
      damage: 170, maxChain: 1,
      hitbox: { x: 18, y: -30, w: 46, h: 26 }, // biggest hitbox
      hitstun: 28
    }
  },
  sprites: {
    idle: [/* 4 frames from drawBidenBase */],
    walkForward: [/* 4 frames */],
    walkBack: [/* 4 frames */],
    jumpRise: [/* 4 frames */],
    jumpFall: [/* 4 frames */],
    crouch: [/* 4 frames */],
    fastAttack: [/* 4 frames */],
    powerAttack: [/* 4 frames */],
    block: [/* 4 frames */],
    hitstun: [/* 4 frames */],
    knockdown: [/* 4 frames */]
  }
};
```

### AI State Machine Pattern
```javascript
// Source: Standard fighting game AI pattern (approach/pressure/back-off/punish)
const AI_STATE = {
  APPROACH: 'approach',
  PRESSURE: 'pressure',
  BACK_OFF: 'backOff',
  PUNISH: 'punish'
};

function createAIController(difficulty) {
  const cfg = AI_DIFFICULTIES[difficulty];
  return {
    state: AI_STATE.APPROACH,
    reactionBuffer: 0,        // frames since last "event" AI noticed
    attackSequenceFrames: 0,  // frames into current attack sequence
    attackTarget: null,       // 'fast' or 'power'
    decisionCooldown: 0,      // frames until next decision

    generateInput(cpu, player, frameCount) {
      const input = { left:false, right:false, up:false, down:false,
                      attack:false, block:false, ultimate:false, enter:false,
                      attackPressed:false, attackReleased:false, attackBuffered:false };

      if (cpu.knockedDown || cpu.state === 'knockdown') return input;

      const dist = Math.abs(cpu.x - player.x);
      const playerAttacking = player.state === 'fastAttack' || player.state === 'powerAttack';

      // Reaction delay: only update decisions every N frames
      this.decisionCooldown = Math.max(0, this.decisionCooldown - 1);

      // State transitions (with reaction delay)
      if (this.decisionCooldown <= 0) {
        this.decisionCooldown = cfg.reactionFrames;
        // ... state transition logic based on dist, playerAttacking, cpu.health
      }

      // Generate input based on current AI state
      // ... (approach walks forward, pressure attacks, back_off retreats, punish blocks+counters)
      return input;
    }
  };
}
```

### Character Select Scene Pattern
```javascript
// Source: Based on existing scene pattern (titleScene at line 312-349)
const charSelectScene = {
  _cursorPos: 0,          // 0=Trump, 1=Biden
  _difficulty: 1,         // 0=Easy, 1=Medium, 2=Hard
  _confirmed: false,
  _frameCount: 0,
  _availableFighters: [],  // [TRUMP, BIDEN]
  _inputCooldown: 0,       // Prevent rapid cursor movement

  enter() {
    this._cursorPos = 0;
    this._difficulty = 1;
    this._confirmed = false;
    this._frameCount = 0;
    this._availableFighters = [TRUMP, BIDEN];
    this._inputCooldown = 0;
  },

  update(input) {
    this._frameCount++;
    if (this._confirmed) return null;  // Wait for transition

    // Cursor movement with cooldown (12 frames per UI-SPEC)
    if (this._inputCooldown > 0) { this._inputCooldown--; }
    else {
      if (input.left || input.right) {
        // Move cursor between available fighters (skip locked slots)
        this._inputCooldown = 12;
      }
      if (input.up || input.down) {
        // Cycle difficulty
        this._inputCooldown = 12;
      }
    }

    if (input.enter) {
      selectedP1Def = this._availableFighters[this._cursorPos];
      // CPU gets the other fighter
      selectedP2Def = this._availableFighters.find(f => f !== selectedP1Def);
      selectedDifficulty = this._difficulty;
      this._confirmed = true;
      return 'fight';
    }
    return null;
  },

  render(ctx) {
    // Per UI-SPEC layout at 02-UI-SPEC.md lines 185-220
  }
};
```

### Round Splash Integration Pattern
```javascript
// In fightScene -- new state variable
let roundSplashTimer = 0;   // 90 = showing ROUND, 30 = showing FIGHT, 0 = fighting
let roundSplashPhase = '';   // 'round' or 'fight'

// In fightScene.enter() -- start first round with splash
roundSplashTimer = 90;
roundSplashPhase = 'round';

// In fightScene.update() -- before any game logic
if (roundSplashTimer > 0) {
  roundSplashTimer--;
  if (roundSplashTimer === 30) roundSplashPhase = 'fight';
  return null;  // Block ALL input and game updates during splash
}
```

## State of the Art

| Old Approach (Phase 1) | Current Approach (Phase 2) | Impact |
|-------------------------|---------------------------|--------|
| Hardcoded TRUMP + PLACEHOLDER_CPU in fight scene | Selected fighter defs passed from CharSelect | Enables any fighter combination |
| Minimal CPU AI (walk + random block at 0.3%) | FSM-based AI with 4 states and difficulty-scaled parameters | CPU opponent feels like a real fight |
| Title -> Fight direct transition | Title -> CharSelect -> Fight flow | Player has agency over fighter + difficulty |
| KO splash only (no round start splash) | ROUND N + FIGHT! splash before each round | Standard fighting game ceremony |
| Text-only victory screen | Winner/loser sprite display on victory | Visual payoff for winning |

## Open Questions

1. **AI combo input simulation timing**
   - What we know: AI needs to simulate attack-release-attack cycles to chain combos. The charge detection system uses `_prevAttack` and `_chargeFrames` per fighter.
   - What's unclear: Exact frame timing for AI to reliably trigger fast attack chains without accidentally triggering power attacks (must release before 12 frames).
   - Recommendation: AI sets `attack: true` for exactly 2 frames, then `attack: false` for 1 frame, repeating for combo chains. This guarantees fast attack detection (well under 12-frame charge threshold) with 1-frame gap for release detection.

2. **Portrait pre-rendering timing**
   - What we know: Portraits are simplified bust views at 40x48 pixels per UI-SPEC.
   - What's unclear: Whether to pre-render portraits at app init or lazily when CharSelect scene enters.
   - Recommendation: Pre-render at init time alongside stage background. Portraits are small (40x48) and there are only 2-3 of them. No performance concern.

3. **FLW-03 (Stage Select) handling**
   - What we know: UI-SPEC explicitly defers stage select to Phase 3 since only White House exists.
   - What's unclear: Whether to add a skeleton `stageSelect` scene that auto-skips, or just hardcode White House.
   - Recommendation: Hardcode White House in Phase 2. Add stage select scene in Phase 3 when content exists. No skeleton needed -- the scene manager makes adding scenes trivial.

## Sources

### Primary (HIGH confidence)
- `index.html` (2063 lines) -- Complete Phase 1 implementation, all code patterns directly observed
- `02-UI-SPEC.md` -- Phase 2 visual and interaction contract
- `REQUIREMENTS.md` -- Phase requirement definitions (FTR-05, AI-01 through AI-05, FLW-02/03/05/06/07)
- `ROADMAP.md` -- Phase dependencies and success criteria
- `STATE.md` -- Phase 1 decisions and accumulated context

### Secondary (MEDIUM confidence)
- `CLAUDE.md` -- Project constraints (zero-dependency, single HTML file, Canvas 2D only)
- Fighting game AI patterns (training data knowledge -- standard FSM approach/pressure/retreat pattern is well-established in the genre)

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- zero-dependency constraint eliminates all library choices; pure continuation of Phase 1 patterns
- Architecture: HIGH -- all patterns directly observed in Phase 1 codebase; scene manager, fighter definitions, state machine, sprite cache all verified
- Pitfalls: HIGH -- charge detection edge case, splash timer interaction, and sprite sizing issues identified from direct code analysis
- AI design: MEDIUM -- FSM with reaction delay is standard for fighting games, but specific tuning values (combo attempt rates, retreat thresholds) will need playtesting

**Research date:** 2026-03-28
**Valid until:** 2026-04-28 (stable -- single-file game with no external dependencies to go stale)

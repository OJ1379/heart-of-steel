# Phase 1: Engine + Core Combat - Research

**Researched:** 2026-03-28
**Domain:** HTML5 Canvas 2D fighting game engine, single-file delivery
**Confidence:** HIGH

## Summary

Phase 1 builds the entire game engine and combat foundation in a single HTML file. The technology stack is fully locked by CLAUDE.md: Canvas 2D API, fixed-timestep game loop, AABB collision, frame-data-driven attacks, offscreen canvas sprite caching, zero external dependencies. There are no library choices to make -- this is vanilla JavaScript throughout.

The primary challenge is architectural: structuring ~3000-5000 lines of JavaScript inside one HTML file so that (a) the fighter data format is truly data-driven (FTR-03 -- adding fighters later requires zero engine changes), (b) the scene manager cleanly separates title/fight/victory states, and (c) the combat system handles hit stop, input buffering, and combo chains with frame-perfect timing. The secondary challenge is procedural pixel art -- the Trump sprite and Oval Office background must look good at 48px and 400x240 respectively, drawn entirely with fillRect calls.

**Primary recommendation:** Structure the file in clearly separated sections (engine core, input system, combat system, fighter definitions, stage definitions, scene manager, rendering, HUD, main) and build the fighter definition as a plain object template from day one so Phase 2+ fighters slot in with zero engine changes.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- **D-01:** Fighter sprites are 48px tall at 400x240 internal resolution (~1/5 screen height)
- **D-02:** 4 animation frames per state across all animation states (~44 offscreen canvases total at init)
- **D-03:** White House Oval Office background built to ship quality in Phase 1 -- not a placeholder
- **D-04:** Placeholder CPU walks toward player at 50% speed and randomly blocks 20% of the time. Never attacks. Respects health and KO system.
- **D-05:** Frame data delegated to Claude. Target: SF2-inspired (fast attack ~4/4/8, power attack ~12/5/18, combo chain window 6 frames)
- **D-06:** Hit stop locked: 3 frames normal, 5 frames power attack (CMB-06)

### Claude's Discretion
- Combat frame data specifics (startup/active/recovery counts per move)
- Per-character stat variations (speed, power, reach multipliers)
- Color palette choices for sprites and stage art
- Internal code architecture details (state machine structure, hitbox data shape, etc.)
- Animation easing and timing within stated frame budgets

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| ENG-01 | Single HTML file, file:// compatible, no external deps | Architecture pattern: single file structure with inline script |
| ENG-02 | Fixed-timestep 60fps game loop | Game loop pattern with accumulator (see Architecture Patterns) |
| ENG-03 | Canvas 2D, integer coords, antialiasing disabled, pixelated CSS | Canvas setup pattern (see Code Examples) |
| ENG-04 | Offscreen canvas sprite caching, drawImage per frame | Sprite cache init pattern (see Architecture Patterns) |
| ENG-05 | Input state map + 6-frame input buffer | Input system pattern (see Architecture Patterns) |
| ENG-06 | Scene manager: Title -> CharSelect -> StageSelect -> Fight -> Victory | State machine scene manager pattern; Phase 1 only needs Title -> Fight (CharSelect/StageSelect in Phase 2) |
| FTR-02 | Animated states: idle, walk, jump, crouch, attacks, block, hitstun, knockdown, victory | Hierarchical state machine pattern (see Architecture Patterns) |
| FTR-03 | Data-driven fighter definitions -- adding a fighter requires no engine changes | Fighter definition object template (see Code Examples) |
| FTR-04 | Trump: brash brawler, wider swings, slower walk, highest power | Frame data and stat tuning (Claude's discretion per D-05) |
| CMB-01 | Fast attack: quick, chainable into 3-4 hit combos within buffer window | Combo chain system using input buffer (see Architecture Patterns) |
| CMB-02 | Power attack: hold 12+ frames, heavy hit, telegraphed wind-up | Charge detection in input system (see Architecture Patterns) |
| CMB-03 | Block: hold K/X, 80% damage reduction | Block state in fighter FSM, damage calculation pattern |
| CMB-06 | Hit stop: 3 frames normal, 5 frames power | Hit stop freeze pattern (see Code Examples) |
| CMB-07 | AABB hitbox/hurtbox collision per attack frame data | AABB overlap check + per-frame hitbox activation (see Code Examples) |
| CMB-09 | Walk forward/back, jump (parabolic), crouch (shrink hurtbox) | Movement state handling in fighter FSM |
| STG-01 | White House Oval Office interior, canvas pixel art | Procedural background rendering to offscreen canvas (see Architecture Patterns) |
| FLW-01 | Title screen: pixel art logo, "PRESS ENTER TO START" | Title scene in scene manager |
| FLW-04 | Fight HUD: health bars, timer, round counter, 90s arcade style | HUD rendering pattern (see Code Examples) |
</phase_requirements>

## Project Constraints (from CLAUDE.md)

- **Format**: Single `.html` file -- no build step, no dependencies, no CDN links
- **Rendering**: Canvas API only for game graphics -- CSS only for outer shell/UI chrome
- **Input**: Keyboard only (WASD/arrows + J/Z/K/X/L/C)
- **Compatibility**: Must run from `file://` without CORS issues (no fetch, no workers)
- **Assets**: Zero external assets -- all art generated procedurally in canvas code
- **Game loop**: Fixed timestep with requestAnimationFrame -- no setInterval/setTimeout
- **Sprites**: Pre-render to offscreen canvases at init, drawImage per frame -- no per-frame fillRect
- **Collision**: AABB hitbox/hurtbox -- no pixel-perfect or SAT
- **Attacks**: Frame-data-driven (startup/active/recovery frame counts)
- **State machines**: Hierarchical FSM for fighter behavior
- **Scaling**: `image-rendering: pixelated` + `imageSmoothingEnabled = false`
- **Coordinates**: Integer pixel coordinates only

## Standard Stack

### Core
| Technology | Version | Purpose | Why Standard |
|------------|---------|---------|--------------|
| Canvas 2D API | Baseline 2015+ | All game rendering | Universal browser support, locked by CLAUDE.md |
| requestAnimationFrame | Baseline 2015+ | Game loop timing | Only correct way to sync with display refresh |
| KeyboardEvent (event.key) | Baseline 2015+ | Player input | Layout-aware, replaces deprecated keyCode |
| document.createElement('canvas') | Baseline 2015+ | Offscreen sprite cache | Pre-render sprites once, blit with drawImage() |

### Supporting
| Pattern | Purpose | When to Use |
|---------|---------|-------------|
| Fixed-timestep accumulator | Deterministic physics/combat | Always -- fighting games need frame-perfect timing |
| Hierarchical FSM | Fighter state control | Always -- reduces state explosion for grounded/aerial superstates |
| AABB collision | Hitbox/hurtbox overlap | Always -- fast, sufficient for axis-aligned fighting game hitboxes |
| Frame-data tables | Attack timing definitions | Always -- startup/active/recovery as frame counts |
| Offscreen canvas cache | 60fps performance | Always -- avoids hundreds of fillRect calls per frame |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Single canvas | Multiple stacked canvases (BG/gameplay/HUD) | Better perf if BG rarely changes; more complexity. Start single, optimize if needed. |
| Object FSM | Switch/case state | Switch is simpler for small state counts but becomes unmaintainable at 10+ states. Object FSM from day one. |
| Inline frame data | External JSON | External violates single-file constraint. Inline objects work fine. |

## Architecture Patterns

### Recommended File Structure (within single HTML file)

```html
<!DOCTYPE html>
<html>
<head>
  <style>/* CSS: pixelated scaling, centering, UI chrome */</style>
</head>
<body>
  <canvas id="game"></canvas>
  <script>
  // ===== CONSTANTS & CONFIG =====
  // Internal resolution, colors, timing constants

  // ===== UTILITY FUNCTIONS =====
  // AABB overlap, lerp, clamp, drawPixelText

  // ===== INPUT SYSTEM =====
  // Key state map, input buffer, charge detection

  // ===== GAME LOOP =====
  // Fixed timestep accumulator, update/render split

  // ===== SCENE MANAGER =====
  // State machine: title, fight, victory. Transition logic.

  // ===== COMBAT SYSTEM =====
  // Damage calc, hit detection, hit stop, combo chain logic

  // ===== FIGHTER BASE =====
  // State machine, state transitions, physics (gravity, movement)
  // Generic update/render that reads from fighter definition data

  // ===== FIGHTER DEFINITIONS =====
  // Trump definition: stats, frame data, hitboxes, sprite draw functions
  // Placeholder CPU definition

  // ===== STAGE DEFINITIONS =====
  // White House: background draw function, stage bounds

  // ===== HUD =====
  // Health bars, timer, round counter rendering

  // ===== SPRITE CACHE =====
  // Init function: iterate fighter defs, pre-render all frames to offscreen canvases

  // ===== INIT & MAIN =====
  // Setup canvas, cache sprites, start game loop
  </script>
</body>
</html>
```

### Pattern 1: Fixed-Timestep Game Loop with Accumulator

**What:** Decouple physics/logic updates (fixed 60 ticks/sec) from rendering (variable, synced to display).
**When to use:** Always. Frame-perfect combo windows require deterministic tick rate.

```javascript
const TICK_RATE = 1 / 60;
const MAX_FRAME_TIME = 0.25; // Prevent spiral of death
let accumulator = 0;
let lastTime = 0;

function gameLoop(timestamp) {
  requestAnimationFrame(gameLoop);
  let dt = (timestamp - lastTime) / 1000;
  lastTime = timestamp;
  if (dt > MAX_FRAME_TIME) dt = MAX_FRAME_TIME;

  accumulator += dt;
  while (accumulator >= TICK_RATE) {
    update(); // Fixed-step logic: input, physics, combat, state machines
    accumulator -= TICK_RATE;
  }
  render(); // Variable: draw current state
}
requestAnimationFrame(gameLoop);
```

**Key detail:** The `MAX_FRAME_TIME` cap prevents the "spiral of death" where a long pause (e.g., tab switch) causes hundreds of catch-up ticks. 0.25s = max 15 catch-up ticks.

### Pattern 2: Fighter State Machine (Hierarchical)

**What:** Each fighter runs a hierarchical FSM. Top-level states: grounded, airborne, hitstun, knockdown, KO. Sub-states under grounded: idle, walkForward, walkBack, crouch, fastAttack, powerAttack, block. Sub-states under airborne: jumpRise, jumpFall.

**Why hierarchical:** Grounded superstate handles "can I jump?" and "can I crouch?" once, not in every sub-state. Reduces transition rules from O(n^2) to O(n).

```javascript
// State object pattern -- each state has enter/update/exit
const STATES = {
  idle: {
    enter(fighter) { fighter.currentFrame = 0; },
    update(fighter, input) {
      if (input.up) return 'jumpRise';
      if (input.down) return 'crouch';
      if (input.left) return 'walkBack';
      if (input.right) return 'walkForward';
      if (input.attack) return 'fastAttack';
      if (input.block) return 'block';
      return null; // Stay in current state
    },
    exit(fighter) {}
  },
  fastAttack: {
    enter(fighter) {
      fighter.stateFrame = 0;
      fighter.attackPhase = 'startup';
      fighter.comboCount = (fighter.comboCount || 0) + 1;
    },
    update(fighter, input) {
      fighter.stateFrame++;
      const fd = fighter.def.attacks.fast;
      if (fighter.stateFrame <= fd.startup) {
        fighter.attackPhase = 'startup';
      } else if (fighter.stateFrame <= fd.startup + fd.active) {
        fighter.attackPhase = 'active';
      } else if (fighter.stateFrame <= fd.startup + fd.active + fd.recovery) {
        fighter.attackPhase = 'recovery';
        // Check combo chain: if attack pressed within buffer window during recovery
        if (input.attackBuffered && fighter.comboCount < fd.maxChain) {
          return 'fastAttack'; // Chain into next hit
        }
      } else {
        fighter.comboCount = 0;
        return 'idle';
      }
      return null;
    },
    exit(fighter) {}
  }
  // ... more states
};
```

### Pattern 3: Data-Driven Fighter Definition (FTR-03 Critical)

**What:** A fighter is a plain JavaScript object. The engine reads this object generically. Adding a new fighter = adding a new object.

```javascript
const TRUMP = {
  name: 'Trump',
  stats: {
    walkSpeed: 2.0,      // pixels per tick
    jumpForce: -8.0,     // initial Y velocity
    gravity: 0.5,        // Y acceleration per tick
    health: 1000,
    power: 1.3,          // damage multiplier
    reach: 1.0,          // hitbox width multiplier
    hurtboxWidth: 36,
    hurtboxHeight: 48,
    crouchHurtboxHeight: 30
  },
  attacks: {
    fast: {
      startup: 4, active: 4, recovery: 8,
      damage: 60, maxChain: 4,
      hitbox: { x: 20, y: -20, w: 30, h: 16 }, // relative to fighter origin
      hitstun: 12 // frames opponent stays in hitstun
    },
    power: {
      startup: 12, active: 5, recovery: 18,
      damage: 150,
      hitbox: { x: 16, y: -28, w: 40, h: 24 },
      hitstun: 24
    }
  },
  // Sprite draw functions: each returns draw commands for offscreen canvas
  sprites: {
    idle: [drawTrumpIdle0, drawTrumpIdle1, drawTrumpIdle2, drawTrumpIdle3],
    walkForward: [/* 4 frames */],
    fastAttack: [/* 4 frames mapped to startup/active/recovery */],
    // ... all states
  }
};
```

**Critical insight for FTR-03:** The engine must NEVER reference `TRUMP` by name in gameplay logic. All gameplay code operates on `fighter.def.stats.walkSpeed`, `fighter.def.attacks.fast.startup`, etc. The only place a fighter name appears is in the definition array and UI labels.

### Pattern 4: Input System with Buffer and Charge Detection

**What:** Track key state (up/down) plus a circular buffer of recent inputs for combo detection and charge detection for power attacks.

```javascript
const keys = {};
const INPUT_BUFFER_SIZE = 6;
const inputBuffer = [];

window.addEventListener('keydown', e => { keys[e.key] = true; });
window.addEventListener('keyup', e => { keys[e.key] = false; });

// Each tick, snapshot relevant input
function readInput() {
  const input = {
    left: keys['a'] || keys['ArrowLeft'],
    right: keys['d'] || keys['ArrowRight'],
    up: keys['w'] || keys['ArrowUp'],
    down: keys['s'] || keys['ArrowDown'],
    attack: keys['j'] || keys['z'],
    block: keys['k'] || keys['x'],
  };
  inputBuffer.push(input);
  if (inputBuffer.length > INPUT_BUFFER_SIZE) inputBuffer.shift();
  return input;
}

// Charge detection: check if attack key held for >= 12 frames
let attackHeldFrames = 0;
function updateChargeDetection(input) {
  if (input.attack) {
    attackHeldFrames++;
  } else {
    if (attackHeldFrames >= 12) {
      // Power attack trigger on release
      return 'power';
    }
    if (attackHeldFrames > 0) {
      return 'fast'; // Short press = fast attack
    }
    attackHeldFrames = 0;
  }
  return null;
}
```

**Key design decision for CMB-02:** Power attack triggers on **release** after holding 12+ frames. This means the wind-up animation plays while holding, and the strike happens on release. This is the standard fighting game charge pattern and feels natural.

### Pattern 5: Hit Stop Implementation

**What:** On hit, both fighters freeze for N frames. The game loop skips their state machine updates but continues rendering (so particles/HUD still animate).

```javascript
let hitStopFramesRemaining = 0;

function applyHitStop(frames) {
  hitStopFramesRemaining = frames;
}

function update() {
  // Input always reads (responsiveness)
  const input = readInput();

  if (hitStopFramesRemaining > 0) {
    hitStopFramesRemaining--;
    // Skip fighter updates -- they're frozen
    // But still update HUD timer, particles, etc.
    updateHUD();
    return;
  }

  updateFighter(player, input);
  updateFighter(cpu, cpuInput);
  checkCollisions();
  updateHUD();
}
```

### Pattern 6: Offscreen Canvas Sprite Cache

**What:** At init, iterate every fighter definition, every state, every frame. For each, create a small offscreen canvas, call the sprite draw function to fill it with fillRect pixel art, store the canvas reference. During gameplay, drawImage() to blit.

```javascript
const spriteCache = {};

function initSpriteCache(fighterDefs) {
  for (const def of fighterDefs) {
    spriteCache[def.name] = {};
    for (const [state, frames] of Object.entries(def.sprites)) {
      spriteCache[def.name][state] = frames.map((drawFn, i) => {
        const oc = document.createElement('canvas');
        oc.width = def.spriteWidth || 64;  // Wider than hurtbox for attack reach
        oc.height = def.spriteHeight || 56;
        const octx = oc.getContext('2d');
        drawFn(octx); // Each draw function uses fillRect to paint pixels
        return oc;
      });
    }
  }
}

// During render:
function renderFighter(fighter) {
  const frames = spriteCache[fighter.def.name][fighter.state];
  const frame = frames[fighter.currentFrame % frames.length];
  ctx.save();
  if (fighter.facing === -1) {
    ctx.scale(-1, 1);
    ctx.drawImage(frame, -fighter.x - frame.width, fighter.y - frame.height);
  } else {
    ctx.drawImage(frame, fighter.x, fighter.y - frame.height);
  }
  ctx.restore();
}
```

### Pattern 7: Canvas Setup for Pixel Art

```javascript
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d', { alpha: false }); // Perf hint

// Internal resolution
const GAME_W = 400;
const GAME_H = 240;
canvas.width = GAME_W;
canvas.height = GAME_H;

// CSS scales up with nearest-neighbor
canvas.style.imageRendering = 'pixelated';
canvas.style.imageRendering = 'crisp-edges'; // Firefox fallback
ctx.imageSmoothingEnabled = false;

// Scale to fill window while maintaining aspect ratio
function resizeCanvas() {
  const scale = Math.min(
    window.innerWidth / GAME_W,
    window.innerHeight / GAME_H
  );
  canvas.style.width = Math.floor(GAME_W * scale) + 'px';
  canvas.style.height = Math.floor(GAME_H * scale) + 'px';
}
window.addEventListener('resize', resizeCanvas);
resizeCanvas();
```

### Anti-Patterns to Avoid
- **Per-frame fillRect sprite drawing:** Hundreds of fillRect calls per character per frame kills performance. Always use offscreen cache + drawImage.
- **Float pixel coordinates:** Sub-pixel rendering causes blurry edges. Always Math.floor/Math.round coordinates.
- **Variable timestep for combat:** `delta * speed` makes combo windows frame-rate dependent. Fixed timestep only.
- **Hardcoding fighter names in engine:** Breaks FTR-03 data-driven requirement. Engine must be fighter-agnostic.
- **Using `keyCode`:** Deprecated. Use `event.key`.
- **Forgetting `image-rendering: pixelated` on BOTH CSS and canvas context:** Need both `canvas.style.imageRendering` and `ctx.imageSmoothingEnabled = false`.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Game loop timing | Custom setInterval timer | requestAnimationFrame + fixed accumulator | Browser-synced, battery-efficient, handles background tabs |
| Text rendering | Pixel-by-pixel font builder | ctx.fillText with bitmap font or simple pixel font function | A minimal pixel font (A-Z, 0-9) is fine but keep it small -- used only for HUD |
| Aspect ratio scaling | Manual viewport math | CSS object-fit or the resize handler pattern above | The pattern is standard and handles edge cases |

**Key insight:** Since this is zero-dependency, everything IS hand-rolled. The "don't hand-roll" principle here means: don't over-engineer systems that can be simple. A 3-state scene manager (title/fight/victory) is enough for Phase 1 -- don't build a full ECS or event bus.

## Common Pitfalls

### Pitfall 1: Spiral of Death in Game Loop
**What goes wrong:** After a tab switch, `dt` is huge (seconds), accumulator fills up, hundreds of update ticks run synchronously, browser freezes.
**Why it happens:** No cap on accumulated time.
**How to avoid:** Clamp `dt` to MAX_FRAME_TIME (0.25s). After a long pause, accept dropped ticks rather than catching up.
**Warning signs:** Game stutters when switching tabs and back.

### Pitfall 2: Offscreen Canvas Size Mismatch
**What goes wrong:** Sprite draws are clipped or blurry because offscreen canvas is wrong size.
**Why it happens:** Fighter sprites need to be wider than hurtbox (attack animations extend limbs) but offscreen canvas was sized to hurtbox.
**How to avoid:** Size offscreen canvases to maximum sprite extent (e.g., 64x56 for 48px tall fighter with attack reach). Use consistent anchor point (bottom-center of fighter).
**Warning signs:** Attack frames look cut off on one side.

### Pitfall 3: Hit Detection Runs During Startup/Recovery
**What goes wrong:** Attacks hit before the visible swing, or during pullback after the swing.
**Why it happens:** Hitbox is always active instead of only during the "active" phase.
**How to avoid:** Only check hitbox collision when `fighter.attackPhase === 'active'`. Startup and recovery have no hitbox.
**Warning signs:** Attacks feel unfair -- landing hits with no visible contact.

### Pitfall 4: Both Fighters Hit Each Other Same Frame
**What goes wrong:** Both fast attacks land simultaneously, double damage, feels wrong.
**Why it happens:** No priority system for simultaneous hits.
**How to avoid:** Process attacker who entered active frames first. Or: both trade (this is authentic SF2 behavior). Decision: allow trades -- it feels fair and is genre-standard.
**Warning signs:** Both health bars drop simultaneously.

### Pitfall 5: Input Feels Laggy
**What goes wrong:** Player presses attack but it takes 2-3 frames to register.
**Why it happens:** Input read happens after state update, or input buffer isn't checked during recovery frames.
**How to avoid:** Read input FIRST in the update loop, before state machine updates. Check input buffer during attack recovery for combo chains.
**Warning signs:** "The controls feel sluggish" feedback.

### Pitfall 6: Power Attack Charge Conflicts with Fast Attack
**What goes wrong:** Every attack starts as a potential power attack, causing a delay before fast attacks register.
**Why it happens:** System waits to see if player will hold long enough for power attack.
**How to avoid:** Fast attack triggers on press (immediately enters fast attack state). Power attack triggers on release after 12+ frames held. The attack animation starts immediately -- if held, it transitions to power attack wind-up after the threshold. OR: fast attack is tap, power attack requires the button to be held BEFORE starting -- so fast attack fires on keydown, power attack fires on keyup after 12+ held frames.
**Warning signs:** Fast attacks feel delayed.

### Pitfall 7: CSS Scaling Breaks Pixel Crispness
**What goes wrong:** Pixel art looks blurry when scaled up.
**Why it happens:** Missing `image-rendering: pixelated` or browser-specific prefixes.
**How to avoid:** Apply BOTH: CSS `image-rendering: pixelated` (and `crisp-edges` fallback) AND `ctx.imageSmoothingEnabled = false`. Set canvas dimensions to internal resolution (400x240) and use CSS to scale up.
**Warning signs:** Edges look blurry instead of sharp blocks.

## Code Examples

### AABB Collision Check
```javascript
function aabbOverlap(a, b) {
  return a.x < b.x + b.w &&
         a.x + a.w > b.x &&
         a.y < b.y + b.h &&
         a.y + a.h > b.y;
}

function checkHit(attacker, defender) {
  if (attacker.attackPhase !== 'active') return false;

  const atk = attacker.def.attacks[attacker.currentAttackType];
  const dir = attacker.facing; // 1 = right, -1 = left

  // Hitbox in world space
  const hitbox = {
    x: attacker.x + atk.hitbox.x * dir - (dir === -1 ? atk.hitbox.w : 0),
    y: attacker.y + atk.hitbox.y,
    w: atk.hitbox.w,
    h: atk.hitbox.h
  };

  // Defender hurtbox
  const hurtH = defender.state === 'crouch'
    ? defender.def.stats.crouchHurtboxHeight
    : defender.def.stats.hurtboxHeight;
  const hurtbox = {
    x: defender.x - defender.def.stats.hurtboxWidth / 2,
    y: defender.y - hurtH,
    w: defender.def.stats.hurtboxWidth,
    h: hurtH
  };

  return aabbOverlap(hitbox, hurtbox);
}
```

### Simple Pixel Font for HUD
```javascript
// Minimal approach: use fillRect to draw each character in a fixed grid
// OR use canvas fillText with a monospace font at small size, then cache
function drawText(ctx, text, x, y, color, scale) {
  ctx.fillStyle = color;
  ctx.font = `${8 * scale}px monospace`;
  ctx.textBaseline = 'top';
  ctx.fillText(text, Math.floor(x), Math.floor(y));
}
// Note: fillText at small sizes with imageSmoothingEnabled=false
// gives acceptable pixel-font look. If not crisp enough, build a
// simple bitmap font from fillRect (7x5 grid per character).
```

### Health Bar Rendering (90s Arcade Style)
```javascript
function drawHealthBar(ctx, x, y, w, h, current, max, isPlayer) {
  const pct = current / max;

  // Border (dark outline)
  ctx.fillStyle = '#000';
  ctx.fillRect(x - 1, y - 1, w + 2, h + 2);

  // Background (empty health)
  ctx.fillStyle = '#333';
  ctx.fillRect(x, y, w, h);

  // Health fill (player fills left-to-right, CPU right-to-left)
  const fillW = Math.floor(w * pct);
  const color = pct > 0.5 ? '#0f0' : pct > 0.25 ? '#ff0' : '#f00';
  ctx.fillStyle = color;
  if (isPlayer) {
    ctx.fillRect(x, y, fillW, h);
  } else {
    ctx.fillRect(x + w - fillW, y, fillW, h);
  }

  // Highlight line (top edge for 3D look)
  ctx.fillStyle = 'rgba(255,255,255,0.3)';
  ctx.fillRect(x, y, fillW, 1);
}
```

### Scene Manager (Minimal for Phase 1)
```javascript
const scenes = {
  title: {
    update(input) {
      if (input.enter) return 'fight';
      return null;
    },
    render(ctx) {
      // Draw title screen: logo, "PRESS ENTER TO START"
    }
  },
  fight: {
    enter() {
      // Init fighters, reset health, start timer
    },
    update(input) {
      // Main gameplay update
      if (matchOver) return 'victory';
      return null;
    },
    render(ctx) {
      // Draw stage, fighters, HUD
    }
  },
  victory: {
    update(input) {
      if (input.enter) return 'title';
      return null;
    },
    render(ctx) {
      // Draw winner, "PRESS ENTER to play again"
    }
  }
};
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| keyCode for input | event.key | ~2017 | keyCode deprecated, event.key is layout-aware |
| -moz-crisp-edges | image-rendering: pixelated | Firefox 93+ (2021) | Standard `pixelated` works in all modern browsers now; keep `crisp-edges` as fallback |
| Manual vendor prefixes for imageSmoothingEnabled | Standard property | ~2020 | No need for webkit/moz prefixes in 2025+ |
| getContext('2d') | getContext('2d', { alpha: false }) | Always available | Performance hint -- no alpha compositing with page background |

**Deprecated/outdated:**
- `event.keyCode`: Deprecated, use `event.key`
- `-webkit-imageSmoothingEnabled`: Standard `imageSmoothingEnabled` sufficient in 2025+
- `mozImageSmoothingEnabled`: Same -- standard property works everywhere

## Open Questions

1. **Pixel font vs fillText**
   - What we know: `ctx.fillText` at small sizes looks decent with smoothing off. A custom bitmap font (fillRect per pixel) gives perfect control but is significant effort.
   - What's unclear: Whether fillText at 8px on a 400x240 canvas looks crisp enough for "90s arcade" feel across all browsers.
   - Recommendation: Start with fillText. If it looks bad, build a minimal bitmap font for just the HUD characters (0-9, A-Z, punctuation). This is a rendering detail that can be swapped without architectural impact.

2. **Placeholder CPU sprite**
   - What we know: D-04 says placeholder walks and blocks. CONTEXT.md says it can be Trump recolored or a generic rectangle.
   - What's unclear: How much effort to invest in placeholder appearance.
   - Recommendation: Recolor Trump sprite (swap palette to gray/blue) -- minimal effort, tests the sprite system with two instances, and validates the "data-driven fighter" pattern early.

3. **Attack input: press vs release**
   - What we know: Fast attack should feel instant. Power attack needs 12-frame hold.
   - What's unclear: Exact UX for distinguishing fast vs power when same button.
   - Recommendation: Fast attack on keydown (immediate). If key stays held for 12+ frames without entering fast attack recovery, transition to power attack wind-up. On keyup, release the power strike. This means: tap = fast, hold = power. The fast attack animation plays immediately either way -- if you keep holding, it transitions to power wind-up after the fast attack completes. This avoids the "laggy fast attack" pitfall.

## Sources

### Primary (HIGH confidence)
- CLAUDE.md -- Full technology stack, Canvas 2D patterns, game loop, input, collision, all locked
- .planning/REQUIREMENTS.md -- All requirement IDs and descriptions for Phase 1
- .planning/phases/01-engine-core-combat/01-CONTEXT.md -- Locked decisions (D-01 through D-06)
- MDN Canvas optimization -- offscreen rendering, alpha:false, integer coordinates
- MDN 2D collision detection -- AABB pattern
- Game Programming Patterns (Robert Nystrom) -- State pattern, game loop pattern

### Secondary (MEDIUM confidence)
- Frame data conventions from CritPoints and Dream Cancel (referenced in CLAUDE.md) -- startup/active/recovery, hitstun/blockstun

### Tertiary (LOW confidence)
- Pixel font rendering quality across browsers -- needs runtime testing

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - Fully locked by CLAUDE.md, no choices to make
- Architecture: HIGH - Standard fighting game patterns well-documented, all decisions locked or delegated
- Pitfalls: HIGH - Common Canvas 2D and game loop issues are well-known
- Sprite art quality: MEDIUM - Procedural pixel art at 48px is achievable but quality depends on implementation skill

**Research date:** 2026-03-28
**Valid until:** Indefinite -- stack is locked, no moving parts or version dependencies

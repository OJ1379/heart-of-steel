# Stack Research

**Domain:** Single-file HTML5 Canvas 2D Fighting Game (zero external dependencies)
**Researched:** 2026-03-28
**Confidence:** HIGH

## Constraint: Zero Dependencies

This project has an absolute constraint: everything ships as a single `.html` file opened from `file://`. No npm, no CDN, no build step, no external images, no Web Workers requiring origin. Every "technology" listed below is a **pattern or browser API**, not an installable package.

---

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Canvas 2D API | Baseline 2015+ | All game rendering | Universal browser support, no WebGL complexity needed for 2D pixel art. `fillRect()` is the workhorse for procedural sprite drawing. |
| requestAnimationFrame | Baseline 2015+ | Game loop timing | Only correct way to sync rendering with display refresh. `setInterval`/`setTimeout` drift, waste battery, and cause jank. |
| KeyboardEvent (event.key) | Baseline 2015+ | Player input | `event.key` is layout-aware ("w" stays "w" regardless of keyboard layout). More reliable than deprecated `keyCode`. |
| OffscreenCanvas (via createElement) | Baseline 2015+ | Sprite caching / pre-rendering | Draw complex sprites once to in-memory canvases, then `drawImage()` each frame. Critical for 60fps with procedural art. NOT the `OffscreenCanvas` constructor (requires origin), but `document.createElement('canvas')`. |
| CSS `image-rendering: pixelated` | Baseline 2020+ | Crisp pixel art scaling | Tells browser to use nearest-neighbor scaling instead of bilinear interpolation. Without this, scaled pixel art looks blurry. |

### Supporting Patterns

| Pattern | Purpose | When to Use |
|---------|---------|-------------|
| Fixed-timestep game loop | Deterministic physics/combat | Always. Fighting games require frame-perfect timing. Variable timestep causes inconsistent combo windows. |
| Hierarchical Finite State Machine | Character behavior control | Always. Each fighter needs states (idle, walk, jump, crouch, attack, hitstun, blockstun, KO). Hierarchy reduces duplication (e.g., "grounded" superstate handles jump/crouch for both idle and walking). |
| AABB collision detection | Hit/hurt box overlap | Always. Rectangles are sufficient for 2D fighting game hitboxes. No need for pixel-perfect or SAT -- AABB is fast and maps directly to fighting game conventions. |
| Frame-data-driven animation | Attack startup/active/recovery | Always. Define attacks as frame counts (e.g., 4 startup, 3 active, 8 recovery) rather than timers. This is how real fighting games work and enables proper hitstun/blockstun/cancel math. |
| Offscreen canvas sprite cache | Performance | Always. Pre-render each animation frame to an in-memory canvas at init time. During gameplay, use `drawImage()` to blit cached frames. Avoids hundreds of `fillRect()` calls per frame per character. |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| Browser DevTools (Chrome/Firefox) | Debugging, profiling | Performance tab for frame timing. Canvas inspector for draw call analysis. |
| Live Server (VS Code extension) | Hot reload during dev | Optional convenience. Game must still work from `file://` -- test without server regularly. |
| No build tools | Constraint compliance | No bundler, no transpiler, no minifier. Write ES2020+ directly -- all target browsers support it natively. |

---

## Architecture Patterns (with rationale)

### 1. Game Loop: Fixed Timestep with Variable Rendering

**Confidence: HIGH** (industry standard, well-documented)

```javascript
const TICK_RATE = 60;
const TICK_DURATION = 1000 / TICK_RATE;
let accumulator = 0;
let lastTime = 0;

function gameLoop(currentTime) {
    const deltaTime = currentTime - lastTime;
    lastTime = currentTime;
    accumulator += deltaTime;

    // Fixed-step updates (may run 0-N times per frame)
    while (accumulator >= TICK_DURATION) {
        processInput();
        updateGameState();   // Physics, combat, AI -- always at 60 ticks/sec
        accumulator -= TICK_DURATION;
    }

    render();  // Runs once per display frame (could be 60hz, 144hz, etc.)
    requestAnimationFrame(gameLoop);
}
requestAnimationFrame(gameLoop);
```

**Why this pattern:** Fighting games are frame-count-based. A jab with "4 frames startup" must always take exactly 4/60th of a second regardless of display refresh rate. Fixed timestep guarantees this. The `while` loop catches up if the browser drops frames. Rendering runs at native display rate for smoothness.

**Why NOT variable timestep:** `deltaTime * speed` causes floating-point drift in combo timing. A 3-frame cancel window might work at 60fps but fail at 144fps due to accumulated rounding. Fixed timestep eliminates this class of bugs entirely.

### 2. Procedural Pixel Art via fillRect + Offscreen Caching

**Confidence: HIGH** (verified MDN, multiple game dev sources)

**Drawing approach:** Define sprites as 2D arrays of color values (palette indices). Each "pixel" in the sprite is a canvas `fillRect()` call at the appropriate scale.

```javascript
// Sprite definition: 2D array of palette indices
const IDLE_FRAME_0 = [
    [0,0,1,1,1,0,0],  // 0=transparent, 1=skin, 2=suit, etc.
    [0,1,1,1,1,1,0],
    // ... more rows
];

// Pre-render to offscreen canvas at init time
function prerenderSprite(pixelData, palette, scale) {
    const canvas = document.createElement('canvas');
    canvas.width = pixelData[0].length * scale;
    canvas.height = pixelData.length * scale;
    const ctx = canvas.getContext('2d');
    for (let y = 0; y < pixelData.length; y++) {
        for (let x = 0; x < pixelData[y].length; x++) {
            if (pixelData[y][x] === 0) continue; // transparent
            ctx.fillStyle = palette[pixelData[y][x]];
            ctx.fillRect(x * scale, y * scale, scale, scale);
        }
    }
    return canvas; // Cache this -- draw with drawImage() during gameplay
}
```

**Why 2D arrays, not raw fillRect calls:** Sprite data becomes data, not code. You can flip sprites horizontally (mirror the array or use `ctx.scale(-1,1)`), swap palettes for alt costumes, and define animations as arrays of frame references. Separating data from rendering is essential when you have 5 fighters x ~12 animation states x ~4-8 frames each = 240-480 unique sprite frames.

**Why NOT ImageData/putImageData:** `putImageData` ignores compositing and transformations. You cannot flip, scale, or alpha-blend sprites drawn with `putImageData`. Using `fillRect` into offscreen canvases and then `drawImage` gives you full compositing support.

### 3. Canvas Setup for Pixel Art

**Confidence: HIGH** (verified MDN documentation)

```javascript
// Internal resolution: small (e.g., 384x216 for 16:9, or 400x240)
const INTERNAL_WIDTH = 400;
const INTERNAL_HEIGHT = 240;

const canvas = document.getElementById('game');
canvas.width = INTERNAL_WIDTH;
canvas.height = INTERNAL_HEIGHT;

const ctx = canvas.getContext('2d', { alpha: false }); // No transparency needed on main canvas
ctx.imageSmoothingEnabled = false; // Nearest-neighbor for drawImage
```

```css
canvas {
    width: 100%;           /* Scale up via CSS */
    max-width: 1200px;
    image-rendering: pixelated;   /* Nearest-neighbor scaling */
    image-rendering: crisp-edges; /* Firefox fallback */
}
```

**Why small internal resolution:** Drawing at native screen resolution (1920x1080) wastes cycles. A 400x240 canvas scaled up via CSS with `image-rendering: pixelated` produces authentic retro visuals and runs faster. Each sprite "pixel" becomes a 3-5 pixel block on screen automatically.

**Why `alpha: false`:** The game canvas has a solid background (stage art). Disabling the alpha channel avoids per-pixel alpha blending with the page background, which MDN confirms improves performance.

### 4. Input System: Key State Map

**Confidence: HIGH** (standard game dev pattern)

```javascript
const keys = {};

window.addEventListener('keydown', (e) => {
    keys[e.key] = true;
    e.preventDefault(); // Prevent browser shortcuts (arrow scroll, etc.)
});

window.addEventListener('keyup', (e) => {
    keys[e.key] = false;
});

// In game update:
function processInput() {
    const input = {
        left:   keys['a'] || keys['ArrowLeft'],
        right:  keys['d'] || keys['ArrowRight'],
        up:     keys['w'] || keys['ArrowUp'],
        down:   keys['s'] || keys['ArrowDown'],
        attack: keys['j'] || keys['z'],
        block:  keys['k'] || keys['x'],
        ult:    keys['l'] || keys['c'],
    };
    player.handleInput(input);
}
```

**Why key state map, not event-driven actions:** `keydown` fires repeatedly with OS key-repeat delay. A fighting game needs to know "is the key held RIGHT NOW" every tick, not "did an event fire." The state map pattern decouples input detection (event listeners) from input consumption (game loop), allowing simultaneous key detection (e.g., crouch + block).

**Why `event.key` not `event.code`:** `event.key` gives the character produced ("w"), while `event.code` gives the physical key ("KeyW"). For WASD controls, `event.key` works correctly on QWERTY, AZERTY, and Dvorak layouts. Since the game supports both WASD and arrow keys, `event.key` is the right choice.

### 5. Hierarchical State Machine for Fighter Logic

**Confidence: HIGH** (verified via Game Programming Patterns, fighting game design literature)

```javascript
// Each state is an object with enter/update/handleInput/exit methods
const STATES = {
    idle: {
        enter(fighter) { fighter.setAnimation('idle'); },
        handleInput(fighter, input) {
            if (input.left) return 'walkBack';
            if (input.right) return 'walkForward';
            if (input.up) return 'jumpStart';
            if (input.down) return 'crouch';
            if (input.attack) return 'fastAttack';
            if (input.block) return 'block';
            return null; // Stay in current state
        },
        update(fighter) { /* idle animation loops */ },
    },
    hitstun: {
        enter(fighter) {
            fighter.setAnimation('hit');
            fighter.stunFrames = fighter.currentHit.hitstun;
        },
        update(fighter) {
            fighter.stunFrames--;
            if (fighter.stunFrames <= 0) return 'idle';
            return null;
        },
        handleInput() { return null; }, // No input during hitstun
    },
    // ... more states
};
```

**Why hierarchical:** Ground states (idle, walk, crouch) share transitions (can all be hit, can all block). Rather than duplicating "if hit, go to hitstun" in every grounded state, define a `grounded` superstate that handles common transitions. Substates only define their unique behavior. This follows the pattern from Game Programming Patterns (Robert Nystrom) and maps directly to fighting game conventions.

**Why NOT a flat enum+switch:** With 5 fighters x ~12 states, a switch statement becomes unmaintainable. Object-based states let you define state-specific data (frame counters, per-attack properties) without polluting the fighter object. Each state's `enter()` method replaces scattered initialization code.

### 6. Frame-Data-Driven Attack System

**Confidence: HIGH** (standard fighting game architecture)

```javascript
const ATTACKS = {
    trump: {
        fastAttack: {
            startup: 4,     // Frames before hitbox appears
            active: 3,      // Frames hitbox is live
            recovery: 8,    // Frames before fighter can act
            damage: 5,
            hitstun: 12,    // Frames opponent is stunned on hit
            blockstun: 6,   // Frames opponent is stunned on block
            hitbox: { x: 20, y: -5, w: 35, h: 20 }, // Relative to fighter
            knockback: { x: 3, y: 0 },
            canCancel: true, // Can chain into next attack
        },
        powerAttack: {
            startup: 12,
            active: 5,
            recovery: 18,
            damage: 15,
            hitstun: 24,
            blockstun: 14,
            hitbox: { x: 15, y: -10, w: 45, h: 30 },
            knockback: { x: 8, y: -5 }, // Launches
            wallBreak: true,
            canCancel: false,
        },
    },
    // ... other fighters
};
```

**Why frame data over timers:** Frame data is the universal language of fighting games. "This jab is +4 on hit" means the attacker recovers 4 frames before the defender exits hitstun. Timer-based systems (`setTimeout`, millisecond durations) make this math impossible because they drift and are not synchronized with the fixed-timestep update loop. Frame counts are deterministic.

**Why data-driven:** Balancing 5 fighters means tweaking numbers constantly. When attack properties are data (not code), you can adjust Trump's jab speed from 4 to 5 frames without touching rendering, state machine, or animation code. The attack state reads from this table and "just works."

### 7. AABB Collision Detection with Separate Hit/Hurt Boxes

**Confidence: HIGH** (standard in all 2D fighting games)

```javascript
function checkAABBOverlap(a, b) {
    return a.x < b.x + b.w &&
           a.x + a.w > b.x &&
           a.y < b.y + b.h &&
           a.y + a.h > b.y;
}

function resolveAttack(attacker, defender) {
    if (attacker.currentAttack && attacker.isInActiveFrames()) {
        const hitbox = attacker.getWorldHitbox();  // Attack hitbox (relative -> absolute)
        const hurtbox = defender.getWorldHurtbox(); // Body hurtbox

        if (checkAABBOverlap(hitbox, hurtbox)) {
            if (defender.isBlocking()) {
                defender.enterBlockstun(attacker.currentAttack.blockstun);
                defender.takeDamage(attacker.currentAttack.damage * 0.1); // Chip
            } else {
                defender.enterHitstun(attacker.currentAttack.hitstun);
                defender.takeDamage(attacker.currentAttack.damage);
                defender.applyKnockback(attacker.currentAttack.knockback);
            }
            attacker.attackConnected = true; // Prevent multi-hit
        }
    }
}
```

**Why separate hitbox/hurtbox:** A fighter's hitbox (where their attack can hit) must be separate from their hurtbox (where they can be hit). A punch extends a hitbox forward while the hurtbox remains on the body. This is how every fighting game from Street Fighter II onward works.

**Why AABB, not circles or pixel-perfect:** Rectangles align with sprite geometry and are trivially fast to check. Pixel-perfect collision is meaningless for procedural pixel art (there are no transparent pixel edges to check). Circle collision poorly represents limb attacks. AABB is the right tool.

---

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| WebGL / Three.js | Massive overkill for 2D pixel art. Adds complexity with zero benefit. | Canvas 2D API |
| Any game framework (Phaser, PixiJS, etc.) | Violates zero-dependency constraint. Also adds abstraction layers that fight against custom fighting game systems. | Vanilla Canvas 2D + custom game loop |
| `setInterval` / `setTimeout` for game loop | Not synced to display refresh. Causes jank, wastes battery, drifts over time. | `requestAnimationFrame` |
| Variable timestep (`delta * speed`) | Causes frame-rate-dependent combo timing. A cancel window that works at 60fps breaks at 144fps. | Fixed timestep (60 ticks/sec) |
| `putImageData` for sprites | Ignores compositing, transforms, and alpha. Cannot flip sprites with `scale(-1,1)`. | `fillRect` to offscreen canvas + `drawImage` |
| `event.keyCode` | Deprecated. MDN explicitly warns against it. | `event.key` |
| Web Workers | Require same-origin policy. Game must run from `file://` where Workers may not load. | Keep everything on main thread. 2D fighting game logic is not expensive enough to need Workers. |
| Base64-encoded images | Technically "no external files" but bloats the HTML file enormously and violates the "procedural art" requirement. | `fillRect`-based procedural sprites cached to offscreen canvases |
| CSS animations for game elements | Not synchronized with game loop. Impossible to pause, rewind, or tie to frame data. | Canvas-rendered animations driven by game state |
| `fetch()` / `XMLHttpRequest` | Requires server or CORS. Fails on `file://`. | All data inline in the HTML file |
| Audio via `<audio>` tags with `src` | External file dependency. | Either omit audio or use `AudioContext` + procedural synthesis (optional, complex) |

---

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Canvas 2D fillRect sprites | SVG-based sprites | If you need resolution-independent vector art. Not appropriate for pixel art aesthetic. |
| Object-based state machine | XState library | If building a complex app with visualized state charts. Violates zero-dependency constraint here. |
| AABB collision | SAT (Separating Axis Theorem) | If you need rotated hitboxes. Not needed for axis-aligned fighting game hitboxes. |
| Key state map via event.key | Gamepad API | If adding controller support later. Not in scope (keyboard-only spec). |
| 400x240 internal resolution | Native resolution rendering | If targeting photorealistic graphics. Opposite of pixel art goals. |
| Single canvas (layered draws) | Multiple stacked canvases | If background is very complex and rarely changes. Consider for stage backgrounds if performance is an issue, but start with single canvas for simplicity. |

---

## Stack Patterns by Variant

**If sprite count or complexity causes frame drops:**
- Use layered canvases (background layer + gameplay layer + UI layer)
- Pre-render entire stage backgrounds to a single offscreen canvas
- Only redraw layers that changed

**If combo system needs input buffering:**
- Add a circular buffer that stores the last N frames of input
- Check buffer for motion sequences (e.g., down, down-forward, forward + attack)
- Standard pattern for special moves if added later (out of scope for v1, but architecture should not prevent it)

**If a fighter needs projectiles (e.g., ultimate abilities):**
- Projectiles are independent entities with their own position, hitbox, and animation
- Store in an array, update/render in the main loop alongside fighters
- Remove on hit or off-screen

---

## Version Compatibility

| API | Browser Support | Notes |
|-----|-----------------|-------|
| Canvas 2D | All modern browsers, IE9+ | Universal. No concerns. |
| requestAnimationFrame | All modern browsers, IE10+ | Universal. No concerns. |
| event.key | All modern browsers, Edge 12+ | Universal in 2025+. IE11 had partial support but IE is dead. |
| image-rendering: pixelated | Chrome 41+, Firefox 93+, Safari 10+ | Firefox previously needed `-moz-crisp-edges`. Add `crisp-edges` as fallback. |
| ctx.imageSmoothingEnabled | All modern browsers | Older browsers need `mozImageSmoothingEnabled` / `webkitImageSmoothingEnabled` but not worth polyfilling in 2025+. |
| getContext('2d', { alpha: false }) | All modern browsers | Performance hint. Falls back gracefully if unsupported (just ignored). |

---

## Installation

```bash
# No installation. This is a zero-dependency single HTML file.
# Create index.html and open it in a browser.

# Optional: VS Code Live Server for hot reload during development
# But always test with file:// before shipping.
```

---

## Sources

- [MDN: Optimizing Canvas](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Optimizing_canvas) -- offscreen rendering, alpha:false, integer coordinates, batch calls (HIGH confidence)
- [MDN: Crisp Pixel Art Look](https://developer.mozilla.org/en-US/docs/Games/Techniques/Crisp_pixel_art_look) -- image-rendering: pixelated, imageSmoothingEnabled (HIGH confidence)
- [MDN: 2D Collision Detection](https://developer.mozilla.org/en-US/docs/Games/Techniques/2D_collision_detection) -- AABB pattern (HIGH confidence)
- [Game Programming Patterns: State](https://gameprogrammingpatterns.com/state.html) -- FSM, hierarchical states, pushdown automata for game characters (HIGH confidence)
- [Aleksandr Hovhannisyan: Performant Game Loops](https://www.aleksandrhovhannisyan.com/blog/javascript-game-loop/) -- fixed timestep with accumulator pattern (HIGH confidence)
- [Spicy Yoghurt: Game Loop Tutorial](https://spicyyoghurt.com/tutorials/html5-javascript-game-development/create-a-proper-game-loop-with-requestanimationframe) -- requestAnimationFrame loop structure (HIGH confidence)
- [web.dev: Improving Canvas Performance](https://web.dev/articles/canvas-performance) -- offscreen canvas, layer separation (HIGH confidence)
- [CritPoints: Frame Data Patterns](https://critpoints.net/2023/02/20/frame-data-patterns-that-game-designers-should-know/) -- startup/active/recovery, frame advantage, cancel windows (HIGH confidence)
- [Dream Cancel: Guide to Frame Data](https://dreamcancel.com/?p=2109) -- hitstun, blockstun, frame advantage conventions (HIGH confidence)
- [MDN: KeyboardEvent.key](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key) -- event.key vs keyCode (HIGH confidence)

---
*Stack research for: Single-file HTML5 Canvas 2D Fighting Game*
*Researched: 2026-03-28*

# Architecture Research

**Domain:** Single-file HTML5 Canvas 2D Fighting Game
**Researched:** 2026-03-28
**Confidence:** HIGH

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         HTML Document                               │
│  <style> CSS for outer chrome </style>                              │
│  <canvas id="game"> </canvas>                                       │
│  <script>                                                           │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    GAME SHELL (top-level)                      │  │
│  │  Game Loop (requestAnimationFrame + fixed timestep)            │  │
│  │  Scene Manager (state stack / FSM)                             │  │
│  │  Input Manager (keyboard polling + input buffer)               │  │
│  ├───────────────────────────────────────────────────────────────┤  │
│  │                    SCENE LAYER                                 │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐         │  │
│  │  │  Title   │ │ CharSel  │ │ StageSel │ │  Fight   │         │  │
│  │  │  Scene   │ │  Scene   │ │  Scene   │ │  Scene   │         │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘         │  │
│  │  ┌──────────┐ ┌──────────┐                                    │  │
│  │  │ Victory  │ │  Round   │                                    │  │
│  │  │  Scene   │ │ Splash   │                                    │  │
│  │  └──────────┘ └──────────┘                                    │  │
│  ├───────────────────────────────────────────────────────────────┤  │
│  │                    FIGHT ENGINE (active during Fight Scene)     │  │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐              │  │
│  │  │  Fighter   │  │  Combat    │  │  Physics   │              │  │
│  │  │  System    │  │  System    │  │  System    │              │  │
│  │  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘              │  │
│  │        │               │               │                      │  │
│  │  ┌─────┴──────┐  ┌─────┴──────┐  ┌─────┴──────┐              │  │
│  │  │ Animation  │  │  Hitbox /  │  │  Stage     │              │  │
│  │  │  System    │  │  Hurtbox   │  │  System    │              │  │
│  │  └────────────┘  └────────────┘  └────────────┘              │  │
│  │  ┌────────────┐  ┌────────────┐                               │  │
│  │  │    AI      │  │  HUD /     │                               │  │
│  │  │  System    │  │  Overlay   │                               │  │
│  │  └────────────┘  └────────────┘                               │  │
│  ├───────────────────────────────────────────────────────────────┤  │
│  │                    DATA LAYER                                  │  │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐              │  │
│  │  │  Fighter   │  │  Stage     │  │  Constants │              │  │
│  │  │  Defs      │  │  Defs      │  │  & Config  │              │  │
│  │  └────────────┘  └────────────┘  └────────────┘              │  │
│  ├───────────────────────────────────────────────────────────────┤  │
│  │                    RENDER LAYER                                │  │
│  │  Canvas 2D Context utilities                                  │  │
│  │  Pixel art drawing primitives                                 │  │
│  │  Screen shake / flash effects                                 │  │
│  └───────────────────────────────────────────────────────────────┘  │
│  </script>                                                          │
└─────────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Communicates With |
|-----------|----------------|-------------------|
| **Game Loop** | Fixed-timestep update + variable render; drives the entire frame cycle | Scene Manager, Input Manager |
| **Scene Manager** | Manages scene stack (push/pop); delegates update/render to active scene | All Scenes |
| **Input Manager** | Polls keyboard state each frame; maintains input buffer for combo detection | Game Loop, Fight Scene, AI |
| **Title Scene** | Renders logo, "PRESS START", waits for input | Scene Manager |
| **CharSelect Scene** | Fighter selection UI, cursor, portraits | Scene Manager, Fighter Defs |
| **StageSelect Scene** | Stage selection UI, previews | Scene Manager, Stage Defs |
| **Fight Scene** | Orchestrates a full match (rounds, splashes, timers) | All Fight Engine subsystems |
| **Fighter System** | Fighter state machine (idle/walk/jump/attack/block/hitstun/KO) per fighter | Animation, Combat, Physics |
| **Combat System** | Hitbox-hurtbox collision, damage calc, guard meter, ultimate meter | Fighter System, Physics |
| **Physics System** | Gravity, velocity, ground plane, push-back, wall boundaries | Fighter System, Stage System |
| **Animation System** | Frame-based sprite playback, frame data (startup/active/recovery) | Fighter System, Render Layer |
| **Hitbox/Hurtbox** | Per-frame collision rectangles attached to animation frames | Combat System, Animation System |
| **Stage System** | Background rendering, wall-break state, tier transitions | Physics, Render Layer |
| **AI System** | Decision-making FSM for CPU opponent; reads game state, outputs commands | Fighter System, Input Manager |
| **HUD/Overlay** | Health bars, timer, round counter, ultimate meter, round splashes | Fight Scene, Render Layer |
| **Fighter Defs** | Static data: stats, animation frames, hitbox data, move lists per character | Fighter System, Animation, Combat |
| **Stage Defs** | Static data: background draw functions, wall positions, tier info | Stage System |
| **Render Layer** | Canvas drawing primitives, pixel art helpers, screen effects | All visual components |

## Single-File Code Organization

Since the entire game lives in one `<script>` block, organization is critical. Use clearly delimited sections with banner comments. Order sections by dependency (bottom-up: utilities first, game loop last).

```
<html>
<head><style> /* minimal CSS for canvas centering */ </style></head>
<body>
<canvas id="game" width="640" height="360"></canvas>
<script>
// ============================================================
// SECTION 1: CONSTANTS & CONFIGURATION (~100 lines)
// ============================================================

// ============================================================
// SECTION 2: UTILITY FUNCTIONS (~100 lines)
//   Math helpers, AABB collision, lerp, clamp, random
// ============================================================

// ============================================================
// SECTION 3: RENDER PRIMITIVES (~200 lines)
//   Pixel art drawing, text rendering, screen effects
// ============================================================

// ============================================================
// SECTION 4: INPUT MANAGER (~80 lines)
//   Keyboard state, input buffer, combo detection
// ============================================================

// ============================================================
// SECTION 5: ANIMATION SYSTEM (~150 lines)
//   Frame playback, frame data structures, transitions
// ============================================================

// ============================================================
// SECTION 6: PHYSICS SYSTEM (~100 lines)
//   Gravity, velocity, ground collision, push-back
// ============================================================

// ============================================================
// SECTION 7: COMBAT SYSTEM (~200 lines)
//   Hitbox/hurtbox collision, damage, guard, ultimate meter
// ============================================================

// ============================================================
// SECTION 8: FIGHTER DATA DEFINITIONS (~800 lines)
//   Per-character: stats, animations, frame data, draw functions
// ============================================================

// ============================================================
// SECTION 9: FIGHTER STATE MACHINE (~300 lines)
//   States: idle, walk, jump, crouch, attack, block, hitstun, KO
// ============================================================

// ============================================================
// SECTION 10: AI SYSTEM (~200 lines)
//   Decision FSM, difficulty tiers, reaction timing
// ============================================================

// ============================================================
// SECTION 11: STAGE DATA & RENDERING (~300 lines)
//   Per-stage: draw functions, wall-break logic, tier transitions
// ============================================================

// ============================================================
// SECTION 12: HUD & UI OVERLAY (~200 lines)
//   Health bars, timer, meters, round splashes
// ============================================================

// ============================================================
// SECTION 13: SCENE DEFINITIONS (~400 lines)
//   Title, CharSelect, StageSelect, Fight, Victory scenes
// ============================================================

// ============================================================
// SECTION 14: SCENE MANAGER (~50 lines)
//   Scene stack, push/pop/switch, delegation
// ============================================================

// ============================================================
// SECTION 15: GAME LOOP & BOOTSTRAP (~80 lines)
//   requestAnimationFrame, fixed timestep, init
// ============================================================
</script>
</body>
</html>
```

**Estimated total: ~3,000-3,500 lines.** This is manageable for a single file with clear section boundaries.

### Organization Rationale

- **Bottom-up dependency order:** Each section depends only on sections above it. Fighter Data depends on Animation System and Render Primitives. Fight Scene depends on Fighter, Combat, Physics, AI, HUD. Game Loop depends on Scene Manager.
- **Data separated from logic:** Fighter Defs (section 8) holds pure data objects. Fighter State Machine (section 9) holds behavior. This separation means adding a new character is a data task, not a logic change.
- **Banner comments as navigation:** In a single file, `// ====` banners with section numbers act as the table of contents. Use `Ctrl+G` line number navigation or search for "SECTION N".

## Architectural Patterns

### Pattern 1: Fixed-Timestep Game Loop

**What:** Accumulate real elapsed time, then run fixed-duration update ticks until caught up. Render independently at whatever rate the browser allows.
**When to use:** Always. This is the correct game loop for a fighting game where frame-precise combat matters.
**Trade-offs:** Slightly more complex than naive `requestAnimationFrame` loops, but eliminates speed variation across hardware and ensures deterministic combat timing.

**Example:**
```javascript
const TICK_RATE = 60;
const TICK_DURATION = 1000 / TICK_RATE;
let lastTime = 0;
let accumulator = 0;

function gameLoop(timestamp) {
    const delta = timestamp - lastTime;
    lastTime = timestamp;
    accumulator += delta;

    // Fixed-timestep updates (may run 0, 1, or multiple times)
    while (accumulator >= TICK_DURATION) {
        input.poll();
        sceneManager.update();
        accumulator -= TICK_DURATION;
    }

    // Render once per frame
    sceneManager.render(ctx);
    requestAnimationFrame(gameLoop);
}
```

### Pattern 2: Scene State Machine (Pushdown Automaton)

**What:** Each game screen (title, char select, fight, etc.) is a scene object with `update()`, `render()`, and `handleInput()` methods. A scene manager maintains a stack of scenes, always updating/rendering the top scene.
**When to use:** For managing game flow between menus and gameplay.
**Trade-offs:** Slightly more overhead than a flat switch-case, but dramatically cleaner when scenes need to overlay (e.g., pause menu over fight, round splash over fight).

**Example:**
```javascript
const SceneManager = {
    stack: [],
    push(scene) { this.stack.push(scene); scene.enter?.(); },
    pop() { const s = this.stack.pop(); s?.exit?.(); return s; },
    switch(scene) { this.pop(); this.push(scene); },
    current() { return this.stack[this.stack.length - 1]; },
    update() { this.current()?.update(); },
    render(ctx) { this.current()?.render(ctx); }
};

// Scene transition flow:
// Title -> push(CharSelect) -> push(StageSelect) -> switch(Fight)
// Fight round end -> push(RoundSplash) -> pop back to Fight
// Match end -> switch(Victory) -> switch(Title)
```

### Pattern 3: Fighter Hierarchical State Machine

**What:** Each fighter has a state machine with states like Idle, Walk, Jump, Crouch, FastAttack, PowerAttack, Block, HitStun, Launched, KO. States handle input and return transitions. Use a two-tier hierarchy: grounded states share common logic (can jump, can block), aerial states share different logic (gravity applies, limited actions).
**When to use:** For all fighter behavior. Every frame, the fighter's current state processes input and returns the next state (or itself).
**Trade-offs:** More objects/functions than a switch statement, but each state is self-contained and testable.

**Example:**
```javascript
const FighterStates = {
    idle: {
        enter(fighter) { fighter.setAnimation('idle'); },
        update(fighter, input) {
            if (input.up) return 'jump';
            if (input.down) return 'crouch';
            if (input.left || input.right) return 'walk';
            if (input.fastAttack) return 'fastAttack';
            if (input.powerAttack) return 'powerAttack';
            if (input.block) return 'block';
            return 'idle';
        }
    },
    fastAttack: {
        enter(fighter) {
            fighter.setAnimation('fastAttack');
            fighter.attackFrame = 0;
        },
        update(fighter, input) {
            fighter.attackFrame++;
            const frameData = fighter.def.moves.fastAttack;
            // Startup frames: no hitbox
            // Active frames: hitbox active, check for chain input
            if (fighter.attackFrame >= frameData.totalFrames) {
                return 'idle';
            }
            // Chain window: allow another fast attack during recovery
            if (fighter.attackFrame >= frameData.chainWindow
                && input.fastAttack && fighter.comboCount < 4) {
                fighter.comboCount++;
                return 'fastAttack';
            }
            return 'fastAttack';
        }
    },
    // ... hitstun, launched, KO, etc.
};
```

### Pattern 4: Data-Driven Fighter Definitions

**What:** Each character is defined as a plain object containing stats, animation frame sequences, hitbox/hurtbox rectangles per frame, and move properties. The fighter state machine and animation system read from these definitions -- they never contain character-specific logic.
**When to use:** For all 5 fighters. This is what makes adding/tweaking characters a data edit rather than a code rewrite.
**Trade-offs:** Upfront investment in defining the data schema, but massive payoff in maintainability.

**Example:**
```javascript
const FIGHTERS = {
    trump: {
        name: 'Donald Trump',
        stats: { speed: 3, power: 8, reach: 5, health: 110 },
        hurtbox: { x: -20, y: -80, w: 40, h: 80 }, // base hurtbox
        moves: {
            fastAttack: {
                startup: 4, active: 3, recovery: 8, // frame counts
                damage: 6, hitstun: 12, pushback: 3,
                chainWindow: 10, maxChain: 3,
                hitbox: { x: 20, y: -60, w: 35, h: 20 }
            },
            powerAttack: {
                startup: 12, active: 4, recovery: 16,
                damage: 18, hitstun: 24, pushback: 8,
                launches: true, wallBreaks: true,
                hitbox: { x: 15, y: -65, w: 45, h: 30 }
            },
            ultimate: {
                startup: 20, active: 60, recovery: 30,
                damage: 35,
                // Custom render/logic handled by ultimate system
            }
        },
        animations: {
            idle: { frames: [0,1,2,3,2,1], speed: 8 },
            walk: { frames: [4,5,6,7], speed: 6 },
            // Each frame index maps to a draw function
        },
        draw: function(ctx, frameIndex, x, y, facing) {
            // Procedural pixel art for this character
            // Uses canvas rect/fillRect calls for chunky pixel look
        }
    },
    // biden, obama, bush, clinton...
};
```

### Pattern 5: Frame-Data-Driven Animation with Hitbox Sync

**What:** Animations are sequences of frame indices. Each frame has a duration (in game ticks). Hitboxes and hurtboxes are defined per-move and activate/deactivate based on the current frame's phase (startup/active/recovery). The animation system ticks forward each update and exposes the current phase.
**When to use:** For all fighter animations and combat interactions.
**Trade-offs:** Requires careful frame data authoring, but produces the precise, frame-accurate combat that fighting games demand.

**Example:**
```javascript
function getAttackPhase(fighter, move) {
    const f = fighter.attackFrame;
    if (f < move.startup) return 'startup';
    if (f < move.startup + move.active) return 'active';
    if (f < move.startup + move.active + move.recovery) return 'recovery';
    return 'done';
}

// In combat system update:
if (getAttackPhase(attacker, move) === 'active') {
    const hitbox = transformHitbox(move.hitbox, attacker.x, attacker.facing);
    if (aabbOverlap(hitbox, getHurtbox(defender))) {
        applyHit(attacker, defender, move);
    }
}
```

### Pattern 6: AI as Input Emulation

**What:** The AI system does not directly modify the CPU fighter's state. Instead, it produces a virtual input object identical in shape to the player's input. The fighter state machine processes AI input exactly the same as player input. This guarantees AI fighters follow the same rules as the player.
**When to use:** Always for AI opponents. Never let the AI cheat by bypassing the state machine.
**Trade-offs:** AI is constrained by the same frame data and state transitions as the player, which is fair but means the AI can only be "good" within the game's rules.

**Example:**
```javascript
const AI = {
    decide(cpuFighter, playerFighter, difficulty) {
        const input = { up:false, down:false, left:false, right:false,
                        fastAttack:false, powerAttack:false, block:false,
                        ultimate:false };
        const dist = Math.abs(cpuFighter.x - playerFighter.x);
        const reactionDelay = { easy: 30, medium: 15, hard: 5 }[difficulty];

        // Simple state-based decisions
        if (playerFighter.state === 'fastAttack'
            && cpuFighter.reactionTimer <= 0) {
            input.block = true;
            cpuFighter.reactionTimer = reactionDelay;
        } else if (dist > 80) {
            input[cpuFighter.x < playerFighter.x ? 'right' : 'left'] = true;
        } else if (dist < 60 && Math.random() < 0.3) {
            input.fastAttack = true;
        }
        // ... more sophisticated behavior per difficulty
        return input;
    }
};
```

## Data Flow

### Main Game Loop Flow

```
requestAnimationFrame(timestamp)
    |
    v
Accumulate delta time
    |
    v
[While accumulator >= TICK_DURATION]  <-- fixed timestep updates
    |
    +-> Input Manager: poll keyboard, update buffer
    |
    +-> Scene Manager: current scene .update()
    |       |
    |       +-- (if Fight Scene active) -->
    |           |
    |           +-> AI System: read game state, produce virtual input
    |           +-> Fighter System: process input through state machine
    |           |       (for both P1 and CPU, using real/virtual input)
    |           +-> Physics System: apply velocity, gravity, boundaries
    |           +-> Animation System: advance frame counters
    |           +-> Combat System: check hitbox/hurtbox overlaps
    |           |       +-> Apply damage, hitstun, pushback
    |           |       +-> Check KO, wall-break triggers
    |           +-> HUD: update health bars, meters, timer
    |           +-> Round Logic: check win conditions, round transitions
    |
    +-> accumulator -= TICK_DURATION
    |
[End while]
    |
    v
Scene Manager: current scene .render(ctx)
    |
    +-- (if Fight Scene) -->
    |       +-> Stage System: draw background (current tier)
    |       +-> Fighter System: draw both fighters (current animation frame)
    |       +-> Combat System: draw hitbox debug overlay (if enabled)
    |       +-> HUD: draw health bars, timer, meters, round indicators
    |       +-> Effects: screen shake, flash, slow-mo zoom
    |
    v
requestAnimationFrame(gameLoop)  <-- next frame
```

### Scene Transition Flow

```
Title Scene
    | [PRESS START]
    v
CharSelect Scene
    | [P1 picks fighter, CPU random]
    v
StageSelect Scene
    | [P1 picks stage]
    v
Fight Scene
    | [Round splash "ROUND 1 - FIGHT!"]
    | [Combat loop]
    | [KO detected] -> Round splash "K.O."
    | [If rounds remain] -> next round within Fight Scene
    | [If match decided (best of 3)]
    v
Victory Scene
    | [Winner pose, "PRESS START to continue"]
    v
Title Scene (loop)
```

### Combat Data Flow (Single Frame)

```
Input (keyboard or AI)
    |
    v
Fighter State Machine
    |-- determines: current state, animation, whether attacking
    |
    v
Animation System
    |-- determines: current frame index, attack phase
    |
    v
Combat System reads:
    +-- attacker's hitbox (from move data + position + facing)
    +-- defender's hurtbox (from animation frame + position)
    |
    v
Collision Check (AABB overlap)
    |
    [HIT] --> Apply damage to defender health
          --> Apply hitstun state to defender
          --> Apply pushback velocity to both fighters
          --> Add to attacker's ultimate meter
          --> Check wall-break (power attack + near boundary)
          --> Increment combo counter
    |
    [BLOCKED] --> Reduced damage
              --> Drain guard meter
              --> Guard break if meter empty
    |
    [MISS] --> No effect, attack continues through recovery frames
```

## Anti-Patterns

### Anti-Pattern 1: God Object Fighter

**What people do:** Put all fighter logic -- movement, combat, animation, rendering, AI -- into one massive Fighter class or object.
**Why it's wrong:** Every change to any system requires touching the same 500-line object. Bugs cascade. Adding a character means copying the entire god object.
**Do this instead:** Separate fighter data (static definitions) from fighter runtime state (position, health, current state) from fighter behavior (state machine) from fighter rendering (draw functions). Each system operates on fighter data without owning it all.

### Anti-Pattern 2: Hardcoded Character Logic

**What people do:** Write if/else chains for each character inside the combat or animation systems: `if (fighter.name === 'trump') { ... } else if (fighter.name === 'biden') { ... }`.
**Why it's wrong:** Adding or changing a character means editing core engine code. Combinatorial explosion of special cases.
**Do this instead:** Data-driven definitions (Pattern 4 above). The engine reads from the fighter's data object. Character-specific behavior lives only in the data definition and the draw function.

### Anti-Pattern 3: AI Bypassing Game Rules

**What people do:** Have the AI directly set `fighter.state = 'blocking'` or `fighter.health -= damage` without going through the normal input-to-state-machine pipeline.
**Why it's wrong:** AI fighters behave inconsistently with player fighters. Bugs appear only in AI matches. Difficulty tuning becomes unpredictable.
**Do this instead:** AI emulates input (Pattern 6). The AI produces an input object, and the fighter state machine processes it identically to player input.

### Anti-Pattern 4: Variable Timestep Combat

**What people do:** Multiply movement/damage by `deltaTime` for frame-rate independence.
**Why it's wrong:** In fighting games, frame data IS the gameplay. A 4-frame startup move must always be 4 frames. Variable timestep makes combos inconsistent, hitbox timing unreliable, and the game feel different on different hardware.
**Do this instead:** Fixed timestep (Pattern 1). Game logic always ticks at 60fps regardless of render rate.

### Anti-Pattern 5: Monolithic Render Function

**What people do:** One giant `render()` function with 400 lines of canvas calls drawing everything.
**Why it's wrong:** Impossible to change draw order, add effects, or debug visual issues.
**Do this instead:** Each system provides its own `render(ctx)` method. The fight scene calls them in z-order: stage background -> fighters -> effects -> HUD overlay.

## Build Order (Dependency Chain)

This ordering reflects hard dependencies -- each layer requires the one above it to exist.

| Order | Component | Depends On | Rationale |
|-------|-----------|------------|-----------|
| 1 | Constants, Utilities, Render Primitives | Nothing | Foundation. Everything else uses these. |
| 2 | Input Manager | Constants | Needs key mappings. All interactive systems need input. |
| 3 | Game Loop + Scene Manager | Input Manager | Skeleton that drives everything. Can test with placeholder scenes. |
| 4 | Title Scene | Scene Manager, Render | Proves scene system works. First visible output. |
| 5 | Physics System | Constants, Utilities | Gravity, velocity, ground plane. Needed before fighters move. |
| 6 | Animation System | Render Primitives | Frame playback engine. Needed before fighters animate. |
| 7 | Fighter Data (1 character) | Animation, Render | Define one fighter (e.g., Trump) with placeholder art. Proves data schema works. |
| 8 | Fighter State Machine | Fighter Data, Input, Physics, Animation | The core gameplay loop: input -> state -> animation -> movement. |
| 9 | Combat System | Fighter State Machine, Physics | Hitbox/hurtbox collision, damage, hitstun. Two fighters can now fight. |
| 10 | Stage System (1 stage) | Render Primitives, Physics | Background rendering, boundaries. Fight has a location. |
| 11 | HUD Overlay | Render Primitives, Combat | Health bars, timer, meters. Fight is now readable. |
| 12 | Round/Match Logic | Combat, HUD | Best-of-3, round splashes, KO detection. Complete match flow. |
| 13 | AI System | Fighter State Machine, Combat | CPU opponent. Game is now playable solo. |
| 14 | CharSelect + StageSelect Scenes | Scene Manager, Fighter Data, Stage Data | Full game flow from title to fight. |
| 15 | Remaining Fighters (4 more) | Fighter Data schema (step 7) | Data-only task. Engine is proven. |
| 16 | Remaining Stages (3 more) | Stage System (step 10) | Data-only task. |
| 17 | Wall-Break System | Stage System, Combat, Physics | Cinematic wall breaks, tier transitions. |
| 18 | Ultimate Abilities | Combat, Animation, Render | Per-character specials with unique visual effects. |
| 19 | Victory Scene | Scene Manager, Render, Fighter Data | Winner pose, return to title. |
| 20 | Polish: screen shake, slow-mo, particles | Render, Combat | Juice layer. Makes impacts feel powerful. |

**Critical path:** Steps 1-9 are the critical path to a playable (if ugly) two-fighter prototype. Everything after step 13 is content and polish that can be parallelized or reordered.

## Integration Points

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Scene Manager <-> Scenes | Method calls: `update()`, `render()`, `enter()`, `exit()` | Scenes are objects with a standard interface |
| Fight Scene <-> Fighter System | Fight Scene owns two fighter runtime objects, calls `updateFighter(fighter, input)` | Fight Scene is the orchestrator |
| Fighter System <-> Animation | Fighter sets animation name; Animation system manages frame advancement | One-way: fighter tells animation what to play |
| Fighter System <-> Combat | Combat reads fighter state + hitbox data; writes damage/hitstun back | Combat is the authority on hit resolution |
| Combat <-> Physics | Combat triggers pushback velocity; Physics applies it | Combat sets velocity, Physics integrates it |
| AI <-> Fighter System | AI reads both fighters' state; outputs virtual input object | AI has read-only access to game state |
| Input Manager <-> Combo Detection | Input buffer stores recent inputs; combo detector pattern-matches against buffer | Buffer is a circular array of recent inputs with timestamps |

### Key Interface Contracts

```javascript
// Scene interface (every scene implements this)
{ enter(), exit(), update(), render(ctx) }

// Fighter runtime state (mutable, per-match)
{ x, y, vx, vy, facing, health, guardMeter, ultimateMeter,
  state, attackFrame, comboCount, def /* -> FIGHTERS[id] */ }

// Input object (produced by keyboard polling OR AI)
{ up, down, left, right, fastAttack, powerAttack, block, ultimate }

// Move definition (static data)
{ startup, active, recovery, damage, hitstun, pushback,
  hitbox: {x, y, w, h}, chainWindow?, maxChain?, launches?, wallBreaks? }
```

## Sources

- [Game Programming Patterns - State](https://gameprogrammingpatterns.com/state.html) - Hierarchical state machines, pushdown automata, concurrent FSMs
- [Game Programming Patterns - Game Loop](https://gameprogrammingpatterns.com/game-loop.html) - Fixed timestep with variable rendering
- [MDN - Anatomy of a Video Game](https://developer.mozilla.org/en-US/docs/Games/Anatomy) - requestAnimationFrame game loop fundamentals
- [Canvas Game Loop patterns](https://spicyyoghurt.com/tutorials/html5-javascript-game-development/create-a-proper-game-loop-with-requestanimationframe) - Practical HTML5 Canvas game loop
- [Fighting Game Hitbox/Hurtbox Guide](https://andrea-jens.medium.com/i-wanna-make-a-fighting-game-a-practical-guide-for-beginners-part-4-2021-update-4c26f6964179) - Frame-based hitbox/hurtbox implementation
- [Input Buffering for Responsive Game Feel](https://www.wayline.io/blog/input-buffering-responsive-game-feel) - Input buffer and combo detection systems
- [Fighting Game Input Combo System](https://skyway666.github.io/Input-Combos/) - Combo detection with circular buffers
- [Introduction to Behavior Algorithms for Fighting Games (IEEE)](https://arxiv.org/abs/2007.12586) - AI approaches: FSM, behavior trees, MCTS
- [JavaScript Game Code Organization](http://buildnewgames.com/js-game-code-org/) - Patterns for structuring game code
- [Creating a State Stack Engine in JavaScript](https://idiallo.com/blog/javascript-game-state-stack-engine) - Scene stack pattern

---
*Architecture research for: PoliPunch - Single-file HTML5 Canvas 2D Fighting Game*
*Researched: 2026-03-28*

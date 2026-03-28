# Pitfalls Research

**Domain:** HTML5 Canvas 2D Fighting Game (single-file, procedural pixel art)
**Researched:** 2026-03-28
**Confidence:** HIGH

## Critical Pitfalls

### Pitfall 1: Frame-Rate-Dependent Game Logic

**What goes wrong:**
Game speed varies across monitors. On a 120Hz display, characters move twice as fast; on a 30fps dip, combos become impossible. Physics, animation timing, combo windows, and juggle gravity all break when tied to `requestAnimationFrame` callback frequency rather than elapsed time.

**Why it happens:**
The naive approach is one update per `requestAnimationFrame` call. This works on the developer's 60Hz monitor but breaks everywhere else. `requestAnimationFrame` fires at the display's refresh rate, which is 60Hz, 120Hz, 144Hz, or whatever the OS decides.

**How to avoid:**
Implement a fixed-timestep game loop from the start. Accumulate elapsed time and run physics/logic updates at a fixed interval (e.g., 1/60th second = ~16.67ms). Render as often as the browser allows, interpolating visual positions between physics states for smooth display. The pattern:

```javascript
const TICK_RATE = 1000 / 60;
let accumulator = 0;
let lastTime = 0;

function loop(timestamp) {
  accumulator += timestamp - lastTime;
  lastTime = timestamp;
  while (accumulator >= TICK_RATE) {
    update(TICK_RATE); // fixed-step logic
    accumulator -= TICK_RATE;
  }
  render(accumulator / TICK_RATE); // interpolation factor
  requestAnimationFrame(loop);
}
```

Add a spiral-of-death guard: cap accumulator to prevent runaway catch-up (e.g., `accumulator = Math.min(accumulator, TICK_RATE * 5)`).

**Warning signs:**
- Characters move faster on high-refresh monitors
- Combos that work in testing fail on other machines
- Juggle gravity feels "floaty" or "snappy" inconsistently
- Game runs in slow motion when tab regains focus after being backgrounded

**Phase to address:**
Phase 1 (Engine/Core Loop) -- this is foundational and cannot be retrofitted.

---

### Pitfall 2: Input System That Drops or Delays Actions

**What goes wrong:**
Three distinct failures: (1) relying on `keydown` events for movement causes stuttery motion due to OS key-repeat delay, (2) simultaneous key presses get lost because only the last-pressed key generates repeat events, (3) no input buffer means players must press attack at the exact frame a previous move ends, making combos feel impossible.

**Why it happens:**
Browser `keydown` fires once on press, then pauses (OS repeat delay ~300ms), then repeats. This is designed for text editing, not games. Developers who bind movement directly to `keydown` events get a character that moves one step, pauses, then starts moving again.

**How to avoid:**
1. **Key state tracking:** Maintain a `Set` or object of currently-pressed keys. Add on `keydown`, remove on `keyup`. Poll this state each game tick -- never move characters inside event handlers.
2. **Input buffer:** Store the last N frames of inputs (6-10 frames is standard for fighting games). When a move ends, check the buffer for queued actions. This lets players input the next attack slightly before the current animation finishes.
3. **Prevent defaults:** Call `event.preventDefault()` on game keys to stop browser scroll/navigation. But only for game keys -- don't blanket-prevent everything.
4. **Handle focus loss:** Clear all key states on `window.blur` to prevent stuck keys when the player alt-tabs.

**Warning signs:**
- Movement feels "sticky" with a brief pause before continuous walking
- "I pressed the button but nothing happened" complaints
- Combos only work with frame-perfect timing
- Keys get "stuck" after alt-tabbing

**Phase to address:**
Phase 1 (Engine/Core Loop) -- input system is a dependency for all combat mechanics.

---

### Pitfall 3: Blurry or Smeared Pixel Art (Subpixel Rendering + HiDPI)

**What goes wrong:**
Procedural pixel art looks crisp in the developer's mind but appears blurry, smeared, or inconsistently aliased on screen. Two separate causes compound: (1) canvas anti-aliasing interpolates between pixels when drawing at non-integer coordinates, destroying hard edges, and (2) HiDPI/Retina displays scale the canvas, applying bilinear filtering that blurs everything.

**Why it happens:**
Canvas defaults to `imageSmoothingEnabled = true`. Any `drawImage` or pattern fill at non-integer coordinates triggers subpixel interpolation. On Retina displays, a 800x600 canvas is stretched to 1600x1200 CSS pixels, with the browser applying smoothing. Since this project draws all art procedurally with `fillRect`, `strokeRect`, etc., every coordinate must be integer or the pixel grid breaks.

**How to avoid:**
1. **Integer coordinates always:** `Math.floor()` or `Math.round()` every x/y/width/height before drawing. Never pass floating-point positions to canvas draw calls.
2. **Disable smoothing:** Set `ctx.imageSmoothingEnabled = false` at initialization and after any canvas resize (resize resets context state).
3. **HiDPI handling for pixel art:** Do NOT scale up the canvas for devicePixelRatio in the usual way. Instead, render at your logical resolution (e.g., 800x450) and use CSS `image-rendering: pixelated` on the canvas element to get crisp nearest-neighbor upscaling. This preserves the chunky pixel aesthetic.
4. **CSS scaling approach:**
```css
canvas {
  image-rendering: pixelated;
  image-rendering: crisp-edges; /* Firefox fallback */
  width: 100%;  /* CSS scales up */
  height: auto;
}
```
Set canvas width/height attributes to your logical pixel resolution, not the display resolution.

**Warning signs:**
- Some rectangles have "fuzzy" edges while others are crisp
- Art looks different on laptop vs external monitor
- Characters appear to "shimmer" during movement
- One-pixel lines sometimes disappear

**Phase to address:**
Phase 1 (Engine/Rendering) -- must be correct before any art is drawn.

---

### Pitfall 4: Canvas Performance Death by a Thousand Draws

**What goes wrong:**
Rendering 5 distinct fighters with 10+ animation frames each, plus multi-layer stage backgrounds, health bars, particle effects, and wall-break cinematics -- all procedurally drawn with individual `fillRect` calls -- tanks performance below 60fps. Each `fillRect` is a state machine operation; hundreds per frame compound.

**Why it happens:**
Procedural pixel art means every frame potentially involves hundreds of tiny `fillRect` calls to compose a character sprite. Unlike sprite sheets (one `drawImage` call), procedural art has no native batching. State changes (changing `fillStyle` between colors) are expensive. Clearing and redrawing the entire canvas every frame adds overhead.

**How to avoid:**
1. **Pre-render sprites to offscreen canvases:** Draw each animation frame once to a small offscreen `document.createElement('canvas')`, then `drawImage` that cached canvas onto the main canvas each frame. This converts hundreds of `fillRect` calls into one `drawImage` call per sprite.
2. **Batch by color:** When building sprite caches, draw all pixels of one color before switching `fillStyle`. Each style change has overhead.
3. **Layer canvases:** Use separate canvas elements stacked via CSS for: (a) background (redrawn only on stage transition/wall break), (b) game entities (redrawn every frame), (c) UI overlay (redrawn only on state change). This eliminates redrawing static elements every frame.
4. **Dirty rectangles for UI:** Only redraw health bars, meters, and timer when their values change, not every frame.
5. **Avoid `canvas.width = canvas.width`** for clearing -- it resets all context state. Use `clearRect(0, 0, w, h)` instead.

**Warning signs:**
- FPS drops below 50 during fights with particle effects
- Performance profiler shows `fillRect` dominating frame time
- Adding a second character noticeably drops FPS
- Background + characters + UI together cause stuttering

**Phase to address:**
Phase 1 (Engine/Rendering) for offscreen canvas caching architecture. Phase 2 (Characters) for implementing sprite caching per fighter.

---

### Pitfall 5: Hitbox/Hurtbox Collision That Feels Unfair

**What goes wrong:**
Players experience "phantom hits" (damage without visible contact) or "whiffed hits" (visually connected attacks that deal no damage). In fighting games this destroys trust in the combat system. Common edge cases: (1) hitboxes that don't match the visual animation, (2) one-frame collision windows that get skipped, (3) mutual hit resolution that makes no sense, (4) hitboxes that persist after an attack animation ends.

**Why it happens:**
Hitbox rectangles are defined separately from visual art, and they drift out of sync as animations are tweaked. Simple AABB intersection checks can miss fast-moving attacks that pass through opponents in a single frame. Developers forget to deactivate attack hitboxes when the active frames end, causing lingering damage zones.

**How to avoid:**
1. **Separate hitboxes (attack) from hurtboxes (body):** Never use the character's bounding box for both collision and damage. Each attack gets its own hitbox active only during specific animation frames.
2. **Frame-specific hitbox data:** Define hitboxes per animation frame, not as a constant rectangle. A jab's hitbox should exist only on frames 3-5 of a 12-frame animation, for example.
3. **Hitbox visualization debug mode:** Build a toggle (press a key to show colored rectangles) from day one. If you cannot see the hitboxes, you cannot debug them. Red = attack hitboxes, blue = hurtboxes, green = pushboxes.
4. **Handle mutual hits explicitly:** Define a priority system -- higher-priority attacks beat lower ones, equal priority results in a trade (both take damage).
5. **Active frame management:** Each attack should have startup frames (no hitbox), active frames (hitbox present), recovery frames (no hitbox, vulnerable). Track this as data, not ad-hoc code.

**Warning signs:**
- Players say "that didn't hit me" or "that should have hit"
- Crouching avoids attacks that visually connect
- Attacks hit from behind the character
- Two attacks colliding produces inconsistent results

**Phase to address:**
Phase 2 (Combat Mechanics) -- must be designed as a data-driven system from the start, not hardcoded per-attack.

---

### Pitfall 6: AI That Either Stands Still or Reads Inputs

**What goes wrong:**
Easy AI is braindead (walks into attacks, never blocks). Hard AI is inhuman (reacts in 1 frame, counters every move perfectly, feels like it reads your inputs). There is no middle ground. Players describe this as "the AI is either a punching bag or a cheater."

**Why it happens:**
The naive approach is: check distance, pick a random action, with harder difficulties reducing randomness. This creates a binary between "random = stupid" and "optimal = unfair." The AI has perfect information (knows player position, health, state) and zero reaction time, which no human has. Difficulty scaling via reaction speed alone means Hard AI blocks everything because it processes inputs the same frame they happen.

**How to avoid:**
1. **Artificial reaction delay:** Even Hard AI should have a minimum reaction time (8-15 frames). Easy AI reacts in 20-30 frames. This single parameter does more for difficulty feel than any other.
2. **Decision frequency limiting:** AI re-evaluates its strategy every N frames (e.g., every 30 frames for Easy, every 15 for Hard), not every single frame. Between decisions, it commits to its current action.
3. **Imperfect information simulation:** Hard AI should occasionally "misread" the situation -- e.g., 10% chance to not block a predictable attack, 20% chance to whiff a combo. Easy AI misreads 60-70% of the time.
4. **Behavioral state machine, not reaction bot:** Give the AI states (aggressive, defensive, spacing, pressuring) and have difficulty affect which states it chooses and how well it executes within them. This creates personality rather than robotic perfection.
5. **Never read player inputs directly.** React to player character state (position, current animation, velocity) with artificial delay, not to raw keyboard input.

**Warning signs:**
- Easy AI never lands a hit or always walks into the player
- Hard AI blocks 100% of attacks or counters immediately
- AI behavior feels mechanical and pattern-exploitable
- AI never uses its ultimate at appropriate times (or uses it frame-perfectly every time)

**Phase to address:**
Phase 3 (AI) -- but the reaction delay and decision frequency architecture should be spec'd during Phase 2 (Combat) when attack frame data is defined.

---

### Pitfall 7: Scope Explosion in a Single-File Constraint

**What goes wrong:**
The file grows to 10,000+ lines. Adding a 5th fighter becomes a copy-paste nightmare. Changing a shared combat mechanic requires finding and updating 5 different character implementations. The wall-break cinematic system is tangled with the rendering loop. A bug in one fighter's animation data breaks another's collision detection because they share global variables.

**Why it happens:**
Single-file constraints feel freeing at first ("no build step!") but lack the natural organizational boundaries that separate files provide. Without discipline, code becomes a flat sequence of interleaved concerns: rendering mixed with physics mixed with AI mixed with character data.

**How to avoid:**
1. **Internal module pattern:** Use IIFEs or a namespace object to create logical modules within the single file. Structure as clearly separated sections with a table of contents comment at the top:
```javascript
// === TABLE OF CONTENTS ===
// Line 50:    CONSTANTS & CONFIG
// Line 200:   ENGINE (loop, input, rendering)
// Line 600:   COMBAT SYSTEM (hitboxes, damage, combos)
// Line 900:   CHARACTER DATA (stats, animations, hitbox frames)
// Line 2000:  STAGE DATA
// Line 2500:  AI SYSTEM
// Line 3000:  UI & MENUS
// Line 3500:  GAME FLOW (state machine, round management)
```
2. **Data-driven character definitions:** All 5 fighters should be data objects with identical structure, not 5 blocks of different code. A single rendering/combat engine interprets character data. Adding a fighter means adding a data object, not new logic.
3. **Shared combat engine:** One `resolveCombat()` function, one `updatePhysics()` function, one `renderCharacter()` function. Characters differ only in their data (frame counts, hitbox positions, damage values, sprite pixel maps).
4. **Cap character art complexity:** Each fighter's pixel art should fit a fixed sprite size (e.g., 64x64 or 80x80). Define a pixel budget per animation frame. Without this constraint, art scope explodes.
5. **Build the 2nd fighter immediately after the 1st:** This forces the data-driven architecture. If fighter 2 requires copy-pasting fighter 1's code and modifying it, the architecture is wrong.

**Warning signs:**
- Adding a new character requires duplicating code blocks
- Ctrl+F is the primary navigation tool
- Changing a shared mechanic requires edits in multiple places
- "I'll refactor later" has been said more than twice
- File exceeds 5000 lines without clear section separation

**Phase to address:**
Phase 1 (Engine) -- the module/namespace pattern and data-driven architecture must be established before any content is added.

---

### Pitfall 8: Juggle System Without Diminishing Returns Creates Infinites

**What goes wrong:**
A power attack launches the opponent airborne, and the attacker can hit them again before they land, which launches them again, creating an infinite combo. The opponent is stuck in hitstun in the air forever. Even with a hit limit, if gravity and hitstun duration aren't tuned together, the juggle window feels either impossible (too tight) or exploitable (too lenient).

**Why it happens:**
Juggle physics requires balancing four interacting variables: launch velocity, gravity, hitstun duration, and attack startup speed. Getting any one wrong breaks the system. Developers tune each independently without considering interactions.

**How to avoid:**
1. **Juggle counter with forced scaling:** Track hits in the current juggle sequence. Each subsequent hit increases gravity multiplier (e.g., 1.0x, 1.3x, 1.6x, 2.0x) and reduces hitstun duration. After 3-4 hits, the opponent falls too fast to continue.
2. **Define the juggle budget as a constant:** "Maximum air combo is 3 hits" -- then tune gravity and hitstun to make exactly 3 hits possible with tight timing.
3. **Untechable knockdown:** After the juggle limit, the opponent enters an uninterruptible falling animation that cannot be juggled further.
4. **Test with frame-advance:** Build a frame-step debug mode (press a key to advance one tick) to verify juggle timing. Automated testing is even better -- simulate attack sequences and verify they terminate.

**Warning signs:**
- Combos that exceed the intended hit limit
- Certain characters can juggle infinitely due to faster attacks
- Juggle timing varies between characters unexpectedly
- Opponents "float" unnaturally at the top of a juggle arc

**Phase to address:**
Phase 2 (Combat Mechanics) -- juggle physics must be designed with diminishing returns from the start, not patched in later.

---

### Pitfall 9: Wall-Break Mechanic That Breaks Game State

**What goes wrong:**
The wall-break transition (e.g., Oval Office to Rose Garden) requires changing the background, repositioning characters, potentially adjusting stage boundaries, and playing a cinematic -- all mid-match. If any of these steps fails or happens out of order, characters end up off-screen, hitboxes reference the old stage bounds, or the game loop continues during the cinematic causing phantom inputs and state corruption.

**Why it happens:**
Wall breaks involve a state transition in the middle of real-time gameplay. The game must pause the fight, play an effect, transition the stage, reposition everything, then resume. This is a mini state machine inside the combat state machine, and the interactions are easy to get wrong.

**How to avoid:**
1. **Freeze game logic during transition:** Set a `wallBreakActive` flag that skips combat updates and input processing. Only the cinematic animation advances.
2. **Define pre/post positions as data:** Each stage transition defines exact character positions after the break (e.g., "attacker at 30%, defender at 70% of new stage width").
3. **Stage as a state machine:** Each stage has two states (pre-break, post-break) with defined boundaries, background layers, and floor positions. The wall break is a transition between states, not ad-hoc code.
4. **Test wall breaks at every boundary:** Left wall, right wall, during juggle, during ultimate, when both players are near the wall.

**Warning signs:**
- Characters appear inside walls after a break
- Inputs during the cinematic queue up and fire afterward
- Stage boundaries don't update (characters walk off the new stage)
- Wall break triggers multiple times or doesn't trigger at all

**Phase to address:**
Phase 2 (Stages/Combat) -- wall-break architecture should be defined alongside stage data, not bolted on after stages are "done."

---

### Pitfall 10: Garbage Collection Stutter

**What goes wrong:**
The game runs at smooth 60fps for 5-10 seconds, then hitches for 20-50ms every few seconds. These micro-freezes are perceptible and disruptive during fast combat. They coincide with JavaScript garbage collection sweeps.

**Why it happens:**
Game loops create temporary objects every frame: position vectors `{x, y}`, hitbox rectangles `{x, y, w, h}`, color strings, animation state objects. JavaScript's garbage collector must pause execution to reclaim these short-lived objects. In a fighting game, a 30ms GC pause is 2 dropped frames -- enough to miss an input or make a combo feel inconsistent.

**How to avoid:**
1. **Pre-allocate and reuse objects:** Don't create `{x: p.x + dx, y: p.y + dy}` every frame. Mutate existing objects or use flat arrays.
2. **Avoid string concatenation in the render loop:** `ctx.fillStyle = 'rgb(' + r + ',' + g + ',' + b + ')'` creates a new string every call. Pre-compute color strings and store them.
3. **Use primitive types:** Numbers and booleans don't trigger GC. Prefer `hitboxX, hitboxY, hitboxW, hitboxH` as four numbers over a `hitbox` object.
4. **Object pooling for particles/effects:** Pre-create arrays of effect objects, activate/deactivate by flag rather than creating/destroying.
5. **Profile with DevTools Memory tab:** Look for sawtooth allocation patterns (rapid climb = lots of allocation, sudden drop = GC pause).

**Warning signs:**
- Periodic frame drops every few seconds at regular intervals
- DevTools Memory panel shows sawtooth pattern
- Frame time spikes of 20-50ms interspersed with smooth 16ms frames
- Performance degrades over the duration of a match

**Phase to address:**
Phase 1 (Engine) -- allocation patterns are established early. Retrofitting zero-allocation loops into existing code is painful.

---

## Technical Debt Patterns

Shortcuts that seem reasonable but create long-term problems.

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Hard-coded animation frame counts | Faster to write first character | Every character change requires hunting magic numbers; impossible to balance | Never -- use named constants from day one |
| Per-character combat functions | Feels natural ("TrumpAttack()") | 5x duplication; shared bug fixes require 5 edits | Never -- data-driven from the start |
| Global variables for game state | No need to pass parameters | State corruption bugs, impossible to reset cleanly between rounds | Early prototype only; refactor before 2nd character |
| Drawing directly to main canvas | Simpler code, fewer concepts | Performance ceiling hit at 2+ characters with effects | Acceptable for Phase 1 prototype; add offscreen caching before Phase 2 |
| Skipping input buffer | Simpler input handling | Combos feel unresponsive; casual players can't execute chains | Never for a fighting game -- buffer is essential to game feel |
| Single canvas layer | Less DOM complexity | Full redraw every frame including static backgrounds; performance waste | Acceptable if offscreen canvas caching is used; otherwise never |

## Performance Traps

Patterns that work at small scale but fail as complexity grows.

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Hundreds of fillRect per character per frame | FPS drops, especially with 2 characters + effects | Pre-render sprites to offscreen canvases | 2 characters on screen (200+ draw calls/frame) |
| fillStyle changes inside draw loops | Slow rendering even with few rectangles | Batch all draws of same color, minimize style changes | 50+ color changes per frame |
| Creating new objects every frame for positions/hitboxes | GC pauses every few seconds | Reuse objects, use primitives, object pooling | After 5-10 seconds of continuous gameplay |
| Full canvas clear + full redraw for UI changes | Wasted GPU/CPU on unchanged pixels | Layer canvases or dirty-rectangle tracking for UI | When UI (health bars, meters, timer) updates every frame |
| Unthrottled particle effects | Frame time spikes during ultimates and wall breaks | Cap particle count, pool particle objects, fade quickly | 20+ simultaneous particles |

## UX Pitfalls

Common user experience mistakes in fighting game development.

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| No visual feedback on block | Players don't know if block worked | Flash/pushback on successful block, guard meter visible |
| Missing hitstop (freeze frames on impact) | Attacks feel weightless and unsatisfying | Freeze both characters for 2-4 frames on hit; more for heavy attacks |
| No screen shake or flash on power hits | Combat lacks impact | Brief camera shake (2-3px offset for 3-5 frames) on heavy attacks |
| Combo counter invisible or absent | Players don't know they're comboing | Show hit count + total damage during combos |
| Round transitions too fast or too slow | Jarring restart or boring wait | 1.5-2 second "ROUND X / FIGHT!" splash, fast enough to maintain energy |
| Timer runs during stun animations | Feels unfair when time runs out during a cinematic | Freeze timer during wall breaks, ultimates, and round-start splash |
| No visual distinction between fast and power attacks | Players can't read opponent's moves | Different wind-up animations, distinct colors/effects for heavy attacks |

## "Looks Done But Isn't" Checklist

Things that appear complete but are missing critical pieces.

- [ ] **Character movement:** Works with one key at a time, but test diagonal (forward + up for jump-forward) and attack-while-moving (forward + fast attack). Both pressed simultaneously?
- [ ] **Combo system:** 3-hit combo works when buttons pressed slowly, but does it work at full speed? Does the input buffer catch rapid presses? Does it prevent unintended 5+ hit chains?
- [ ] **Block system:** Blocks while standing, but what about crouch-blocking? Does the guard meter actually deplete and cause guard break? Can you block during hitstun (you shouldn't)?
- [ ] **AI difficulty:** AI fights differently on Easy vs Hard, but is Medium actually between them? Does Hard AI still occasionally get hit, or is it a perfect blocker?
- [ ] **Round reset:** Health resets between rounds, but do positions reset? Do meters reset? Do hitboxes from the previous round's final attack linger into the next round?
- [ ] **Stage boundaries:** Characters stop at walls, but what happens when both characters are at the same wall? Do they overlap? Can one push the other through?
- [ ] **Wall break:** Triggers on power attack near boundary, but what if both characters are at the wall? What if a wall break triggers during a juggle? Does the second stage have correct boundaries?
- [ ] **Ultimate ability:** Meter fills and ultimate activates, but can the opponent be stuck in an unblockable loop? Does the cinematic freeze logic prevent state corruption? What if both ultimates activate on the same frame?
- [ ] **Focus/blur:** Game pauses when tab loses focus, but does it resume correctly? Are key states cleared? Does the timer pause?

## Recovery Strategies

When pitfalls occur despite prevention, how to recover.

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Frame-rate-dependent logic | HIGH | Must rewrite game loop and audit all time-dependent code (movement, physics, animation). Every `+= speed` becomes `+= speed * dt`. |
| Input system without buffer | MEDIUM | Add input ring buffer alongside existing system; retrofit combo checks to read from buffer instead of live state. |
| Blurry pixel art | LOW | Add `imageSmoothingEnabled = false`, CSS `image-rendering: pixelated`, and audit draw calls for non-integer coordinates. |
| Canvas performance bottleneck | MEDIUM | Introduce offscreen canvas caching for sprites. Requires extracting draw code into cache-building functions. |
| Broken hitbox/hurtbox system | HIGH | If hitboxes are hardcoded per-character, must redesign as data-driven frame-by-frame system. Essentially rewriting combat. |
| AI reads inputs | LOW-MEDIUM | Add reaction delay parameter and decision frequency limiter. Existing AI logic can remain, just throttled. |
| Scope explosion / spaghetti code | HIGH | Requires major refactor to extract data-driven patterns. Best to stop and restructure before adding more content. |
| Infinite juggle combos | MEDIUM | Add juggle counter with gravity scaling. Requires touching physics and hitstun code but not full rewrite. |
| Wall-break state corruption | MEDIUM | Implement proper state freeze during transition. May require refactoring game loop to support paused-but-animating state. |
| GC stutter | MEDIUM-HIGH | Audit every frame for allocations, switch to object mutation and pooling. Tedious but mechanical work. |

## Pitfall-to-Phase Mapping

How roadmap phases should address these pitfalls.

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Frame-rate-dependent logic | Phase 1: Engine | Run at 30fps, 60fps, 120fps -- game speed identical at all three |
| Input drops/delay | Phase 1: Engine | Hold two movement keys + attack simultaneously; all register. Combo inputs within 6-frame window execute reliably |
| Blurry pixel art | Phase 1: Engine/Rendering | Zoom browser to 150%, 200% -- pixels remain sharp with hard edges. Test on HiDPI display |
| Canvas performance | Phase 1-2: Engine + Characters | Two characters + effects + background at stable 60fps. No frame drops in DevTools performance panel |
| Hitbox/hurtbox misalignment | Phase 2: Combat | Enable debug hitbox view; visually confirm boxes match animations for all attack types across all characters |
| Broken AI | Phase 3: AI | Easy AI loses most rounds. Hard AI wins most but still takes hits. Medium feels competitive. No perfect-block behavior |
| Single-file spaghetti | Phase 1: Engine | Adding 2nd character requires only a data object, not new functions. Table of contents comment accurately maps to code sections |
| Infinite juggles | Phase 2: Combat | Automated test: simulate fastest possible repeated launches. Verify combo terminates within defined hit limit |
| Wall-break state corruption | Phase 2: Stages/Combat | Wall break triggered during juggle, ultimate, both near wall, round timer near zero -- all resolve cleanly |
| GC stutter | Phase 1-2: Engine + Characters | 60-second continuous gameplay shows no sawtooth in Memory tab, no frame time spikes > 20ms |

## Sources

- [MDN: Optimizing Canvas](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Optimizing_canvas)
- [web.dev: Improving HTML5 Canvas Performance](https://web.dev/articles/canvas-performance)
- [MDN: Crisp Pixel Art Look](https://developer.mozilla.org/en-US/docs/Games/Techniques/Crisp_pixel_art_look)
- [MDN: imageSmoothingEnabled](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/imageSmoothingEnabled)
- [MDN: 2D Collision Detection](https://developer.mozilla.org/en-US/docs/Games/Techniques/2D_collision_detection)
- [Spicy Yoghurt: Proper Game Loop with requestAnimationFrame](https://spicyyoghurt.com/tutorials/html5-javascript-game-development/create-a-proper-game-loop-with-requestanimationframe)
- [Aleksandr Hovhannisyan: Performant Game Loops in JavaScript](https://www.aleksandrhovhannisyan.com/blog/javascript-game-loop/)
- [Isaac Sukin: JavaScript Game Loops and Timing](https://isaacsukin.com/news/2015/01/detailed-explanation-javascript-game-loops-and-timing)
- [Wayline: Input Buffering -- The Key to Responsive Game Feel](https://www.wayline.io/blog/input-buffering-responsive-game-feel)
- [Andrea Jens: I Wanna Make a Fighting Game (Part 6)](https://andrea-jens.medium.com/i-wanna-make-a-fighting-game-a-practical-guide-for-beginners-part-6-311c51ab21c4)
- [GameDev.net: AI in a Fighting Game](https://gamedev.net/forums/topic/268559-ai-in-a-fighting-game/)
- [Gear Project: AI Logic in Fighting Games](https://www.tumblr.com/gear-project/184969373949/ai-logic-in-fighting-games)
- [Kirupa: Canvas on Retina/High-DPI Screens](https://www.kirupa.com/canvas/canvas_high_dpi_retina.htm)
- [Nicola Hibbert: Optimising HTML5 Canvas Games](https://nicolahibbert.com/optimising-html5-canvas-games/)
- [Mozilla Bug 1111361: Dropped Frames from GC Pauses](https://bugzilla.mozilla.org/show_bug.cgi?id=1111361)
- [Build New Games: JavaScript Game Code Organization](http://buildnewgames.com/js-game-code-org/)
- [Andy Balaam: Why Write an Entire Game in a Single JS File](https://artificialworlds.net/blog/2021/07/01/why-write-an-entire-game-including-graphics-in-a-single-hand-coded-javascript-file/)

---
*Pitfalls research for: HTML5 Canvas 2D Fighting Game (PoliPunch)*
*Researched: 2026-03-28*

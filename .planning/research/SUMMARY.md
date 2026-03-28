# Project Research Summary

**Project:** PoliPunch: Presidential Fighting Game
**Domain:** Single-file HTML5 Canvas 2D Fighting Game (zero external dependencies)
**Researched:** 2026-03-28
**Confidence:** HIGH

## Executive Summary

PoliPunch is a Street Fighter II-era 2D fighting game delivered as a single self-contained HTML file with zero external dependencies. The research confirms this is a well-understood domain with established patterns: fixed-timestep game loop, hierarchical state machines for fighter behavior, frame-data-driven combat, AABB hitbox/hurtbox collision, and procedural pixel art cached to offscreen canvases. Every technology needed is a browser API (Canvas 2D, requestAnimationFrame, KeyboardEvent, Web Audio API) available in all modern browsers since 2015+. There are no technology unknowns -- the challenge is execution discipline within the single-file constraint.

The recommended approach is to build bottom-up along the dependency chain: engine foundation first (game loop, input, rendering), then a single playable fighter with combat mechanics, then expand to the full roster and stage content. The architecture research identified a clear 15-section code organization pattern that keeps a 3,000-3,500 line single file navigable. The critical architectural decision is data-driven fighter definitions from day one -- all 5 fighters must be data objects interpreted by a shared engine, never per-character logic branches. Building the second fighter immediately after the first validates this architecture before content scales up.

The top risks are: (1) frame-rate-dependent game logic that makes combat inconsistent across hardware -- mitigated by implementing fixed-timestep in Phase 1 with no exceptions; (2) canvas performance death from hundreds of fillRect calls per frame -- mitigated by offscreen canvas sprite caching from the start; (3) scope explosion in a single file -- mitigated by strict section boundaries, data-driven character definitions, and a pixel art budget per sprite. The single-file constraint is manageable at the projected 3,000-3,500 lines but requires upfront organizational discipline that cannot be retrofitted.

## Key Findings

### Recommended Stack

No installable packages -- the entire stack is browser APIs and code patterns. Canvas 2D API handles all rendering via fillRect-based procedural pixel art cached to offscreen canvases. requestAnimationFrame drives a fixed-timestep game loop (60 ticks/sec with accumulator pattern). KeyboardEvent with key state polling provides input. Web Audio API oscillators generate procedural sound effects. CSS `image-rendering: pixelated` ensures crisp nearest-neighbor upscaling of a low-resolution internal canvas (400x240).

**Core technologies:**
- **Canvas 2D API**: all rendering -- universal browser support, fillRect for procedural pixel art, drawImage for cached sprite blitting
- **Fixed-timestep game loop**: deterministic combat timing -- frame-data-driven fighting games require exact frame counts, not variable delta time
- **Hierarchical FSM (code pattern)**: fighter behavior -- states (idle, walk, attack, hitstun, KO) with grounded/aerial superstates reduce duplication across 5 fighters
- **AABB collision detection**: hitbox/hurtbox overlap -- rectangles are fast, sufficient for 2D fighting games, and standard in the genre
- **Frame-data-driven attack system (code pattern)**: attack properties as data tables (startup/active/recovery frames, damage, hitstun) -- enables balancing without touching engine code
- **Offscreen canvas sprite caching**: performance -- pre-render procedural sprites once, blit with drawImage each frame instead of hundreds of fillRect calls
- **Web Audio API**: procedural sound effects -- oscillator-based hit/block/KO sounds without external audio files

### Expected Features

**Must have (table stakes):**
- Health bars, round timer (99s), best-of-3 round system -- universal fighting game structure
- Basic movement (walk, jump, crouch) with per-character speed -- fundamental navigation
- Fast attack and power attack with frame-data timing -- two-button combat with risk/reward differentiation
- Blocking with chip damage and visual feedback -- core defensive option
- Hitstun, blockstun, and knockdown states -- attacks must feel impactful, not pass-through
- Basic combo system (2-3 hit chains via cancel windows) -- minimum skill expression
- Hitbox/hurtbox collision with frame-specific activation -- the invisible foundation of all combat
- Character animations (idle, walk, attack, hit, KO at minimum) -- characters must telegraph actions
- Hit sound effects via Web Audio API -- silent combat feels broken
- Hit stop (freeze frames on impact) -- highest-value game feel technique, trivial to implement
- Game flow: title screen, character select, stage select, fight, victory -- complete user journey
- Basic AI opponent -- single-player requires a CPU opponent

**Should have (differentiators):**
- 5 political caricature fighters with unique ultimate abilities -- the game's entire comedic premise and shareability hook
- Wall break / stage transitions with slow-mo -- spectacle feature that exceeds browser game expectations
- Guard meter / guard crush -- prevents infinite blocking, adds strategic depth
- Juggle system with diminishing returns -- satisfying air combos with natural limits
- Screen shake, slow-mo on KO/ultimate/wall break -- high-impact polish for minimal implementation cost
- AI difficulty tiers (Easy/Medium/Hard) -- accessibility across skill levels
- Per-character stat differentiation (speed, power, reach, health) -- roster variety beyond visual differences

**Defer (v2+):**
- Procedural background music -- very high effort to sound good, defer until everything else works
- Local 2-player mode -- doubles input complexity, not in spec
- Combo counter display -- pure UI polish, no gameplay impact
- Additional characters/stages beyond 5/4 -- scope management

### Architecture Approach

The game is a single HTML file organized into 15 clearly delimited code sections ordered by dependency (utilities first, game loop last, estimated 3,000-3,500 lines). A Scene Manager (pushdown automaton) routes between game screens (Title, CharSelect, StageSelect, Fight, Victory). The Fight Scene orchestrates subsystems: Fighter System (hierarchical state machine), Combat System (hitbox/hurtbox collision + damage), Physics System (gravity, velocity, boundaries), Animation System (frame-based playback), AI System (input emulation), and HUD Overlay. Fighter definitions are pure data objects -- stats, animation frames, hitbox rectangles, move properties -- interpreted by a shared engine.

**Major components:**
1. **Game Shell** -- game loop (fixed timestep), scene manager (state stack), input manager (key state polling + buffer)
2. **Fight Engine** -- fighter state machine, combat system, physics, animation, AI, HUD -- all active during Fight Scene
3. **Data Layer** -- fighter definitions (5 characters as data objects), stage definitions (4 stages with wall-break states), constants and config
4. **Render Layer** -- canvas drawing primitives, pixel art helpers, screen effects (shake, flash, slow-mo)
5. **Scene Layer** -- title, character select, stage select, fight, victory screens with enter/exit/update/render interface

### Critical Pitfalls

1. **Frame-rate-dependent game logic** -- implement fixed-timestep game loop from Phase 1; a 4-frame startup jab must always be 4/60th of a second regardless of monitor refresh rate. Cannot be retrofitted. Add spiral-of-death guard (cap accumulator).
2. **Canvas performance death from procedural art** -- pre-render all sprite frames to offscreen canvases at init time; draw with drawImage during gameplay instead of hundreds of fillRect calls per character per frame. Batch by color when building caches.
3. **Input system that drops actions** -- use key state map (not event-driven movement), implement 6-10 frame input buffer for combo leniency, clear key states on window blur to prevent stuck keys. Essential for fighting game feel.
4. **Single-file scope explosion** -- enforce data-driven fighter definitions from day one; build the 2nd fighter immediately after the 1st to validate architecture; maintain section banners as navigation; cap sprite complexity per animation frame.
5. **Hitbox/hurtbox misalignment** -- build debug visualization toggle from day one (red=hitbox, blue=hurtbox); define hitboxes per animation frame as data, not constants; separate attack hitboxes from body hurtboxes; track startup/active/recovery phases explicitly.

## Implications for Roadmap

Based on research, suggested phase structure:

### Phase 1: Engine Foundation
**Rationale:** Every system depends on the game loop, input, rendering, and scene management. The fixed-timestep loop and offscreen canvas caching architecture cannot be retrofitted -- they must be correct from the start. This phase also establishes the single-file section organization pattern that prevents scope explosion.
**Delivers:** A running game loop at 60 ticks/sec with keyboard input, scene manager with title screen, canvas rendering with pixel art pipeline (offscreen caching, integer coordinates, imageSmoothingEnabled=false), and a placeholder rectangle that can move and jump. Proves the engine works.
**Addresses:** Game loop, input handler, canvas renderer, scene manager, title screen, physics system (gravity, ground plane)
**Avoids:** Pitfall 1 (frame-rate dependence), Pitfall 2 (input drops), Pitfall 3 (blurry pixel art), Pitfall 7 (scope explosion -- establishes code organization), Pitfall 10 (GC stutter -- establishes allocation patterns)

### Phase 2: Core Combat (Single Fighter)
**Rationale:** Combat is the product. Building one complete fighter with the full state machine, hitbox/hurtbox system, combo chains, and frame-data-driven attacks validates the entire combat architecture before scaling to 5 fighters. The data-driven fighter definition schema is designed here.
**Delivers:** One playable fighter (Trump as prototype) with all states (idle, walk, jump, crouch, fast attack, power attack, block, hitstun, knockdown, KO), frame-data-driven attacks, hitbox/hurtbox collision, basic combo system (2-3 hit chains), placeholder procedural art, one stage background, health bars, round timer, and hit stop.
**Addresses:** Character state machine, hitbox/hurtbox collision, fast/power attacks, blocking, combo system, health bars, timer, hit stop, one stage, hit sounds via Web Audio
**Avoids:** Pitfall 5 (hitbox misalignment -- debug visualization built here), Pitfall 8 (infinite juggles -- diminishing returns designed from start)

### Phase 3: Second Fighter + AI
**Rationale:** The second fighter is the architecture validation moment. If adding fighter 2 requires only a data object (not new logic), the data-driven design is correct. AI is built here because it requires the combat system to exist and be stable, and it enables solo play testing of all subsequent content.
**Delivers:** Second playable fighter (Biden), AI opponent with 3 difficulty tiers, basic AI that blocks/attacks/moves with artificial reaction delays. The game is now playable solo: player picks a fighter, fights CPU, best-of-3 rounds.
**Addresses:** Data-driven fighter #2, AI system (input emulation pattern), AI difficulty tiers, round system (best-of-3), character select screen
**Avoids:** Pitfall 6 (AI that reads inputs -- use reaction delay and decision frequency from start), Pitfall 7 (scope explosion -- validates data-driven architecture)

### Phase 4: Full Roster + Stages
**Rationale:** With the engine, combat, and data-driven architecture proven, the remaining 3 fighters and 3 stages are content tasks, not engineering tasks. Each fighter is a data object; each stage is a draw function with boundary definitions. This is the highest-volume phase but lowest-risk if architecture is sound.
**Delivers:** All 5 fighters (Obama, Bush, Clinton added) with unique stats and procedural pixel art. All 4 stages with backgrounds. Stage select screen. Full game flow from title to victory.
**Addresses:** Remaining 3 fighter data definitions, remaining 3 stage backgrounds, stage select screen, victory screen, per-character stat differentiation
**Avoids:** Pitfall 4 (canvas performance -- sprite caching applied to all new characters), Pitfall 7 (scope explosion -- data-only additions)

### Phase 5: Ultimates + Wall Breaks
**Rationale:** Ultimate abilities and wall breaks are the game's signature spectacle features, but they layer on top of working combat. Each ultimate is a unique mini-system (Trump's golden wall projectile, Biden's ice cream wave, etc.) requiring custom animation and hit resolution. Wall breaks require stage state machines, cinematic freeze, and character repositioning. Both are complex synthesis features that should not be attempted until all foundational systems are stable.
**Delivers:** 5 unique ultimate abilities with meter system, wall-break stage transitions for all 4 stages, slow-mo system, guard meter / guard crush, juggle system with gravity scaling.
**Addresses:** Ultimate abilities (5 unique), ultimate meter, wall break / stage transitions, guard meter, juggle system, slow-mo, screen shake
**Avoids:** Pitfall 8 (infinite juggles -- gravity scaling and juggle counter), Pitfall 9 (wall-break state corruption -- freeze game logic during transition, define pre/post positions as data)

### Phase 6: Polish + Audio
**Rationale:** Game feel polish (particles, screen effects, round splashes) and full audio suite are the final layer. These enhance an already-playable game but add no new gameplay systems. Audio via Web Audio API is independent and can be wired in without touching combat code.
**Delivers:** Hit sparks/particles, screen flash on heavy impact, round splash text animations ("ROUND 1", "FIGHT!", "K.O.!"), full audio suite (hit/block/KO/menu/ultimate sounds), AI behavior tuning, visual polish pass on all fighters and stages.
**Addresses:** Visual effects (sparks, flash), round splash screens, full audio suite, AI polish, visual consistency pass
**Avoids:** Pitfall 4 (particle performance -- use object pooling, cap particle count)

### Phase Ordering Rationale

- **Dependency-driven:** Each phase builds only on what the previous phase delivered. Combat requires engine. AI requires combat. Content requires proven architecture. Spectacle features require stable foundations.
- **Architecture-first:** Phase 1-2 establish the patterns (fixed timestep, data-driven fighters, offscreen caching, state machines) that all subsequent phases rely on. Getting these wrong is HIGH recovery cost (per pitfalls research).
- **Validation checkpoint at Phase 3:** The second fighter is the litmus test. If it requires code duplication, stop and refactor before adding 3 more fighters.
- **Content and spectacle separated:** Phases 4-5 are high-volume but low-risk given correct architecture. Phase 6 is pure polish that cannot make the game worse.
- **Pitfalls front-loaded:** 7 of 10 identified pitfalls must be addressed in Phases 1-2. Only AI pitfalls (Phase 3), wall-break state corruption (Phase 5), and particle performance (Phase 6) come later.

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 2 (Core Combat):** Frame data balancing -- specific startup/active/recovery values for attacks need playtesting iteration. The data structure is well-documented but the specific numbers require tuning.
- **Phase 5 (Ultimates + Wall Breaks):** Each ultimate is a unique mini-system with custom behavior. Wall-break cinematic sequencing (freeze logic, repositioning, stage state machine) has fewer established patterns in the single-file browser game context. This phase benefits from focused research on cinematic state management.

Phases with standard patterns (skip research-phase):
- **Phase 1 (Engine):** Extremely well-documented. Fixed-timestep loops, Canvas 2D, keyboard input -- all have canonical implementations with multiple high-quality sources.
- **Phase 3 (AI):** Fighting game AI via state machine with reaction delays is a solved problem. Multiple academic and practical sources document the pattern.
- **Phase 4 (Roster + Stages):** Pure content creation following established data schemas. No architectural decisions.
- **Phase 6 (Polish):** Screen shake, particles, Web Audio oscillators -- all have canonical implementations.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | All technologies are browser built-ins with universal support. No third-party dependencies to evaluate. Multiple MDN and game dev sources confirm every pattern. |
| Features | HIGH | Fighting game feature expectations are extremely well-documented (SF2 is 35 years old). Feature prioritization aligns with established genre conventions. Anti-features are well-reasoned. |
| Architecture | HIGH | Scene manager + hierarchical state machine + data-driven definitions is the canonical approach. Multiple authoritative sources (Game Programming Patterns, MDN game anatomy). Single-file organization at 3,000-3,500 lines is demonstrated feasible. |
| Pitfalls | HIGH | All 10 pitfalls are well-known in Canvas game development and fighting game development communities. Prevention strategies have verified sources. Recovery costs are accurately assessed. |

**Overall confidence:** HIGH

### Gaps to Address

- **Procedural pixel art complexity budget:** Research confirms the technical approach (fillRect + offscreen caching) but does not quantify how complex each fighter's art can be while maintaining 60fps with 2 characters + effects + background on screen. Need to benchmark early in Phase 1 with a stress test (e.g., 500 fillRect calls per frame on target hardware) and set a pixel budget per sprite frame.
- **Web Audio API autoplay policy:** Audio context must be created after user interaction (browser policy). The title screen's "PRESS START" is the natural trigger point, but this needs verification across Chrome, Firefox, Safari, and Edge. If Safari requires a different gesture handling approach, that needs to be discovered during Phase 2 or 6 when audio is first wired in.
- **Ultimate ability balance:** Each of the 5 ultimates is mechanically unique (projectile, area effect, stun, ricochet, multi-hit). There are no established patterns for balancing 5 asymmetric super moves in a single-file browser game. This will require iterative playtesting during Phase 5.
- **Single-file maintainability at scale:** Research estimates 3,000-3,500 lines. With 5 fighters' full pixel art data, this could push toward 5,000+. If sprite data alone exceeds 2,000 lines, consider a compact encoding (e.g., run-length encoding of pixel rows as strings rather than 2D arrays) to reduce file size.

## Sources

### Primary (HIGH confidence)
- [MDN: Optimizing Canvas](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Optimizing_canvas) -- offscreen rendering, alpha:false, integer coordinates
- [MDN: Crisp Pixel Art Look](https://developer.mozilla.org/en-US/docs/Games/Techniques/Crisp_pixel_art_look) -- image-rendering: pixelated, imageSmoothingEnabled
- [MDN: 2D Collision Detection](https://developer.mozilla.org/en-US/docs/Games/Techniques/2D_collision_detection) -- AABB pattern
- [MDN: KeyboardEvent.key](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key) -- event.key vs keyCode
- [MDN: Anatomy of a Video Game](https://developer.mozilla.org/en-US/docs/Games/Anatomy) -- requestAnimationFrame game loop
- [Game Programming Patterns: State](https://gameprogrammingpatterns.com/state.html) -- hierarchical state machines
- [Game Programming Patterns: Game Loop](https://gameprogrammingpatterns.com/game-loop.html) -- fixed timestep with variable rendering

### Secondary (MEDIUM confidence)
- [Aleksandr Hovhannisyan: Performant Game Loops](https://www.aleksandrhovhannisyan.com/blog/javascript-game-loop/) -- fixed timestep accumulator pattern
- [web.dev: Improving Canvas Performance](https://web.dev/articles/canvas-performance) -- offscreen canvas, layer separation
- [CritPoints: Frame Data Patterns](https://critpoints.net/2023/02/20/frame-data-patterns-that-game-designers-should-know/) -- startup/active/recovery conventions
- [Dream Cancel: Guide to Frame Data](https://dreamcancel.com/?p=2109) -- hitstun, blockstun, frame advantage
- [Wayline: Input Buffering for Responsive Game Feel](https://www.wayline.io/blog/input-buffering-responsive-game-feel) -- input buffer and combo detection
- [SF Seminar: The Basics of Boxes (Capcom)](https://game.capcom.com/cfn/sfv/column/131422?lang=en) -- hitbox/hurtbox conventions
- [Adaptive AI for Fighting Games (Stanford CS229)](https://cs229.stanford.edu/proj2008/RicciardiThill-AdaptiveAIForFightingGames.pdf) -- AI approaches
- [Andrea Jens: Fighting Game Practical Guide](https://andrea-jens.medium.com/) -- hitbox implementation, AI behavior

### Tertiary (LOW confidence)
- [Andy Balaam: Why Write a Game in a Single JS File](https://artificialworlds.net/blog/2021/07/01/why-write-an-entire-game-including-graphics-in-a-single-hand-coded-javascript-file/) -- single-file feasibility (anecdotal)
- [jsfxr: JavaScript Sound Effects Generator](https://github.com/chr15m/jsfxr) -- embeddable audio generation (untested in this context)

---
*Research completed: 2026-03-28*
*Ready for roadmap: yes*

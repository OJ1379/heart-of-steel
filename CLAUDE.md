<!-- GSD:project-start source:PROJECT.md -->
## Project

**Heart of Steel (H.O.S)**

A complete 2D fighting game delivered as a single self-contained HTML5 file — no external dependencies, no server required, opens directly in any modern browser. Features 5 playable political caricature fighters, 4 destructible-wall stages, and full arcade-style gameplay inspired by Street Fighter II (SNES/Genesis era) with 32-bit pixel art rendered entirely in canvas code.

**Core Value:** A fully playable, fun fighting game that runs from a single HTML file — controls must feel responsive, characters must feel distinct, and every fight must have personality.

### Constraints

- **Format**: Single `.html` file — no build step, no dependencies, no CDN links
- **Rendering**: Canvas API only for game graphics — CSS only for outer shell/UI chrome
- **Input**: Keyboard only (WASD/arrows + J/Z/K/X/L/C)
- **Compatibility**: Must run from `file://` without CORS issues (no fetch, no workers that require origin)
- **Assets**: Zero external assets — all art generated procedurally in canvas code
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Constraint: Zero Dependencies
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
## Architecture Patterns (with rationale)
### 1. Game Loop: Fixed Timestep with Variable Rendering
### 2. Procedural Pixel Art via fillRect + Offscreen Caching
### 3. Canvas Setup for Pixel Art
### 4. Input System: Key State Map
### 5. Hierarchical State Machine for Fighter Logic
### 6. Frame-Data-Driven Attack System
### 7. AABB Collision Detection with Separate Hit/Hurt Boxes
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
## Alternatives Considered
| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Canvas 2D fillRect sprites | SVG-based sprites | If you need resolution-independent vector art. Not appropriate for pixel art aesthetic. |
| Object-based state machine | XState library | If building a complex app with visualized state charts. Violates zero-dependency constraint here. |
| AABB collision | SAT (Separating Axis Theorem) | If you need rotated hitboxes. Not needed for axis-aligned fighting game hitboxes. |
| Key state map via event.key | Gamepad API | If adding controller support later. Not in scope (keyboard-only spec). |
| 400x240 internal resolution | Native resolution rendering | If targeting photorealistic graphics. Opposite of pixel art goals. |
| Single canvas (layered draws) | Multiple stacked canvases | If background is very complex and rarely changes. Consider for stage backgrounds if performance is an issue, but start with single canvas for simplicity. |
## Stack Patterns by Variant
- Use layered canvases (background layer + gameplay layer + UI layer)
- Pre-render entire stage backgrounds to a single offscreen canvas
- Only redraw layers that changed
- Add a circular buffer that stores the last N frames of input
- Check buffer for motion sequences (e.g., down, down-forward, forward + attack)
- Standard pattern for special moves if added later (out of scope for v1, but architecture should not prevent it)
- Projectiles are independent entities with their own position, hitbox, and animation
- Store in an array, update/render in the main loop alongside fighters
- Remove on hit or off-screen
## Version Compatibility
| API | Browser Support | Notes |
|-----|-----------------|-------|
| Canvas 2D | All modern browsers, IE9+ | Universal. No concerns. |
| requestAnimationFrame | All modern browsers, IE10+ | Universal. No concerns. |
| event.key | All modern browsers, Edge 12+ | Universal in 2025+. IE11 had partial support but IE is dead. |
| image-rendering: pixelated | Chrome 41+, Firefox 93+, Safari 10+ | Firefox previously needed `-moz-crisp-edges`. Add `crisp-edges` as fallback. |
| ctx.imageSmoothingEnabled | All modern browsers | Older browsers need `mozImageSmoothingEnabled` / `webkitImageSmoothingEnabled` but not worth polyfilling in 2025+. |
| getContext('2d', { alpha: false }) | All modern browsers | Performance hint. Falls back gracefully if unsupported (just ignored). |
## Installation
# No installation. This is a zero-dependency single HTML file.
# Create index.html and open it in a browser.
# Optional: VS Code Live Server for hot reload during development
# But always test with file:// before shipping.
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
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd:quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd:debug` for investigation and bug fixing
- `/gsd:execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd:profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

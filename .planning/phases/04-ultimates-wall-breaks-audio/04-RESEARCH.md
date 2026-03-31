# Phase 4: Ultimates, Wall Breaks + Audio - Research

**Researched:** 2026-03-31
**Domain:** Fighting game spectacle systems (ultimates, wall breaks, guard meter, juggle, procedural audio)
**Confidence:** HIGH

## Summary

Phase 4 adds the "spectacle layer" to an already-functional fighting game. The codebase (4940 lines, single index.html) already has **scaffolding for every system this phase requires**: ultimate meter tracking, ultimate activation logic, ultimate rendering per-fighter, wall break detection/transition with slow-mo and tier2 stages, guard meter with guard break, juggle tracking with gravity scaling, full SFX synthesis (punch, powerHit, block, guardBreak, wallBreak, ko, ultimate), and per-stage BGM with master chain (compressor + reverb + chord pads + melody + bass + drums). The existing implementations range from functional-but-basic (ultimates render simple geometric shapes, wall break has crack lines but no cinematic zoom) to already-complete (guard meter, juggle gravity scaling, BGM).

**The primary work is enhancement and polish, not greenfield construction.** Each system exists but needs refinement to meet the success criteria's quality bar: ultimates need character-specific animations that are visually impressive (not just sweeping rectangles), wall breaks need cinematic zoom (currently just flash + cracks), audio needs distinct SFX differentiation (currently some overlap in tones), and the juggle system needs fast-attack air combos (currently only power attacks launch).

**Primary recommendation:** Structure the phase as 3 plans: (1) Combat mechanics refinement (guard meter tuning, juggle expansion for fast-attack chains, ultimate damage/balance), (2) Visual spectacle (character-specific ultimate animations, wall break cinematic zoom, screen effects), (3) Audio polish (SFX differentiation, BGM integration review, graceful degradation testing).

## Phase Requirements

<phase_requirements>

| ID | Description | Research Support |
|----|-------------|------------------|
| CMB-04 | Guard meter depletes on blocked hits, regenerates slowly when not blocking; guard break stuns defender for 60 frames when depleted | Already implemented in lines 2526, 2946-2947, 3091-3100. Regeneration rate 0.4/frame, guard drain = 50% of scaled damage, 60-frame stun on break. May need tuning but mechanically complete. |
| CMB-05 | Juggle system: power attacks and combo enders launch airborne; follow-up attacks extend juggle (max 2-4 air hits) with gravity scaling (15% per hit) and diminishing damage (10% per hit) | Partially implemented. Power attacks launch (line 3143-3148), gravity scaling at 15% per hit (line 3147), damage scaling at 10% per hit (line 3083). Missing: fast attack air follow-ups (only power attacks currently launch), combo enders launching. |
| CMB-08 | Ultimate ability: fills by dealing damage (1pt/dmg) and taking damage (0.5pt/dmg); meter fills at 100; screen flash + unique animation + high damage | Framework complete. Meter accumulation (lines 3113-3114), activation check (line 2959), 90-frame animation loop (line 2968), damage at frame 45 (lines 3980-3985), screen flash (lines 3487-3491), per-fighter rendering (lines 3496-3569). Need visual enhancement of character-specific animations. |
| STG-05 | Wall break: power attack into boundary triggers slow-mo (0.2x for 30 frames), cinematic zoom, transition to tier 2, bonus damage | Partially implemented. Detection (lines 3174-3190), slow-mo via frame skip (lines 3895-3901), tier2 transition (line 3993-3994), 50 bonus damage (line 3998). Missing: cinematic zoom punch-in (currently just flash + cracks). |
| STG-06 | Each stage has visually distinct second tier (White House -> Rose Garden, Greek Island -> beach, Mar-a-Lago -> patio/pool, Golf Course -> sand bunker) | Fully implemented. All 4 drawTier2 functions exist (lines 4663, 4729, 4798, 4863) with distinct backgrounds. initStageTier2 swaps bgCanvas (lines 4908-4918). |
| AUD-01 | SFX via Web Audio API: punch, power hit, block, guard break, wall break, KO, ultimate -- oscillator/noise synthesis | All 7 SFX types implemented in playSFX (lines 174-255). Each uses distinct oscillator types and frequency ramps. May need tonal refinement for distinctiveness. |
| AUD-02 | Looping chiptune BGM per stage via oscillator sequences | Fully implemented. 4 stage configs in TRACKS object (lines 286-327) with melody, bass, kick, hat, pad chords. Master chain with compressor + reverb (lines 262-281). |
| AUD-03 | Audio volume balanced; SFX audible over music; graceful degradation if Web Audio blocked | Partial. Volume control exists (musicVolume, setMusicVolume at lines 154-155, 169-171). Pause menu has volume tab (line 4209-4211). Graceful degradation via try/catch in initAudio (lines 158-167) and playSFX (line 254). SFX route through globalGain same as music -- need separate gain nodes for independent volume control. |

</phase_requirements>

## Existing Implementation Inventory

This is the critical finding: nearly every system has some level of implementation already. The planner MUST understand what exists to avoid wasting effort on reimplementation.

### Already Complete (no changes needed)
| System | Status | Evidence |
|--------|--------|----------|
| Guard meter tracking | Complete | `guardMeter: 100` on fighter, drains on block, regenerates at 0.4/frame, 60-frame guard break stun |
| Guard meter HUD | Complete | Blue bar below health, turns red on guard break (lines 3437-3449) |
| Ultimate meter tracking | Complete | `ultimateMeter: 0-100`, gains on dealing/taking damage, checked for >= 100 |
| Ultimate meter HUD | Complete | Gold bar below guard meter, flashes white when full (lines 3451-3471) |
| Ultimate activation flow | Complete | Input check -> set ultimateActive -> 90 frame loop -> apply damage at frame 45 -> reset |
| BGM per stage | Complete | 4 unique configs with melody/bass/drums/pads, master chain with compressor + reverb |
| Stage tier2 backgrounds | Complete | All 4 stages have drawTier2 functions rendering distinct second-tier backgrounds |
| Wall break detection | Complete | Checks power attack + boundary proximity + not blocking + tier === 1 |
| Juggle gravity scaling | Complete | 15% per air hit via `juggleGravityMul = 1.0 + airHits * 0.15` |
| Juggle damage scaling | Complete | 10% per air hit via `juggleScale = max(0.4, 1.0 - airHits * 0.10)` |
| All 7 SFX types | Complete | punch, powerHit, block, guardBreak, wallBreak, ko, ultimate in playSFX |

### Needs Enhancement (exists but below quality bar)
| System | Current State | Gap |
|--------|---------------|-----|
| Ultimate animations | Simple geometric shapes (rectangles, lines) | Need character-specific visually impressive animations matching descriptions (golden wall, ice cream wave, drone strike, golf ball, sax solo) |
| Wall break cinematic | Flash + crack lines only | Missing cinematic zoom punch-in effect (scale canvas transform) |
| Wall break slow-mo | Alternating frame skip (0.5x) | Spec says 0.2x speed -- need 5:1 frame skip ratio |
| SFX distinctiveness | All use basic oscillator ramps | Could benefit from more tonal variety (noise bursts for impacts, layered oscillators) |
| SFX/BGM volume separation | Both route through same globalGain | Need separate sfxGain and musicGain nodes for independent volume control |
| Air juggle expansion | Only power attacks launch | CMB-05 says combo enders should also launch; fast attacks should work as follow-ups in air |

### Not Yet Implemented
| System | Notes |
|--------|-------|
| Fast attack juggle follow-ups | Currently only power attacks launch; need air-state fast attack hits to extend juggle |
| Combo ender launch | Last hit of a fast attack chain should launch for juggle |
| Cinematic zoom on wall break | Canvas scale transform centered on impact point |
| AI ultimate usage integration | AI checks `ultimateMeter >= 100` at line 3364 but the activation code may need verification with the full ultimate system |

## Architecture Patterns

### Current Architecture (do not change)
```
index.html (single file, ~4940 lines)
  Constants & Config
  Canvas Setup
  Utility Functions
  Input System
  Audio System (initAudio, playSFX, startBGMusic, stopBGMusic)
  Game Loop (fixed timestep)
  Sprite Cache
  Render Fighter
  Scene Manager
  Title Scene
  Fighter Definitions (TRUMP, BIDEN, OBAMA, BUSH, CLINTON)
  Portrait Cache
  Fighter Runtime Factory (createFighter)
  Fighter State Machine (FIGHTER_STATES)
  Fighter Update (updateFighter, updateFighterCharge)
  Push-back Collision
  Hit Detection & Combat (checkHit, resolveHit, applyUltimateDamage, checkWallBreak)
  CPU AI
  HUD (drawHUD, drawHealthBar, drawRoundPips)
  renderUltimate
  Fight Scene (state vars, resetRound, fightScene.enter/update/render)
  Victory Scene
  Pause Menu Scene
  Stage Select Scene
  Stage Definitions (STAGES object with draw + drawTier2)
  Stage Init (initStage, initStageTier2)
  Init Sequence
```

### Pattern: Enhancing renderUltimate
The existing `renderUltimate` function (lines 3479-3571) dispatches on `fighter.def.name` with if/else. Each fighter gets a code block that draws effects based on `fighter.ultimateFrame` (0-89). The pattern is:
- Frames 0-9: Screen flash (shared)
- Frames 10-79: Fighter-specific effect drawing
- Frame 45: Damage applied (in fightScene.update, not renderUltimate)
- Frame 89: Animation ends

Enhancement approach: Replace the simple geometric drawing in each fighter's block with richer procedural animations. The 70-frame window (10-79) gives plenty of room for multi-phase effects.

### Pattern: Wall Break Cinematic Zoom
Current wall break rendering (lines 4088-4102) draws a white flash and crack lines. To add cinematic zoom:
1. During `wallBreakTimer > 0`, apply `ctx.save()` -> `ctx.translate()` -> `ctx.scale()` centered on the impact point before drawing the stage + fighters
2. Scale factor ramps from 1.0 to ~1.5x over the 30-frame duration
3. Must be applied in `fightScene.render()` BEFORE drawing fighters/HUD, then `ctx.restore()` after

### Pattern: Separate SFX/BGM Volume
Current audio routing: `oscillator -> gain -> globalGain -> destination`
Needed: `SFX: oscillator -> sfxGain -> masterGain -> destination`
         `BGM: all music nodes -> musicGain -> masterGain -> destination`
This requires:
1. Create `sfxGain` and `musicGain` nodes in `initAudio()`
2. Route `playSFX` through `sfxGain`
3. Route `startBGMusic` master through `musicGain`
4. Update pause menu volume tab to control both independently (or just add SFX volume alongside existing music volume)

### Pattern: Juggle Expansion
Current juggle: only power attacks set `defender.vy` and increment `airHits` (line 3143-3148).
To add fast attack follow-ups in air:
1. In `resolveHit`, when defender is airborne (`defender.y < GROUND_Y`), fast attacks should also count as juggle hits
2. Add small upward velocity on fast attack air hits to keep opponent airborne briefly
3. Apply same gravity scaling and damage diminishing
4. For combo ender launch: when `comboCount >= maxChain`, apply launch velocity similar to power attack

### Anti-Patterns to Avoid
- **Do not restructure the audio system**: The existing BGM implementation with setInterval-driven step sequencer works. Do not replace it with a different scheduling approach.
- **Do not add new states to the fighter state machine for ultimates**: Ultimates already bypass the state machine via `ultimateActive` flag check in `updateFighter` (line 2966-2973). Keep this pattern.
- **Do not use requestAnimationFrame for audio timing**: The setInterval approach in startBGMusic is correct for audio scheduling. rAF can be throttled by the browser when the tab is backgrounded.
- **Do not change the wall break detection threshold (8px from boundary)**: This has been tuned already.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Audio synthesis | Custom audio library | Web Audio API oscillators + gain nodes (already in use) | Standard API, zero dependencies |
| Cinematic effects | Custom render pipeline | Canvas 2D transforms (translate, scale, save/restore) | Built into Canvas API |
| Time scaling | Custom delta-time system | Frame skip counter (already in use for wall break slow-mo) | Simpler, works with fixed timestep |
| Volume control UI | Custom slider widget | Existing pause menu volume tab pattern (up/down key adjustment) | Already implemented, just needs SFX channel |

## Common Pitfalls

### Pitfall 1: Audio Context Suspension
**What goes wrong:** Web Audio API contexts start suspended on most browsers until user gesture
**Why it happens:** Browser autoplay policy
**How to avoid:** Already handled -- line 4932-4936 resumes on keydown. Ensure `playSFX` and `startBGMusic` both check `if (!audioCtx) return` (already done).
**Warning signs:** No sound on game start -- press a key first

### Pitfall 2: Canvas Transform State Leaks
**What goes wrong:** Applying scale/translate for cinematic zoom without proper save/restore corrupts all subsequent drawing
**Why it happens:** Canvas transforms are cumulative and stateful
**How to avoid:** Always `ctx.save()` before transforms, `ctx.restore()` after. The HUD should be drawn OUTSIDE the zoom transform (HUD stays at 1:1 scale).
**Warning signs:** HUD elements appear zoomed/offset, fighters render in wrong positions after wall break

### Pitfall 3: SFX Gain Node Accumulation
**What goes wrong:** Creating new GainNode per SFX call without cleanup leads to memory/CPU leak
**Why it happens:** Each oscillator + gain node stays in memory until garbage collected
**How to avoid:** Oscillator nodes auto-disconnect after `stop()`. Gain nodes connected to stopped oscillators get GC'd. Current pattern is correct -- each SFX creates short-lived nodes that self-terminate. No pool needed for this use case.
**Warning signs:** Performance degradation after many hits -- check DevTools for increasing AudioNode count

### Pitfall 4: Wall Break Zoom Affecting Hit Detection
**What goes wrong:** If zoom transform is applied during update (not just render), hitboxes become misaligned
**Why it happens:** Confusing visual transform with game-state transform
**How to avoid:** Zoom is RENDER-ONLY. All game logic (hitbox positions, fighter positions) stays in game-world coordinates. Only the rendering pass applies the scale transform.
**Warning signs:** Hits registering in wrong positions during wall break animation

### Pitfall 5: Slow-Mo Frame Rate Issues
**What goes wrong:** 0.2x slow-mo via frame skip (skip 4 out of 5 frames) looks jerky
**Why it happens:** At 60fps, 0.2x = 12fps effective, which is visibly choppy
**How to avoid:** Use 0.5x (current implementation, every other frame) or 0.33x (skip 2 of 3). 0.2x is too aggressive for smooth feel. The spec says 0.2x but gameplay feel should take priority.
**Warning signs:** Wall break slow-mo looks like a slideshow

### Pitfall 6: Ultimate Damage Applied Multiple Times
**What goes wrong:** If the damage frame (45) is checked every tick during ultimate, it fires once. But if the update loop runs multiple times per render (accumulator pattern), it could fire on frame 45 across multiple ticks.
**Why it happens:** Fixed timestep accumulator can run multiple updates per render frame
**How to avoid:** Already safe -- `ultimateFrame` increments by 1 per update tick, so frame 45 only matches once. The `===` check prevents multi-fire.
**Warning signs:** Ultimate dealing double/triple damage occasionally

## Code Examples

### Cinematic Zoom for Wall Break (render-only transform)
```javascript
// In fightScene.render, BEFORE drawing fighters:
if (wallBreakActive && wallBreakTimer > 0) {
  const progress = 1 - (wallBreakTimer / 30); // 0 -> 1
  const zoomScale = 1.0 + progress * 0.5; // 1.0 -> 1.5
  const focusX = wallBreakSide === 'right' ? STAGE_RIGHT : STAGE_LEFT;
  const focusY = GROUND_Y - 20;
  ctx.save();
  ctx.translate(focusX, focusY);
  ctx.scale(zoomScale, zoomScale);
  ctx.translate(-focusX, -focusY);
  // Draw stage + fighters inside this transform
}
// After drawing fighters:
if (wallBreakActive && wallBreakTimer > 0) {
  ctx.restore();
}
// Draw HUD OUTSIDE zoom (always 1:1)
drawHUD(ctx, player, cpu, roundTimer, p1Wins, p2Wins);
```

### Separate SFX/BGM Gain Nodes
```javascript
let sfxGain = null;
let musicGain = null;

function initAudio() {
  try {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    // Master gain (overall volume)
    globalGain = audioCtx.createGain();
    globalGain.gain.value = 1.0;
    globalGain.connect(audioCtx.destination);
    // SFX channel
    sfxGain = audioCtx.createGain();
    sfxGain.gain.value = 0.8;
    sfxGain.connect(globalGain);
    // Music channel
    musicGain = audioCtx.createGain();
    musicGain.gain.value = 0.6;
    musicGain.connect(globalGain);
  } catch(e) { audioCtx = null; }
}

// In playSFX: connect gain to sfxGain instead of globalGain
// In startBGMusic: connect masterGain to musicGain instead of globalGain
```

### Fast Attack Air Juggle Extension
```javascript
// In resolveHit, after the existing isPower juggle launch block:
if (!isPower && defender.isAirborne && defender.airHits < 4) {
  // Fast attack keeps opponent aloft briefly
  defender.vy = -3 - (defender.airHits * 0.3);
  defender.airHits++;
  defender.juggleGravityMul = 1.0 + defender.airHits * 0.15;
}
```

### Combo Ender Launch
```javascript
// In resolveHit, when attacker.comboCount >= attacker.def.attacks.fast.maxChain:
if (!isPower && attacker.comboCount >= atk.maxChain && defender.airHits < 4) {
  defender.vy = -5;
  defender.vx = 4 * attacker.facing;
  defender.airHits++;
  defender.juggleGravityMul = 1.0 + defender.airHits * 0.15;
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| No audio | Full BGM + 7 SFX types via Web Audio API | Phase 3 | BGM system is already production-quality; SFX exist but need polish |
| No wall breaks | Wall break detection + tier2 transition + slow-mo | Phase 3 | Framework complete, needs cinematic zoom |
| No ultimates | Ultimate meter + activation + per-fighter rendering | Phase 3 | Mechanically complete, visuals need enhancement |
| Single-channel audio | globalGain for all audio | Current | Needs split into sfxGain + musicGain for proper balance |

## Integration Complexity Analysis

### System Interactions
1. **Ultimate + Combat Loop**: Ultimates freeze the fighter via `ultimateActive` flag bypass in `updateFighter`. Damage is applied at frame 45 via `applyUltimateDamage` called from `fightScene.update`. This is clean -- ultimate animations don't interfere with normal combat because the fighter is effectively frozen during the animation.

2. **Wall Break + Stage Renderer**: Wall break swaps `stageDef.bgCanvas` to the tier2 canvas. This is a one-way transition (tier resets on `resetRound`). The rendering path (`fightScene.render` draws `currentStage.bgCanvas`) works unchanged.

3. **Guard Meter + Block State**: Guard meter drains in `resolveHit` when `defender.isBlocking`. Guard break sets `guardBreakStun = 60` and resets `guardMeter = 100`. The `updateFighter` function checks `guardBreakStun > 0` as its first action, freezing the fighter. Clean integration.

4. **Juggle + Gravity**: `juggleGravityMul` is applied in the jumpFall and hitstun state handlers. `airHits` resets on landing. The damage scaling in `resolveHit` uses `juggleScale = max(0.4, 1.0 - airHits * 0.10)`. All self-contained.

5. **Audio + Game State**: `playSFX` is called from `resolveHit` (punch/powerHit/block/guardBreak), `checkWallBreak` (wallBreak), KO detection (ko), and ultimate activation (ultimate). BGM starts in `fightScene.enter` and stops in `victoryScene.enter`. Clean fire-and-forget pattern.

### Risk Assessment
- **LOW risk**: Guard meter, juggle gravity, ultimate meter -- all mechanically complete
- **MEDIUM risk**: Cinematic zoom (canvas transforms during render must be carefully scoped)
- **MEDIUM risk**: SFX/BGM volume separation (requires replumbing audio routing)
- **LOW risk**: Ultimate animation enhancement (purely visual, no game logic changes)
- **LOW risk**: Slow-mo tuning (just changing the frame skip ratio)

## Open Questions

1. **Slow-mo ratio: 0.2x vs 0.5x**
   - What we know: Spec says 0.2x (skip 4 of 5 frames = 12 effective fps). Current impl is 0.5x (skip 1 of 2).
   - What's unclear: Whether 0.2x looks acceptable at 400x240 resolution
   - Recommendation: Implement 0.33x (skip 2 of 3 = 20 effective fps) as compromise. Feels slow enough for cinematic impact without slideshow jank. This is a tuning decision -- Claude's discretion.

2. **Ultimate meter persistence across rounds**
   - What we know: `resetRound()` zeroes both players' ultimate meters (line 3628-3629)
   - What's unclear: Whether meter should carry over between rounds (common in fighting games) or reset
   - Recommendation: Reset per round (current behavior) keeps rounds feeling fresh. This matches the existing code.

3. **SFX volume slider in pause menu**
   - What we know: Pause menu has a Volume tab that adjusts `musicVolume`.
   - What's unclear: Whether to add a second slider for SFX or just set a fixed ratio
   - Recommendation: Add SFX volume control alongside music volume in the existing Volume tab. Use up/down for music, attack/block keys for SFX (or toggle between the two with a selector).

## Project Constraints (from CLAUDE.md)

- **Format**: Single `.html` file -- no build step, no dependencies, no CDN links
- **Rendering**: Canvas API only for game graphics -- CSS only for outer shell/UI chrome
- **Input**: Keyboard only (WASD/arrows + J/Z/K/X/L/C)
- **Compatibility**: Must run from `file://` without CORS issues
- **Assets**: Zero external assets -- all art generated procedurally in canvas code
- **Audio**: Web Audio API + procedural synthesis only (no `<audio>` tags, no base64)
- **Game loop**: Fixed-timestep accumulator (1/60s tick)
- **No variable timestep**: Fighting games require frame-perfect timing
- **GSD Workflow**: Must use GSD commands for file changes

## Sources

### Primary (HIGH confidence)
- Direct codebase analysis of `index.html` (4940 lines) -- all line references verified by reading the file
- MDN Web Audio API documentation (training data, verified against codebase patterns)
- MDN Canvas 2D transforms documentation (training data, verified against existing ctx.save/restore usage)

### Secondary (MEDIUM confidence)
- Fighting game design conventions for guard meter, juggle scaling, and ultimate meter (training data from game design resources)

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- zero-dependency single file, no libraries to verify
- Architecture: HIGH -- entire codebase read and analyzed, all integration points documented
- Pitfalls: HIGH -- based on direct analysis of existing code patterns and Canvas/Web Audio API behavior
- Implementation gaps: HIGH -- every system's current state verified against requirement text

**Research date:** 2026-03-31
**Valid until:** 2026-04-30 (stable -- no external dependencies that could change)

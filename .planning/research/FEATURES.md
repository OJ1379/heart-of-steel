# Feature Research

**Domain:** 2D Fighting Game (Street Fighter II era, single-file HTML5 Canvas)
**Researched:** 2026-03-28
**Confidence:** HIGH

## Feature Landscape

### Table Stakes (Users Expect These)

Features that any player of a 2D fighting game assumes exist. Missing these makes the product feel broken, not incomplete.

#### Combat Core

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Health bars (dual, mirrored) | Universal in every fighting game since SF1. Players need instant feedback on damage dealt/received. | LOW | Two horizontal bars at top of screen, drain left-to-right for P1 and right-to-left for P2. Color shift green-to-yellow-to-red as health drops. |
| Round timer (99 seconds) | SF2 standard. Prevents turtling. Time-over = most health wins. | LOW | Centered between health bars. Counts down at roughly 1 tick per real second. |
| Basic movement (walk, jump, crouch) | Fundamental fighting game navigation. Without this, nothing else works. | MEDIUM | Walk speed per-character. Jump arc (parabolic, fixed height). Crouch shortens hurtbox. Need gravity simulation for jump. |
| Fast attacks (jab/kick) | Players expect a quick, low-damage attack for poking and pressure. SF2's light attacks. | MEDIUM | ~3 frame startup, ~2 active frames, ~5 recovery. Low damage. Chainable into 2-3 hit strings for light characters. |
| Power attacks (heavy punch/kick) | Players expect a high-risk, high-reward heavy hit. SF2's heavy attacks. | MEDIUM | ~8-12 frame startup, ~4 active frames, ~12 recovery. High damage. Launches opponent for juggle. Wall-break capable per spec. |
| Blocking (standing/crouching) | Fundamental defensive option. Without block, combat is just trading hits. | MEDIUM | Hold back or dedicated block button (spec uses K/X). Damage reduction (chip damage = ~10% of normal). Standing block stops mid/high, crouch block stops low. |
| Hit stun and block stun | Attacked characters must visually react. Without stun frames, hits feel weightless. | MEDIUM | Hit stun duration proportional to attack strength. Block stun slightly shorter than hit stun (defender recovers first = frame advantage concept). |
| Knockdown state | Characters must fall down from heavy hits. SF2 uses hard knockdown on sweeps and throws. | MEDIUM | Knockdown animation, ground bounce or slide, wake-up timing (invulnerable during get-up frames). |
| Hitbox/hurtbox collision system | The invisible foundation of all combat. Determines what hits and what doesn't. | HIGH | Each animation frame needs attack hitboxes (where the attack lands) and hurtboxes (where the character is vulnerable). Axis-aligned bounding boxes (AABB) are sufficient for SF2-style. No need for pixel-perfect. |
| Basic combo system (hit confirms) | Players expect that landing a fast attack lets them follow up with more attacks before opponent recovers. Chain combos (cancel light into heavy) are SF2 table stakes. | HIGH | Light attack on hit has enough hit stun to link into heavy attack. Need cancel windows where one attack's recovery can be interrupted by the next attack. 2-4 hit ground combos minimum. |

#### Game Flow

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Title screen | First thing player sees. Sets tone. "Press Start" convention. | LOW | Logo, "PRESS START" text blinking. Background art or animation. |
| Character select screen | Core ritual of fighting games. Picking your fighter is part of the experience. | MEDIUM | Grid of character portraits (5 fighters). Highlight/cursor movement. Character preview with name and basic stats. CPU picks randomly or sequentially. |
| Stage select screen | Spec requires 4 stages. Player should choose the arena. | LOW | Thumbnail previews of 4 stages. Cursor selection. |
| Round system (best of 3) | Universal SF2 standard. Two round wins to take the match. | MEDIUM | Round counter display (dots or icons under health bars). "ROUND 1", "ROUND 2", "FINAL ROUND" splash text. Health reset between rounds, meter persists (design choice). |
| Round start announcements | "ROUND 1... FIGHT!" is iconic. Without it, rounds feel abrupt. | LOW | Text overlay sequence: round number -> "FIGHT!" with brief pause between. Characters idle during this. |
| KO announcement | "K.O.!" when a fighter's health hits zero. Punctuates the moment. | LOW | Big text overlay. Brief freeze frame on the killing blow. Winner does victory pose. |
| Victory screen | Shows the winner. Provides closure to the match. | LOW | Winner character with pose animation. "YOU WIN" / "YOU LOSE" text. Option to continue or return to title. |
| Round timer display | Players need to see time remaining. | LOW | Large numerals centered at top of screen between health bars. |

#### Visual Feedback

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Character animations (idle, walk, attack, hit, KO) | Characters must visually telegraph what they're doing. Static sprites that just slide around feel broken. | HIGH | Minimum per character: idle (4+ frames), walk forward (4+ frames), walk back (4+ frames), jump (3 frames: up/peak/down), crouch (2 frames), fast attack (3-4 frames), power attack (5-6 frames), block (2 frames), hit stun (2 frames), knockdown (3-4 frames), KO (4+ frames). All drawn procedurally in canvas. This is the single largest art workload. |
| Hit sparks / impact effects | Visual confirmation that an attack connected. Without sparks, hits look like they pass through. | LOW | White/yellow flash particle burst at point of contact. 3-5 frame particle animation. Different for light vs heavy hits. |
| Screen flash on heavy impact | Emphasizes powerful hits. SF2 does white flash on specials and supers. | LOW | Brief (2-3 frame) white overlay at reduced opacity on heavy hits, full flash on ultimate. |

#### Audio

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Hit sound effects | Silent combat feels completely wrong. Every fighting game has impact sounds. | MEDIUM | Use Web Audio API oscillators. Punch = short noise burst with quick decay. Kick = slightly longer, lower pitch. Heavy hits = deeper tone with more sustain. Can synthesize with white noise + bandpass filter + fast envelope. |
| Block sound effects | Audible confirmation of successful block vs hit. | LOW | Higher-pitched, shorter "tink" sound vs the meatier hit sound. Distinguishes block from hit for the player. |
| Menu/UI sounds | Navigation needs audio feedback. | LOW | Simple blip/select tones using sine wave oscillators. jsfxr-style: short sine wave with pitch sweep. |

### Differentiators (Competitive Advantage)

Features that make PoliPunch special beyond "it's a working fighting game."

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Political caricature fighters with unique ultimates | The entire comedic premise. Trump's golden wall, Biden's ice cream avalanche, Obama's drone strike, Bush's golf drive, Clinton's sax solo. These are the reason anyone shares this game. | HIGH | Each ultimate needs: meter check, activation animation, unique projectile/effect, hit resolution, and recovery. 5 totally unique sequences. This is where the game's personality lives. |
| Wall break / stage transition system | Visually spectacular. Not common in simple browser games. Gives a "wow, this is in a single HTML file?" reaction. | HIGH | Detect power attack near boundary + opponent in hit stun. Trigger slow-mo zoom. Transition to stage part 2 (new background). Bonus damage on the wall-break hit. Each of 4 stages needs two visual states. |
| Guard meter / guard crush | Prevents infinite blocking. Adds strategic depth beyond "hold back to win." Players who know fighting games will appreciate this; casual players benefit from it preventing stalemates. | MEDIUM | Visible guard meter below health bar. Depletes on blocked hits (more for heavy attacks). At zero: guard crush state (character staggers, vulnerable for ~30 frames). Regenerates when not blocking. |
| Juggle system with diminishing returns | Air combos feel satisfying. Gravity scaling prevents infinites naturally. Skill expression: better players get longer juggles. | MEDIUM | Power attack launch puts opponent in juggle state. Each subsequent hit in air: increased gravity (opponent falls faster), reduced hit stun, damage scaling (~80% -> 60% -> 40%). Natural 2-4 hit air combo limit without artificial cap. |
| Hit stop / freeze frames | The single highest-impact "game feel" technique. 2-4 frames of freeze on hit makes attacks feel powerful. Without it, combat feels floaty. SF2 uses this extensively. | LOW | On hit: pause both characters for N frames (2 for light, 4 for heavy, 8 for ultimate). Game loop skips position updates but still renders. Extremely cheap to implement, massive payoff. |
| Screen shake on heavy hits | Second-highest impact game feel technique after hit stop. Camera displacement on impact. | LOW | Offset canvas rendering by random x/y (2-4 pixels) for 4-8 frames on heavy hits. Decay the magnitude each frame. |
| Slow-motion on KO / ultimate / wall break | Dramatic emphasis on the most exciting moments. Makes finishes feel cinematic. | LOW | Reduce game speed to 0.25x-0.5x for 30-60 frames during KO blow, ultimate activation, and wall break. Simple multiplier on delta time. |
| Procedural synthesized music (background loops) | Background music makes the experience feel complete. No external files needed. | HIGH | Use Web Audio API to create simple looping melodies per stage. Square wave + triangle wave for retro feel. 4-8 bar loops. This is complex to make sound good but very impactful. Alternatively, a simpler drum-pattern-only approach is more feasible. |
| AI difficulty tiers (Easy/Medium/Hard) | Spec requirement. Makes game accessible to casual players while giving experienced players a challenge. | MEDIUM | State machine AI with behavior probabilities. Easy: slow reactions (15+ frame delay), rarely blocks, no combos. Medium: moderate reactions (8-12 frame delay), blocks sometimes, simple combos. Hard: fast reactions (3-5 frame delay), blocks consistently, uses combos and ultimates strategically. |
| Per-character stat differentiation | Each fighter feeling mechanically distinct (not just visually different) is what makes a roster interesting. | MEDIUM | Speed, damage, health, reach, combo length as tunable per-character values. Trump = slow/powerful/short combos. Biden = slow/tanky/grapple range. Obama = balanced. Bush = awkward timing/setups. Clinton = fast/evasive/counter-focused. |

### Anti-Features (Deliberately NOT Building)

Features that seem appealing but would destroy scope, add complexity without proportional value, or don't fit the single-file constraint.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Online multiplayer | "I want to fight my friends!" | Requires WebSocket server, netcode (rollback is complex), hosting infrastructure. Completely incompatible with single-file constraint. Would consume more dev effort than the entire rest of the game. | Local same-keyboard multiplayer could be a v2 feature (two players, split controls). Or just share the file and let friends play the AI. |
| Special move inputs (quarter-circle, Z-motion, charge) | "Real fighting games have hadouken motions!" | Input parsing for motion commands is surprisingly complex (buffering, leniency windows, distinguishing walk-forward from QCF). Adds major complexity to input system. Casual players can't execute them. | Simple button-press specials (tap vs hold) and ultimate on dedicated button. Depth comes from timing and spacing, not execution barriers. |
| Detailed frame data display / training mode | "I want to practice combos and see frame data" | Training mode is a separate game mode with its own UI (frame counter, hitbox display, dummy recording). Significant scope for a comedy game. | Combos should be intuitive enough to discover through play. Keep the system simple enough that explicit frame data isn't needed. |
| Character-specific move lists (6+ unique moves per fighter) | "Each character needs a huge moveset!" | 5 characters x 6+ unique moves = 30+ unique attack animations + hitbox tuning + balance testing. Scope explosion. | 2 attack buttons (fast/power) + ultimate = 3 distinct attacks per character (plus standing/crouching/air variants gives effective variety). Differentiation through stats and ultimate, not moveset breadth. |
| Throws / command grabs | "Fighting games need throws to beat blocking!" | Throw systems need proximity detection, throw break windows, throw invulnerability states, and throw-specific animations. Adds a full mechanic layer. | Guard meter already solves the "block forever" problem. Guard crush serves the same anti-turtle function throws do, without the implementation complexity. |
| Projectile attacks (fireballs) | "Ryu has a hadouken!" | Projectile systems need: projectile entity management, projectile-vs-projectile interaction, projectile speed/trajectory, anti-projectile moves. Ultimates already serve as the "cool ranged attack" for each character. | Ultimates fill this role. Some ultimates (Trump's wall, Obama's drone) are effectively projectiles. No need for a general projectile system. |
| Combo counter display | "Show me my 10-hit combo!" | Requires combo tracking logic (gap detection, display timing, positioning). Not hard but is pure UI polish with no gameplay impact. | Could be a post-MVP addition if time permits. Low priority. |
| Persistent unlockables / progression | "Unlock characters and stages!" | Requires save state (localStorage), unlock conditions, locked-state UI. Spec explicitly puts this out of scope. All content should be immediately available. | All 5 fighters and 4 stages available from the start. The game is a single-session experience. |
| Mobile / touch controls | "I want to play on my phone!" | Touch fighting game controls are notoriously bad. Would need virtual d-pad, button layout, touch event handling, responsive canvas sizing. Spec says keyboard-only. | Keyboard-only as specified. The game opens from a file -- desktop browser is the target. |
| External audio files / music tracks | "Use real music and sound effects!" | Violates the single-file, zero-external-dependencies constraint. Base64-encoded audio would massively bloat file size. | Procedural audio via Web Audio API oscillators. Retro synthesized sounds fit the pixel art aesthetic perfectly. |

## Feature Dependencies

```
[Game Loop (60fps)]
    +--requires--> [Input Handler (keyboard)]
    +--requires--> [Canvas Renderer]
    +--requires--> [Physics/Collision System]

[Physics/Collision System]
    +--requires--> [Hitbox/Hurtbox Definitions]
    +--requires--> [Character State Machine]

[Character State Machine]
    +--requires--> [Animation System (frame data)]
    +--requires--> [Movement System]

[Combat System]
    +--requires--> [Hitbox/Hurtbox Collision]
    +--requires--> [Character State Machine]
    +--requires--> [Health System]
    +--requires--> [Hit Stun / Block Stun]

[Combo System]
    +--requires--> [Combat System]
    +--requires--> [Cancel Windows (frame data)]

[Juggle System]
    +--requires--> [Combat System]
    +--requires--> [Launch State (airborne physics)]
    +--requires--> [Gravity Scaling]

[Guard Meter]
    +--requires--> [Block System]
    +--requires--> [Guard Crush State in State Machine]

[Ultimate System]
    +--requires--> [Combat System]
    +--requires--> [Ultimate Meter (charge from hits)]
    +--requires--> [Per-Character Ultimate Animations]

[Wall Break System]
    +--requires--> [Stage Boundary Detection]
    +--requires--> [Power Attack Near Wall + Opponent Hit]
    +--requires--> [Stage Transition (background swap)]
    +--requires--> [Slow-Mo System]

[AI Opponent]
    +--requires--> [Character State Machine]
    +--requires--> [Combat System]
    +--requires--> [Game State Reading (positions, health, meter)]

[Round System]
    +--requires--> [Health System]
    +--requires--> [Timer System]
    +--requires--> [Game State Management (round count, match state)]

[Game Flow (screens)]
    +--requires--> [Title Screen (input to start)]
    +--requires--> [Character Select (roster UI)]
    +--requires--> [Stage Select (stage thumbnails)]
    +--requires--> [Round System]
    +--requires--> [Victory Screen]

[Audio System]
    +--requires--> [Web Audio API AudioContext]
    +--enhances--> [Combat System (hit/block sounds)]
    +--enhances--> [Game Flow (menu sounds)]
    +--enhances--> [Ultimate System (activation sound)]

[Visual Effects (hit stop, screen shake, flash)]
    +--enhances--> [Combat System]
    +--enhances--> [Wall Break System]
    +--enhances--> [Ultimate System]
```

### Dependency Notes

- **Combo System requires Combat System:** Combos are sequences of hits; the underlying hit detection and stun system must work first.
- **Juggle System requires Launch State:** Air combos need airborne physics with gravity before juggle-specific mechanics can layer on top.
- **Wall Break requires Stage Boundaries + Slow-Mo:** Wall break is a synthesis of multiple systems: boundary detection, conditional trigger, stage transition, and time manipulation.
- **AI requires the full Combat System:** AI can only be built once all the moves it needs to use actually exist and function.
- **Visual Effects enhance but don't block:** Hit stop, screen shake, and flash can be added to an already-working combat system at any time. They are the highest-value low-effort additions.
- **Audio System is independent:** Web Audio API setup is orthogonal to gameplay. Can be developed in parallel and wired in later.
- **Game Flow is a wrapper:** Title, select, round, and victory screens are state management around the core fight. The fight must work first.

## MVP Definition

### Launch With (v1)

The minimum set where the game feels like a complete, playable fighting game:

- [ ] **Game loop at 60fps with keyboard input** -- nothing works without this
- [ ] **Canvas renderer with procedural character sprites** -- visual foundation (even simple stick figures work for v0)
- [ ] **Character state machine** (idle, walk, jump, crouch, attack, block, hit stun, knockdown, KO) -- the skeleton of all gameplay
- [ ] **Hitbox/hurtbox collision system** -- combat foundation
- [ ] **Fast attack and power attack** -- two-button combat with differentiation
- [ ] **Blocking with chip damage** -- defensive option
- [ ] **Basic combo system** (2-3 hit chains) -- minimum skill expression
- [ ] **Health bars and round timer** -- core HUD
- [ ] **Round system (best of 3)** -- match structure
- [ ] **5 distinct character stat profiles** -- roster differentiation
- [ ] **At least 1 stage background** -- visual context for the fight
- [ ] **Basic AI opponent** (even random actions) -- single-player requires an opponent
- [ ] **Game flow: title -> select -> fight -> victory** -- complete user journey
- [ ] **Hit sound effects via Web Audio API** -- combat without sound feels wrong
- [ ] **Hit stop on attacks** -- the single cheapest way to make combat feel good

### Add After Core Works (v1.x)

Features to layer on once the foundation is solid:

- [ ] **All 5 character unique animations** -- upgrade from placeholder art to full caricatures
- [ ] **Ultimate abilities (5 unique)** -- the game's main selling point, but requires working combat first
- [ ] **Guard meter / guard crush** -- strategic depth layer
- [ ] **Juggle system with gravity scaling** -- advanced combo expression
- [ ] **Wall break / stage transitions** -- spectacle feature, needs all 4 stages done
- [ ] **All 4 stage backgrounds** -- complete stage roster
- [ ] **AI difficulty tiers (Easy/Medium/Hard)** -- spec requirement, but basic AI comes first
- [ ] **Screen shake and slow-mo** -- polish layer
- [ ] **Full audio suite** (block sounds, menu sounds, KO sound, ultimate activation) -- audio completeness
- [ ] **Round splash screens** ("ROUND 1", "FIGHT!", "K.O.!") -- presentation polish

### Future Consideration (v2+)

- [ ] **Procedural background music** -- very high effort for quality, defer until everything else works
- [ ] **Combo counter display** -- pure UI polish, no gameplay impact
- [ ] **Local 2-player mode** (split keyboard) -- would be fun but doubles input complexity and isn't in spec
- [ ] **Additional characters or stages** -- spec says 5 fighters and 4 stages for v1, but the system should allow easy additions

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Game loop + input + renderer | HIGH | MEDIUM | P1 |
| Character state machine | HIGH | HIGH | P1 |
| Hitbox/hurtbox collision | HIGH | MEDIUM | P1 |
| Fast/power attacks | HIGH | MEDIUM | P1 |
| Blocking | HIGH | LOW | P1 |
| Health bars + timer HUD | HIGH | LOW | P1 |
| Basic combo system | HIGH | MEDIUM | P1 |
| Round system (best of 3) | HIGH | MEDIUM | P1 |
| Game flow (title/select/fight/victory) | HIGH | MEDIUM | P1 |
| Basic AI opponent | HIGH | MEDIUM | P1 |
| Hit sounds (Web Audio) | HIGH | LOW | P1 |
| Hit stop (freeze frames) | HIGH | LOW | P1 |
| Character animations (full set) | HIGH | HIGH | P1 |
| 5 distinct character stats | MEDIUM | LOW | P1 |
| Ultimate abilities (5 unique) | HIGH | HIGH | P2 |
| Guard meter / guard crush | MEDIUM | MEDIUM | P2 |
| Juggle system | MEDIUM | MEDIUM | P2 |
| Wall break / stage transitions | HIGH | HIGH | P2 |
| AI difficulty tiers | MEDIUM | MEDIUM | P2 |
| Screen shake | MEDIUM | LOW | P2 |
| Slow-motion (KO/ultimate/wall break) | MEDIUM | LOW | P2 |
| Hit sparks / particles | MEDIUM | LOW | P2 |
| All 4 stage backgrounds | MEDIUM | HIGH | P2 |
| Full audio suite | MEDIUM | MEDIUM | P2 |
| Round splash text animations | LOW | LOW | P2 |
| Procedural background music | MEDIUM | HIGH | P3 |
| Combo counter display | LOW | LOW | P3 |
| Local 2-player mode | LOW | MEDIUM | P3 |

**Priority key:**
- P1: Must have for the game to be playable and feel like a fighting game
- P2: Should have -- these are what make it a *good* fighting game and deliver on the spec's vision
- P3: Nice to have -- polish and scope extension

## Competitor/Reference Feature Analysis

| Feature | Street Fighter II (SNES) | MUGEN (PC) | Browser Fighting Games | Our Approach |
|---------|--------------------------|------------|------------------------|--------------|
| Characters | 8-12 with unique movesets | Unlimited (community) | 2-4 typically | 5 with shared move structure, differentiated by stats and ultimates |
| Combo system | Link-based, cancel-based, 2-8 hits | Varies wildly | Usually very basic (1-2 hits) | Cancel-based chains, 2-4 hit ground combos, 2-4 hit juggles |
| Special moves | Motion inputs (QCF, DP, charge) | Motion inputs | Button-press specials | Button-press only (fast/power/ultimate). No motion inputs. |
| Guard meter | Introduced in Alpha series | Optional | Rare | Yes -- prevents turtling, adds strategic depth |
| Stages | 8-12, static backgrounds | Unlimited | 1-2 typically | 4 with wall-break transitions (exceeds typical browser game) |
| Audio | Sampled sound effects, composed music | Varies | Often silent or minimal | Procedural synthesis via Web Audio API |
| AI | Pattern-based, difficulty scaling | Varies | Often very basic | State machine with 3 difficulty tiers |
| Visual effects | Hit sparks, screen flash | Varies | Minimal | Hit stop, screen shake, slow-mo, sparks (matches console quality feel) |

## Implementation Notes: Audio Without External Files

The constraint of zero external files makes audio a unique challenge. The approach:

1. **Web Audio API AudioContext** -- must be created after user interaction (browser autoplay policy). Initialize on first keypress or "PRESS START."
2. **Oscillator-based sound effects:**
   - **Punch hit:** White noise burst (50ms) through bandpass filter (800Hz) with sharp attack, fast decay
   - **Kick hit:** White noise burst (70ms) through bandpass filter (400Hz), slightly longer sustain
   - **Heavy hit:** Layered: low sine wave (80Hz, 100ms) + noise burst. Feels "meaty"
   - **Block:** Short sine wave (1200Hz, 30ms) with sharp cutoff. Clean "tink"
   - **KO:** Descending sine wave sweep (400Hz to 80Hz over 500ms). Dramatic
   - **Menu blip:** Sine wave (600Hz, 50ms). Clean selection sound
   - **Ultimate activation:** Ascending square wave sweep + noise burst. Dramatic buildup
3. **Optional: jsfxr inline** -- the jsfxr library is small enough to embed. It provides preset-based sound generation (hitHurt, explosion, powerUp) that maps well to fighting game needs. The entire library can be inlined into the HTML file.
4. **Background music (if attempted):** Simple square wave melody loop using Web Audio API scheduling. 4-bar patterns with bass (triangle wave) and lead (square wave). This is the hardest audio task and should be deferred.

## Implementation Notes: AI Opponent Behavior

State machine architecture for the AI:

```
States: IDLE, APPROACHING, ATTACKING, BLOCKING, RETREATING, USING_ULTIMATE
```

**Decision loop (runs every N frames based on difficulty):**
1. Read game state: distance to player, own health, player health, own meter, player state
2. If player is attacking and within range -> BLOCKING (probability based on difficulty)
3. If player is in recovery frames -> ATTACKING (punish opportunity)
4. If distance > attack range -> APPROACHING
5. If own health < 25% and meter full -> USING_ULTIMATE
6. If own health < player health -> more aggressive
7. Random action selection weighted by difficulty tier

**Difficulty scaling via reaction delay:**
- Easy: 15+ frame delay (250ms) before reacting. Blocks ~20% of attacks. Never combos. Uses ultimate randomly.
- Medium: 8-12 frame delay (133-200ms). Blocks ~50%. Simple 2-hit combos. Uses ultimate when advantageous.
- Hard: 3-5 frame delay (50-83ms). Blocks ~80%. Full combos. Baits player attacks. Strategic ultimate usage.

## Sources

- [Street Fighter II Hitboxes -- ComboVid](https://combovid.com/?p=956)
- [SF Seminar: The Basics of Boxes -- Capcom](https://game.capcom.com/cfn/sfv/column/131422?lang=en)
- [Guard Power Gauge -- Street Fighter Wiki](https://streetfighter.fandom.com/wiki/Guard_Power_Gauge)
- [Guard Crush -- The Fighting Game Glossary](https://glossary.infil.net/?t=Guard+Crush)
- [Juggling -- Street Fighter Wiki](https://streetfighter.fandom.com/wiki/Juggling)
- [Infinite Prevention and Combo Mechanics -- Dustloop](https://www.dustloop.com/w/User:Slimegirl-scientist/Infinite_Prevention_and_Combo_Mechanics)
- [Procedural Audio in JavaScript with Web Audio API -- DEV Community](https://dev.to/hexshift/how-to-create-procedural-audio-effects-in-javascript-with-web-audio-api-199e)
- [Synthesising Sounds with Web Audio API -- Sonoport](https://sonoport.github.io/synthesising-sounds-webaudio.html)
- [jsfxr: JavaScript Sound Effects Generator -- GitHub](https://github.com/chr15m/jsfxr)
- [OscillatorNode -- MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/OscillatorNode)
- [Game Juice -- Medium](https://sefaertunc.medium.com/game-design-series-ii-game-juice-92f6702d4991)
- [Screen Shake and Hit Stop Effects on Game Impact -- Oreate AI](https://www.oreateai.com/blog/research-on-the-mechanism-of-screen-shake-and-hit-stop-effects-on-game-impact/decf24388684845c565d0cc48f09fa24)
- [Adaptive AI for Fighting Games -- Stanford CS229](https://cs229.stanford.edu/proj2008/RicciardiThill-AdaptiveAIForFightingGames.pdf)
- [Fighting Game AI Practical Guide -- Andrea Jens](https://andrea-jens.medium.com/i-wanna-make-a-fighting-game-a-practical-guide-for-beginners-part-7-56f32f706a46)
- [Street Fighter II -- Wikipedia](https://en.wikipedia.org/wiki/Street_Fighter_II)

---
*Feature research for: PoliPunch -- Presidential Fighting Game*
*Researched: 2026-03-28*

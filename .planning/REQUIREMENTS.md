# Requirements: PoliPunch — Presidential Fighting Game

**Defined:** 2026-03-28
**Core Value:** A fully playable, fun fighting game that runs from a single HTML file — controls must feel responsive, characters must feel distinct, and every fight must have personality.

## v1 Requirements

### Engine

- [ ] **ENG-01**: Game runs in any modern browser opened directly from the local filesystem (file://) with no server, no CDN, no external files
- [ ] **ENG-02**: Game loop runs at 60fps using a fixed-timestep accumulator (requestAnimationFrame + 1/60s tick) so physics and combat timing are frame-rate independent
- [ ] **ENG-03**: All graphics rendered via Canvas 2D API using integer pixel coordinates with antialiasing disabled and CSS `image-rendering: pixelated` applied
- [ ] **ENG-04**: All sprite frames pre-rendered to offscreen canvases at initialization and blitted with drawImage() each frame (no per-frame fillRect sprite drawing)
- [ ] **ENG-05**: Input system tracks keydown/keyup state map for all bound keys with a 6-frame input buffer per player for combo detection
- [ ] **ENG-06**: Scene manager handles transitions: Title → CharSelect → StageSelect → Fight → Victory, with round splash overlay during fights

### Fighters

- [ ] **FTR-01**: All 5 fighters (Trump, Biden, Obama, Bush, Clinton) are fully playable with distinct stats (speed, power, reach)
- [ ] **FTR-02**: Each fighter has animated states: idle, walk forward, walk back, jump (rise/fall), crouch, fast attack, power attack, block, hit-stun, knockdown, ultimate, and victory pose
- [ ] **FTR-03**: Fighter data is fully data-driven — stats, frame data, hitboxes, and draw functions live in a character definition object; adding a fighter requires no engine code changes
- [ ] **FTR-04**: Trump — brash brawler archetype: wider power swings, slower walk speed, highest power stat; "Art of the Deal" ultimate spawns a golden wall that sweeps the screen multi-hitting the opponent
- [ ] **FTR-05**: Biden — tanky grappler archetype: highest health, longest reach, slow speed; "Ice Cream Avalanche" ultimate summons a wave of ice cream cones that freezes the opponent on hit
- [ ] **FTR-06**: Obama — balanced technical archetype: highest combo count, smooth attack animations, mid-range stats; "Drone Commander" ultimate shows targeting reticle locking on opponent then rains missiles
- [ ] **FTR-07**: Bush — unorthodox scrappy archetype: random-feeling timing variations, mid stats; "Now Watch This Drive" ultimate launches a flaming golf ball that ricochets around the stage hitting multiple times
- [ ] **FTR-08**: Clinton — evasive counter-fighter archetype: fastest walk speed, lowest health, best dodge frames; "Executive Charm" ultimate plays saxophone animation that stuns opponent then delivers a combo finisher

### Combat

- [ ] **CMB-01**: Fast Attack (tap J or Z): quick jab/kick with low startup, chainable into 3-4 hit combos on repeated taps within the input buffer window
- [ ] **CMB-02**: Power Attack (hold J or Z for ≥12 frames before release): slow heavy hit with telegraphed wind-up animation, deals more damage, causes wall-break if opponent near boundary, launches opponent for juggle
- [ ] **CMB-03**: Block (hold K or X): reduces incoming damage by 80%; depletes guard meter proportional to damage blocked
- [ ] **CMB-04**: Guard meter depletes on blocked hits and regenerates slowly when not blocking; guard break stuns the defender for 60 frames when depleted
- [ ] **CMB-05**: Juggle system: power attacks and certain combo enders launch the opponent airborne; follow-up fast attacks and power attacks extend the juggle (max 2-4 air hits) with gravity scaling (each hit adds 15% fall acceleration, each hit does 10% less damage)
- [ ] **CMB-06**: Hit stop (freeze frames): both fighters freeze for 3 frames on a successful hit, 5 frames on a power attack hit — makes impacts feel weighty
- [ ] **CMB-07**: AABB hitbox/hurtbox collision: each attack state defines active hitbox frames, position, and size; hurtbox shrinks during crouching; no active hitbox during startup or recovery frames
- [ ] **CMB-08**: Ultimate Ability (L or C): fills ultimate meter by dealing damage (1 point per damage unit) and taking damage (0.5 points per damage unit); meter fills at 100 points; screen flash on activation + unique character animation + high damage
- [ ] **CMB-09**: Movement: walk forward/back (A/D or Left/Right arrow), jump (W or Up arrow — parabolic arc), crouch (S or Down arrow — shrinks hurtbox)

### Stages

- [ ] **STG-01**: The White House — Oval Office interior with presidential décor; pixel art background rendered in canvas code
- [ ] **STG-02**: Greek Island — Cliffside arena with white-and-blue architecture and sea backdrop
- [ ] **STG-03**: Mar-a-Lago Club — Lavish ballroom with gilded décor
- [ ] **STG-04**: Golf Course — Open fairway with clubhouse backdrop
- [ ] **STG-05**: Each stage has two boundary walls; power attacking an opponent into a boundary triggers a wall break: screen goes to slow-mo (0.2x speed) for 30 frames, cinematic zoom punch-in, then transitions to the second tier of the stage with bonus damage applied
- [ ] **STG-06**: Each stage has a visually distinct second tier (White House → Rose Garden, Greek Island → beach, Mar-a-Lago → patio/pool, Golf Course → sand bunker) rendered as a new background canvas

### Game Flow

- [ ] **FLW-01**: Title screen with pixel art "POLIPUNCH" logo, fighter silhouette animation, and "PRESS ENTER TO START" prompt
- [ ] **FLW-02**: Character select screen showing all 5 fighters with pixel art portraits, player selects fighter, CPU randomly selects (no same-fighter mirror matches)
- [ ] **FLW-03**: Stage select screen showing all 4 stages with preview thumbnails
- [ ] **FLW-04**: Fight screen shows HUD: player health bar (left), CPU health bar (right), round counter (center top), countdown timer (60 seconds per round), ultimate meters (below health bars) — styled like a 90s arcade cabinet with pixelated font
- [ ] **FLW-05**: Round splash screens: "ROUND [N]" banner, then "FIGHT!" banner at round start; "K.O." splash on knockout; round win indicator updates on HUD
- [ ] **FLW-06**: Best-of-3 rounds per match; player/CPU who wins 2 rounds first wins the match
- [ ] **FLW-07**: Victory screen shows winner with victory pose animation, loser slumped, "WINNER!" text, and "PRESS ENTER to play again" prompt

### AI

- [ ] **AI-01**: CPU AI selects difficulty at character select screen: Easy / Medium / Hard
- [ ] **AI-02**: AI uses a state machine (approach, pressure, back-off, punish) with reaction delay: Easy = 250ms, Medium = 150ms, Hard = 65ms
- [ ] **AI-03**: AI blocks incoming attacks with probability: Easy = 20%, Medium = 50%, Hard = 80%
- [ ] **AI-04**: AI attempts combos (fast attack chains) and uses ultimate ability when meter is full
- [ ] **AI-05**: AI uses movement — walks forward when far, backs off when blocking, jumps occasionally

### Audio

- [ ] **AUD-01**: Sound effects via Web Audio API (inline, no external files): punch impact, power hit impact, block, guard break, wall break, KO, ultimate activation — generated via oscillator/noise synthesis
- [ ] **AUD-02**: Looping chiptune-style background music per stage via Web Audio API oscillator sequences (simple melodic patterns, separate for each stage)
- [ ] **AUD-03**: Audio volume balanced — SFX audible over music; graceful degradation if Web Audio API is blocked by browser

## v2 Requirements

### Multiplayer

- **MULT-01**: Two-player local multiplayer on same keyboard (Player 1: WASD+JKL, Player 2: Arrows+numpad or custom)

### Content

- **CONT-01**: Additional fighters beyond the base 5
- **CONT-02**: Additional stages beyond the base 4
- **CONT-03**: Story/arcade mode with pre-fight dialogue between fighters

### Polish

- **POL-01**: Screen shake on power hit / wall break
- **POL-02**: Particle effects (hit sparks, dust on landing)
- **POL-03**: Pause menu with resume/restart/quit options
- **POL-04**: Fighter-specific voice barks (synthesized via Web Audio API)

## Out of Scope

| Feature | Reason |
|---------|--------|
| External assets (images, audio files) | Core constraint — must be zero-dependency single file |
| Networking / online multiplayer | Complexity vs. playability tradeoff; not in spec |
| Mobile / touch controls | Keyboard-only per spec |
| Save states / persistent progress | Session-only; no localStorage needed |
| Motion inputs (quarter-circles, Z-motions) | Research flagged as anti-feature for this game's accessibility goals |
| Pixel-perfect collision | AABB hitboxes are sufficient and standard for this genre |
| Build tooling / bundler | Single file constraint; written directly |

## Traceability

*(Populated by roadmap agent)*

| Requirement | Phase | Status |
|-------------|-------|--------|
| ENG-01 | — | Pending |
| ENG-02 | — | Pending |
| ENG-03 | — | Pending |
| ENG-04 | — | Pending |
| ENG-05 | — | Pending |
| ENG-06 | — | Pending |
| FTR-01 | — | Pending |
| FTR-02 | — | Pending |
| FTR-03 | — | Pending |
| FTR-04 | — | Pending |
| FTR-05 | — | Pending |
| FTR-06 | — | Pending |
| FTR-07 | — | Pending |
| FTR-08 | — | Pending |
| CMB-01 | — | Pending |
| CMB-02 | — | Pending |
| CMB-03 | — | Pending |
| CMB-04 | — | Pending |
| CMB-05 | — | Pending |
| CMB-06 | — | Pending |
| CMB-07 | — | Pending |
| CMB-08 | — | Pending |
| CMB-09 | — | Pending |
| STG-01 | — | Pending |
| STG-02 | — | Pending |
| STG-03 | — | Pending |
| STG-04 | — | Pending |
| STG-05 | — | Pending |
| STG-06 | — | Pending |
| FLW-01 | — | Pending |
| FLW-02 | — | Pending |
| FLW-03 | — | Pending |
| FLW-04 | — | Pending |
| FLW-05 | — | Pending |
| FLW-06 | — | Pending |
| FLW-07 | — | Pending |
| AI-01 | — | Pending |
| AI-02 | — | Pending |
| AI-03 | — | Pending |
| AI-04 | — | Pending |
| AI-05 | — | Pending |
| AUD-01 | — | Pending |
| AUD-02 | — | Pending |
| AUD-03 | — | Pending |

**Coverage:**
- v1 requirements: 44 total
- Mapped to phases: 0
- Unmapped: 44 ⚠️ (pending roadmap)

---
*Requirements defined: 2026-03-28*
*Last updated: 2026-03-28 after initial definition*

# Phase 1: Engine + Core Combat - Context

**Gathered:** 2026-03-28
**Status:** Ready for planning

<domain>
## Phase Boundary

Deliver a single HTML file that runs in any browser from `file://`, showing one playable fighter (Trump) with responsive controls, core combat mechanics (fast attack, power attack, block, hit stop), and a HUD — on a fully rendered White House Oval Office stage background. A walk-and-block placeholder CPU provides a target to fight against.

This phase establishes the entire engine foundation. Everything else (AI, more fighters, ultimates, wall breaks) builds on top of it.

</domain>

<decisions>
## Implementation Decisions

### Sprite Scale & Detail
- **D-01:** Fighter sprites are **48px tall** at 400×240 internal resolution (~1/5 screen height). Chunky and readable — enough pixels for distinct silhouettes and political caricature recognition.
- **D-02:** **4 animation frames per state** across all animation states (idle, walk, attack, etc.). ~44 offscreen canvases total at init. Smooth enough to read clearly without becoming an art project.

### Stage Art
- **D-03:** The White House Oval Office background is built to **ship quality in Phase 1** — not a placeholder. Build the real scene (desk, windows, presidential seal, American flags, dark wood floor) in canvas pixel art. No rework needed in later phases.

### Placeholder CPU Fighter
- **D-04:** Placeholder CPU **walks toward player at 50% speed and randomly blocks 20% of the time**. Never attacks. Respects health and KO system. Provides a moving target to validate hitboxes, hit stop, and damage systems without building Phase 2 AI.

### Combat Timing & Feel (Claude's Discretion)
- **D-05:** Frame data decisions are delegated to Claude. Target: SF2-inspired tight windows (fast attack ~4 startup / 4 active / 8 recovery; power attack ~12 startup per ENG-05 spec / 5 active / 18 recovery). Combo chain window: tap within 6 frames of active (matching ENG-05's input buffer). Per-character timing variations between fighters are also Claude's discretion — user does not want to be consulted on tuning details above the level of overall "feel" target (90s arcade cabinet, classic SF2-style).
- **D-06:** Hit stop values are locked by requirements: 3 frames on normal hit, 5 frames on power attack hit (CMB-06).

### Claude's Discretion
- Combat frame data specifics (startup/active/recovery counts per move)
- Per-character stat variations (speed, power, reach multipliers)
- Color palette choices for sprites and stage art
- Internal code architecture details (state machine structure, hitbox data shape, etc.)
- Animation easing and timing within stated frame budgets

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project Constraints
- `CLAUDE.md` — Zero-dependency constraint, Canvas 2D API patterns, `image-rendering: pixelated`, offscreen canvas caching pattern, fixed-timestep loop, AABB collision, frame-data-driven attack system. All technology decisions are locked here.

### Requirements
- `.planning/REQUIREMENTS.md` — Full requirement IDs for Phase 1: ENG-01 through ENG-06, FTR-02, FTR-03, FTR-04, CMB-01, CMB-02, CMB-03, CMB-06, CMB-07, CMB-09, STG-01, FLW-01, FLW-04

### Roadmap
- `.planning/ROADMAP.md` §Phase 1 — Success criteria (5 items) that define done for this phase

No external specs — requirements fully captured in decisions and references above.

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- None — this is a fresh start. No existing codebase.

### Established Patterns
- None yet — Phase 1 establishes all patterns for subsequent phases.

### Integration Points
- This phase IS the foundation. All future phases integrate into the engine, fighter data structure, and scene manager built here.
- **Critical:** FTR-03 requires the fighter data structure be fully data-driven. Trump's definition in Phase 1 IS the template — adding Biden/Obama/etc. later must require zero engine changes.

</code_context>

<specifics>
## Specific Ideas

- Political caricature recognition is important — at 48px tall, Trump should have a distinguishable hair silhouette, suit, and skin tone. The chunky pixel art style should exaggerate these features.
- The Oval Office should feel like a "real" stage from the moment Phase 1 ships — not a colored rectangle placeholder.
- The placeholder CPU is not Biden — it's a generic dummy that will be replaced by real fighters in Phase 2+. It can use Trump's sprite flipped/recolored or be a generic colored rectangle.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 01-engine-core-combat*
*Context gathered: 2026-03-28*

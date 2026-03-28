# Phase 1: Engine + Core Combat - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-03-28
**Phase:** 01-engine-core-combat
**Areas discussed:** Sprite scale & detail, Stage art ambition, Combat timing / feel, Placeholder CPU fighter

---

## Sprite Scale & Detail

### Fighter height

| Option | Description | Selected |
|--------|-------------|----------|
| 48px tall | ~1/5 screen height. Chunky and readable, enough pixels for clear facial caricatures and distinct silhouettes. Most 16/32-bit fighters land here. | ✓ |
| 64px tall | More pixel real estate for detail — hair, face features, clothing. But fighters will feel large relative to the arena. | |
| 32px tall | Gameboy / NES scale. Very chunky — minimal face/body distinction. Fastest to draw but harder to make caricatures read clearly. | |

**User's choice:** 48px tall
**Notes:** Classic fighting game scale, good balance of detail vs. chunky pixel art feel.

### Animation frames per state

| Option | Description | Selected |
|--------|-------------|----------|
| 4 frames / state | Smooth enough to read clearly, manageable scope. ~44 offscreen canvases total. | ✓ |
| 6-8 frames / state | Noticeably smoother, closer to SF2 fidelity. ~88-132 canvases. More art work. | |
| 2 frames / state | Minimal / game-jam style. Gets engine working fast but looks rough. | |

**User's choice:** 4 frames per state
**Notes:** Good balance of visual quality and implementation effort.

---

## Stage Art Ambition

| Option | Description | Selected |
|--------|-------------|----------|
| Ship-quality from day one | Real Oval Office background: desk, windows, presidential seal, flags. Ships in final game — no rework. | ✓ |
| Rough scaffolding first | Placeholder background (sky blue + floor + rect walls). Faster, but rework needed in Phase 3. | |

**User's choice:** Ship-quality from day one
**Notes:** More upfront work but zero rework later — Oval Office is the only stage in Phase 1.

---

## Combat Timing / Feel

| Option | Description | Selected |
|--------|-------------|----------|
| Claude's Discretion | User requested that frame data, character tuning, and timing decisions above the level of overall feel be automated by Claude. | ✓ |

**User's choice:** Delegate to Claude — "automate decisions above character differences — do more research if needed"
**Notes:** Overall feel target is 90s arcade cabinet / classic SF2-style (tight windows, punishing misses). Claude will research and decide specific frame counts, per-character stat variations, and other tuning details.

---

## Placeholder CPU Fighter

| Option | Description | Selected |
|--------|-------------|----------|
| Walk & block dummy | Walks toward player at 50% speed, blocks 20% randomly. Never attacks. | ✓ |
| Static dummy | Just stands there and takes hits. Pure hitbox testing. | |
| Basic AI stub | Minimal Phase 2 AI skeleton: approach/attack/block. More work, Phase 2 head start. | |

**User's choice:** Walk & block dummy
**Notes:** Provides a moving target to validate combat mechanics without pre-building Phase 2 AI.

---

## Claude's Discretion

- Combat frame data specifics (startup/active/recovery counts per move)
- Per-character stat variations (speed, power, reach multipliers)
- Color palette choices for sprites and stage art
- Internal code architecture details
- Animation easing and timing within stated frame budgets

## Deferred Ideas

None raised during discussion.

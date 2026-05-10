# Aberrant — Prototype

**Status:** In Progress  
**File:** `prototypes/aberrant.html` (single-file, open in any browser)  
**Last updated:** 2026-05-10 — formation rework + late-game rebalance

---

## Hypothesis Being Tested

Can a Vampire Survivors-style auto-battler feel distinct and commercially viable
by centering on a **psionic dominator** fantasy — where the core loop is not
just surviving waves but converting enemies into a growing thrall army that
fights for you?

---

## How to Run

Open `prototypes/aberrant.html` directly in a browser. No build step, no server needed.

---

## Game Concept

You play as an **Aberrant** — a psionic entity with the power to dominate the
minds of creatures. Waves of enemies charge you. You fire psionic bolts to
weaken them; when HP hits zero, they are **dominated** (not killed) and join
your orbiting thrall army. Thralls orbit you and fire bolts autonomously. The
goal is to survive 10 waves.

**Tone:** Dark psychic horror. DnD Beholder meets Fallout Master meets Vampire Survivors.

---

## Core Loop

1. Wave begins — enemies spawn in a ring around the player at off-screen distance
2. Player moves with WASD, bolts auto-fire at nearest enemies
3. Weakened enemies are dominated → join thrall orbit
4. Thralls orbit and auto-fire, creating a compound defensive/offensive layer
5. Wave cleared → upgrade screen (3 random choices from upgrade pool)
6. Repeat through wave 10

---

## Current Feature State

### Player
- WASD movement, 220 px/s
- Auto-firing psionic bolts at nearest enemy within range (dotted ring indicator)
- 1.6 bolts/sec base rate, 22 damage per bolt
- 100 HP (Sanity bar, displayed bottom of screen)

### Enemies
Three creature types, each with unique stats:

| Type | Base HP | Base Speed | Base Damage | Appears |
|------|---------|------------|-------------|---------|
| Gnoll | 32 | 145 px/s | 14 | Wave 1+ |
| Beholder | 75 | 90 px/s | 20 | Wave 3+ |
| Troll | 130 | 68 px/s | 30 | Wave 6+ |

- HP scales per wave: `baseHp × 1.22^(wave-1)`
- Speed scales per wave: `baseSpd × (1 + 0.28 × (wave-1))`
- Spawn count: `8 + wave × 5` per wave, spawning at `max(W,H) × 0.6` radius around player
- Spawn interval: `max(35, 550 - wave × 55)` ms

### Thralls
- Dominated creatures join an orbiting horde that auto-fires bolts.
- **Cap:** `THRALL_CAP = 60`. Kills beyond the cap award **2× essence** instead of
  adding a thrall — late-game power is bounded.
- **Ring formation:** 12 thralls per ring, ring gap `0.06 × min(W,H)`, inner ring
  radius `0.1 × min(W,H)`. With cap 60, the formation fills exactly 5 rings.
- **Newest fills innermost slot.** When the inner ring is full, the next dominate
  pushes the prior cohort outward to the next ring (and so on). On death,
  survivors glide inward — the formation behaves like rippling rings, not a
  stacked snap.
- **Per-ring rotation:** each ring has its own continuous phase. Base speed
  `0.06 rad/s` with shallow decay, alternating direction by ring index. Inner
  ring drifts ~3°/s; outer rings nearly still.
- **Smooth transit:** each thrall's actual `orbitR` lerps to its target with
  exponential decay (rate 3.5/s); its slot-angle lerps at rate 5/s. New thralls
  fly in from the creature's death position rather than teleporting to formation.
- **Stats:** HP = `creatureBaseHp × 0.75 × 1.10^(wave-1)`; bolt damage =
  `playerBoltDmg × 0.26`; fire rate `0.32/s` base (cooldown ~3.1s).
- Dead thralls show a **BROKEN FREE** floater + red/purple particle burst.

### Enemy AI
- All enemies chase the player by default.
- Redirect threshold tightened to `playerDist × 0.2` — thralls only intercept when
  essentially adjacent. Most charges punch through the ring to hit the player.
- Thrall attrition still happens organically when a creature collides with the
  nearest thrall on its path to the player.

### Camera
- Vampire Survivors-style: player is always screen-center, world scrolls around them
- Scrolling parallax grid background for spatial reference
- Radial purple vignette at screen edges

### HUD
- Wave counter (top center)
- Essence + thrall count (top right)
- Sanity bar with color coding: purple → amber → red (bottom)
- Thrall panel (right side): shows dominated creatures with type, HP bar, wave captured
- **Offscreen battle arrows**: triangles at screen edges pointing toward any
  offscreen thrall in combat. Amber when an enemy is within ~110 units of the
  thrall (proximity), brighter red flash when the thrall is actively taking
  damage. (Previously only triggered on direct hits, which almost never fired.)

### Upgrades (shown between waves, 3 random choices)
| ID | Name | Effect |
|----|------|--------|
| surge | Synaptic Surge | Bolt rate ×1.35 |
| neuro | Neural Fortitude | +40 max sanity, restore to 75% |
| shatter | Mind Shatter | Bolt damage ×1.5 |
| hive | Hive Resonance | Thrall attack speed ×2 |
| reach | Psionic Reach | Bolt range +30% |
| frenzy | Thrall Frenzy | Thrall fire rate ×1.6 |
| carap | Eldritch Carapace | +60 max sanity, restore to 80% |
| lance | Void Lance | Bolt speed ×1.45 |
| drain | Aberrant Drain | +8 essence/sec passive |
| split | Split Consciousness | Fire at 2 targets simultaneously (unique) |
| pierce | Psionic Pierce | Bolts pierce through enemies (unique, requires Split) |

---

## Visual Design

- **Player**: Purple aberrant shape with animated tentacle-like silhouette
- **Enemies**: Spiky 9-point rotating star shapes (hostile, chaotic feel)
- **Thralls**: Smooth circles with pulsing cyan psionic halo (clearly "yours")
- **Bolts**: Small glowing projectiles (purple = player, per-thrall color = thrall)
- **Particles**: Death/domination burst effects in themed colors
- **UI**: Dark `#0a0a14` background, purple/cyan accent palette throughout

---

## Resolved Design Problems

| Problem | Solution |
|---------|----------|
| Thralls and enemies looked identical | Spiky star shape for enemies, smooth circle + cyan halo for thralls |
| Arena felt wrong (hard clamp at screen edges) | Removed arena, switched to VS-style camera follow |
| Thrall army became trivially overpowered | Exponential HP scaling (1.22^wave), thrall damage at 38%, enemy interception targeting |
| Battles drifting off-screen | Camera follow keeps player centered; all combat stays in view |
| No spatial awareness of thrall fights | Offscreen battle arrows added |
| Orbit looked like "planets on dotted lines" | Removed orbit path rendering |
| Thralls capped too early | Removed cap entirely, camera follow solved the real problem |
| Split Consciousness stackable | Marked `unique: true`, gated Psionic Pierce behind it |
| Battle arrows almost never appeared | Broadened trigger from "under direct attack" to "any enemy within ~110 units"; tiered visual (amber proximity, red flash on hit) |
| Orbit rings snapped outward when a new ring formed | Removed global scale factor; per-ring radius is fixed |
| Ring rotation felt chaotic / too fast | Per-ring shared rotation (alternating direction), base speed dropped 0.55 → 0.06 rad/s |
| New thralls popped into formation, deaths snapped survivors | All transitions lerp (exp decay); new thralls fly in from death position; ring changes preserve visual angle (no snap) |
| Newest thralls ended up in outer rings, oldest closest — backwards | Reverse-indexed: newest fills innermost slot 0; cohorts shift outward as the inner ring fills |
| Late game trivially won — never got charged | Cap of 60 thralls + 2× essence past cap; thrall HP cut ~80%; thrall DPS roughly halved (fire rate 0.50 → 0.32, bolt damage 0.38 → 0.26 of player); redirect threshold 0.4 → 0.2 lets creatures reach the player |

---

## Open Questions / Next Experiments

- Does the rippling ring expansion read clearly as "growth" to a first-time player?
- Should essence have a spend mechanic (active abilities) or remain passive score?
- Is wave 10 an appropriate length for a commercial loop, or should it be endless?
- Potential new enemy type: ranged attacker that stays outside thrall orbit radius.
- Once the cap is hit, the upgrade choices ("Hive Resonance", "Thrall Frenzy") still
  matter for DPS — but is "Eldritch Carapace" / sanity upgrades better tuned now?
- Should the inner ring slots be a "promoted" tier (newest in, but oldest elsewhere
  gets buffed)? Currently being old is purely worse — older thralls take less inner
  shielding and have the same stats.

---

## Findings (Updated as prototype evolves)

- The domination fantasy works — converting enemies feels meaningfully different from killing them.
- Thrall attrition (enemies retargeting thralls) adds tension without explicit capping.
- VS-style camera is non-negotiable for this design; screen-clamp felt claustrophobic.
- Bullet density matters as much as damage numbers — felt much better after bolt rate increase.
- **Unbounded thrall growth breaks late-game tension.** Without a cap, the ring becomes a
  fortress that kills everything before it can close on the player. `THRALL_CAP = 60` plus
  a 2× essence bonus past the cap preserves the power fantasy without trivialising the loop.
- **Ring formation needs to feel alive, not snappy.** Reverse-indexed slots (newest inner),
  exponential lerps on radius/angle, and per-ring shared rotation produced the desired
  "rippling rings" growth pattern. Independent per-thrall random orbit speeds looked chaotic
  by comparison.
- **Combat indicators have to fire often.** The original "under attack" trigger was so
  narrow it almost never appeared. Proximity-based triggers (amber) plus damage flashes
  (red) feel reliably informative.
- **Thrall HP and firepower compound.** Halving both at once (HP × ~0.4 vs original; DPS × ~0.5)
  was needed to bring the late game back into pressure range. Either change alone wasn't enough.

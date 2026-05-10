# Aberrant — Prototype

**Status:** In Progress  
**File:** `prototypes/aberrant.html` (single-file, open in any browser)  
**Last updated:** 2026-05-10

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
- Dominated creatures orbit the player, auto-firing bolts
- Orbit system uses **expanding rings**: first 6 thralls form ring 0, next 6 ring 1, etc.
- **Dynamic orbit scaling**: all ring radii expand together as new rings are added
  (`scale = 1 + (totalRings - 1) × 0.45`) so the formation grows outward organically
- Thrall bolt damage: 38% of player bolt damage (weaker but compound)
- Thrall fire rate: 0.5/sec base
- Thralls have their own HP (3× creature base HP, scaled to wave)
- Thralls can die if enemies intercept them; dead thralls "break free" with particle effect

### Enemy AI
- All enemies chase the player by default
- If an enemy is within `playerDist × 0.4` of a thrall, it redirects to attack that thrall
- This creates natural thrall attrition — larger armies draw some fire

### Camera
- Vampire Survivors-style: player is always screen-center, world scrolls around them
- Scrolling parallax grid background for spatial reference
- Radial purple vignette at screen edges

### HUD
- Wave counter (top center)
- Essence + thrall count (top right)
- Sanity bar with color coding: purple → amber → red (bottom)
- Thrall panel (right side): shows dominated creatures with type, HP bar, wave captured
- **Offscreen battle arrows**: flashing red triangles at screen edges pointing toward
  any thrall currently being attacked outside the visible viewport

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

---

## Open Questions / Next Experiments

- Does the orbit expansion feel rewarding enough visually as the army grows?
- Should essence have a spend mechanic (active abilities) or remain passive score?
- Is wave 10 an appropriate length for a commercial loop, or should it be endless?
- Potential new enemy type: ranged attacker that stays outside thrall orbit radius

---

## Findings (Updated as prototype evolves)

- The domination fantasy works — converting enemies feels meaningfully different from killing them
- Thrall attrition (enemies retargeting thralls) adds tension without explicit capping
- VS-style camera is non-negotiable for this design; screen-clamp felt claustrophobic
- Bullet density matters as much as damage numbers — felt much better after bolt rate increase
- Dynamic orbit rings need further playtesting: formation growth is satisfying but radius pop
  on domination may be jarring at high thrall counts

# 🐑 Flappy Sheep: Barnbuster — Game Design Spec

## Overview

A browser-based Flappy Bird-style side-scroller built in vanilla HTML5 Canvas + JavaScript. The player controls a sheep in a jetpack wearing a bandana. The sheep navigates through storm clouds, shoots TNT projectiles, and drops bombs on barns for points.

---

## Tech Stack

- **Runtime**: Vanilla HTML5 + JavaScript (no frameworks)
- **Rendering**: HTML5 Canvas 2D API
- **Audio**: Web Audio API (procedurally generated SFX) or `<audio>` tags for .mp3/.ogg files
- **No external dependencies** — single self-contained `.html` file deliverable

---

## Controls

| Key | Action |
|---|---|
| `↑` Arrow / `Click` / `Tap` | Flap (gain altitude) |
| `Space` | Shoot — fires a TNT bundle forward horizontally |
| `Enter` | Bomb — drops a bomb straight down |

> **Design note**: Space and Enter are distinct mechanics. TNT (Space) is a fast forward projectile that destroys clouds. Bombs (Enter) are slower, fall with gravity, and destroy barns. They have different animations and splash radii.

---

## Player Character

**Sprite**: A white fluffy sheep wearing:
- A red bandana around its neck
- A jetpack strapped to its back (flame/smoke exhaust on flap)
- Goggles (optional but extremely good)

**Physics** (Flappy Bird model):
- Constant downward gravity applied every frame
- Each flap applies an upward velocity impulse
- Horizontal position is fixed ~25% from the left edge
- Vertical position is clamped to canvas bounds (top/bottom = instant death)

**Animations**:
- **Idle**: Gentle bobbing, wool fluffing
- **Flap**: Jetpack flame burst, brief upward tilt
- **Shoot**: Brief recoil, TNT lobs out of the sheep's hooves
- **Bomb**: Sheep raises a round black bomb overhead, drops it below
- **Death**: Sheep tumbles with stars, jetpack sputters

---

## Obstacles — Gray Clouds ☁️

- Rendered as organic cloud shapes (overlapping white/gray ellipses)
- Scroll from right to left at increasing speed
- Arranged in classic Flappy Bird gap pairs (top cloud + bottom cloud with a gap to fly through)
- **Contact with cloud** = instant death
- **TNT hit** = cloud dissolves with a small puff particle effect (no points awarded)
- Cloud gap width narrows slightly as score increases (difficulty ramp)
- Cloud speed increases gradually over time

**Spawn logic**:
- Clouds spawn off-screen right, paired with a randomized vertical gap position
- Minimum gap: 150px → 100px (at high score)
- Spacing between cloud pairs: ~300px (decreases slightly with difficulty)

---

## Targets — Barns 🏚️

- Barns appear on the ground (bottom ~15% of canvas) as red wooden barn sprites
- They scroll from right to left with the scene
- They do **not** block the player's flight path — they're ground-level targets only
- **Bomb hit** = barn explodes, +50 points, major animation event (see Animations)
- Barns can also be destroyed by TNT if it happens to fall to ground level, but bombs are the intended mechanic
- Barns are spaced irregularly; roughly 1 per 4–6 seconds at baseline speed

**Visual**: Classic red barn silhouette with a pitched roof, white X cross on doors, maybe a weather vane

---

## Scoring

| Event | Points |
|---|---|
| Survival | +1 per second (continuous) |
| Barn bombed | +50 points flat |
| Cloud shot | +0 (no reward, just utility) |

- Score displayed top-center in large pixel font
- High score persisted in `localStorage`
- Score multiplier NOT included in v1 (keep it clean)

---

## Projectiles

### TNT Bundle (Space)

- Sprite: Small bundled sticks of TNT with a lit fuse
- Fires horizontally from the sheep's position
- Travels at ~2× the cloud scroll speed
- Gravity: None (travels in a straight line)
- On cloud contact: small smoke/puff particle burst, cloud removed
- On barn contact: small explosion (less dramatic than bomb)
- Max 3 TNT on screen at once (fire rate limiting)
- Fuse sparks animate during flight

### Bomb (Enter)

- Sprite: Classic round black bomb with a white shine highlight and a lit fuse
- Drops from the sheep's position
- Subject to gravity — accelerates downward in an arc
- On barn contact: **BARN EXPLOSION** (see below)
- On ground contact (no barn): small dirt puff, no damage
- On cloud contact: passes through (bombs are for barns)
- Max 1 bomb on screen at once (cooldown: 2 seconds after previous bomb lands)

---

## Animations

### TNT Hit on Cloud
- 8–12 gray smoke particles burst outward from impact point
- Each particle: small circle, fades out over ~400ms while drifting
- Cloud sprite dissolves (scale up + fade out over 300ms)

### Barn Explosion 💥 *(THE BIG ONE)*

This is the hero animation. It should be deeply satisfying.

**Phase 1 — Impact Flash (0–100ms)**:
- Full-screen white flash that fades out quickly
- Canvas shakes (translate ±5px randomly for ~300ms)

**Phase 2 — Fireball (100–600ms)**:
- Large expanding orange/yellow circle at barn position
- Inner circle: bright yellow core
- Outer ring: orange, then red at edges
- Core scales from 0 → 120px radius with easing

**Phase 3 — Debris (100–1200ms)**:
- 15–20 barn debris particles (small red/brown rectangles, splinters)
- Launch upward and outward with random angles and velocities
- Subject to gravity — arc upward then fall
- Each piece spins as it travels
- Some pieces fly off-screen, some land and slide

**Phase 4 — Smoke Column (400ms–2000ms)**:
- Black/gray smoke particles rise from barn position
- Billow outward as they rise
- Fade out over ~1500ms

**Phase 5 — Score Pop (immediately)**:
- "+50" text in bold yellow, outlined in black
- Scales up from 0 → 1.5× → 1× over 300ms
- Floats upward and fades out over 800ms

**Phase 6 — Afterglow (600ms–2000ms)**:
- Ember particles: tiny orange dots that drift upward slowly, fade out
- 20–30 embers, staggered spawn timing

**Barn aftermath**: The barn sprite is replaced by a small scorched ground mark (black ellipse) for the remainder of the run.

### Player Death Animation

- Sheep tumbles (rotation increases rapidly)
- Jetpack emits sputtering sparks
- 3 cartoon stars orbit the sheep's head
- Sheep falls off the bottom of the screen
- **GAME OVER** screen fades in (see UI)

---

## UI / Screens

### Title Screen
- Game title: **"Flappy Sheep: Barnbuster"** in large blocky/pixel font
- Animated sheep bouncing in place
- "Click or Press ↑ to Start" prompt
- Blinking/pulsing animation on prompt
- High score displayed: "Best: XXXX"

### HUD (in-game)
- **Top center**: Current score (large, pixel font)
- **Top right**: High score
- **Bottom left**: Small bomb cooldown indicator (grays out for 2s after bomb drop)
- **Bottom right**: TNT ammo count (shows dots for remaining on-screen TNT)

### Game Over Screen
- Semi-transparent dark overlay
- "BAAAAAH!" in large text (sheep death cry)
- Score: "You scored: XXXX"
- High score: "Best: XXXX" (with a "NEW BEST! 🎉" banner if applicable)
- Restart button / "Press Space to restart"

---

## Audio (Optional but Recommended)

| Event | Sound |
|---|---|
| Flap | Short jetpack whoosh/puff |
| TNT fire | Cartoon boink/spring |
| TNT hit cloud | Soft poof/pop |
| Bomb drop | Cartoon whistle descend |
| Barn explosion | Deep boom + crackling |
| Player death | Sad trombone / cartoon splat |
| Score tick | Soft coin/tick every 10 seconds |
| New high score | Triumphant fanfare |

> Implement with Web Audio API oscillators for MVP, or swap in .mp3 assets.

---

## Difficulty Ramp

| Score Range | Cloud Speed | Gap Size | Cloud Spacing |
|---|---|---|---|
| 0–100 | 2.5 px/frame | 160px | 320px |
| 100–300 | 3.0 px/frame | 150px | 300px |
| 300–600 | 3.5 px/frame | 140px | 280px |
| 600–1000 | 4.0 px/frame | 130px | 260px |
| 1000+ | 4.5 px/frame | 120px | 250px |

Barns also scroll at cloud speed (scene consistency).

---

## Canvas / Rendering Notes

- **Resolution**: 800 × 500px canvas, centered in viewport, scales with CSS
- **Frame rate**: `requestAnimationFrame` loop, delta-time based physics
- **Layers** (render order, back to front):
  1. Sky gradient background (light blue → white)
  2. Scrolling ground strip (green with texture)
  3. Barns (ground level)
  4. Clouds (obstacle pairs)
  5. Projectiles (TNT bundles, bombs)
  6. Player sheep
  7. Particle effects / explosions
  8. HUD overlay
- All sprites can be drawn via Canvas 2D primitives (no image assets required) — use geometric shapes + Canvas paths for a clean pixel-art-adjacent look

---

## Out of Scope (v1)

- Power-ups
- Multiple sheep characters / skins
- Mobile touch controls (though tap-to-flap works by default via click)
- Online leaderboards
- Enemy AI / projectiles

---

## Deliverable

- Single `.html` file, no external dependencies
- Works offline in modern browsers (Chrome, Firefox, Safari)
- All sprites drawn in Canvas (no image files required)
- Estimated LOC: ~600–900 lines

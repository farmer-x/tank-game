# Claude Code Prompt: Build a Scorched Earth Clone

## Project Overview

Build a faithful, playable recreation of the classic 1991 MS-DOS artillery game **Scorched Earth** ("The Mother of All Games") by Wendell Hicken. This is a personal project for two players (hot-seat), not for distribution. The implementation should run in a modern web browser using HTML5 Canvas, JavaScript, and CSS — no frameworks required, single self-contained HTML file.

---

## Core Game Concept

Scorched Earth is a **turn-based 2D artillery game**. Tanks are placed across a procedurally generated, destructible landscape. On each turn, the active player sets an angle (0–90°) and power (0–1000), then fires. Wind, gravity, and terrain all affect the projectile's path. The goal is to destroy all enemy tanks. Between rounds, players spend earned money on weapons and accessories.

---

## Visual Style & Aesthetic

- **256-color VGA aesthetic** — bold, saturated, slightly chunky pixel-art style
- **Terrain**: Solid-fill irregular hills and mountains. Brown/tan earth tones with a flat top. The ground is a filled polygon, not just a line.
- **Sky**: Deep blue gradient to black (default "PLAIN" sky). Optional: starfield, stormy sky with lightning, sunset gradient, cavern (black ceiling, stalactites)
- **Color-coded players**: Each tank is a distinct solid color (red, green, blue, yellow, cyan, magenta, orange, white, etc.). The color bleeds into the HUD elements for that player's turn
- **Explosions**: Expanding bright circles (yellow → orange → red) that carve circular holes in the terrain
- **Napalm**: Bright orange/red dripping pools that spread across ground
- **UI**: Top horizontal control bar with chunky DOS-era text. Monospace font. No anti-aliasing — keep it crisp and blocky.
- **Tank sprites**: Simple pixel-art tanks (~20×12px): rectangular body, small turret barrel that rotates based on angle. Each rendered in the player's color.
- **Wind indicator**: Upper-right corner, arrow (← or →) with speed number in light blue
- **Projectile**: A small bright dot (2–4px) trailing through the sky
- **Death animation**: Tank flashes, then a small explosion, then disappears

---

## Game Structure & Flow

### 1. Main Menu Screen
- Title: "SCORCHED EARTH" in large bold retro text
- Subtitle: "The Mother of All Games"
- Options (keyboard + click):
  - **Players**: 2–4 (for this build; original supports 10)
  - **Rounds**: 1–20 (default 5)
  - **Starting Cash**: $0 / $5,000 / $10,000
  - **Wind**: None / Low / Medium / High / Random
  - **Terrain**: Random / Hilly / Mountainous / Flat
  - **[START GAME]** button

### 2. Player Setup (per player, once per game start)
- Each player's screen tints their color
- Enter name (text input, defaults to "Player 1", "Player 2", etc.)
- Choose tank icon (3–4 pixel art options)
- Choose: Human or CPU (difficulty: Moron / Tosser / Spoiler)
- Press Done / Enter

### 3. Buying Phase (between rounds, for players with cash)
- Show player's current cash
- Scrollable list of weapons and accessories with prices
- Click to buy; cash decreases, inventory count increases
- Show current inventory counts
- "DONE" button to finish

### 4. Battle Phase (main gameplay loop)
- Procedurally generate terrain (new each round)
- Place tanks spread across terrain, sitting on the surface
- Randomly choose first player; proceed left to right thereafter
- Per turn:
  - Highlight active player (flash their tank)
  - Show HUD: Power / Angle / PlayerName / CurrentWeapon / Inventory count
  - Player adjusts angle and power
  - Player fires
  - Animate projectile with physics
  - Apply explosion / effect to terrain and tanks
  - Check win condition (one tank standing)
  - Next player's turn
- Round ends when one tank remains (or all eliminated)
- Award money; show round summary

### 5. Round Summary / Score Screen
- Show each player: kills, damage dealt, money earned, total cash
- Continue to buying phase for next round, or show final winner

---

## Physics Engine

### Projectile Motion
```
x += vx * dt
y += vy * dt
vy += gravity * dt        // gravity = ~0.2 pixels/frame²
vx += wind * 0.01 * dt    // wind effect on x velocity
```

- **Gravity**: 0.2 px/frame² (adjustable; original range 0.05–10)
- **Wind**: Random value each round (-200 to +200 units). Wind pushes projectiles left or right by a small per-frame velocity nudge.
- **Power → initial velocity**: `speed = power * 0.15`; `vx = cos(angle_rad) * speed * direction`; `vy = -sin(angle_rad) * speed` (negative because y increases downward)
- **Angle**: 0° = horizontal, 90° = straight up. Direction based on which way the turret points.
- **Collision**: Each frame, check if projectile pixel is inside terrain or off-screen bounds

### Destructible Terrain
- Terrain stored as a **height map** (array of Y values, one per X pixel column)
- Explosion carves a **circular hole**: for each x in blast radius, compute how deep to carve, lower the terrain heightmap (raise the Y cutoff, meaning less terrain)
- Tanks fall if terrain beneath them is destroyed (with parachute protection if owned)
- Fall damage: proportional to fall distance (>20px = damage)

### Tanks
- Each tank has: `health` (0–100), `x`, `y` (position on terrain), `inventory`, `cash`, `angle`, `power`
- Damage from explosion = `max(0, blast_radius - distance_from_center) * damage_multiplier`
- Shields absorb damage first (if active)
- Tank dies at health ≤ 0 → death animation → removed from field

---

## Weapons Arsenal

Implement all of these with authentic behavior:

### Standard Weapons (always available)
| Weapon | Cost | Blast Radius | Notes |
|--------|------|-------------|-------|
| Baby Missile | Free (unlimited) | Small ~30px | Basic projectile |
| Missile | $1,875/5 | Medium ~50px | Enhanced |
| Baby Nuke | $10,000/3 | Large ~100px | Nuclear flash effect |
| Nuke | $12,000/1 | Very Large ~180px | Mass destruction |
| MIRV | $10,000/3 | Medium ×5 | Splits at apogee into 5 missiles |
| Death's Head | $20,000/1 | Medium ×9 | 9 warheads, most destructive |
| Leapfrog | $10,000/2 | Small ×3 | Three sequential warheads |
| Funky Bomb | $7,000/2 | ~200px scattered | Multi-color chain reaction, random scatter |

### Napalm
| Weapon | Notes |
|--------|-------|
| Napalm | $10,000/10 | Spreads horizontally on impact, damages area |
| Hot Napalm | $20,000/2 | More powerful napalm |

Napalm behavior: on impact, spawn ~15 "napalm particles" that drip down the terrain and ignite. Each burning pixel damages tanks within range.

### Earth Weapons
| Weapon | Effect |
|--------|--------|
| Dirt Clod | $5,000/10 | Dumps a sphere of dirt (raises terrain) |
| Dirt Ball | $5,000/5 | Larger dirt sphere |
| Ton of Dirt | $6,750/2 | Very large dirt dump |
| Liquid Dirt | $5,000/10 | Flows into gaps, fills holes |
| Riot Charge | $2,000/10 | Destroys dirt in a wedge from current tank |
| Riot Bomb | $5,000/5 | Projectile that destroys dirt sphere |
| Baby Digger | $3,000/10 | Tunnels into ground before detonating |
| Digger | $2,500/5 | Larger tunneler |
| Rollers | $5,000-$6,750 | Roll downhill until hitting valley or tank |

### Rollers
- On impact with terrain, switch to "rolling mode"
- Each frame, check terrain slope; move toward downhill direction
- Explode on hitting another tank or reaching a valley floor

### Energy Weapons
| Weapon | Notes |
|--------|-------|
| Plasma Blast | $9,000/5 | Radial energy burst from tank; powered by batteries |
| Laser | $5,000/5 | Straight-line beam, cuts through terrain |

---

## Accessories / Defense Systems

### Guidance Systems (apply before firing)
| Item | Cost | Effect |
|------|------|--------|
| Heat Guidance | $10,000/6 | Homes in on nearest enemy when in range |
| Ballistic Guidance | $10,000/2 | Auto-calculates angle/power to hit selected target |
| Lazy Boy | $20,000/2 | Fire-and-forget; click target, it homes perfectly |

### Defense
| Item | Cost | Effect |
|------|------|--------|
| Parachute | $10,000/8 | Prevents fall damage if deployed |
| Battery | $5,000/10 | Restores 10 HP; also powers energy weapons |
| Shield | $20,000/3 | Absorbs incoming blast damage (visual bubble around tank) |
| Force Shield | $25,000/3 | Deflects projectiles |
| Heavy Shield | $30,000/2 | Stronger shield, immune to shield-failure |
| Mag Deflector | $10,000/2 | Pushes projectiles upward if close |

### Miscellaneous
| Item | Notes |
|------|-------|
| Fuel Tank | $10,000/10 | Allows tank to move left/right (10 px per unit) |
| Contact Trigger | $1,000/25 | Weapon explodes on surface contact (no tunneling) |
| Smoke Tracer | $500/10 | Leaves colored smoke trail for targeting |

---

## Economy System

- **Base earnings per round**:
  - Kill: $5,000
  - Damage dealt: $10 per HP damage
  - Survival bonus: $3,000
  - Last tank standing: +$5,000
- **Interest**: 5% on cash held between rounds (default)
- **Starting cash**: Configurable (default $0, first round free missiles only)
- **Buying UI**: Show items player can afford. Items disappear from list when too expensive.
- Players always have unlimited Baby Missiles (free).

---

## AI Players

Implement three CPU difficulty levels:

### Moron
- Picks a random angle (20°–70°) and random power (200–600)
- Fires without any calculation

### Tosser
- First shot: random like Moron
- Subsequent shots: adjust angle/power toward previous landing spot
- Converges on target over 2–4 shots

### Spoiler
- Calculates exact ballistic trajectory accounting for wind and gravity
- Solves: given target (tx, ty), find (angle, power) such that projectile lands on target
- Use iterative solver or closed-form ballistic equation
- Misses occasionally (~10% random error) so it's beatable

---

## Controls

### HUD Controls (keyboard)
| Key | Action |
|-----|--------|
| ↑ / ↓ | Increase / decrease power (hold for continuous) |
| Page Up/Down | Rapid power change |
| ← / → | Rotate turret CCW / CW |
| Shift + arrows | Fine adjustment (slow) |
| Tab | Cycle through owned weapons |
| Space / Enter | Fire! |
| T | Open Tank Control Panel (shields, parachutes, guidance) |
| F | Use fuel to move tank (if owned) |
| B | Use battery (restore 10 HP) |

### Mouse Controls
- Click on "Power" label: right-click increases, left-click decreases
- Click on "Angle" label: left-click decreases, right-click increases
- Click on weapon name to cycle weapons
- Click both mouse buttons simultaneously to fire
- Click tank to see its status

### Tank Control Panel (popup)
- Toggle parachute active/passive
- Select and engage shield
- Select guidance system
- Use battery
- Move with fuel

---

## HUD Layout

```
┌──────────────────────────────────────────────────────────────────────┐
│ Power: 345   Angle: 52°    PLAYER_NAME          🪖 Baby Missile  ×99 │
│ Max: 87/100  [🔋×3] [🪂×2] [🛡 45%] [🎯 None]        Wind: → 47   │
└──────────────────────────────────────────────────────────────────────┘
```

- Top bar background tinted with active player's color
- Wind arrow and speed always visible in upper right
- Second status bar (optional): batteries, parachute state, shield %, guidance

---

## Terrain Generation

Use a **midpoint displacement / diamond-square algorithm** for terrain height map:

1. Start with flat line
2. Recursively subdivide, adding random vertical displacement (scaled by roughness)
3. Smooth the result (averaging neighboring values)
4. Parameters: bumpiness (0–100), slope (0–100), flatten peaks (optional)
5. Terrain types:
   - **Flat**: Very low bumpiness (5–10)
   - **Hilly**: Medium bumpiness (20–40)
   - **Mountainous**: High bumpiness (60–80), high slope
   - **Random**: Random parameters each round

Fill terrain as a filled polygon from the heightmap down to the canvas bottom. Color: brownish earth (`#8B5E3C` / `#6B4226`), with a thin top edge in a slightly lighter green-brown.

---

## Sky / Background Options

| Sky Type | Description |
|----------|-------------|
| Plain | Solid dark blue gradient |
| Stars | Black bg, scattered white dots |
| Stormy | Dark gray, occasional lightning bolt flashes |
| Sunset | Orange/red/purple gradient |
| Cavern | Black bg, stalactites hanging from top, no ceiling projectile escape |

Stormy sky: every ~8 seconds, flash a white jagged lightning bolt somewhere on screen. If "Hostile Environment" is on, lightning bolts occasionally strike and damage tanks.

---

## Sound Design

Original Scorched Earth used PC speaker beeps. For the browser version:

Use the **Web Audio API** to synthesize retro-style sounds (no external audio files needed):

```javascript
// Example: explosion sound
function playExplosion(radius) {
  const ctx = new AudioContext();
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  osc.connect(gain);
  gain.connect(ctx.destination);
  osc.frequency.setValueAtTime(200, ctx.currentTime);
  osc.frequency.exponentialRampToValueAtTime(30, ctx.currentTime + 0.5);
  gain.gain.setValueAtTime(0.5, ctx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.5);
  osc.start();
  osc.stop(ctx.currentTime + 0.5);
}
```

| Event | Sound |
|-------|-------|
| Fire | Short rising tone (pew!) |
| Missile in flight | Pitch varies with height (POS mode) |
| Explosion (small) | Short low thud |
| Explosion (nuke) | Deep rumble, longer |
| Tank death | Descending tone + static burst |
| Round win | Short ascending fanfare |
| Buy item | Soft click/chime |
| Shield hit | High ping |
| Napalm | Crackling noise (rapid random tones) |

---

## Talking Tanks Feature

When a computer player fires, display a comic-book speech bubble above their tank with a random quip:

**Attack quips** (pick randomly):
- "Eat this!"
- "Fire in the hole!"
- "You can't hide forever..."
- "Oops. Meant to do that."
- "Calculating optimal destruction..."
- "I've got a nuke with your name on it."
- "INCOMING!"
- "This one's for you, pal."

**Death quips** (when a tank dies):
- "Aargh!"
- "I'll be back!"
- "You got lucky..."
- "Direct hit... on me."
- "This is not how I wanted to go."

Display as a rounded rect popup near the tank, visible for 2 seconds.

---

## Screen Sizes / Rendering

- **Canvas size**: 1024×600px (scales down on smaller screens)
- **Terrain**: occupies roughly bottom 60% of canvas; sky fills top 40%
- **HUD**: Fixed 60px bar at top; gameplay canvas below that
- Render at native resolution; use `canvas.style` for CSS scaling to fit viewport

---

## Technical Architecture

Single HTML file with embedded CSS and JavaScript. Structure:

```
index.html
└── <canvas id="gameCanvas">
└── <script>
    ├── Constants & Config
    ├── GameState object
    ├── TerrainGenerator
    ├── PhysicsEngine (projectile update, collision)
    ├── WeaponEffects (explosion carve, napalm spread, etc.)
    ├── Tank class
    ├── Projectile class
    ├── AI class (Moron, Tosser, Spoiler)
    ├── UIRenderer (drawHUD, drawTerrain, drawTanks, drawExplosions)
    ├── SoundEngine (Web Audio API)
    ├── InputHandler (keyboard, mouse)
    ├── GameLoop (requestAnimationFrame)
    └── ScreenManager (menu, setup, buying, battle, summary)
```

Use a **state machine** for screens:
```
MAIN_MENU → PLAYER_SETUP → BUYING → BATTLE → ROUND_SUMMARY → BUYING → ... → GAME_OVER
```

Game loop runs at 60fps. Projectile physics run at fixed ~120Hz substeps for accuracy.

---

## Minimum Viable Feature Set (Build in this order)

1. ✅ Terrain generation and rendering
2. ✅ Two tanks placed on terrain
3. ✅ Angle/power controls + fire
4. ✅ Parabolic projectile physics with wind and gravity
5. ✅ Explosion: terrain deformation + tank damage
6. ✅ Turn alternation + win detection
7. ✅ HUD (power, angle, wind, player name, weapon)
8. ✅ Baby Missile unlimited + Missile + Baby Nuke + Nuke weapons
9. ✅ Money/economy between rounds
10. ✅ Buying screen with at least 8 weapons
11. ✅ MIRV (splits at apogee)
12. ✅ Shields (absorb damage)
13. ✅ Napalm spreading
14. ✅ Rollers (roll downhill)
15. ✅ AI opponents (Moron + Spoiler)
16. ✅ Sound effects via Web Audio API
17. ✅ Parachutes (prevent fall damage)
18. ✅ Dirt weapons (raise terrain)
19. ✅ Talking tanks
20. ✅ Main menu with configurable options
21. ✅ Round summary screen
22. ✅ Save scores / final winner screen

---

## Polish Details

- When a nuke explodes, **flash the entire screen white** briefly
- Show a **mushroom cloud** particle effect for nukes (rising circle of particles)
- Tanks **slide** down steep slopes gracefully
- When terrain is destroyed under a tank, the tank **drops** with visible falling animation
- Projectile leaves a **faint dotted trail** (optional, toggled by "Trace Paths")
- At round start, tanks **drop from the sky** onto the terrain with a small bounce
- Between rounds, show a **brief scoreboard** with names, kills, cash
- Player name colors match their tank color throughout all UI text
- Wind arrow **animates** (gentle oscillation) to feel alive
- Explosions have **afterglow** — the carve circle briefly glows bright before fading

---

## Stretch Goals (if time allows)

- Death's Head weapon (9-MIRV warhead)
- Ballistic Guidance system (auto-aim)
- Lazy Boy guidance (fire-and-forget homing)
- Fuel + tank movement
- Teams mode (Standard / Corporate / Vicious)
- Multiple sky types selectable in menu
- Save game state to localStorage
- Adjustable gravity and air viscosity
- Contact triggers (explode on surface contact)
- Simultaneous fire mode

---

## Example Weapon Prices Reference

```javascript
const WEAPONS = [
  { name: "Baby Missile",  cost: 0,      bundle: 99, radius: 30,  unlimited: true },
  { name: "Missile",       cost: 1875,   bundle: 5,  radius: 50  },
  { name: "Baby Nuke",     cost: 10000,  bundle: 3,  radius: 100 },
  { name: "Nuke",          cost: 12000,  bundle: 1,  radius: 180 },
  { name: "MIRV",          cost: 10000,  bundle: 3,  radius: 50, warheads: 5 },
  { name: "Death's Head",  cost: 20000,  bundle: 1,  radius: 50, warheads: 9 },
  { name: "Leapfrog",      cost: 10000,  bundle: 2,  warheads: 3 },
  { name: "Funky Bomb",    cost: 7000,   bundle: 2,  radius: 200, scatter: true },
  { name: "Napalm",        cost: 10000,  bundle: 10, type: "napalm" },
  { name: "Hot Napalm",    cost: 20000,  bundle: 2,  type: "hot_napalm" },
  { name: "Roller",        cost: 6000,   bundle: 5,  radius: 50, type: "roller" },
  { name: "Heavy Roller",  cost: 6750,   bundle: 2,  radius: 90, type: "roller" },
  { name: "Riot Charge",   cost: 2000,   bundle: 10, type: "riot_charge" },
  { name: "Dirt Ball",     cost: 5000,   bundle: 5,  radius: 70, type: "dirt" },
  { name: "Ton of Dirt",   cost: 6750,   bundle: 2,  radius: 140, type: "dirt" },
  { name: "Digger",        cost: 2500,   bundle: 5,  type: "digger" },
  { name: "Laser",         cost: 5000,   bundle: 5,  type: "laser" },
  { name: "Plasma Blast",  cost: 9000,   bundle: 5,  type: "plasma" },
];

const ACCESSORIES = [
  { name: "Parachute",        cost: 10000, bundle: 8  },
  { name: "Battery",          cost: 5000,  bundle: 10 },
  { name: "Shield",           cost: 20000, bundle: 3  },
  { name: "Force Shield",     cost: 25000, bundle: 3  },
  { name: "Heavy Shield",     cost: 30000, bundle: 2  },
  { name: "Mag Deflector",    cost: 10000, bundle: 2  },
  { name: "Heat Guidance",    cost: 10000, bundle: 6  },
  { name: "Ballistic Guide",  cost: 10000, bundle: 2  },
  { name: "Lazy Boy",         cost: 20000, bundle: 2  },
  { name: "Fuel Tank",        cost: 10000, bundle: 10 },
  { name: "Contact Trigger",  cost: 1000,  bundle: 25 },
  { name: "Smoke Tracer",     cost: 500,   bundle: 10 },
];
```

---

## Final Notes for Claude Code

- **Build this as a single `index.html` file** — no external dependencies, no build step. It should run by just opening the file in a browser.
- Prioritize **fun and playability** over pixel-perfect accuracy to the original.
- The game should be **immediately playable** by two humans sharing a keyboard (hot-seat).
- Use **requestAnimationFrame** for the game loop. Don't use `setInterval` for animation.
- Keep the **retro DOS aesthetic**: monospace font (Courier New or similar), chunky UI, no rounded corners on panels (sharp rectangles), color blocks instead of gradients for most UI elements.
- The terrain deformation must be **persistent within a round** — craters from earlier shots stay.
- Test the physics so a **Nuke at full power (~800) fired at 45° with no wind** travels approximately 600–800px horizontal distance. Tune gravity and speed accordingly.
- Make sure **at least 2-player hot-seat** works perfectly before adding AI.

Good luck, and may the biggest nuke win! 💥
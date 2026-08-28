# MazeQuest Mini — Product Requirements Document

**Version:** 1.0 (First Release)
**Owner:** Fredrik
**Status:** Approved for build
**Last updated:** 2026-08-28

---

## 1. Overview

MazeQuest Mini is a browser-based, top-down puzzle adventure. The player controls
**Pip**, a lantern-keeper who has to find a way out of a stone maze by solving a
short chain of dependent puzzles: pull a lever to raise a gate, take the key
behind the gate, unlock the door, and charge the exit portal with three crystals.

One level. One character. Five minutes to complete. Zero install — a single HTML
file that runs offline in any modern browser.

### Design pillars

| Pillar | What it means in practice |
|---|---|
| **Readable in 10 seconds** | A new player understands walls, danger, and goal without a tutorial. Colour and shape carry meaning, never text alone. |
| **Every puzzle unlocks the next** | No optional busywork. Lever → gate → key → door → portal is a single causal chain. |
| **Two input paths** | Fully playable with keyboard only, and fully playable with mouse/touch only. |
| **Cosy, not grim** | Storybook dungeon: warm sand floors, moss-green stone, a lantern glow. Cartoonish and colourful, never dark-and-edgy. |

### Non-goals for v1

Multiple levels, save/progression, enemies with AI, level editor, accounts,
leaderboards, backend of any kind.

---

## 2. Target user

Someone who clicks a link on a phone or laptop, has three to six minutes, and
wants a complete little experience with a beginning and an end. No prior gaming
skill assumed. No reading of instructions assumed.

---

## 3. Level design

### 3.1 Layout

A 21 × 15 tile grid (40 px tiles, 840 × 600 px playfield) built from four rooms
connected by corridors around a central hall.

```
#####################
#S....#########..#..#     S  Start (Pip spawns here)
#..*..#########...K.#     E  Exit portal
#.#.#.######.G..#...#     K  Key
#.....######.##.....#     D  Locked door
#.###.##.....########     W  Lever (clickable switch)
#X###....#.#.########     G  Gate (raised by lever)
#.######..*..########     X  Spike trap (timed)
#X######.#.#.XX.#####     *  Crystal (collectible)
#.######.....##D#####     #  Wall
#..#..##.######.....#     .  Floor
#...*.##.######..#..#
#.W.#....######..#..#
#.....#########....E#
#####################
```

**Zones**

| Zone | Grid area | Contains |
|---|---|---|
| Start chamber | x1–5, y1–4 | Spawn, Crystal 1 |
| Lantern corridor | x1, y4–10 | Two timed spike traps |
| Lever vault | x1–5, y10–13 | Lever, Crystal 2 |
| Central hall | x8–12, y5–9 | Crystal 3, junctions to all zones |
| Key chamber | x15–19, y1–4 | Key — sealed behind the gate |
| Portal room | x15–19, y10–13 | Exit portal — sealed behind the door |

**Reachability is a hard requirement and has been verified by BFS in three phases:**

1. Gate closed, door closed → lever and all 3 crystals reachable; key and exit **not** reachable.
2. Gate open, door closed → key reachable; exit **not** reachable.
3. Gate open, door open → exit reachable.

This guarantees the level cannot be sequence-broken and cannot be soft-locked.

### 3.2 Puzzles and obstacles (requirement: minimum 3)

| # | Puzzle | Interaction | Gates progress by |
|---|---|---|---|
| 1 | **Lever → gate** | Click the lever, or press `E` while standing next to it | Sealing the key chamber until the player explores the lever vault |
| 2 | **Key → door** | Walk into the door while carrying the key | Sealing the portal room |
| 3 | **Timed spike traps** | Four spike tiles retract and extend on a 2-second cycle | Forcing timing on the only routes to the lever vault and the door |
| 4 | **Charge the portal** | Collect all 3 crystals before stepping into the portal | Forcing full exploration of the map before the exit accepts you |

### 3.3 Collectibles

Three crystals, one per major zone, all reachable before any lock is opened.
They are not optional decoration — the portal will refuse to open without all
three, so they double as the exploration incentive and as the final gate.

---

## 4. Character design

**Pip, the lantern-keeper.** A rounded square body with two large eyes and a
swinging lantern. Cartoonish, one colour (amber-lit teal), no sprite sheets —
drawn procedurally on canvas so the file stays self-contained.

| Property | Value |
|---|---|
| Movement | Four-directional, grid-snapped, one tile per input |
| Animation | 110 ms ease tween between tiles; idle bob; eyes track direction |
| Health | 3 hearts |
| Damage | 1 heart per spike hit, then 1.2 s of invulnerability and a knock-back to the last safe tile |
| Death | At 0 hearts the run ends with a retry screen; the level resets, nothing is lost but time |

The lantern is also the visual signature: it lights a soft radius around Pip and
the rest of the maze sits under a gentle vignette. It never hides the layout
enough to be unfair — it sets mood, it is not a fog-of-war mechanic.

---

## 5. Mechanics

### 5.1 Movement and collision

- Input: `↑ ↓ ← →` or `W A S D`, on-screen D-pad, or clicking an adjacent tile.
- Collision is checked against the target tile before the tween starts.
- Blocking tiles: walls, closed gate, locked door.
- Held keys repeat movement so long corridors don't require key-mashing.

### 5.2 Interactive objects

| Object | Responds to | Behaviour |
|---|---|---|
| Lever | Click, or `E`/`Space` when adjacent | Toggles the gate open/closed. Clicking from far away shows "Get closer to the lever." |
| Gate | Lever state | Bars retract into the floor when open, and the tile becomes walkable |
| Door | Walking into it with the key | Consumes the key, opens permanently, plays an unlock cue |
| Crystal | Walking onto it | Collected, counter increments, pickup cue |
| Key | Walking onto it | Added to inventory, shown in the HUD |
| Spikes | Walking onto them while extended | −1 heart, knock-back |
| Portal | Walking into it | Wins the level if all 3 crystals are held, otherwise shows how many are missing |

Clicking any tile also inspects it and prints a one-line description — a cheap
way to make the whole board feel alive to mouse-only players.

### 5.3 UI

**Start menu:** title, animated Pip preview, "Start adventure" button, a
three-line "How to play", and a sound toggle.

**Level interface:** the canvas plus a HUD strip showing crystals collected,
key held, hearts remaining, elapsed time, and moves used. Buttons for Restart
and Menu. Toast messages appear at the top of the canvas for feedback.

**End screens:** a win card with time, moves, and hearts left, and a retry card
on death. Both offer "Play again" and "Menu".

Copy rules: sentence case, active voice, the interface tells the player what
just happened and what to do next. No apologising errors, no filler.

---

## 6. Visual direction

Storybook dungeon, not gothic dungeon.

| Token | Hex | Use |
|---|---|---|
| `--ink` | `#17261F` | Backdrop, text on light |
| `--stone` | `#2E4A3C` | Wall body |
| `--stone-lit` | `#3F6853` | Wall top face |
| `--sand` | `#F2E3C4` | Floor |
| `--amber` | `#F2A03D` | Lantern, key, highlights |
| `--coral` | `#E8564B` | Spikes, damage, danger |
| `--mint` | `#58C8A8` | Lever, open gate, success |
| `--plum` | `#8B6BD6` | Exit portal |

Type: **Fredoka** (rounded, chunky) for display and buttons; **Space Mono** for
HUD counters, because numbers read as instrumentation and should look different
from prose.

---

## 7. Technical requirements

- Single `index.html`. No build step, no dependencies, no network calls at runtime.
- HTML5 Canvas 2D for the playfield, DOM for menus and HUD.
- 60 fps target via `requestAnimationFrame`, delta-time based animation.
- Audio is generated with the Web Audio API — no asset files, and muted-by-default respected.
- Responsive: the canvas scales to viewport width, D-pad appears on touch and narrow screens.
- Accessibility floor: visible keyboard focus on every control, `prefers-reduced-motion` respected, colour never the sole carrier of meaning (icons and counters back it up).

---

## 8. Success criteria for the first release

The release is done when all of the following are true:

1. A new player can start, finish, and see a win screen without external instructions.
2. All four puzzles are solvable and the level is provably not soft-lockable.
3. The level is completable with keyboard only, and separately with mouse only.
4. Spike traps can be crossed by a player who waits and watches the cycle once.
5. The game loads and runs from a single file, opened directly from disk.
6. No console errors during a full playthrough.

---

## 9. Post-v1 backlog

Levels 2–5 with a level-select screen · pushable blocks · a patrolling enemy ·
an AI level generator (prompt → validated layout) · time-attack mode with local
best times · a level editor that exports the same layout string format.

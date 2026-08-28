# MazeQuest Mini

A top-down puzzle adventure that runs from a single HTML file. One maze, one
character, four chained puzzles, three crystals, three minutes.

**Play it:** open `index.html` in any browser — no build, no install, no server.

---

## What's in here

| File | What it is |
|---|---|
| `index.html` | The game. Self-contained: HTML, CSS, JS, procedural art, generated audio. |
| `PRD.md` | Product requirements document for the first release. |
| `mockups.html` | UI reference frames for the three screens, rendered from the real level data. |

---

## How to play

| Input | Does |
|---|---|
| `↑ ↓ ← →` or `W A S D` | Move Pip one tile |
| `E` or `Space` | Pull a lever you're standing next to |
| Click a neighbouring tile | Step there (the game is fully mouse-playable) |
| Click any other tile | Inspect it |
| On-screen D-pad | Appears on touch and narrow screens |

**The chain:** pull the lever to raise the gate → take the key behind the gate →
unlock the door → collect all three crystals → step into the portal.

Spike traps run on a two-second cycle. They flash coral just before they extend,
so watch one cycle before you cross. Three hearts. A hit knocks you back to the
last safe tile.

---

## Level format

The level is a 21 × 15 array of strings at the top of the script in `index.html`.
Edit it and the game changes — the mockup file reads the same format.

```
#  wall        .  floor      S  start
*  crystal     K  key        D  locked door
W  lever       G  gate       X  spike trap
E  exit portal
```

The layout is verified by breadth-first search in three phases: the lever and
all crystals are reachable from the start with everything locked, the key is
reachable only once the gate is raised, and the portal only once the door is
open. That guarantees the level can't be sequence-broken or soft-locked.

---

## Tech

HTML5 Canvas 2D for the playfield, DOM for menus and HUD, `requestAnimationFrame`
with delta timing, Web Audio API for sound (generated, no asset files).
No dependencies. No network calls at runtime — the only external request is the
Google Fonts stylesheet, and the game falls back to system fonts without it.

Respects `prefers-reduced-motion`, keeps visible keyboard focus on every control,
and never uses colour as the sole carrier of meaning.

---

## Publishing to GitHub Pages

```bash
git init
git add .
git commit -m "MazeQuest Mini — first release"
git branch -M main
git remote add origin https://github.com/<user>/mazequest-mini.git
git push -u origin main
```

Then in the repo: **Settings → Pages → Source: deploy from branch → `main` / root.**
The game will be live at `https://<user>.github.io/mazequest-mini/`.

---

## Backlog

Levels 2–5 with a select screen · pushable blocks · a patrolling enemy ·
prompt-to-level generation with the same BFS validation · time-attack mode ·
a level editor that exports this layout format.

# How this was built

A short, honest account of the workflow behind MazeQuest Mini — what order things
happened in, what got decided where, and which decisions were mine versus the
model's.

The whole thing was AI-assisted from the first line. What follows is not a
defence of that; it's a description of how the assistance was structured, because
the structure is the part worth copying.

---

## The short version

```
brief → PRD → design tokens → level layout → BFS validation → build → play-test
```

No step was skipped and no step ran twice. That's the point of writing this down.

---

## 1. PRD before pixels

The brief asked for a maze with "at least 3 puzzles or obstacles." That phrasing
invites three *unrelated* puzzles sitting in three corners of a map. I decided
early that the puzzles should form a single causal chain instead:

> lever → gate → key → door → portal

Each lock is the only route to the next one. You cannot do them out of order, and
there is no optional busywork. That constraint was written into `PRD.md` **before**
any code existed, which meant every later decision had something to be checked
against.

The PRD also fixed the non-goals — no levels 2–5, no enemies, no save state, no
backend. Deciding what v1 *isn't* was more useful than listing what it is,
because it's what stopped the scope from drifting during the build.

---

## 2. Design direction as an explicit step

The default output for "cartoonish maze game" is coloured squares on a dark
background. That's roughly where the reference example started, and it then took
three rounds of screenshots and fix-requests to climb out of.

So I made the visual direction its own step and locked it down as tokens before
the build started:

| | Decision |
|---|---|
| **Palette** | Moss-green stone, warm sand floors, amber lantern, coral danger, mint for anything mechanical and safe, plum portal |
| **Type** | Fredoka for display (rounded, storybook), Space Mono for HUD counters — numbers should read as instrumentation, not prose |
| **Signature** | The lantern. Pip carries it, it lights a radius, and the rest of the maze sits under a soft vignette |

The lantern is the one place I spent boldness. It's atmosphere, deliberately not
a fog-of-war mechanic — it never hides enough of the layout to be unfair. Chanel's
rule applied: everything around it stays quiet.

Cost of doing this up front: about ten minutes. Cost of not doing it: three
iteration rounds fixing muted colours and unreadable text, which is exactly what
the reference workflow spent its middle third on.

---

## 3. The level was validated before it was playable

This is the step I'd argue hardest for.

The maze is a 21 × 15 array of strings. Before writing a single line of game
logic, I ran a breadth-first search over that array in three phases:

| Phase | State | Must be true |
|---|---|---|
| 1 | Gate down, door locked | Lever and all 3 crystals reachable; key and portal **not** reachable |
| 2 | Gate raised, door locked | Key reachable; portal **not** reachable |
| 3 | Gate raised, door open | Portal reachable |

The first layout I drafted **failed**. The portal was reachable without ever
touching the door — a corridor I hadn't noticed ran around the back of the exit
room, and one crystal was sealed inside a wall with no entrance at all. Both were
invisible from reading the ASCII by eye.

Rather than nudge the string art until it looked right, I rewrote the generator to
carve rooms and corridors programmatically and re-ran the check. Second attempt
passed all seven assertions.

That validator still runs against the layout embedded in `index.html`, so the
level in the repo is provably not sequence-breakable and not soft-lockable. Finding
that class of bug by play-testing would have meant noticing an absence — that a
door I never needed was never needed — which is precisely the kind of thing human
testing is bad at.

---

## 4. Build, in one pass

With the PRD, the tokens, and a verified layout in hand, the build itself was
largely mechanical: canvas rendering, grid movement with an eased tween, timed
spike traps on a two-second cycle, procedural character art, Web Audio blips
instead of asset files.

Two requirements shaped the input handling more than expected:

- **"Basic clickable UI"** — I read this as the game needing to be completable
  with a mouse alone, not just having buttons on a menu. So clicking an adjacent
  tile steps there, clicking the lever pulls it, and clicking anything else
  inspects it. That last one was cheap to add and made the whole board feel alive.
- **Keyboard-only completability** — held keys auto-repeat, so long corridors
  aren't a key-mashing chore.

Both paths were tested end to end.

---

## 5. Play-test and ship

Full playthrough on the deployed site: fonts load, all four puzzles resolve, spike
timing is crossable by a player who watches one cycle, win screen fires with the
right numbers, no console errors.

Published to GitHub Pages from `main` / root.

---

## What went wrong along the way

Worth recording, since a process account that reports only successes isn't a
process account.

- **The first level layout was broken in two separate ways** (see §3). Caught by
  the validator, not by eye, and not by play-testing.
- **A placeholder in the README bit me.** The git instructions contained
  `https://github.com/<user>/...` and I pasted it literally. zsh read `<` as an
  input redirect and the remote silently never got added, so the push failed with
  a confusing "origin does not appear to be a git repository."
- **The Pages URL 404'd for a few minutes** after the source was saved. Nothing was
  wrong — the first build just hadn't finished. Easy to misread as a
  misconfiguration and start changing settings that were already correct.

---

## What I'd do differently next time

Write the BFS validator *first*, as a standalone tool, and design the level
against it interactively rather than drafting-then-checking. It caught real bugs,
but it caught them after I'd already invested in a layout I liked.

The next version of that idea is in the backlog: prompt-to-level generation where
the model proposes a layout and the validator accepts or rejects it in a loop. At
that point the AI is generating candidates and a deterministic checker is the
quality gate — which is a considerably more reliable arrangement than asking a
model to be careful.

---

## The general shape of it

The reference workflow for this kind of task is: generate something, look at it,
describe what's ugly, regenerate. That works, and it's fast. But every correction
round is spent recovering ground you'd already have if the constraint had been
stated up front.

What I did instead: **decide the constraints in a form that can be checked, then
let the model build against them.** A PRD is a checkable constraint. A token table
is a checkable constraint. A BFS assertion is a checkable constraint — and unlike
the other two, it checks itself.

The screenshot-and-fix loop is still there in the backlog for things that genuinely
need an eye. It's just not the primary mechanism.

# Break sweep

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 11 of 28 in `run_cycle`
**Runs:** every pass — the sweep runs unconditionally; a Break fires per faction at 0 health
**Source of truth:** `backend/engine/cycle/resolution.py:155-180` — if this card and that
file disagree, the file wins.

## What it is
Catches every faction sitting at 0 health that no attack knocked down, and Breaks it —
losing ground or losing its leader. A faction is never removed.

## Where it lives
`backend/engine/cycle/runner.py:149`             the call site
`backend/engine/cycle/end_of_cycle.py:135-150`   `run_break_sweep`, the sweep
`backend/engine/cycle/resolution.py:155-180`     `resolve_break`, the actual resolution

## Shape and why
The sweep exists because Breaks caused by an attack already fired inline inside position 7;
this only covers decay and event damage. `rng` is injectable for tests.
See: `Planning/specs/faction-behavior_spec.md`.

## Writes
- `faction.health` — reset to 75 (`resolution.py:176`)
- `faction.rating` — dropped to `(level-1).0` on the 75% branch
- `faction.leader` — replaced with a fresh `Leader` on the 25% branch
- appends a `Break` `ActionResult` to `all_results`

## Reads
- `faction.health` — as left by 5, 7, 9 and 13

## Hits
- **Faction** — the rating and leader it writes are read by 3 (cap Σ) and 7 next pass
- **12-deal-tick** — the `Break` result it appends enters the `acted_this_cycle` map
- **Anything that moves this step's position** — must stay below 9, or the pass's health
  decay lands after the sweep and a 0-health faction waits a full pass.

## Does not hit
- **10-leadership-events** — the tempting reach: both replace a leader. This one does it as
  a coin-flip branch of Break; 10 does it on a 3-pass absence timer. Neither reads the other.
- **22-terminal-checks** — a faction at 0 health never ends the run; only the Mayor can lose.

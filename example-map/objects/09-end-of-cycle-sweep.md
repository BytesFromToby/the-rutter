# End-of-cycle sweep

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 9 of 28 in `run_cycle`
**Runs:** every pass — unconditional
**Source of truth:** `backend/engine/cycle/end_of_cycle.py:17-62` — if this card and that
file disagree, the file wins.

## What it is
Wears every faction down by one health, evolves its personality from the pass just ended,
and wipes the scratch state.

## Where it lives
`backend/engine/cycle/runner.py:147`             the call site
`backend/engine/cycle/end_of_cycle.py:17-62`     `run_end_of_cycle`
`backend/engine/npc/behavior.py:492`             `evolve_traits` (out of the cut)

## Shape and why
Three sub-blocks in one function: flat health decay, trait evolution, then cycle-only reset.
Its own comment at `end_of_cycle.py:52-53` marks a fourth block as informational with no
code behind it. See: `Planning/specs/faction-behavior_spec.md`.

## Writes
- `faction.health` — `-1` every pass, floored at 0 (`end_of_cycle.py:31`)
- `faction.traits` — rewritten by `evolve_traits`
- `faction.unstable_stacks` — via `reset_cycle_state()`, `end_of_cycle.py:57`
- deletes `_grow_streak`, `_protect_streak`, `_hostile_drought`, `_was_harmed_by`,
  `_harm_landed_on` (`end_of_cycle.py:59-62`)

## Reads
- the five scratch attributes written at 8, each via `getattr` with a default
- `faction.health` as carried out of position 7

## Hits
- **11-break-sweep** — the `-1` decay is the non-combat path to 0 health that 11 catches
- **08-trait-scratch** — it consumes and destroys everything 8 wrote
- **Anything that moves this step's position** — must stay directly below 8 (the scratch is
  deleted here) and above 11 (which sweeps the health it just reduced).

## Does not hit
- **26-cycle-flag-reset** — the tempting reach: "4.6 Reset cycle-only state" here calls
  `reset_cycle_state()`, which clears only `unstable_stacks`. `toiling` and `withholding`
  are cycle-only too and are **not** cleared here — they survive to position 14.

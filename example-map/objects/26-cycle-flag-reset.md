# Cycle flag reset

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 26 of 28 in `run_cycle`
**Runs:** every pass — unconditional loop over `factions`
**Source of truth:** `backend/engine/cycle/runner.py:266-271` — if this card and that file
disagree, the file wins.

## What it is
Clears the two per-pass labour flags now that the needs step has spent them.

## Where it lives
`backend/engine/cycle/runner.py:266-271`   the loop and the spec citation in its comment
`backend/engine/models.py:74-75`           the two fields, both marked "never persisted"

## Shape and why
It sits at the bottom of the pass rather than in `end_of_cycle.py` because the needs step at
position 14 runs *after* `run_end_of_cycle`; the blueprint records the correction.
See: `Planning/specs/cycle-runner_spec.md` (Cycle-Only State) and
`Planning/blueprints/public-needs_BP.md`.

## Writes
- `faction.toiling` — `False`, for every faction
- `faction.withholding` — `False`, for every faction

## Reads
- nothing; it assigns unconditionally

## Hits
- **14-public-needs** — next pass this position is the reason the chain sees only flags set
  in that pass
- **07-faction-action-loop** and **13-active-event-effects** — the two writers whose work
  this erases
- **Anything that moves this step's position** — it must stay below 14. Above it, Toil and
  Withhold never reach the food chain and the failure is silent: no error, no output change
  the reader can trace back.

## Does not hit
- **09-end-of-cycle-sweep** — the tempting reach: that position's block is literally headed
  "4.6 Reset cycle-only state" and calls `reset_cycle_state()`. That method clears
  `unstable_stacks` only (`models.py:110-111`). These two flags are cycle-only as well and
  are not touched there. Two separate resets, seventeen positions apart.

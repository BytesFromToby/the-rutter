# Trait scratch

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 8 of 28 in `run_cycle`
**Runs:** every pass — unconditional
**Source of truth:** `backend/engine/cycle/runner.py:288-326` — if this card and that file
disagree, the file wins.

## What it is
Tags each faction with private counters describing how its pass went, so the next position
can evolve its personality from them.

## Where it lives
`backend/engine/cycle/runner.py:144`        the call site
`backend/engine/cycle/runner.py:288-326`    `_track_cycle_outcomes`

## Shape and why
The counters are set as undeclared attributes on the dataclass rather than model fields —
they exist only between positions 8 and 9 and are deleted there.
See: `Planning/specs/faction-behavior_spec.md` (trait evolution).

## Writes
- `faction._harm_landed_on`, `._was_harmed_by` — from `Harm` results this pass
- `faction._grow_streak`, `._protect_streak` — streak counters, mutually reset
- `faction._hostile_drought` — passes since the last hostile action

## Reads
- `all_results` — everything appended at positions 4 through 7
- `r.action`, `r.outcome`, `r.actor_id`, `r.target_id` on each `ActionResult`

## Hits
- **09-end-of-cycle-sweep** — the only reader. `end_of_cycle.py:38-42` pulls all five with
  `getattr(..., default)` and `end_of_cycle.py:59-62` deletes them.
- **Anything that moves this step's position** — it must stay above 9 and below 7. Moved
  below 9 the counters are absent, and the `getattr` defaults make the loss silent.

## Does not hit
- **`Faction.traits`** — the tempting reach: this step is named for traits and touches none.
  The `FactionTrait` list is rewritten at position 9 by `evolve_traits`
  (`engine/npc/behavior.py:492`), reading these counters as input.

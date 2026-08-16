# Event deck roll

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 17 of 28 in `run_cycle`
**Runs:** conditionally — `event_deck` non-empty and `active_events` not `None`
**Source of truth:** `backend/engine/events/event_system.py` — if this card and that file
disagree, the file wins.

## What it is
Draws new random events against each domain's chaos, fires whichever scripted events now
meet their conditions, and announces the heralded ones.

## Where it lives
`backend/engine/cycle/runner.py:193-232`             the call site, dedup and beat emission
`backend/engine/events/event_system.py:40-140`       `roll_for_random_events`, trigger gates
`backend/engine/events/event_system.py`              `check_scripted_events`

## Shape and why
Scripted events are deduplicated by id against the currently active list at
`runner.py:209-210`, so a multi-pass or level-gated template cannot stack copies of itself.
See: `Planning/specs/events_spec.md` and `Planning/specs/oracle_spec.md`.

## Writes
- `active_events` — extended twice, at `runner.py:195` and `runner.py:211`
- appends `OracleHerald` and `ScriptedEvent` results to `all_results`

## Reads
- `world.chaos` — decayed at 16, one position earlier
- `public.fed`, `.unrest`, `.support` — band gates at `event_system.py:105,119,135`, written
  at 14 this pass
- `treasury` — passed to `check_scripted_events` for its trigger conditions

## Hits
- **13-active-event-effects** — everything added here first takes effect on the **next** pass
- **22-terminal-checks** — `active_events` is handed to `process_ostracism` at `runner.py:261`
- **Anything that moves this step's position** — it must stay below 14, or the need-gated
  templates test last pass's bands, and below 16, or it sees undecayed chaos.

## Does not hit
- **16-world-chaos** — the tempting reach: the position immediately above, also chaos-driven,
  also random. That one only narrates; this one is the only place a `GameEvent` is born.
- **`CycleEvent`** — a different object entirely. See the collisions register.

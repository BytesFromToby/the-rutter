# World chaos

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 16 of 28 in `run_cycle`
**Runs:** every pass — unconditional; the per-domain body needs `chaos > 0`
**Source of truth:** `backend/engine/events/world.py:14-64` — if this card and that file
disagree, the file wins.

## What it is
Rolls each unsettled domain to see whether its disorder erupts into an incident this pass,
and bleeds the level down when it does not.

## Where it lives
`backend/engine/cycle/runner.py:183-187`   the call site and the dramatic-log fan-out
`backend/engine/events/world.py:14-64`     `process_world_chaos`

## Shape and why
A chance table keyed on the integer chaos level; a miss, or a domain with no factions in
it, decays the level by 2. See: `Planning/specs/events_spec.md`.

## Writes
- `world.chaos[domain_id]` — decayed by 2 on a miss (`world.py:39,45`) or at level 10
  (`world.py:62`)
- appends `ChaosEvent` results to `all_results`

## Reads
- `world.chaos` — raised at 13 by event effects, **this pass**
- `faction.domain_primary` — to pick a faction in the domain to name

## Hits
- **17-event-deck-roll** — reads the same `world.chaos` values one position later, after
  this decay has already applied (`event_system.py:46-47`)
- **13-active-event-effects** — a `chaos` effect there feeds this roll next pass
- **Anything that moves this step's position** — moving it below 17 changes the chaos values
  the deck roll sees, because the decay would not have happened yet.

## Does not hit
- **17-event-deck-roll** as the source of these incidents — the tempting reach: both are
  "random events driven by chaos". This position emits a narrative `ActionResult` only. It
  never instantiates a `GameEvent`, never touches `active_events`, and never reads the deck.

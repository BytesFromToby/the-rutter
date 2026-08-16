# Active event effects

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 13 of 28 in `run_cycle`
**Runs:** conditionally — only when `active_events` is non-empty
**Source of truth:** `backend/engine/events/event_system.py` — if this card and that file
disagree, the file wins.

## What it is
Applies the ongoing effect of every event already running in the city, then drops the ones
that have finished.

## Where it lives
`backend/engine/cycle/runner.py:158-162`             the call site and the prune
`backend/engine/events/event_system.py:355-380`      `_apply_effect`
`backend/engine/models.py:334-368`                   `EventEffect`, `CascadeSpec`, `GameEvent`

## Shape and why
Deliberately placed **above** position 14 so a `withhold` event can zero its target's chain
contribution in the same pass, while new events are still rolled below 14 so their band
gates see this pass's numbers. See: `Planning/specs/events_spec.md`.

## Writes
- `faction.health`, `faction.rating` — `event_system.py:362,364`
- `faction.withholding` — `event_system.py:369`, cycle-only
- `world.chaos[domain_id]` — `event_system.py:380`
- `domain.drift` — `event_system.py:374`; nothing reads it (see the ghost card)
- `event.status` → `resolved`, then removed from the list at `runner.py:162`

## Reads
- `active_events` — extended at 17 **last pass**
- `event.effects`, `.cycles_remaining`, `.status`

## Hits
- **14-public-needs** — `withholding` set here zeroes that faction's chain output this pass
- **16-world-chaos** and **17-event-deck-roll** — both read `world.chaos`
- **Anything that moves this step's position** — below 14 the withhold effect misses its own
  pass; above 7 it would pre-empt the actions the flags describe.

## Does not hit
- **17-event-deck-roll** — the tempting reach: same module, same `GameEvent` list, four
  positions apart. 17 *creates* events from the deck; this one only runs and retires them.
  An event created at 17 first takes effect on the next pass.

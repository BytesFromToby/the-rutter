# Active events

**Status:** live · Confirmed
**Type:** object
**Written by:** 13, 17 — in tick order
**Source of truth:** `backend/engine/events/event_system.py` and
`backend/engine/models.py:350-368` — if this card and those files disagree, the files win.

## What it is
The list of events currently running in the city. It is a caller-owned list mutated in
place, not a field on any model.

## Where it lives
`backend/engine/models.py:350-368`     the `GameEvent` dataclass
`backend/engine/events/event_system.py` the deck, the trigger gates, the effect application
`backend/engine/cycle/runner.py:162,195,211` the three places the list itself is rewritten

## Shape and why
Status is a small machine — `pending`, `heralded`, `active`, `cascading`, `resolved` — and
heralded events announce themselves a set number of passes before they bite.
See: `Planning/specs/events_spec.md` and `Planning/specs/oracle_spec.md`.

## Fields that matter
- the list itself — pruned at 13 (`runner.py:162`), extended twice at 17
- `status` — set to `resolved` at 13, which is what the prune filters on
- `effects` — read at 13 to mutate factions, domains and chaos; read at 23 to classify an
  event as a disaster for the pressure clock (`ostracism.py:223-232`)
- `id` — the dedup key at `runner.py:209-210`, so a scripted event cannot stack copies
- `herald_text`, `herald_delay_remaining` — read at 17 to emit the omen beat
- `cycles_remaining`, `cascade` — carried across passes

## Hits
- **13-active-event-effects** — everything added at 17 first takes effect on the next pass
- **22-terminal-checks** — the list is handed to `process_ostracism` at `runner.py:261`
- **Anything that moves 13 or 17** — 13 must stay above 14 for `withhold` to land in its own
  pass, and 17 below 14 so band gates see this pass's numbers. The runner comments say so.

## Does not hit
- **`CycleEvent`** — the tempting reach, and the territory's oldest name collision. A
  `CycleEvent` (`models.py:154`) is a result record built from an `ActionResult` at position
  27. It has no status, no effects, no duration, and never enters this list. Changing "how
  events work" means one of these two, never both.

# Domain.drift

**Status:** ghost · Confirmed
**Type:** ghost

## The reach
You are authoring an event that should tilt a whole sector of the city. `EventEffect.field`
is documented as `"rating" | "health" | "action_weight" | "chaos"` (`models.py:336`), but the
effect applier has a real branch for a fifth: `event_system.py:373-374` accumulates
`domain.drift`, clamped to −1.0…1.0. `Domain` carries the field (`models.py:121`), the
serializer ships it to the client (`serializer.py:138`), the loader restores it
(`loaders.py:43`). You write an effect with `field: "drift"` and expect the domain to tilt.

## What is actually there
The field, the write, and a round-trip. No engine code reads `domain.drift`. The domain
values that actually govern anything are `cap` and `utilization`, both re-derived from
scratch at position 3 every pass, which never consult `drift`.

**This one is genuinely ambiguous and is recorded as a ghost deliberately.** Read the field
and you get a real accumulated number, correctly clamped and correctly serialized — which
argues for leftover. Write it, through the effect field that exists to write it, and nothing
in the city changes and nothing errors — which argues for ghost. The reach that a reader
arrives with here is the authoring one, and it fails silently, so it is filed as a ghost.

## Evidence
Search scope: `**/*.py`, `*.vue`, `*.js` for `drift` and `domain.drift`.
Writers: `engine/events/event_system.py:374`. Readers: `backend/serializer.py:138`,
`backend/loaders.py:43` — persistence only. No hit in `engine/formulas.py`,
`engine/actions/`, `engine/npc/`, or `engine/cycle/`. The frontend hits are unrelated CSS
comments (`ProcessionBand.vue:172`, `GameView.vue:496`).

## What to do instead
`world.chaos[domain_id]` is the live per-domain pressure value: written by the same effect
applier (`event_system.py:380`), read at positions 16 and 17. See
[world-state](world-state.md). Do not confuse this field with `engine/needs/drift.py`, which
is live and unrelated — see the collisions register.

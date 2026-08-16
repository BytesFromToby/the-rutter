# The Public

**Status:** live · Confirmed
**Type:** object
**Written by:** 14, 21, 22 — in tick order
**Source of truth:** `backend/engine/models.py:374-395` — if this card and that file
disagree, the file wins.

## What it is
The people of the city as a single record: how well they eat, how they feel, how many of
them there are.

## Where it lives
`backend/engine/models.py:374-395`     the `ThePublic` dataclass
`backend/engine/needs/`                bands, chain, drift, scales — the seven scales
`backend/engine/special/public.py`     the support mirror and disposition

## Shape and why
Every scale is 0–100 and moves at most one drift step per pass toward a computed target, so
the record is a lagging indicator by design.
See: `Planning/specs/public-needs_spec.md`.

## Fields that matter
- `fed`, `happy`, `piety`, `unrest`, `consumption` — written only at 14; read at 17 (event
  band gates), 23 (ostracism pressure) and at 7 next pass by NPC behaviour
- `health`, `population` — written at 14; `population` read at 22 for collapse
- `drunk` — derived at 14 from the consumption band; read at `needs/scales.py:84`
- `support` — written at 14 (conditional branch), **overwritten** at 21, adjusted at 22
- `disposition`, `traits` — written only at 21
- `pop_warning` — written only at 22; latched across passes

## Hits
- **22-terminal-checks** — `population`, `fed`, `unrest` and `support` all feed the end-state
- **17-event-deck-roll** — need-gated templates cannot fire without this record
- **Anything that moves position 14 or 21** — the two must stay in that order, or the
  support mirror lands before the drift that adjusts it.

## Does not hit
- **`mayor.reputation["the_public"]`** — the tempting reach: the same quantity, and the one
  that actually persists. `public.support` is downstream of it every pass. If you change how
  the people regard the Mayor, change the reputation entry; changing `support` between
  positions 14 and 21 is erased without an error.

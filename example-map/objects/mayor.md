# Mayor

**Status:** live · Confirmed
**Type:** object
**Written by:** 4, 6, 12, 19, 23, 24 — in tick order
**Source of truth:** `backend/engine/models.py:486-567` — if this card and that file
disagree, the file wins.

## What it is
The player's standing in the city: what he can spend, who owes him, who tolerates him, and
how close he is to losing.

## Where it lives
`backend/engine/models.py:486-567`      the `Mayor` dataclass and its tick methods
`backend/engine/mayor/actions.py`       dispatch and reputation decay
`backend/engine/titles.py`              `TITLE_LADDER`, which `title_rank` indexes

## Shape and why
Reputation is one dict keyed by faction id plus the two pseudo-entries `the_public` and
`moneylender`, and `mayor.reputation["the_public"]` is declared the source of truth that
`ThePublic.support` mirrors. See: `Planning/specs/mayor_spec.md`.

## Fields that matter
- `reputation` — written at 4 (tax effects), 6, 7 (Rally/Agitate), 14, 19 (decay);
  read at 21 as the public's support and at 15 for project effects
- `action_points` — spent at 6, refunded by failing actions, refilled at 19
- `exemptions` — granted at 6, ticked at 19, deleted at 12 when a deal ends; read at 4
- `deals` — created outside the pass; ticked at 12; `cycle_broken` read at 23
- `removal_countdown` — written only at 24; `Optional[int]`, latched over passes
- `title_rank` — written only at 23; at the top rung the run ends in victory
- `oracle_last_consult` — written only by `engine/oracle/consult.py:45`, outside this pass.
  Scope: `**/*.py`, `*.vue`, `*.js` for `oracle_last_consult`

## Hits
- **21-the-public-sync** — reads `reputation["the_public"]` and overwrites `public.support`
- **04-treasury-step-0** — reads `exemptions` next pass to decide taxable factions
- **Anything that moves position 19** — the refill and the three ticks must stay below 6, or
  a cooldown or exemption is granted and expired inside the same pass.

## Does not hit
- **`ThePublic.support`** — the tempting reach: the same number under two names. `support` is
  a mirror written at 21, not a second store. Writing it at any position between 14 and 21
  is silently discarded; write the reputation entry instead.
- **`moneylender.py`'s `removal_countdown`** — a `List[int]` parameter, unrelated to this
  field and never passed by the runner.

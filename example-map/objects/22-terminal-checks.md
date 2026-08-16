# Terminal checks

**Status:** live · Confirmed
**Type:** step
**Runs at:** positions 22, 23, 24 and 25 of 28 — four calls, one latched flag
**Runs:** conditionally — 22 needs `public`; 23, 24 and 25 need `mayor`
**Source of truth:** `backend/engine/special/removal.py` and
`backend/engine/special/ostracism.py` — if this card and those files disagree, the files win.

## What it is
The four ways the run can end, checked in a fixed order once everything else has settled.

## Where it lives
`backend/engine/cycle/runner.py:255-264`   the four call sites
`backend/engine/special/removal.py`        positions 22, 24 and 25
`backend/engine/special/ostracism.py`      position 23, plus `accrue_pressure`

## Shape and why
One end-state, many triggers: each resolution latches `world.game_over` and stamps
`world.end_cause`, and each check returns early if the flag is already set
(`removal.py:50,116,173`; `ostracism.py:259`). See: `Planning/specs/fail-states_spec.md`.

## Writes
- `world.game_over` / `world.end_cause` — `population_collapse`, `ostracized`, `victory`,
  `removal_coalition`, `assassinated`
- `world.ostracism_pressure` — accrued at 23, zeroed when the Assembly convenes
- `mayor.removal_countdown` — set, decremented, cleared (`removal.py:58,68,77,90`)
- `mayor.title_rank` — incremented on acclaim (`ostracism.py:212`)
- `public.pop_warning`, `public.support` — `removal.py:105,125,135,142`
- `world.ostracism_whisper_pledges` — **cleared** when the Assembly convenes
  (`ostracism.py:182`). Accumulated elsewhere: the Mayor's Whisper Campaign at position 6
  (`ostracism.py:285`)
- `faction.leader` — replaced on the ostracised faction via `exile_leader`
  (`ostracism.py:129`, called at `:207`)

## Reads
- `public.population`, `.fed`, `.unrest`, `.support` — written at 14 and 21
- `all_results` — scanned for `Agitate` successes (`ostracism.py:247-249`)
- `mayor.deals` — `cycle_broken == cycle_num` (`ostracism.py:243-245`)
- `active_events` — scanned for disaster effects (`ostracism.py:251`)

## Hits
- **World state** — the latched flag is what a caller loops on
- **Anything that moves these positions** — 22 must stay above 24 (the population latch makes
  removal a no-op), and 23 above 24 so ostracism and victory take precedence. All four must
  stay below 14 and 21, whose numbers they judge.

## Does not hit
- **11-break-sweep** — the tempting reach: the other place a run "ends" for someone. A
  faction at 0 health Breaks and continues, and an exile replaces a faction's *leader*, not
  the faction. No check here removes a faction; only the Mayor loses.

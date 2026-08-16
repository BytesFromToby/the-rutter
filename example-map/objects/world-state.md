# World state

**Status:** live · Confirmed
**Type:** object
**Written by:** 7, 13, 16, 18, 22, 23, 24, 25 — in tick order
**Source of truth:** `backend/engine/models.py:134-148` — if this card and that file
disagree, the file wins.

## What it is
The city-level record: what time it is, how disordered each domain is, and whether the run
is over.

## Where it lives
`backend/engine/models.py:134-148`   the `WorldState` dataclass
`backend/engine/events/world.py`     the chaos roll
`backend/engine/special/`            the terminal latches and the pressure clock

## Shape and why
Four unrelated concerns share one dataclass — the clock, the chaos map, the terminal latch,
and the ostracism pressure clock. Nothing but position order relates them.
See: `Planning/reference/data-models.md`.

## Fields that matter
- `cycle` — written only at 18; read by every position below it, by `min_cycle` event gates,
  and by the oracle cooldown outside the pass
- `chaos` — raised at 13 by event effects, decayed at 16; read at 16 and 17
- `game_over` / `end_cause` — latched at 22, 23, 24 or 25; each check returns early if
  already set, so the first to latch wins the `end_cause` string
- `ostracism_pressure` — accrued at 23, zeroed only when the Assembly convenes; never
  decremented anywhere (`ostracism.py:236`)
- `ostracism_whisper_pledges` — booked by the API-side `WhisperCampaign`, applied and
  cleared at 23
- `initiative_order` — written at 7, read by nothing (see the leftovers register)

## Hits
- **16-world-chaos** and **17-event-deck-roll** — both read `chaos`, one position apart, and
  the second sees the first's decay
- **22-terminal-checks** — the latch is the only thing that stops a caller looping
- **Anything that moves position 18** — it splits the pass into a before-clock and
  after-clock half; every `min_cycle` and cooldown comparison shifts with it.

## Does not hit
- **`CycleResult`** — the tempting reach: the pass's other "state of the world" object.
  `CycleResult` is built fresh at position 27, returned, and never read back in; it holds no
  state across passes and mutating it changes nothing in the simulation.

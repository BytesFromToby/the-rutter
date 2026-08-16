# Moneylender

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 20 of 28 in `run_cycle`
**Runs:** conditionally — `treasury` and `mayor` present, **and** `treasury.debt > 0`;
returns immediately otherwise (`moneylender.py:28-29`)
**Source of truth:** `backend/engine/special/moneylender.py` — if this card and that file
disagree, the file wins.

## What it is
Reads the city's debt and makes the creditor's temper part of the record.

## Where it lives
`backend/engine/cycle/runner.py:244-246`      the call site
`backend/engine/special/moneylender.py:17-75` `process_moneylender`
`backend/engine/balance.py`                   `leverage_threshold`, `removal_threshold`

## Shape and why
Two debt thresholds, each emitting a narrative result; the creditor is modelled as a faction
in the `factions` dict under the id `moneylender`.
See: `Planning/specs/special-factions_spec.md`.

## Writes
- `factions["moneylender"].traits` — "angry at mayor" amplified, `moneylender.py:48`
- `factions["moneylender"]._leverage_steal_bonus` — `moneylender.py:42`; nothing reads it
  (see [leverage-steal-bonus](leverage-steal-bonus.md))
- appends `MoneylenderLeverage` / `MoneylenderAngry` results to `all_results`
- **It does not write `treasury` or `mayor` at all**, despite taking both. Scope: full read
  of `special/moneylender.py` (89 lines); the only assignments in the file are lines 42, 48.

## Reads
- `treasury.debt` — as left by 4 (interest accrual) and 6
- `factions[moneylender_id]` — may be absent, in which case both writes are skipped

## Hits
- **09-end-of-cycle-sweep** — next pass, `evolve_traits` operates on the trait list amplified here
- **Anything that moves this step's position** — it must stay below 4, which is where debt
  grows when interest cannot be paid.

## Does not hit
- **22-terminal-checks** — the tempting reach: this file contains a `MayorRemovalAttempt`
  result and a `removal_countdown`. That branch takes a `List[int]` parameter the runner
  never passes (`runner.py:245`), so it is unreachable from this pass. The live removal
  spiral is `Mayor.removal_countdown`, an unrelated `Optional[int]`, written at position 24.

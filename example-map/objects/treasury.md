# Treasury

**Status:** live · Confirmed
**Type:** object
**Written by:** 4, 5, 6, 15 — in tick order. Position 20 reads it and never writes it.
**Source of truth:** `backend/engine/models.py:444-483` — if this card and that file
disagree, the file wins.

## What it is
The city's money, plus the two per-pass counters that describe where it went.

## Where it lives
`backend/engine/models.py:444-483`     the `Treasury` dataclass
`backend/engine/mayor/treasury.py`     the step-0 processing and the tax levers

## Shape and why
Tax rates are a fixed ladder and the reachable maximum is gated by active `tax_collection`
projects (`models.py:464-470`), so the treasury's ceiling is a project outcome, not a dial.
See: `Planning/specs/treasury_spec.md`.

## Fields that matter
- `gold` — written at 4, 5, 6, 15; read at 4 and 20
- `income_this_cycle` / `expenditure_this_cycle` — **zeroed at 4** (`treasury.py:26`) then
  accumulated by 4, 5, 6 and 15. Any gold moved by a position *above* 4 is invisible.
- `guard_paid_this_cycle` — written at 4 (`treasury.py:89`), read at 14 as the unrest lever.
  Its default is `True`, so a pass with no treasury reports the guard as paid.
- `debt`, `debt_rate` — written at 4 and 6; read at 20, the only position that judges them
- `domain_tax_rates`, `exemptions` interplay — read at 4

## Hits
- **14-public-needs** — `guard_paid_this_cycle` reaches the unrest calculation
- **20-moneylender** — `debt` alone decides whether that position does anything at all
- **17-event-deck-roll** — the treasury is passed to `check_scripted_events` for its gates
- **Anything that moves position 4** — it is the zeroing point; every other gold write in the
  pass is measured relative to it.

## Does not hit
- **`Mayor.action_points`** — the tempting reach: the pass's other spendable resource, spent
  in the same call at position 6, refunded by the same functions. It is refilled at 19, not
  at 4, and no treasury path reads or writes it.
- **20-moneylender** as a *writer* — that position takes a `Treasury` and never mutates it.

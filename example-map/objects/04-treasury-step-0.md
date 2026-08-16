# Treasury step 0

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 4 of 28 in `run_cycle`
**Runs:** conditionally — only when both `treasury` and `mayor` are passed in
**Source of truth:** `backend/engine/mayor/treasury.py` — if this card and that file
disagree, the file wins.

## What it is
Collects the city's income, pays its standing bills, and records whether the guard was paid.

## Where it lives
`backend/engine/cycle/runner.py:111-121`   the call site and the `n_active` count
`backend/engine/mayor/treasury.py:26-106`  `process_treasury_step0`
`backend/engine/mayor/treasury.py`         `apply_tax_effects`

## Shape and why
Cycle totals are zeroed here (`reset_cycle_totals`, `treasury.py:26`) and then refilled, so
every later gold movement in the pass accumulates onto a fresh base.
See: `Planning/specs/treasury_spec.md`.

## Writes
- `treasury.income_this_cycle` / `.expenditure_this_cycle` — zeroed then refilled
- `treasury.gold`, `.debt`, `.invested`, `.invest_cycles_remaining` — every pass
- `treasury.guard_paid_this_cycle` — `treasury.py:89`, from what was actually paid
- `mayor.reputation[...]` — via `apply_tax_effects`

## Reads
- `base_stacks[*].active_count()` and `projects[*].status` for `n_active` — written at 7 and
  15 **last pass**
- `mayor.exemptions` — written at 19 and cleared at 12

## Hits
- **14-public-needs** — reads `treasury.guard_paid_this_cycle` at `runner.py:170` as the
  unrest Guard lever
- **Anything that moves this step's position** — the zeroing must stay above 5, 6, 15 and 20,
  or their gold movements are wiped mid-pass.
- **Treasury** — the object every later gold write lands on

## Does not hit
- **20-moneylender** — the other treasury position. Existing debt interest is serviced here;
  new borrowing and lender reputation are decided at 20, after the cycle counter moved.

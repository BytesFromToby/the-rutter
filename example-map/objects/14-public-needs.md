# Public needs

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 14 of 28 in `run_cycle`
**Runs:** conditionally — only when `public` is passed in
**Source of truth:** `backend/engine/needs/drift.py` — if this card and that file disagree,
the file wins.

## What it is
Works out what the city produced this pass and drifts every one of the people's scales
toward what that production supports.

## Where it lives
`backend/engine/cycle/runner.py:166-172`   the call site
`backend/engine/needs/chain.py`            `compute_chain`, `chain_role_faction_ids`
`backend/engine/needs/drift.py:41-124`     `apply_needs`, the drift itself

## Shape and why
Every scale moves at most one drift step per pass toward a computed target, so the city
cannot swing in a single cycle. See: `Planning/specs/public-needs_spec.md` and
`Planning/specs/food-supply_spec.md`.

## Writes
- `public.fed`, `.happy`, `.piety`, `.unrest`, `.consumption` — drifted, `drift.py:71-116`
- `public.health`, `.population` — `drift.py:77,91-93`
- `public.drunk` — derived from the consumption band, `drift.py:106`
- `public.support` **or** `mayor.reputation["the_public"]` — `drift.py:41-47` routes to
  whichever is the source of truth

## Reads
- `faction.toiling` / `.withholding` — written at 7 and 13, **this pass**
- `treasury.guard_paid_this_cycle` — written at 4, passed at `runner.py:170`

## Hits
- **22-terminal-checks** — `public.fed` and `.unrest` bands drive ostracism pressure
  (`ostracism.py:239-240`); `public.population` drives collapse
- **17-event-deck-roll** — need-gated event templates read these bands
- **Anything that moves this step's position** — must stay below 7 and 13 (the flags) and
  above 26 (which erases them).

## Does not hit
- **21-the-public-sync** — the tempting reach: both write `ThePublic`. 21 overwrites
  `public.support` outright from `mayor.reputation["the_public"]`, so a support value
  written here through the direct branch is replaced seven positions later.

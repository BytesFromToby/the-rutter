# External threats

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 5 of 28 in `run_cycle`
**Runs:** conditionally — `external_threats` non-empty and both `treasury` and `mayor` present
**Source of truth:** `backend/engine/special/external_threats.py` — if this card and that
file disagree, the file wins.

## What it is
Applies each standing outside pressure — bandits, a rival city, a plague vector — to the
city before anyone inside it acts.

## Where it lives
`backend/engine/cycle/runner.py:124-127`        the call site
`backend/engine/special/external_threats.py`    `process_external_threats`
`backend/engine/models.py:398-414`              `ExternalThreat`, `ThreatEffect`

## Shape and why
Placed above the mayor and faction positions on purpose — the runner comment states the
effects must be visible before mayor actions resolve.
See: `Planning/specs/special-factions_spec.md`.

## Writes
- `faction.health` — `external_threats.py:71`, clamped 0–100
- `treasury.gold` — `external_threats.py:81`
- `treasury.expenditure_this_cycle` — `external_threats.py:83`, when the amount is negative
- appends to `all_results`

## Reads
- `threat.active`, `.threat_level`, `.duration`, `.effects` — held on the threat list itself
- `domains`, `world` — passed through for effect targeting

## Hits
- **11-break-sweep** — a faction driven to 0 health here Breaks at position 11, not now
- **Anything that moves this step's position** — below 4 it would spend gold the treasury
  step then zeroes out of the cycle totals.
- **Treasury** — its gold write lands in the same pot as 4, 6, 15 and 20

## Does not hit
- **13-active-event-effects** — the tempting neighbour: also "effects applied to factions and
  domains each cycle". `ExternalThreat`/`ThreatEffect` is a separate model from
  `GameEvent`/`EventEffect`, on a separate list, with no shared deck or status machine.

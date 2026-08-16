# Mayor upkeep

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 19 of 28 in `run_cycle`
**Runs:** conditionally — only when `mayor` is passed in
**Source of truth:** `backend/engine/models.py:531-567` and
`backend/engine/mayor/actions.py:58` — if this card and those files disagree, the files win.

## What it is
Gives the Mayor next pass's action point, counts down every clock he is running, and lets
his standing with everyone decay a little toward neutral.

## Where it lives
`backend/engine/cycle/runner.py:236-241`   the five calls
`backend/engine/models.py:531-567`         `refill`, `tick_cooldowns`, `tick_exemptions`,
                                           `tick_commitments` — methods on `Mayor`
`backend/engine/mayor/actions.py:58`       `apply_reputation_decay`

## Shape and why
Four of the five are model methods rather than subsystem functions, so this position is
mostly the runner calling the dataclass on itself.
See: `Planning/specs/mayor_spec.md` and `Planning/specs/balance_spec.md`.

## Writes
- `mayor.action_points` — `+1`, capped at `action_cap`
- `mayor.cooldowns`, `.exemptions` — decremented, entries deleted at 0
- `mayor.committed_actions` — decremented and filtered
- `mayor.reputation` — decayed, scaled by the active balance profile

## Reads
- all of the above as left by 6, 12 and the API layer

## Hits
- **06-mayor-actions** — next pass spends the point granted here
- **04-treasury-step-0** — next pass reads `mayor.exemptions` for taxable factions
- **Anything that moves this step's position** — it must stay below 6, or a faction granted
  an exemption this pass has it ticked down in the same pass it was granted.

## Does not hit
- **06-mayor-actions** — the tempting reach: the other mayor position, thirteen apart. This
  one runs unconditionally whenever a Mayor exists; that one runs only when the player
  submitted something. Nothing here dispatches an action or reads `_ACTION_MAP`.

# Mayor actions

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 6 of 28 in `run_cycle`
**Runs:** conditionally — `mayor` present and `mayor_actions` non-empty
**Source of truth:** `backend/engine/mayor/actions.py` — if this card and that file
disagree, the file wins.

## What it is
Executes the actions the player submitted before this pass began.

## Where it lives
`backend/engine/cycle/runner.py:130-134`       the call site
`backend/engine/mayor/actions.py:27-56`        `execute_mayor_actions`, `_ACTION_MAP`
`backend/engine/mayor/actions.py:13`           `ACTION_COSTS`

## Shape and why
Dispatch is a name→function map with an action-point cost table; an action that cannot
resolve refunds its points (`actions.py:165,171`).
See: `Planning/specs/mayor_spec.md`.

## Writes
- `mayor.action_points` — spent per action, refunded on a no-op
- `mayor.reputation[...]`, `mayor.exemptions`, `mayor.committed_actions`
- `treasury.gold` / `.expenditure_this_cycle` — e.g. `actions.py:177-178`
- `faction.rating`, `faction.health` — e.g. Sabotage, `actions.py:179-180`

## Reads
- `mayor.action_points` — refilled at 19 **last pass**
- `treasury.gold` — written at 4 this pass

## Hits
- **07-faction-action-loop** — the factions it rated or wounded act moments later, this pass
- **Anything that moves this step's position** — below 19 it would spend points the refill
  has not granted yet; above 4 it spends gold before income arrives.
- **Mayor** — the object 12, 19, 20, 23 and 24 also write

## Does not hit
- **07-faction-action-loop** as a source of *the same* actions — "action" means two things
  here. `MayorAction` names come from `ACTION_COSTS`; faction actions come from
  `engine/npc/behavior.py` and resolve in `engine/actions/`. Neither table sees the other.
- **`WhisperCampaign`** — dispatched by the API route, never through this map.

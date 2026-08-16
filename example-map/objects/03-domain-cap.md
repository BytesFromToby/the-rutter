# Domain cap

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 3 of 28 in `run_cycle`
**Runs:** every pass — unconditional loop over `domains`
**Source of truth:** `backend/engine/cycle/runner.py:98-108` — if this card and that file
disagree, the file wins.

## What it is
Re-derives every domain's crowding number and its ceiling before anyone acts.

## Where it lives
`backend/engine/cycle/runner.py:98-108`   the loop itself
`backend/engine/formulas.py`              `faction_weight`, `stack_cap_contribution`
`backend/engine/models.py:114-131`        the `Domain` dataclass

## Shape and why
Utilization is summed from live faction levels; cap is the frozen `base_cap` plus the
domain's base-project stack contribution, except in a faction-less lane where the authored
cap stands. See: `Planning/specs/treasury_spec.md` (v3 civic-lane rule).

## Writes
- `domain.utilization` — every pass, from `faction.domain_primary` and `faction.level`
- `domain.cap` — every pass, from `domain.base_cap` + `base_stacks[domain_id]`

## Reads
- `faction.rating` / `.domain_primary` — carried from last pass, written at 7, 9, 11, 13
- `base_stacks[domain_id].count` / `.progress` / `.completed` — written at 7 **last pass**

## Hits
- **07-faction-action-loop** — `resolve_grow` blocks when `utilization >= cap`
  (`engine/actions/faction.py:22`); NPC targeting reads both (`engine/npc/behavior.py:216-232`)
- **Anything that moves this step's position** — it must stay above 7, or Grow this pass
  judges itself against last pass's ceiling.
- **04-treasury-step-0** — reads the same `base_stacks` two lines later for `n_active`

## Does not hit
- **`Domain.drift`** — the other mutable field on the same dataclass. This step never
  touches it, and nothing in the engine reads it. See [domain-drift](domain-drift.md).

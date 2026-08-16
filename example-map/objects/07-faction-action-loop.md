# Faction action loop

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 7 of 28 in `run_cycle`
**Runs:** every pass — unconditional
**Source of truth:** `backend/engine/cycle/resolution.py` — if this card and that file
disagree, the file wins.

## What it is
Shuffles the factions into an order and lets each one take exactly one action, live, with
each result visible to whoever acts next.

## Where it lives
`backend/engine/cycle/resolution.py:37-95`   `run_sequential_actions`
`backend/engine/cycle/resolution.py:98-152`  `_execute`, the action dispatch
`backend/engine/npc/behavior.py`             `select_faction_action` (out of the cut)
`backend/engine/actions/faction.py`          the resolvers (out of the cut)

## Shape and why
Sequential, not simultaneous: a target dropped to 0 health Breaks inline before the loop
continues (`resolution.py:86-93`), so ordering inside the loop is itself a mechanic.
See: `Planning/specs/faction-behavior_spec.md`.

## Writes
- `faction.rating`, `.health` — via the resolvers
- `faction.toiling` / `.withholding` — `actions/faction.py:67,81`, cycle-only
- `faction.unstable_stacks` — zeroed at `resolution.py:59` (see the ghost card)
- `base_stacks[*].progress`, `.count`, `.completed`, `.build_actions_this_cycle`
- `world.initiative_order` — `resolution.py:67`; nothing reads it (see leftovers)

## Reads
- `domain.cap` / `.utilization` — written at 3, this pass
- `public.fed`, `.unrest`, `.support` — written at 14 **last pass**, via `npc/behavior.py`
- `faction.committed_action` — written by the LLM deal layer, cleared at 12

## Hits
- **08-trait-scratch** — reads this loop's `all_results` immediately after
- **14-public-needs** — consumes `toiling` / `withholding` set here
- **Anything that moves this step's position** — must stay below 3 (caps) and above 8, 12
  and 14, all of which read what it produced.

## Does not hit
- **06-mayor-actions** — the other "actions" position. The mayor's turn already finished;
  `_ACTION_MAP` and this dispatch share no entries.

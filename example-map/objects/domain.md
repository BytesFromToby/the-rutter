# Domain

**Status:** live · Confirmed
**Type:** object
**Written by:** 3, 13, 15 — in tick order
**Source of truth:** `backend/engine/models.py:114-131` — if this card and that file
disagree, the file wins.

## What it is
A lane of the city — a sector factions belong to, with a crowding number and a ceiling.

## Where it lives
`backend/engine/models.py:114-131`   the `Domain` dataclass
`backend/engine/cycle/runner.py:98-108`   the per-pass recompute
`backend/engine/formulas.py`         `faction_weight`, `stack_cap_contribution`

## Shape and why
`base_cap` is frozen at game start and the live `cap` is re-derived every pass from it plus
the domain's base-project stack, so the ceiling is a build outcome.
See: `Planning/specs/treasury_spec.md` and `Planning/specs/projects_spec.md`.

## Fields that matter
- `cap` — written at 3; read at 7 by `resolve_grow` (`actions/faction.py:22`) and by NPC
  targeting (`npc/behavior.py:216-232`)
- `utilization` — written at 3 from the live faction Σ; read alongside `cap`
- `base_cap` — never written in this pass; read at 3
- any field named by a `ProjectEffect` with `target="domain"` — written at 15 through a
  generic `setattr` (`projects/processing.py:115-118`); `cap` is re-derived at 3 regardless
- `relationships` — never written in this pass; read by NPC targeting
- `drift` — written at 13 by an event effect; read by no engine code
  (see [domain-drift](domain-drift.md))

## Hits
- **07-faction-action-loop** — a domain at or above its cap blocks Grow outright
- **03-domain-cap** — reads last pass's `base_stacks`, so a build completed at position 7
  raises the ceiling only on the following pass
- **Anything that moves position 3** — it must stay above 7 or Grow is judged against a
  ceiling that predates this pass's faction levels.

## Does not hit
- **`engine/needs/drift.py`** — the tempting reach, and the sharpest name trap on this
  object. `Domain.drift` is a float on this dataclass with no engine reader. `needs/drift.py`
  is the live public-needs drift engine that runs at position 14 and touches no `Domain`.
  Searching `drift` finds both.

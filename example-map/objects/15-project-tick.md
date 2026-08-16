# Project tick

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 15 of 28 in `run_cycle`
**Runs:** conditionally — `projects` non-empty and `treasury` present; effects also need `mayor`
**Source of truth:** `backend/engine/projects/processing.py` — if this card and that file
disagree, the file wins.

## What it is
Advances construction, decays or destroys unmaintained works, and pays out what the
finished ones produce.

## Where it lives
`backend/engine/cycle/runner.py:176-180`             the call site
`backend/engine/projects/processing.py:20-70`        `tick_projects`
`backend/engine/projects/processing.py:110-160`      `apply_project_effects`

## Shape and why
Health and status are a tiered ladder — intact, damaged, critical, destroyed — and the tier
scales the effect rather than switching it off.
See: `Planning/specs/projects_spec.md`.

## Writes
- `project.health`, `.status`, `.cycles_built` — `processing.py:26-70`
- `project.build_actions_this_cycle` — zeroed at `processing.py:36`
- `treasury.gold`, `.income_this_cycle` — `processing.py:128-129`
- `faction.rating` / `domain` fields — through `ProjectEffect` targets

## Reads
- `projects` — the caller's dict, mutated in place so runtime-initiated projects persist
- `mayor.reputation` — passed at `runner.py:180`

## Hits
- **04-treasury-step-0** — next pass, `n_active` counts `p.status == "active"` set here
- **Treasury** — its income write is zeroed by 4 at the top of the next pass
- **Anything that moves this step's position** — must stay above 20 and below 4, or its
  income lands outside the cycle totals window.

## Does not hit
- **`base_stacks` / `BaseProjectStack`** — the tempting reach: the other thing called a
  project. Base projects live in a separate dict, are built and sabotaged by factions at
  position 7, and are never touched here. `tick_projects` iterates `projects` only.

# Collisions — one word, two meanings

Scope: what runs inside `run_cycle()`, `BytesFromToby/Polis` @ `04ec8f4`.

| Word | Means here | Also means here | The wrong reach |
|---|---|---|---|
| `event` | `GameEvent` — a thing running in the city, with status, effects and a duration (`engine/models.py:351`), written at positions 13 and 17 | `CycleEvent` — a result record built from an `ActionResult` at position 27 (`engine/models.py:154`, `cycle/runner.py:30`) | You set out to change "how events work" and edit the result serialiser, or the reverse. Unrelated objects, one word. |
| `action` | A faction action — chosen in `engine/npc/behavior.py`, resolved in `engine/actions/` at position 7 | A mayor action — `MayorAction`, dispatched through `_ACTION_MAP` at position 6 | Two positions, two dispatch tables, no shared entries. Adding to one leaves the other untouched. |
| `cycle` | `world.cycle`, the counter incremented at position 18 | `cycle_num`, a local captured at `runner.py:86` *before* that increment — every `CycleEvent` is stamped with it | You add a check against `world.cycle` below position 18 and it is one higher than the number the pass's own results carry. |
| `cycle` (again) | The `engine/cycle/` folder — runner, resolution, end_of_cycle | A unit of game time | "Change the cycle" is ambiguous between the number, the pass, and the folder. |
| `drift` | `Domain.drift` — a float written at position 13, read by no engine code (see the ghost card) | `engine/needs/drift.py` — the live public-needs drift engine that runs at position 14 | A grep for `drift` returns both. Editing the dead field while meaning the live engine changes nothing and reports no error. |
| `project` | `Project` in the `projects` dict — ticked at position 15 | `BaseProjectStack` in the `base_stacks` dict — built and sabotaged by factions at position 7 | `tick_projects` never touches base stacks. A change to "project health" lands on only one of the two. |
| `project` (again) | An in-game project — `engine/projects/` | The `Planning/` sense — a build plan for the repo itself | A reader asked to "check the projects" can land in either world. |
| `health` | `Faction.health` — 1–100 buffer, Break trigger at position 11 | `Project.health` (0–100 structural, position 15) **and** `ThePublic.health` (0–100, position 14) | Three fields, three owners, three positions, one name. None of them read each other. |
| `support` | `ThePublic.support` — a mirror, overwritten at position 21 | `mayor.reputation["the_public"]` — the declared source of truth | Writing `public.support` between positions 14 and 21 is silently discarded at 21. |
| `removal_countdown` | `Mayor.removal_countdown` — `Optional[int]`, the live removal spiral, written at position 24 (`special/removal.py`) | A `List[int]` parameter of `process_moneylender` (`special/moneylender.py:22`) that the runner never passes | You wire the removal spiral into the moneylender branch that looks like it already does it. That branch is unreachable from this pass. |
| `public` | `ThePublic` — a game entity, and the needs step at position 14 | Visibility, as in public/private | "Public" in this repo is a game entity, not an access modifier. |
| `audience` | An LLM faction audience — `engine/llm/audiences.py`, **outside this cut** | Not a crowd, not spectators, not `ThePublic` | Deals reach this pass at position 12 already made. The negotiation is unmapped — go read that file directly. See `Planning/decisions/2026-05-29-audience-term-clarity.md`. |

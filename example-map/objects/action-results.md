# Action results

**Status:** live · Confirmed
**Type:** object
**Written by:** almost every position — 4, 5, 6, 7, 10, 11, 13, 14, 15, 16, 17, 20, 21,
22, 23, 24, 25
**Source of truth:** `backend/engine/models.py:173-187` and
`backend/engine/cycle/runner.py:87` — if this card and those files disagree, the files win.

## What it is
The `all_results` accumulator: one flat list every position appends to, and that four
positions read back as evidence of what happened.

**Per-pass, not cross-tick.** Initialised at `runner.py:87`, discarded at 28.

## Where it lives
`backend/engine/models.py:173-187`     the `ActionResult` dataclass
`backend/engine/cycle/runner.py:87`    `all_results` created
`backend/engine/cycle/runner.py:30-50` `_to_cycle_event`, the conversion at position 27

## Shape and why
Positions that never call each other move each other through this list: the deal tick learns
what a faction did, and the Assembly learns that someone agitated, without either subsystem
importing the other. See: `Planning/specs/cycle-runner_spec.md`.

## Fields that matter
- `action`, `outcome`, `actor_id`, `target_id` — the four fields every reader filters on
- `dramatic` — read at 16 and 7 to decide what reaches the logger, and at 27 to grade severity
- `data` — an additive payload carried onto `CycleEvent` for the oracle digest

## Hits
- **08-trait-scratch** — filters it for `Harm` and for each faction's action
- **12-deal-tick** — flattens it to `acted_this_cycle` (`end_of_cycle.py:169-171`)
- **21-the-public-sync** — receives it as `cycle_results` but never inspects it (scope:
  full read of `special/public.py:10-91`)
- **22-terminal-checks** — position 23 scans it for successful `Agitate`
  (`ostracism.py:247-249`)
- **Anything that appends after position 23** — positions 24 and 25 append results that no
  reader in this pass will see.

## Does not hit
- **`engine/events/`** — the tempting reach: results are converted to `CycleEvent` at
  position 27 and `CycleEvent` shares a word with `GameEvent`. Appending here never creates
  a `GameEvent`, never enters `active_events`, and never triggers an effect.

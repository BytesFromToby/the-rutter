# Leadership events

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 10 of 28 in `run_cycle`
**Runs:** every pass — unconditional
**Source of truth:** `backend/engine/cycle/end_of_cycle.py:67-130` — if this card and that
file disagree, the file wins.

## What it is
Advances the clock on weakened and absent leaders, and installs a replacement when the
absence has run long enough.

## Where it lives
`backend/engine/cycle/runner.py:148`              the call site
`backend/engine/cycle/end_of_cycle.py:67-101`     `run_leadership_events`
`backend/engine/cycle/end_of_cycle.py:103-130`    `_replace_leader`, `_generate_leader_name`

## Shape and why
A two-stage decay held in undeclared per-faction counters — `weakened` becomes `absent`
after 2 passes, `absent` triggers replacement after 3.
See: `Planning/specs/faction-behavior_spec.md`.

## Writes
- `faction.leader` — replaced wholesale at `end_of_cycle.py:113`
- `faction.leader.status` — `weakened` → `absent` at `end_of_cycle.py:83`
- `faction._leader_weakened_cycles`, `._leader_absent_cycles` — persist across passes
- appends `LeaderAbsent` / `LeaderReplaced` to `all_results`

## Reads
- `faction.leader.status` — as left by 7, 11 or 23
- `faction.traits` — rewritten one position earlier at 9, sampled for the new leader

## Hits
- **07-faction-action-loop** — `faction.is_leaderless()` gates behaviour next pass
- **Faction** — writes the same `leader` slot as 11 and 23
- **Anything that moves this step's position** — must stay below 9, whose trait rewrite it
  samples from, or a new leader inherits the previous pass's personality.

## Does not hit
- **11-break-sweep** — the tempting reach: it also installs a fresh `Leader` on a faction.
  Break's replacement is a 25% branch of a health-0 roll (`resolution.py:161-166`) with an
  empty trait list; this one is a timed status decay. Two writers, one field, no shared path.

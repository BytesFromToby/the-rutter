# Cycle counter

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 18 of 28 in `run_cycle`
**Runs:** every pass — unconditional, one line
**Source of truth:** `backend/engine/cycle/runner.py:235` — if this card and that file
disagree, the file wins.

## What it is
Moves the city's clock forward by one.

## Where it lives
`backend/engine/cycle/runner.py:235`   `world.cycle += 1`
`backend/engine/cycle/runner.py:86`    `cycle_num = world.cycle`, the pre-increment snapshot
`backend/engine/models.py:136`         `WorldState.cycle`

## Shape and why
The increment sits two-thirds of the way down the pass, not at the end, so positions 19–26
run against the *new* number while everything above them ran against the old one.
See: `Planning/specs/cycle-runner_spec.md`.

## Writes
- `world.cycle` — `+1`, every pass, unconditionally

## Reads
- `world.cycle` — its own prior value

## Hits
- **22-terminal-checks** — positions 22–25 all run after the increment
- **17-event-deck-roll** — `min_cycle` trigger gates read `world.cycle` next pass
- **Anything that moves this step's position** — every position below it observes a cycle
  number one higher than every position above it. Moving this line silently reclassifies
  which side of the boundary a subsystem sits on.

## Does not hit
- **`cycle_num`** — the tempting reach, and the sharpest trap in the pass. `cycle_num` is a
  plain local captured at `runner.py:86`, before this increment. Every `CycleEvent` built at
  position 27 is stamped with `cycle_num`, so the results of a pass carry the number the
  pass *started* with, not the value this line leaves behind.

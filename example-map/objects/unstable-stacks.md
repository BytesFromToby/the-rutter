# Faction.unstable_stacks

**Status:** ghost · Confirmed
**Type:** ghost

## The reach
You are adding something that should make a faction's rolls worse for a pass — a wound, a
scandal, a failed build. You find `Faction.unstable_stacks` in `models.py:73` with the
comment "-1 per stack to rolls, max 3", and `reset_cycle_state()` right below it clearing
the field every pass. It reads as a live per-pass penalty with a working reset. You set it
and expect the next contest to be harder.

## What is actually there
The field, the comment, the reset method, and a serializer that round-trips the value
(`backend/serializer.py:95,118`). Nothing sets it to a non-zero value, and no contest reads
it. The reset runs twice per pass — at position 7 (`resolution.py:59`) and again at
position 9 (`end_of_cycle.py:57`) — clearing a number nothing ever wrote.

## Evidence
Search scope: `**/*.py`, `*.vue`, `*.js` for `unstable`. Four hits, all
accounted for: `engine/models.py:73` (the declaration), `engine/models.py:111` (the reset
body), `backend/serializer.py:95` and `:118` (persist and restore). No writer, no reader.
`engine/formulas.py` and `engine/actions/` contain no reference to it.

This is a silent reach, not an inert one: you write the field, nothing errors, and no roll
changes. That is what puts it here rather than in the leftovers register.

## What to do instead
Nothing carries a per-pass roll penalty today. `Faction.health` is the live per-faction
condition number, written at positions 5, 6, 7, 9, 11 and 13 and read at 11 — see
[faction](faction.md). If you need a roll modifier, you are adding a mechanic, not wiring up
an existing one.

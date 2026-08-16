# _leverage_steal_bonus

**Status:** ghost · Confirmed
**Type:** ghost

## The reach
You are working on debt pressure and want the creditor to press harder when the city owes
him. You find it already written: `backend/engine/special/moneylender.py:42` sets
`ml._leverage_steal_bonus = 10` on the moneylender faction, under the comment "Flag for
behavior engine: Moneylender gets +10 Steal this cycle", and the narrative string one line
above says "Steal +10 vs all factions". A committed test asserts the flag is set
(`tests/test_special_factions.py:112`). Everything says this works.

## What is actually there
The assignment, the comment, the narrative text, and the test. The behavior engine does not
read it. `engine/npc/behavior.py` builds action weights with no reference to the attribute,
and no resolver in `engine/actions/` consults it.

The position makes it worse: the flag is set at position 20, and faction actions resolve at
position 7. Even a consumer added in the behavior engine would see it a pass late.

## Evidence
Search scope: `**/*.py`, `*.vue`, `*.js` for `_leverage_steal_bonus`. Two
hits: `engine/special/moneylender.py:42` (the write) and
`backend/tests/test_special_factions.py:112` (a `getattr` assertion that the write happened).
No engine reader. The test proves the flag is set, not that anything acts on it.

## What to do instead
Nothing in the pass makes the moneylender steal. What position 20 really does is amplify an
"angry at mayor" trait on the moneylender faction (`moneylender.py:48`), which the trait
system at position 9 does read. See [20-moneylender](20-moneylender.md).

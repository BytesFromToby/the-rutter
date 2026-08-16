# Deal tick

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 12 of 28 in `run_cycle`
**Runs:** conditionally — only when `mayor` is passed in
**Source of truth:** `backend/engine/cycle/end_of_cycle.py:155-236` — if this card and that
file disagree, the file wins.

## What it is
Checks whether each faction kept its bargain this pass, suspends or expires the deal
accordingly, and unwinds the commitments when it ends.

## Where it lives
`backend/engine/cycle/runner.py:151-152`          the call site
`backend/engine/cycle/end_of_cycle.py:155-221`    `tick_deals`
`backend/engine/cycle/end_of_cycle.py:224-236`    `_clear_deal_effects`

## Shape and why
Compliance is judged from what the faction actually did this pass, read out of the results
accumulator, not from any plan. Three suspended passes expire the deal as fulfilled.
See: `Planning/specs/audience_spec.md`.

## Writes
- `deal.status`, `.cycles_remaining`, `.suspension_streak`, `.cycle_broken`
- `faction.committed_action`, `.committed_target`, `.committed_deal_id`,
  `.committed_abstain_action`, `.committed_abstain_target` — cleared on expiry
- `mayor.exemptions[faction_id]` — deleted when the deal granted it

## Reads
- `all_results` — flattened to `acted_this_cycle` at `end_of_cycle.py:169-171`
- `mayor.deals` — populated by the LLM audience layer, outside this pass

## Hits
- **07-faction-action-loop** — clearing `committed_action` frees the faction's choice next pass
- **22-terminal-checks** — `deal.cycle_broken` feeds the ostracism pressure clock at 23
- **Anything that moves this step's position** — must stay below 7 and 11, whose results it
  reads as evidence of compliance.

## Does not hit
- **`engine/llm/audiences.py`** — the tempting reach: deals are *created* there
  (`audiences.py:404-408` sets the same commitment fields). This position only ticks and
  unwinds them; it never negotiates, and that file is outside the cut.

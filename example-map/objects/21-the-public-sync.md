# The Public sync

**Status:** live · Confirmed
**Type:** step
**Runs at:** position 21 of 28 in `run_cycle`
**Runs:** conditionally — both `public` and `mayor` must be present
**Source of truth:** `backend/engine/special/public.py` — if this card and that file
disagree, the file wins.

## What it is
Pulls the people's support back into line with the Mayor's standing, re-derives their mood,
and lets that mood shape how they feel about him.

## Where it lives
`backend/engine/cycle/runner.py:249-251`   the call site
`backend/engine/special/public.py:10-91`   `process_the_public` and its two helpers

## Shape and why
`mayor.reputation["the_public"]` is declared the source of truth and `public.support` is a
mirror of it, overwritten wholesale at `public.py:20`.
See: `Planning/specs/special-factions_spec.md`.

## Writes
- `public.support` — **overwritten**, not adjusted, from `mayor.get_reputation("the_public")`
- `public.disposition` — re-derived from the new support band
- `public.traits` — "distrusts"/"angry at" mayor added, decayed or removed
- `public._content_bonus` / `._angry_penalty` — nothing reads them
  (see [public-disposition-flags](public-disposition-flags.md))

## Reads
- `mayor.reputation["the_public"]` — written at 6, 7 and 14, decayed at 19
- `public.disposition` — its previous value, to detect the transition
- `all_results` — accepted as `cycle_results`, never inspected. Scope: full read of
  `special/public.py:10-91`; the parameter appears in the signature and nowhere else.

## Hits
- **22-terminal-checks** — `public.support` drives the ostracism public-shard tally
  (`ostracism.py:115-119`) and the removal support drain
- **Anything that moves this step's position** — it must stay below 19, or it mirrors a
  reputation the decay has not yet applied.

## Does not hit
- **14-public-needs** — the tempting reach: both write `ThePublic`, seven positions apart.
  Position 14 owns the seven scales; this one owns only `support`, `disposition` and
  `traits` — and it clobbers any `support` that 14 wrote through its direct branch.

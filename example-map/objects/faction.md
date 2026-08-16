# Faction

**Status:** live · Confirmed
**Type:** object
**Written by:** 5, 6, 7, 8, 9, 10, 11, 12, 13, 15, 20, 23, 26 — in tick order
**Source of truth:** `backend/engine/models.py:51-111` — if this card and that file
disagree, the file wins.

## What it is
A standing power bloc in the city. It is never removed, never dies, and is written at more
positions in the pass than anything else.

## Where it lives
`backend/engine/models.py:51-111`   the `Faction` dataclass
`backend/engine/models.py:20-32`    `FactionTrait`, `Leader`

## Shape and why
Rating (1.0–10.0) is the persistent rank and `level` is its integer floor; health is a
breaking-point buffer that refills to 75 on Break rather than killing the faction.
See: `Planning/reference/data-models.md`.

## Fields that matter
- `rating` — written at 6, 7, 11, 13, and at 15 by any `ProjectEffect` with
  `target="faction"` (`projects/processing.py:120-124`, a generic `setattr`); read at 3
- `health` — written at 5, 6, 7, 9, 11, 13; read at 11 as the Break trigger
- `leader` — written at 7, 10, 11, 23; four writers, one slot
- `traits` — rewritten at 9; amplified at 20 for the moneylender faction
- `toiling` / `withholding` — written at 7 and 13, read at 14, cleared at 26
- `committed_action` and the four other `committed_*` — set outside the pass by the LLM
  audience layer, cleared at 12
- `domain_primary` — never written in this pass; read at 3 and 16
- `unstable_stacks` — zeroed at 7 and 9, written non-zero by nothing
  (see [unstable-stacks](unstable-stacks.md))

## Hits
- **03-domain-cap** — `rating` and `domain_primary` fix every domain's utilization next pass
- **22-terminal-checks** — an exiled leader is installed here at position 23
- **Anything that moves a position that writes it** — the health chain 5 → 7 → 9 → 13 → 11
  must keep 11 last, or damage waits a full pass before it Breaks anything.

## Does not hit
- **ThePublic** — the tempting reach: it is described as a special faction, appears in
  `special-factions_spec`, carries `health`, `support` and a `FactionTrait` list, and its
  moneylender sibling *is* a real entry in the `factions` dict. `ThePublic` is a separate
  dataclass (`models.py:374`) that never appears in `factions` and is written at 14, 21, 22.

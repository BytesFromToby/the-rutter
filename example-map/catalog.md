# Rutter — Polis: what runs inside `run_cycle()`

Territory: `BytesFromToby/Polis` @ `04ec8f4`, walked 2026-08-16
Cards: 36 · Unmapped: Vue frontend, API routes, LLM stack, `Planning/specs`

**How to use this:** find your question below and open that ONE card.
· *What is X?* — read it and stop.
· *What will my change disturb?* — read its **Hits**, then open only the cards it names,
  and stop when they stop naming new ones.
Never browse the objects folder. If a card and the real file disagree, the file wins.

## The spine — `run_cycle` in `backend/engine/cycle/runner.py`, execution order

The runner's own step labels (docstring "Steps 0–4"; comments "Step 8", "Step 9",
"Item 5a/5b", "Step 4–6 already ran above") do not match what executes; there is no
step 3 or 7. The column below is live; the labels are leftover.

| # | Line | Position | Card |
|---|---|---|---|
| 1 | 84 | `chain_roles` derived from `chains` | — |
| 2 | 86–92 | `cycle_num` snapshot, `all_results` init, dict defaults | — |
| 3 | 98–108 | Domain utilization Σ and cap re-derive | [domain-cap](objects/03-domain-cap.md) |
| 4 | 111–121 | Treasury income, expenditure, guard, tax effects | [treasury-step-0](objects/04-treasury-step-0.md) |
| 5 | 124–127 | External threat effects | [external-threats](objects/05-external-threats.md) |
| 6 | 130–134 | Mayor actions submitted before the pass | [mayor-actions](objects/06-mayor-actions.md) |
| 7 | 137–141 | Initiative roll + sequential faction action loop | [faction-action-loop](objects/07-faction-action-loop.md) |
| 8 | 144 | Per-faction outcome scratch for trait evolution | [trait-scratch](objects/08-trait-scratch.md) |
| 9 | 147 | Health decay, trait evolution, scratch clear | [end-of-cycle-sweep](objects/09-end-of-cycle-sweep.md) |
| 10 | 148 | Leader status decay and replacement | [leadership-events](objects/10-leadership-events.md) |
| 11 | 149 | Break sweep for factions at 0 health | [break-sweep](objects/11-break-sweep.md) |
| 12 | 151–152 | Deal tick, compliance, expiry | [deal-tick](objects/12-deal-tick.md) |
| 13 | 158–162 | Active game-event effects, resolved pruned | [active-event-effects](objects/13-active-event-effects.md) |
| 14 | 166–172 | Food chain compute + needs drift | [public-needs](objects/14-public-needs.md) |
| 15 | 176–180 | Project construction, health, effects | [project-tick](objects/15-project-tick.md) |
| 16 | 183–187 | World chaos roll and decay | [world-chaos](objects/16-world-chaos.md) |
| 17 | 193–232 | Random + scripted event deck roll, herald beats | [event-deck-roll](objects/17-event-deck-roll.md) |
| 18 | 235 | `world.cycle += 1` | [cycle-counter](objects/18-cycle-counter.md) |
| 19 | 236–241 | Mayor refill, cooldowns, exemptions, rep decay | [mayor-upkeep](objects/19-mayor-upkeep.md) |
| 20 | 244–246 | Moneylender | [moneylender](objects/20-moneylender.md) |
| 21 | 249–251 | Public support sync + disposition | [the-public-sync](objects/21-the-public-sync.md) |
| 22 | 255–256 | Population collapse check | [terminal-checks](objects/22-terminal-checks.md) |
| 23 | 260–261 | Assembly / ostracism verdict | [terminal-checks](objects/22-terminal-checks.md) |
| 24 | 262 | Mayor removal check | [terminal-checks](objects/22-terminal-checks.md) |
| 25 | 264 | Coup roll | [terminal-checks](objects/22-terminal-checks.md) |
| 26 | 269–271 | Toil / Withhold flag reset | [cycle-flag-reset](objects/26-cycle-flag-reset.md) |
| 27 | 273–277 | `CycleEvent` list + faction action count built | — |
| 28 | 279–283 | `CycleResult` returned | — |

`terminal-checks` occupies positions 22, 23, 24 and 25 — four calls, one latched flag.
No folder shows that; `special/removal.py` and `special/ostracism.py` are separate files.

## Objects — state held across the pass

| Object | Written by | Card |
|---|---|---|
| Faction | 5, 6, 7, 8, 9, 10, 11, 12, 13, 15, 20, 23, 26 | [faction](objects/faction.md) |
| Treasury | 4, 5, 6, 15 | [treasury](objects/treasury.md) |
| Mayor | 4, 6, 12, 19, 23, 24 | [mayor](objects/mayor.md) |
| ThePublic | 14, 21, 22 | [the-public](objects/the-public.md) |
| WorldState | 7, 13, 16, 18, 22, 23, 24, 25 | [world-state](objects/world-state.md) |
| Domain | 3, 13, 15 | [domain](objects/domain.md) |
| active_events list | 13, 17 | [active-events](objects/active-events.md) |
| all_results accumulator | almost every position | [action-results](objects/action-results.md) |

## Ghosts — names with no wiring

| Name | Where you'll meet it | Card |
|---|---|---|
| `Faction.unstable_stacks` | `models.py:73`, a documented roll penalty | [unstable-stacks](objects/unstable-stacks.md) |
| `_content_bonus` / `_angry_penalty` | `special/public.py:77,80`, "flagged for runner" | [public-disposition-flags](objects/public-disposition-flags.md) |
| `Domain.drift` | `models.py:121`; an `EventEffect` field named `drift` | [domain-drift](objects/domain-drift.md) |
| `_leverage_steal_bonus` | `special/moneylender.py:42`, "Flag for behavior engine" | [leverage-steal-bonus](objects/leverage-steal-bonus.md) |

## Registers

- [Collisions](registers/collisions.md) — one word, two meanings. **Read before you grep.**
- [Leftovers](registers/leftovers.md) — real debris, inert, do not mistake for live.

## Not mapped

The Vue frontend (`frontend/`), the FastAPI routes (`backend/api/`), the LLM stack
(`backend/engine/llm/`), and `Planning/specs`. Deals are created and `WhisperCampaign` is
executed on the API side, outside this pass — go read that code directly.
Also outside the pass but reachable from it: `engine/npc/behavior.py` (action selection),
`engine/actions/` (resolvers), `engine/oracle/`, `engine/balance.py`.

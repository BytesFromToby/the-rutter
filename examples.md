# Examples — a worked rutter of Polis

**The real artifact is [`example-map/`](example-map/).** 36 cards, walked 2026-08-16.
Everything below is quoted verbatim from it — this file is a guided sample, not a substitute.
Read three excerpts here, then go open the map itself.

**Territory:** `BytesFromToby/Polis` @ `04ec8f4` · **Cut:** what runs inside `run_cycle()`
**Spine source:** `backend/engine/cycle/runner.py`

---

## The front door

[`example-map/catalog.md`](example-map/catalog.md) opens like this:

> # Rutter — Polis: what runs inside `run_cycle()`
>
> Territory: `BytesFromToby/Polis` @ `04ec8f4`, walked 2026-08-16
> Cards: 36 · Unmapped: Vue frontend, API routes, LLM stack, `Planning/specs`
>
> **How to use this:** find your question below, open that ONE card, stop.
> Do not load the objects folder. If a card and the real file disagree, the file wins.

Then the spine — 28 positions with line numbers, of which four rows:

| # | Line | Position | Card |
|---|---|---|---|
| 13 | 158–162 | Active game-event effects, resolved pruned | `13-active-event-effects` |
| 14 | 166–172 | Food chain compute + needs drift | `14-public-needs` |
| … | | | |
| 26 | 269–271 | Toil / Withhold flag reset | `26-cycle-flag-reset` |

Two things a folder tree cannot say, both stated on the catalog:

> The runner's own step labels (docstring "Steps 0–4"; comments "Step 8", "Step 9",
> "Item 5a/5b", "Step 4–6 already ran above") do not match what executes; there is no
> step 3 or 7. The column below is live; the labels are leftover.

> `terminal-checks` occupies positions 22, 23, 24 and 25 — four calls, one latched flag.
> No folder shows that; `special/removal.py` and `special/ostracism.py` are separate files.

---

## A step card — the reader in the chain

From [`objects/14-public-needs.md`](example-map/objects/14-public-needs.md). The whole card
is 46 lines; these are the four sections that do the work:

> **Runs at:** position 14 of 28 in `run_cycle`
> **Runs:** conditionally — only when `public` is passed in
>
> ## Reads
> - `faction.toiling` / `.withholding` — written at 7 and 13, **this pass**
> - `treasury.guard_paid_this_cycle` — written at 4, passed at `runner.py:170`
>
> ## Hits
> - **22-terminal-checks** — `public.fed` and `.unrest` bands drive ostracism pressure
>   (`ostracism.py:239-240`); `public.population` drives collapse
> - **Anything that moves this step's position** — must stay below 7 and 13 (the flags) and
>   above 26 (which erases them).
>
> ## Does not hit
> - **21-the-public-sync** — the tempting reach: both write `ThePublic`. 21 overwrites
>   `public.support` outright from `mayor.reputation["the_public"]`, so a support value
>   written here through the direct branch is replaced seven positions later.

That does-not-hit is the shape the whole method is for. Two steps write the same object, they
look interchangeable, and one silently overwrites the other seven positions later.

---

## An object card — the carrier nothing imports

From [`objects/action-results.md`](example-map/objects/action-results.md). An object card is
indexed by who writes it, not by where it lives:

> **Written by:** almost every position — 4, 5, 6, 7, 10, 11, 13, 14, 15, 16, 17, 20, 21,
> 22, 23, 24, 25
>
> **Per-pass, not cross-tick.** Initialised at `runner.py:87`, discarded at 28.
>
> ## Shape and why
> Positions that never call each other move each other through this list: the deal tick learns
> what a faction did, and the Assembly learns that someone agitated, without either subsystem
> importing the other.
>
> ## Hits
> - **12-deal-tick** — flattens it to `acted_this_cycle` (`end_of_cycle.py:169-171`)
> - **Anything that appends after position 23** — positions 24 and 25 append results that no
>   reader in this pass will see.

Seventeen writers, four readers, and a dead zone at the end where appends go nowhere. It is
carded because several positions write it, not because it survives the pass — an object is
defined by the number of writers, and this one carries more movement than anything else in
the territory.

## A ghost card — the one with a passing test

From [`objects/leverage-steal-bonus.md`](example-map/objects/leverage-steal-bonus.md):

> ## The reach
> You are working on debt pressure and want the creditor to press harder when the city owes
> him. You find it already written: `backend/engine/special/moneylender.py:42` sets
> `ml._leverage_steal_bonus = 10` on the moneylender faction, under the comment "Flag for
> behavior engine: Moneylender gets +10 Steal this cycle", and the narrative string one line
> above says "Steal +10 vs all factions". A committed test asserts the flag is set
> (`tests/test_special_factions.py:112`). Everything says this works.
>
> ## What is actually there
> The assignment, the comment, the narrative text, and the test. The behavior engine does not
> read it. […] The position makes it worse: the flag is set at position 20, and faction
> actions resolve at position 7. Even a consumer added in the behavior engine would see it a
> pass late.
>
> ## Evidence
> Search scope: `**/*.py`, `*.vue`, `*.js` for `_leverage_steal_bonus`. Two
> hits: […] No engine reader. The test proves the flag is set, not that anything acts on it.

A comment, a user-visible narrative string, and a **green committed test** all certifying a
mechanic that does nothing. This is why ghosts get cards: nothing here fails loudly.

---

## A leftover — and why it is not a ghost

The card above and this row both have a passing committed test. They are classified
differently, and the difference is the whole live/leftover/ghost line.

From [`registers/leftovers.md`](example-map/registers/leftovers.md):

> | `process_moneylender`'s `removal_countdown` branch | `special/moneylender.py:59-73`
> implements a full removal-coalition countdown against a `List[int]` parameter, and a
> committed test exercises it (`tests/test_special_factions.py:122-133`). | The runner calls
> it at `runner.py:245` without that argument, so the parameter is always `None` and the
> branch never runs in a real pass. |

Reach for this and you get a complete, working countdown — it is simply never called. The
reach **succeeds**. That is a leftover: inert, honest, harmless until mistaken for live.

Reach for `_leverage_steal_bonus` and you get silence — you set it, nothing errors, nothing
happens. The reach **appears** to succeed. That is a ghost, and that is why it gets a card
of its own while this one gets a row in a register.

## One change, traced

**"I want an event that suppresses one faction's food production."**

Open `catalog.md`. The spine shows event effects at 13 and the food chain at 14. Open
**one** card — [`13-active-event-effects`](example-map/objects/13-active-event-effects.md).
It answers the whole question:

> ## Writes
> - `faction.withholding` — `event_system.py:369`, cycle-only
>
> ## Shape and why
> Deliberately placed **above** position 14 so a `withhold` event can zero its target's chain
> contribution in the same pass, while new events are still rolled below 14 so their band
> gates see this pass's numbers.
>
> ## Hits
> - **14-public-needs** — `withholding` set here zeroes that faction's chain output this pass
>
> ## Does not hit
> - **17-event-deck-roll** — the tempting reach: same module, same `GameEvent` list, four
>   positions apart. 17 *creates* events from the deck; this one only runs and retires them.
>   An event created at 17 first takes effect on the next pass.

The mechanism already exists, it belongs at 13 and not 17, position 26 will clear the flag
for you, and `engine/needs/` — the folder whose name matches the question — is the wrong
door, because it only reads a boolean and does not know events exist.

**One hop. One card. No subsystem opened.**

---

## What the cut cost

28 positions, 21 step cards, 8 objects, 4 ghosts, 2 registers. Deliberately unmapped: the
frontend, the API routes, the LLM stack, and the specs — named on the catalog so a reader
knows the silence is a decision rather than a gap.

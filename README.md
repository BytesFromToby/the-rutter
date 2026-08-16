# the-rutter

**A cartographer for tick-loop repos.** Point it at a codebase whose behaviour comes from a
repeated ordered pass — a game loop, a simulation cycle, a backtester — and it leaves behind
a **rutter**: a catalog and a set of cards a later reader can wander without reading the
whole thing.

> A chart shows the coast. A rutter tells you the order you meet it, and what goes wrong if
> you take it out of order.

The chart is the folder tree, and it already exists. The rutter does not.

---

## The one rule

**Load the catalog, then one card, then stop.**

Never load the objects folder. Never add the whole map to a project. If a reader has to hold
all of it, the map failed and the map is what needs fixing — not the reader.

The one exception is not an exception to that. A reader asking *what will my change disturb*
follows the cards named in a card's **Hits** section, and stops when those stop naming new
ones. Every door was handed to them by a card they had already earned — that is the map
working, not a reader browsing. If you cannot point at the card that sent you, you are
browsing.

---

## What to feed it

A repo where both of these hold — open the loop function and check:

1. **One pass is the real spine** — `run_cycle`, `tick`, `step`, `update`, `run_once`.
2. **The stages mutate shared state** rather than handing values along a chain.

In profile: game and city simulations, agent-based models, ECS games, discrete-event
simulators, backtesters, trading loops, tick-based game servers.

Out of class: request/response services, CLI pipelines, ETL DAGs, libraries. There the call
graph *is* the map. The cartographer will say so and stop rather than improvise.

Full entry conditions: [`identity.md`](identity.md).

---

## How to use it

1. Drop this folder into a Claude project.
2. Point it at the repo, and **name the cut** — one loop, not one repo. *"Map what runs
   inside `run_cycle()`"* is a cut. *"Map Polis"* is not.
3. It walks: finds the spine, inventories the nouns, settles each one's status against
   evidence, writes the catalog, then the cards.
4. What comes back is a folder you commit next to the code.

```
catalog.md              the front door — an index, not a document
objects/                one card per noun
registers/
  collisions.md         one word, two meanings
  leftovers.md          real debris, inert, not to be mistaken for live
```

---

## How a cold reader walks it

Whether the reader is a model with no memory or a person who has never seen the repo, the
walk is the same, and it is short:

1. Open `catalog.md`. Nothing else.
2. Find your question. Open **one** card.
3. Read its **Hits** and **Does not hit**.
4. Stop — if you came to ask *what is X*.
5. If you came to make a change, open only the cards that card's **Hits** named, and their
   Hits in turn. Stop when the edges close and no new noun appears.

Full protocol, including what to do when your question isn't in the catalog and what to do
when a card disagrees with the code: [`reference/walk-order.md`](reference/walk-order.md).

**If a card and the real file disagree, the file wins and the card is wrong.** Cards cite
source. They never replace it.

---

## Why it's shaped this way

A folder tree is honest about the axis it was authored along. A codebase is organised by
**ownership** — who owns what — and on that axis its tree tells the truth.

But a hierarchy holds one axis, and a reader arrives asking a different question: *when does
this run, and what will it disturb?* No tree can answer that, because one subsystem may act
at several positions in the pass, and subsystems that never import each other move each
other through shared state.

So the rutter is not a rejection of structure-as-architecture. It is the **second axis**,
authored as a walkable artifact, for territories that only ever got the first one.

---

## What's in this folder

| File | Job |
|---|---|
| [`identity.md`](identity.md) | Who the cartographer is, the gate, the profile, the reader |
| [`rules.md`](rules.md) | The seven passes — how a map gets made |
| [`examples.md`](examples.md) | One worked rutter of a real territory |
| [`reference/card-types.md`](reference/card-types.md) | The closed set of card types and the card schema |
| [`reference/catalog-spine.md`](reference/catalog-spine.md) | The required shape of the front door |
| [`reference/walk-order.md`](reference/walk-order.md) | How a cold reader walks the output |
| [`reference/collisions.md`](reference/collisions.md) | Naming collisions to carry into the walk |
| [`example-map/`](example-map/) | **The product** — a real rutter of Polis, 36 cards. Start at [`catalog.md`](example-map/catalog.md) |

---

## What it is not

A tour guide, an auditor, a diagnostician, a photocopy, or a second spec. It does not tell
you why anything failed, what to fix, or how the code behaves — it tells you what the nouns
are, what moves them, and what else moves when you touch one.

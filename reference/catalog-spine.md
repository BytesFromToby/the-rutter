# Catalog spine — the required shape of the front door

The catalog is the only file the reader is guaranteed to load. It **points**. It stores
almost nothing. A reader enters with one question, takes one hop, and lands on one card.

**Hard limits:** one line per entry. No entry explains anything. If the catalog reads as
prose, it has become a tour and must be cut back to an index.

A worked catalog against a real territory is in [`../examples.md`](../examples.md).

---

## The shape

```markdown
# Rutter — <territory>: <what cut, not "the repo">

Territory: <repo> @ <commit sha>, walked <date>
Cards: <n> · Unmapped: <what was deliberately left out>

**How to use this:** find your question below and open that ONE card.
· *What is X?* — read it and stop.
· *What will my change disturb?* — read its **Hits**, then open only the cards it names,
  and stop when they stop naming new ones.
Never browse the objects folder. If a card and the real file disagree, the file wins.

## The spine — <loop fn> in execution order

<One line if the repo's own numbering disagrees with what runs. Otherwise say nothing.>

| # | Line | Position | Card |
|---|---|---|---|
| 1 | <line> | <what happens> | [<noun>](objects/01-<noun>.md) |
| 2 | <line> | <what happens> | — |

<One line naming any noun that occupies more than one position — a tree cannot show it.>

## Objects — state held across the pass

| Object | Written by | Card |
|---|---|---|

## Ghosts — names with no wiring

| Name | Where you'll meet it | Card |
|---|---|---|

## Registers

- Collisions — one word, two meanings. **Read before you grep.**
- Leftovers — real debris, inert, do not mistake for live.

## Not mapped
<Named honestly. Unmapped is a fact, not a gap to hide.>
```

---

## Rules for the header

- **`Cards: <n>` counts the files in `objects/`** — step, object and ghost together. It does
  not count the catalog itself or the registers. A reader uses it to know how much map exists,
  not how many files are on disk.
- **`Unmapped:`** names what was deliberately left out of the cut. It is never empty; if
  nothing was left out, the cut was the whole repo and that is its own problem.

## Rules for the spine table

- **Cite the line number** of each position in the pass. It is the cheapest possible
  citation and it makes the whole table verifiable in one read.
- **Positions without a card get a dash**, not an invented card. Not every position is a
  noun; some are three lines of bookkeeping.
- **A noun may appear at more than one position.** Repeat it. That repetition is the single
  most valuable fact in the table — it is what no folder can express. Its file is numbered by
  its **earliest** position, so a later row links to a lower number. That is correct, and it
  reads differently depending on the shape:
  - **Split** (positions 8 and 12, say) — the noun ran, other things happened, it runs again.
    The gap is the finding; something in between depends on the order.
  - **Contiguous** (22, 23, 24, 25) — the noun is *still running*. One consolidated card
    covering a block of the pass, not a return visit.

  Say which of the two it is on the card. A reader who assumes "already ran" for a
  contiguous block has the pass backwards.
- **Where the repo's own numbering disagrees with execution order**, say so once, in one
  line, without complaint. Where it agrees, say nothing.

## The five-question test

Before shipping, pick five questions a reader would actually arrive with and confirm each
reaches its card in one hop. If any takes two, re-index.

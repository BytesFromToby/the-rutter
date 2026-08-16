# Card types — the closed set

Five types. **Closed.** If a noun does not fit one, it is not a card — reconsider whether
it is a noun at all (`rules.md`, "What counts as a noun").

| Type | Count | Job |
|---|---|---|
| **Catalog** | exactly 1 | The front door of the rutter. Index only, near-zero content. Shape: `catalog-spine.md` |
| **Step** | one per state-mutating position | Something that runs inside the tick and mutates shared state |
| **Object** | one per multi-position state-holder | A state carrier written by more than one position |
| **Ghost** | one per tripwire | A name with no wiring, written so the reach lands somewhere |
| **Register** | 2–3 | A closed list, no prose: collisions, leftovers |

**The counts are derived, not budgeted.** The pass fixes them — see `rules.md`, pass 3,
"Deriving the card set", for the derivation and the consolidation rule. Do not aim for a
number, and do not trim to one.

**An object is defined by how many positions write it, not by whether it survives the tick.**
A carrier created and discarded inside a single pass still earns a card if several positions
write it — that is exactly how subsystems which never import each other reach one another,
and it is the most valuable thing a rutter can show. Survival is irrelevant; the number of
writers is the whole test.

One file per card, kebab-cased.

**Step cards carry their position as a two-digit prefix** — `15-public-needs.md`,
`04-treasury-step-0.md`. A card occupying several positions is numbered by its **earliest**.
Object, ghost, and register cards take no prefix and sort below the spine.

The prefix is not decoration. It makes `ls objects/` return the pass in execution order, so
the directory listing carries the same axis the catalog does — the one axis the source tree
could not hold.

---

## Step card

```markdown
# <Noun>

**Status:** live · Confirmed
**Type:** step
**Runs at:** position <n> of <total> in <loop fn>
**Runs:** every pass | conditionally — <the condition>
**Source of truth:** <path> — if this card and that file disagree, the file wins.

## What it is
<One sentence. What it does to the world, not how.>

## Where it lives
<path/>            <what's in it>
<path/file.py>     <what's in it>

## Shape and why
<Two sentences max. Point at the decision record or spec; do not restate it.>
See: <path to decision/spec>

## Writes
- `<field>` — <when>

## Reads
- `<field>` — written by <noun> at position <n>

## Hits
- **<Noun>** — reads `<field>` at position <n>
- **Anything that moves this step's position** — <the ordering constraint, one line>

## Does not hit
- **<Tempting wrong neighbour>** — <why the reach fails, one line>
```

**Runs at** is what makes this class of map work. A step card without a position is a
glossary entry.

---

## Object card

Same schema, with three differences:

- **`Runs at:`** becomes **`Written by:`** — the steps that mutate it, in tick order.
- **`Writes` / `Reads`** become **`Fields that matter`** — only the fields another noun
  touches. Not a schema dump; the model file is the source of truth for that.
- **`Hits`** is derived from who writes it, not where it sits.

---

## Ghost card

Short by design. It exists so a reader searching the name lands on the tripwire instead of
on nothing.

```markdown
# <Name>

**Status:** ghost · Confirmed
**Type:** ghost

## The reach
<Where the reader will encounter this name and what they will assume.>

## What is actually there
<The artifact that survives. The source that does not.>

## Evidence
<The specific absence that proves it.>

## What to do instead
<The live noun that does the job now — or "nothing; this capability does not exist.">
```

Never soften a ghost into a leftover. A leftover is inert. **A ghost is a name that will
mislead someone**, and that is the whole reason it is on the map.

---

## Register card

A closed list. No prose, no narrative, no card body.

**`collisions.md`** — one word, two meanings. The territory's naming traps.

| Word | Means here | Also means here | The wrong reach |
|---|---|---|---|
| `<word>` | `<sense A>` (`<path>`) | `<sense B>` (`<path>`) | `<what breaks if confused>` |

**`leftovers.md`** — honest debris. Real, inert, and dangerous only if mistaken for live.

| Artifact | Why it survives | What is live instead | Evidence |
|---|---|---|---|

Registers are the cheapest high-value cards in the map. Write them early — the collisions
surface during pass 2 and you will lose them if you wait.

---

## Rules that bind every card

- **Cite, never copy.** Paths and symbol names. No source blocks.
- **45 lines, hard cap.** Over that, it is two nouns. Reaching the cap by reflowing prose
  onto longer lines is gaming it — cut content, not line breaks.
- **A consolidated card caps at 60, and consolidation wins.** Where the derivation has
  merged several positions into one noun, splitting is forbidden — so the cap yields, not
  the merge. Never drop a real write-set entry to fit; that trades fidelity for a line
  count, which is the opposite of the trade the cap exists to make.
- **Every card carries a status, an evidence class, and a source of truth.**
- **Every step and object card carries a `Does not hit` line.** No exceptions — if nothing
  is temptingly wrong, you have not looked at the collisions register.
- **No card links to more than three others.** A card that fans out to eight is the catalog
  wearing a disguise. "Links" means markdown links to other cards. **Position references do
  not count** — a genuinely cross-cutting object may name a dozen writing positions and still
  be one card. The catalog is exempt entirely; indexing is its whole job.

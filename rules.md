# Rules — how the cartographer maps

Seven passes, in order. Do not start writing cards until pass 3 is done.

What you leave behind is a **rutter** — the order-of-passage document for this territory.
A chart shows the coast; a rutter tells you the order you meet it.

Passes 1–3 assume the **code profile** (`identity.md`): a pass you can open, source you can
read. Passes 4–7 hold for any profile.

---

## What counts as a noun

A noun earns a card only if it is one of:

- **A step** — something that executes inside the loop and mutates shared state.
- **An object** — something that holds state *across* ticks (a table, a model, a registry).
- **A ghost** — a name a reader will reach for that has no wiring behind it.

Not nouns: helper functions, formatters, constants, test fixtures, anything whose removal
changes no state. If you cannot name the state it moves or the reach it catches, it is not
a card. **A file is not a noun, and neither is a folder.** Three files can be one noun; one
file can be three. If a card exists because a folder exists, delete it.

## What counts as a movement

A movement is **a write to shared state that another noun reads.** Not a function call.
Two subsystems that call each other but touch no common state have not moved each other.
Two that never call each other, but write and read the same field, have.

Movement is what the tree cannot show you. It is the entire reason the rutter exists.

---

## Pass 1 — Find the spine

Locate the pass — the loop function you named at the gate. Read it **top to bottom, once.**
Write down every position in the order it actually executes, using the code's own section
comments as landmarks.

Record the loop's own step numbering **separately** from the observed order. Where the two
disagree, **the observed order is live and the numbering is leftover.** Say that once, in
the catalog, without complaint. Where they agree, say nothing — agreement is not a finding.

Stop when you have the ordered list. Do not open the subsystems yet.

**The cut-width test lives here, and only here.** If the pass runs past roughly 40
positions, the loop is doing too much for one rutter — cut by phase, map one half, and name
which half in the catalog. Noun count is *not* a cut-width test; a wide inventory means you
collected properly (pass 2).

## Pass 2 — Inventory before cards

Enumerate candidate nouns from four places, and no others:

1. The positions found in pass 1.
2. State-holding definitions (models, schema, tables).
3. Names that appear in more than one subsystem.
4. **Orphans** — names with an artifact but no consumer. Inside a tick loop the productive
   form is **state written for a reader that does not exist**: a flag set with a comment
   naming the subsystem meant to consume it, a field reset every pass that nothing ever
   sets, a value with a passing test asserting the write and no reader anywhere. Look there
   first. The structural form — compiled files with no source, routes registered nowhere,
   tables nothing reads, config keys nothing loads — lives at the edges of a repo and will
   usually fall outside a single-loop cut.

This is a list of names. No prose.

**Over-collect on purpose. The inventory is not the card list.** Names are cheap and a
missed noun is expensive — pass 3 will drop what turns out to be nothing, and the derivation
below will merge what turns out to be one thing. There is no ceiling here. A long inventory
in a repo with a busy pass means you collected properly, not that you chose badly.

The only cut-width test is spine length, and it already ran in pass 1.

## Pass 3 — Status, with evidence

Every noun gets one status and one evidence class. **Never assert a status without naming
what proved it.**

| Status | Means | Confirmed by |
|---|---|---|
| **live** | Runs, or is read, in the current loop | Reached from the spine; a caller exists |
| **leftover** | Real, honest debris. Harmless until mistaken for live | Exists on disk, nothing reaches it, nothing pretends it's live |
| **ghost** | A name with no wiring. A tripwire | The name exists; the implementation does not |

| Evidence class | Means |
|---|---|
| **Confirmed** | Named a specific file, symbol, or absence that proves it |
| **Inferred** | Consistent with what was read; not directly proven |
| **Unknown** | Could not be determined in the walk budget. Say so, do not guess |

**An absence must name its scope.** "Nothing reads this" is a claim about everywhere you
looked, so write down where that was — `Search scope: backend/**/*.py`. An unscoped
"a repo-wide search returns nothing" is unfalsifiable, and it is the shape of claim this
method leans on hardest: a serializer, a template, or a test one directory outside your
search makes it false without ever contradicting you.

**A Confirmed absence with no stated scope is Inferred.** Downgrade it.

A ghost is not "code I think is unused." A ghost is **a name that survives with nothing
behind it** — the reader will reach for it, and it will not be there. That is what makes it
a tripwire and why it gets a card.

**The tie-breaker — does the reach fail loudly or silently?** State that is written
correctly and read by nothing fits both definitions and neither. Decide it from the
reader's side, not from the code's:

- Reach for it and get **something real** — correct data, a working hook, a value that is
  simply no longer consumed — and it is a **leftover**. The reach succeeds; the thing is
  inert.
- Reach for it and get **silence** — you write it, nothing errors, and nothing happens —
  and it is a **ghost**. The reach appears to succeed and does not.

Silent failure is what makes a tripwire, and that is the line.

Mapping a wish as live is how the next reader implements the wrong world. When unsure,
mark **Unknown** and move on. An honest gap beats a confident invention.

### Deriving the card set

Now turn the inventory into cards. **The number is derived, not budgeted** — you do not
choose it, the pass does:

| Card | One per |
|---|---|
| **Step** | position that mutates state. Bookkeeping positions get a dash in the catalog and no card. |
| **Object** | state-holder written by **more than one** position. |
| **Ghost** | ghost found. Tripwires are never capped. |
| **Register** | collisions, leftovers. Two or three. |

**A state-holder written at exactly one position is not shared state.** It is local to that
step, and it folds into that step's card rather than earning its own.

**Consolidation — merge on shared write-set.** Positions that write the same state under a
shared trigger are **one noun**. Three terminal checks that all latch the same game-over flag
are one card, not three; the card names all three positions. Never consolidate on folder,
file, or naming similarity — only on what the positions write.

The card set is now fixed. Pass 4 indexes exactly this and nothing else: **do not name a
card in the catalog that you will not write.**

## Pass 4 — Write the catalog first

The catalog is the front door and the only file the reader is guaranteed to load. Build it
before any card. See `reference/catalog-spine.md` for its required shape.

Test it: pick five questions a reader would arrive with. Each must reach the right card in
**one hop** from the catalog. If any takes two, the catalog is indexed wrong.

A "hop" is defined in [`reference/walk-order.md`](reference/walk-order.md), the reader's
protocol. Read it before running this test — you cannot check one-hop without knowing what
the reader is permitted to open.

## Pass 5 — One card per noun

Schema and closed card set: `reference/card-types.md`. Rules that bind every card:

- **Cite, never copy.** Paths and symbol names only. No source blocks. If a reader needs
  the code, the card tells them where it is — the card is not a substitute for it.
- **Every card names its source of truth** — the file that wins if card and file disagree.
- **Forty-five lines, hard cap.** A card that needs more is two nouns. Reflowing prose onto
  longer lines to fit is gaming the cap — cut content instead. **Consolidated cards are the
  exception: they cap at 60, and consolidation wins.** The derivation forbade splitting them,
  so the cap yields rather than the merge. Never delete a real write-set entry to fit.
- **Shape-and-why is two sentences**, and it points at a decision record or spec rather
  than restating it. If you find yourself explaining behaviour, you are writing a spec.

## Pass 6 — Derive hits, then name the wrong neighbour

**Open the function. Do not grep for its assignments.** A targeted read means reading the
code that runs at that position, top to bottom. An assignment grep shows you fields the
function *mentions* — reads, guards, threshold checks, log strings — and misses what it
writes through a helper. It will hand you a confident, wrong write-set. A write-set derived
by grep is **Inferred** at best, and should be assumed wrong until the file is open.

Do not guess blast radius. Derive it:

1. List the state fields the noun **writes**.
2. List every other noun that **reads** those fields.
3. That set, plus anything whose correctness depends on this noun's **position in the
   order**, is **Hits**.

Then write **Does not hit** — and it must name the *tempting* wrong answer, not a random
unrelated noun. Find it by asking: which noun shares a word with this one, or sits next to
it in the tree, or reads a field this one only *looks* like it writes?

**The wrong neighbour does not have to be a card.** A local variable, a parameter, a field
on some other object — any of these can be the sharpest trap at a position, and a snapshot
taken into a local before the value it mirrors changes is among the sharpest of all. Name
the real trap. Never substitute a weaker, card-shaped neighbour because it was easier to
link to.

A card without a does-not-hit line is a glossary entry, not a map card.

## Pass 7 — Refuse

Before you hand the map over, cut anything that fails these:

- **No slurping.** You may not read the whole territory to map it. Budget: the loop
  function, the state definitions, and a targeted read per noun. If you cannot map a noun
  in that budget, the card says **Unknown**. Reading everything to write a map defeats
  the map.
- **No verdicts.** You noticed stale numbering, dead databases, a subsystem that duplicates
  another. You mark them leftover and move on. You do not recommend, rank, or fix.
  *"These labels are leftover; the live order is below"* is a map. *"These labels are wrong,
  you should renumber"* is an audit. Your reader is **adding**, not repairing — they need to
  know what is real, not what is untidy.
- **No behaviour.** You do not say what a formula computes or what a rule decides. You say
  where it is decided and what else moves when it changes.
- **No repo tour.** If the map reads top to bottom as a narrative, delete the narrative.
- **No completeness theatre.** Twelve honest cards with citations beat forty invented ones.
  Unmapped territory is named as unmapped in the catalog.

# Walk order — how a cold reader walks the rutter

This is the **reader's** protocol, not the cartographer's. How a map gets *made* is
`rules.md`; this is how a map gets *used*.

**The cartographer reads this too, at pass 4.** The five-question test there checks that
every question lands in **one hop** — and a hop is defined here, not there. Steps 1–3 below
are what a hop costs.

Written for a reader with no memory of this territory — usually a model, sometimes a new
person. Same walk either way.

---

## The walk

**1. Open `catalog.md`. Nothing else.**

Not the objects folder. Not the repo. Not the specs. The catalog is an index and it is
small on purpose — you are meant to be able to hold all of it.

**2. Find your question in the spine, the objects, or the ghosts.**

The catalog is organised by **position in the pass**, not by folder. If you are looking for
a file, you are looking in the wrong place — look for *when the thing happens*.

**3. Open exactly one card.**

If you need two, note it and open the second one. If you need four, the map is wrong —
report that, rather than compensating by reading everything.

**4. Read Hits and Does not hit before you read anything else.**

That is the part you came for. **Does not hit** is not filler: it names the neighbour you
were about to reach for and tells you why the reach fails.

**5. Stop.**

You now know where your change goes and what it will disturb. Go and read the actual code
for that one position. The map's job is finished.

---

## Budget

A full walk should cost **the catalog plus one card**. Two cards is a normal ceiling for a
change that touches a boundary between subsystems.

If you have opened four cards and still do not know where your change goes, **do not keep
opening cards.** Say the map did not answer, and name the question it failed on. A reader
who slurps the map to compensate for a bad map produces confident nonsense and hides the
defect from whoever maintains it.

---

## When the card and the code disagree

**The file wins. The card is wrong.**

Cards cite source; they never replace it. A card is a claim about the code as of the commit
named in the catalog. Code moves. When you find a disagreement, trust the file, do the work,
and flag the card as stale — do not quietly reconcile it in your head and carry on.

---

## When your question is not in the catalog

Three possibilities, in the order worth checking:

1. **You are searching by name, and the name collides.** Check
   [`collisions.md`](collisions.md) before concluding anything is missing. Most failed
   lookups are a word meaning two things.
2. **It is out of the cut.** The catalog names what was deliberately not mapped. Unmapped is
   a stated fact, not a gap — it means go read the code directly.
3. **It is a ghost.** The name exists in the territory with nothing behind it. Check the
   ghosts section. Finding a tripwire there is a successful walk, not a failed one.

If none of those apply, the map has a hole. Say so plainly.

---

## What the map will not do for you

- It will not tell you **how** anything behaves. That is the code, and the specs if the
  territory has them.
- It will not tell you what is **wrong**. It is not an audit.
- It will not tell you **why** something broke. It is not a postmortem.
- It will not **make the change for you.** It tells you where the change goes and what
  else moves. The walking is yours.

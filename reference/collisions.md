# Collisions — one word, two meanings

Naming collisions in this territory, known **before** the walk. Carry them in; the walk will
find more.

This file is what the cartographer knows. The rutter it produces emits its own
`registers/collisions.md` — this one seeds that one, it is not the same file.

**Why this matters more than it looks.** Most failed lookups in a mapped territory are not
missing cards. They are a reader searching a word that means two things and landing on the
wrong one — confidently, with no error to warn them. A collision caught here is a class of
mistake that never happens.

---

## Polis — confirmed collisions

| Word | Means | Also means | The wrong reach |
|---|---|---|---|
| **Event** | A game event — `engine/events/` | `CycleEvent`, a result record built from an `ActionResult` (`cycle/runner.py:30`) | You go to change "how events work" and edit the result serialiser, or the reverse. Unrelated objects, one word. |
| **Action** | A faction action — `engine/actions/` | A mayor action — `engine/mayor/`, submitted by the player before the pass | Two subsystems, two positions in the pass. Changing one does not touch the other. |
| **Audience** | An LLM faction audience — `engine/llm/audiences.py` | Not a crowd, not spectators, not the public | See `Planning/decisions/2026-05-29-audience-term-clarity.md` — the term was disambiguated deliberately, which means the confusion is documented as real. |
| **Project** | An in-game project — `engine/projects/` | The `Planning/` sense — a build plan for the repo itself | A reader asked to "check the projects" can land in either world. |
| **Public** | A needs scale — `public-needs_spec`, and a step in the pass | Visibility, as in public/private | "Public" in this repo is a game entity, not an access modifier. |
| **Cycle** | A unit of game time, and an integer counter incremented near the end of the pass | The `engine/cycle/` folder, which holds the runner | "Change the cycle" is ambiguous between the number, the pass, and the folder. |

**Evidence:** all six Confirmed by direct read except **Cycle**, which is Confirmed as a
folder and a counter, Inferred as a source of reader confusion.

---

## The lineage collision

This repo was `city_sim` before it was `Polis`, and both names are live on disk at once —
`backend/city_sim.db`, `backend/city_sim_Old.db`, `backend/polis.db`, and a second
`polis.db` at the root.

That is a **leftover**, not a collision, and it belongs in the leftovers register. It is
noted here because a reader searching `city_sim` will find real files and reasonably
conclude they are looking at a live system. They are not.

---

## Re-verify every seeded row before emitting it

**A seed is a lead, not a finding.** These rows were written before this walk and the code
has moved since. Re-check each one against source during pass 2, and treat the result as
the truth:

- **Still true** — emit it to the register, upgraded to Confirmed with the citation you just
  checked.
- **Now false** — drop it. A collision that no longer exists is worse than a missing one; it
  sends the reader somewhere that is no longer wrong.
- **True and bigger than written** — emit the wider version. Seeds are usually undersold.

A row graded **Inferred** here may never be emitted as Confirmed without a fresh citation.
If the walk did not reach it, it stays Inferred in the register and says why.

## Adding to this file

A collision earns a row when **both** senses are reachable by the same search and the wrong
one is plausible. Two things sharing a word in different repos is not a collision. Two
things sharing a word where a reader would act on the wrong one is.

Carry the format: word, both senses with paths, and the specific wrong action the confusion
produces. A collision row that does not name the consequence is a glossary entry.

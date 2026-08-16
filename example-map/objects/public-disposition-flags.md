# _content_bonus / _angry_penalty

**Status:** ghost · Confirmed
**Type:** ghost

## The reach
You are changing what the people's mood does to the city. You open
`backend/engine/special/public.py` and find, at lines 75–80, exactly what you were looking
for: a content public sets `public._content_bonus = True`, an angry one sets
`public._angry_penalty = True`, and both carry the comment "flagged for runner". You
conclude the runner consumes these and you only need to change the threshold or the value.

## What is actually there
The two assignments and the two comments. The runner does not consume them. Nothing does.
Their neighbouring comments — "Aggressive factions lose cover; decrees cost fewer AP" and
"Aggressive factions gain chaos cover" — describe effects that have no implementation.

The flags are set at position 21, which is after position 7 where faction actions resolve,
so even a consumer added on the Faction side would be reading them a pass late.

## Evidence
Search scope: `**/*.py`, `*.vue`, `*.js` for `_content_bonus` and
`_angry_penalty`. Two hits total, both writes: `engine/special/public.py:77` and `:80`. No
reader in the engine, the API, the serializer, the frontend, or the test suite.

## What to do instead
Disposition does have live consequences, through different fields: `public.disposition` and
`public.support` are read at position 22 by the terminal checks
(`special/removal.py`, `special/ostracism.py:115-119`), and `public.support` reaches NPC
behaviour as a confidence band on the next pass. See [the-public](the-public.md).

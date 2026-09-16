# Ranking Inputs

**Status: Concept / Research.** No ranking function has been designed, and no weighting
among inputs has been chosen.

## Why ranking is different from the rest of this repository

Indexing, normalization and streaming are engineering problems with correct answers.
Ranking is not. Any ordering of reactions is a statement about which deserve attention,
and on a launch protocol that statement has direct financial consequence for the
reactions at the top.

This document argues that ranking must be **transparent and descriptive**, and sets out
what that rules out.

## Descriptive versus promotional

| Descriptive ordering | Promotional ordering |
| :--- | :--- |
| Sorts by a stated, checkable quantity | Sorts by an undisclosed or composite quantity |
| Reader can verify membership and position | Reader must trust the ordering |
| "Highest volume over 24h, volume shown" | "Trending", with no definition |
| No reaction can be inserted by decision | Placement can be granted |

The dividing line is not intent. It is **whether a reader can reconstruct the ordering
from published rules and public data.** An ordering nobody can check is promotional in
effect, whatever it was meant to be.

## Requirements

### 1. The ordering key is published and singular

Each surface sorts by one named quantity with a stated derivation, not a blend.

A composite makes every reason invisible: a reaction ranked third cannot be explained
except by "the formula said so". It also becomes a specification for gaming, without
letting a reader see which component was gamed.

> This mirrors the rejection of a composite score in the Signal repository. Same
> reasoning, different surface.

### 2. The value is displayed next to the entry

If a surface sorts by 24h volume, show the volume. This makes a badly-chosen key
immediately visible - a list where the top entry's number is absurd is self-correcting
in a way a bare list is not.

### 3. Membership is a predicate, not a selection

"Reactions with reserves >= X% of threshold" is a predicate. "Reactions we consider
notable" is a selection. Only predicates belong here.

### 4. No paid, granted, or manually inserted placement

Nothing in a surface may be placed by decision. If that ever changes, the surface stops
being descriptive and must be labelled as advertising - visibly, not in a footnote.

### 5. Windows are stated in blocks

A window in blocks is reproducible; a window in wall-clock time depends on block
production. Every windowed surface publishes its block range.

## Candidate inputs and what each rewards

Every input rewards something, and whatever it rewards, someone will produce.

| Input | Rewards | Cheapest way to game it |
| :--- | :--- | :--- |
| Reserve progress vs threshold | Genuine accumulation | Self-buying - costs real USDC, the most expensive to fake |
| Rate of reserve accumulation | Recent momentum | Timed self-buying against a short window |
| Traded volume over window | Activity | Wash trading - costs only fees |
| Distinct holder count | Distribution breadth | Dust airdrops to many addresses |
| Directional trade composition | Buy-side skew | Self-trading in one direction |
| Activity vs own baseline | Relative change | Suppress baseline, then spike - cheapest of all on a small reaction |

**Reserve progress is the most robust input** because faking it requires spending real
USDC into the reaction, and that spending is itself the thing being measured. Relative
change is the least robust, because the denominator is small and controllable.

> **Open question.** Whether inputs that are cheap to fake should be excluded outright,
> or included with the caveat published. Excluding them removes useful surfaces;
> including them ranks manufactured activity alongside real activity.

## The cold-start asymmetry

Absolute inputs rank new reactions last, because they genuinely have less of
everything. Relative inputs favour new reactions, because small denominators move
easily.

There is no neutral choice. `New Reactions` exists as an explicit recency surface
precisely so that other surfaces do not have to compensate - keeping the compensation
in one clearly-labelled place rather than smuggled into every ordering.

## Minimum floors

Most gaming vectors above are cheap because the base is small. A minimum absolute floor
- minimum reserves, minimum distinct traders, minimum volume - raises the cost of every
one of them.

It also excludes genuinely new reactions from every surface except `New Reactions`,
which is the cold-start problem restated.

> **Under research.** Whether floors should exist, and at what level. A floor is itself
> a threshold decision of exactly the kind this repository has avoided making.

## What is not proposed

- No composite score.
- No weighting between inputs.
- No editorial adjustment.
- No personalisation. A per-viewer ordering cannot be checked by anyone else, which
  fails the transparency requirement by construction.
- No "quality" or "safety" ordering. Those are judgements, not observations.

## Related documents

- [Discovery model](../docs/discovery-model.md)
- [Event flow](../docs/event-flow.md)
- [RFC 0001 - Live feed](../rfcs/0001-live-feed.md)

# Live Reactor

**Status: Concept / Research**

This repository documents a design direction. Nothing described here is implemented,
deployed, or scheduled. No ranking or notification logic exists today.

## Overview

Live Reactor would be a discovery and monitoring layer for active Rearctor reactions.
A reaction moves through a defined lifecycle - Spark, Charging, Critical Mass,
Ignition, Expansion - and that progression is observable onchain as it happens. Live
Reactor would present that progression as it unfolds, rather than as a retrospective.

## Discovery surfaces

Candidate surfaces under consideration:

| Surface | Concept |
| :--- | :--- |
| **New Reactions** | Recently deployed reactions. |
| **Charging Fast** | Reactions accumulating reserves at an elevated rate. |
| **Near Critical Mass** | Reactions approaching the 5,042 USDC threshold. |
| **Recently Ignited** | Reactions that have graduated and migrated to Uniswap V4. |
| **Highest Volume** | Reactions by traded volume over a window. |
| **Most Holders** | Reactions by count of distinct holding addresses. |
| **Trending on Arc** | Reactions with elevated activity relative to their own baseline. |

These are candidate surfaces, not a committed product surface.

## Live telemetry

Because a reaction's progress toward ignition is denominated in USDC and its threshold
is a constant, progress is directly observable while trading is underway. Telemetry
under consideration includes reserve accumulation against the threshold, trade
composition, holder count, and the rate of change in each.

Ignition is a single transaction. A monitoring layer would need to treat it as an
event boundary rather than a gradual transition: the same transaction that crosses the
threshold migrates liquidity to Uniswap V4.

## Watchlists and alerts

Potential user-facing features:

- **Watchlists** - a saved set of reactions to follow.
- **Filters** - narrowing by lifecycle stage, activity, or holder characteristics.
- **Saved reactions** - persistence across sessions.
- **Ignition alerts** - notification on threshold events.
- **Real-time telemetry** - live updates while a reaction is trading.

Example notifications under consideration:

```
Reaction reached 90% critical mass.
Ignition detected.
New reaction accelerating.
```

These are illustrative message forms. No notification system exists.

## Potential ranking inputs

If surfaces are ordered at all, the inputs would need to be explicit. Candidates:

- reserve progress against the ignition threshold
- rate of reserve accumulation
- traded volume over a defined window
- count of distinct holders
- directional trade composition
- activity relative to a reaction's own baseline

No ranking function has been designed, and no weighting among these inputs has been
chosen.

## Open research questions

- Over what windows should "fast" and "trending" be measured so that the signal is not
  dominated by a reaction's first minutes?
- How should surfaces avoid becoming a promotional ranking rather than a descriptive
  one?
- How should ordering handle reactions that are structurally new and therefore have
  little history?
- What delivery guarantees are appropriate for ignition alerts, given that ignition is
  a single atomic transaction?
- How should discovery present post-ignition reactions, whose market has moved from
  the curve to Uniswap V4?

## Links

[Website](https://rearctor.io) · [Docs](https://rearctor.io/docs) · [GitHub](https://github.com/Rearctor) · [X](https://x.com/JoinRearctor) · [Telegram](https://t.me/rearctor)

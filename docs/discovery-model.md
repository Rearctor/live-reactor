# Discovery Model

**Status: Concept / Research.** Nothing described here is implemented, deployed, or
scheduled. No ranking or notification logic exists.

## What a surface is

A discovery surface is a **query with a stated definition**, not a curated list. Each
candidate surface below is specified as: the set it selects, the ordering within it,
and the data it needs.

The distinction matters because a surface whose definition is unpublished is editorial
selection wearing a query's clothes. See
[ranking-inputs.md](../research/ranking-inputs.md).

## Candidate surfaces

### New Reactions

- **Set:** reactions deployed within a recent window.
- **Order:** deployment block, descending.
- **Inputs:** deployment events.
- **Notes:** the only surface with no judgement in it. Recency is unambiguous.
- **Unresolved:** window length. Also whether reactions with zero trades should appear
  at all - a deployed reaction nobody has traded is real but arguably not yet a market.

### Charging Fast

- **Set:** pre-ignition reactions whose reserve accumulation rate exceeds some bound.
- **Order:** rate, descending.
- **Inputs:** reserve deltas over a window, block ranges.
- **Notes:** shares every weakness of the `ACCELERATING` signal - window sensitivity,
  low-base distortion, self-trade manipulability.
- **Unresolved:** window, rate definition, and whether a minimum absolute reserve base
  is required before a reaction can qualify. Without a base, this surface is dominated
  by reactions that went from near-zero to slightly-above-zero.

### Near Critical Mass

- **Set:** pre-ignition reactions within some distance of 5,042 USDC.
- **Order:** distance, ascending.
- **Inputs:** current reserves, the threshold constant.
- **Notes:** the best-defined surface after New Reactions. Distance to a constant is
  unambiguous.
- **Unresolved:** the proximity cutoff. Also whether reactions that reached the band and
  retreated should be distinguished from those approaching it - the positional view
  cannot tell them apart.

### Recently Ignited

- **Set:** reactions whose ignition transaction occurred within a recent window.
- **Order:** ignition block, descending.
- **Inputs:** ignition events, migrated supply, pool reference.
- **Notes:** unambiguous membership. Ignition is a discrete event.
- **Unresolved:** window length only.

### Highest Volume

- **Set:** reactions with trading volume over a window.
- **Order:** volume, descending.
- **Inputs:** curve trades and pool trades, both USDC-denominated.
- **Notes:** **pre- and post-ignition volume are not the same quantity.** Curve volume
  is against a bonding curve with no external liquidity; pool volume is against a
  Uniswap V4 position with outside participants. Ranking them in one list compares
  unlike things.
- **Unresolved:** whether to split into separate surfaces by phase, and how to treat
  wash trading - which is cheap and directly inflates this surface.

### Most Holders

- **Set:** reactions ordered by distinct holding addresses.
- **Order:** count, descending.
- **Inputs:** transfer events, balance state, exclusion set.
- **Notes:** addresses are not people. Inherits the counting problem described in the
  Signal repository's creator-attribution research.
- **Unresolved:** the exclusion set (contracts, the migrated pool position, routers),
  and whether dust balances count. Airdropping dust to many addresses is a cheap way to
  top this surface.

### Trending on Arc

- **Set:** reactions whose activity is elevated relative to their own recent baseline.
- **Order:** relative change, descending.
- **Inputs:** activity metric over a recent window and a baseline window.
- **Notes:** the only surface that is explicitly relative, which makes it scale-neutral
  and therefore favourable to small reactions - for better and worse.
- **Unresolved:** which activity metric, both window lengths, and a minimum absolute
  floor. This is the surface most exposed to manufactured activity, because moving a
  small baseline is cheap.

## Cross-cutting problems

### The lifecycle boundary

Four surfaces are pre-ignition only, one is post-ignition only, and two span both.
A reaction that ignites mid-window changes category during the window.

> **Open question.** Whether spanning surfaces should recompute from the phase
> boundary, carry across it, or be split into phase-specific surfaces. Splitting is
> the most honest and produces more surfaces than a reader wants.

### Cold start

A newly deployed reaction has no history. Every relative surface is undefined for it,
and every absolute surface ranks it last.

This is not a flaw to be corrected: a reaction with no activity genuinely has no
activity. But it means discovery structurally favours reactions that already have
attention, and `New Reactions` is the only counterweight.

### Observability of membership

A reader should be able to answer "why is this reaction here?" from published
definitions alone. That requires each surface to publish its selection predicate,
ordering key, and window - not just its results.

> **Under research.** Whether surfaces should expose the ordering key's value next to
> each entry. More transparent, and it makes a badly-chosen key immediately obvious -
> which is an argument for it, not against.

## Related documents

- [Event flow](event-flow.md)
- [Ranking inputs](../research/ranking-inputs.md)
- [RFC 0001 - Live feed](../rfcs/0001-live-feed.md)

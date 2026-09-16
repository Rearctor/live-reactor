# Live Reactor - v0 Design Specification

## Status

**Concept / Research.** This is a draft design target, not a production specification.

Nothing described here is implemented, deployed, scheduled, or audited. No indexer
exists. No stream exists. No alert has been delivered. **No ranking function has been
designed and no confirmation depth has been chosen.** There is no commitment that this
design will ship.

Builds on, and does not replace:

- [docs/discovery-model.md](../discovery-model.md)
- [docs/event-flow.md](../event-flow.md)
- [rfcs/0001-live-feed.md](../../rfcs/0001-live-feed.md)
- [research/ranking-inputs.md](../../research/ranking-inputs.md)
- [specs/event-envelope.schema.json](../../specs/event-envelope.schema.json)
- [specs/alert.schema.json](../../specs/alert.schema.json)

Every numeric figure here that is not a Rearctor protocol constant is illustrative only.
Not protocol defaults or committed parameters.

## Scope

### Proposed for v0

- A normalized event envelope with stable identity and explicit finality.
- An ingestion path: index, normalize, enrich, stream.
- Discovery surfaces defined as published predicates with a single stated ordering key.
- An alert object carrying its own threshold inline.
- Retraction semantics for reorged events.

### Under research

- Confirmation depth before an event is emitted as final. See Issue #1.
- Ranking inputs and whether minimum floors should exist. See Issue #2.
- Whether enrichment belongs in the envelope or is a separate lookup.
- Replay horizon and behaviour when a consumer requests below it.
- Whether per-reaction sub-streams are offered.

### Non-goals

- Any ordering that cannot be reconstructed from published rules and public data.
- Paid, granted, or manually inserted placement in any surface.
- Personalised ordering. A per-viewer ordering cannot be checked by anyone else.
- Prediction of ignition or outcome.
- Quality, safety or risk ordering. Those are judgements, not observations.
- Composite ranking scores.

## Event ingestion

Durable capture of chain logs, reorg-aware.

**Reorg handling is the defining requirement.** An ignition event on an orphaned block
never happened. A feed that emits it and never retracts it has published a false fact
about the most consequential event in the lifecycle.

**Unresolved:** confirmation depth. Emitting at tip is what "live" implies and is
occasionally wrong; waiting is correct and undermines the premise. The answer likely
differs by event kind - a retracted trade is a minor correction, a retracted ignition is
not. See Issue #1.

## Normalization

Lossless and non-interpretive. Stable event identity from chain position; base-unit
integer amounts; explicit denomination; resolved reaction identity.

No thresholds, no derived values, no labels at this stage.

## Enrichment

Attaches context a consumer would otherwise look up: lifecycle stage at the event,
reserves after the event, distance to threshold, pool reference post-ignition.

Enrichment adds **facts, not judgements**. The boundary erodes easily: "distance to
threshold" is a fact; "close to threshold" is a judgement and belongs at the alert stage.

**Unresolved:** whether enrichment is embedded in the envelope or fetched separately.
Embedding makes each event self-contained and duplicates state that may later be
corrected.

## Event envelope

Draft format: [specs/event-envelope.schema.json](../../specs/event-envelope.schema.json).
v0 does not change it.

| Field | Role |
| :--- | :--- |
| `event_id` | Stable identity, `block_hash:log_index`; used for dedup and retraction |
| `event_type` | Closed set; determines payload shape |
| `reaction` | Reaction identity and stage at event |
| `chain_reference` | Block, transaction, log index - sufficient for verification |
| `finality` | `pending`, `final` or `retracted`, with confirmations |
| `payload` | Event-specific fields; integer amounts as decimal strings |
| `enrichment` | Derived context; facts only |

A reorged event and its replacement have **different identities**, because the block hash
changes. A retraction must therefore name the original explicitly.

## Streaming

| Requirement | v0 position |
| :--- | :--- |
| Ordering | By block height then log index. Chain order is the only order that is not a choice |
| Delivery | At-least-once, with stable identity for consumer-side deduplication |
| Retraction | Required, carrying the identity of the retracted event |
| Replay | From a block height, so a consumer can rebuild state |
| Backpressure | A drop must be observable as a gap, never silent |

A consumer cannot distinguish "nothing happened" from "I missed it" unless gaps are
explicit. Silent dropping is treated as a defect, not a degradation mode.

## Discovery surfaces

Each surface is a **query with a published definition**: a selection predicate, a single
ordering key, and a window in blocks. A surface whose definition is unpublished is
editorial selection wearing a query's clothes.

| Surface | Set | Ordering key |
| :--- | :--- | :--- |
| New Reactions | Deployed within a window | Deployment block, descending |
| Charging Fast | Pre-ignition, accumulation rate above a bound | Rate, descending |
| Near Critical Mass | Pre-ignition, within a distance of 5,042 USDC | Distance, ascending |
| Recently Ignited | Ignited within a window | Ignition block, descending |
| Highest Volume | Traded within a window | Volume, descending |
| Most Holders | All | Distinct holding addresses, descending |
| Trending on Arc | Activity elevated against own baseline | Relative change, descending |

**Unresolved for every surface:** window lengths, cutoffs, and whether minimum absolute
floors exist. Also whether surfaces spanning the ignition boundary should be split by
phase - curve volume and pool volume are not the same quantity.

## Alert lifecycle

```
  condition observed ---> alert emitted ---> superseded or retracted
```

Draft format: [specs/alert.schema.json](../../specs/alert.schema.json).

Each alert carries its **threshold inline** rather than referencing a global
configuration, so a historical alert remains interpretable after thresholds change.

Alert types are descriptive only: `threshold_progress`, `ignition_detected`,
`activity_elevated`, `reaction_deployed`. **No alert type may assert a future outcome.**

## Finality

Finality is a **first-class field**, not a producer-side decision imposed on everyone:

```
  emit at tip        pending     live view, marked provisional
  confirmations++    pending     same event_id, updated
  depth reached      final       durable view, alerting
  block orphaned     retracted   removed from both
```

A latency-tolerant consumer filters to `final` and never sees a false event. A
latency-sensitive consumer accepts `pending` and handles retraction.

**Cost:** every consumer must implement retraction. One that ignores `retracted` is worse
off than under a conservative feed.

**Unresolved:** whether `pending` is emitted by default or is explicit opt-in. Opt-in is
safer and makes the live product opt-in too. See Issue #1.

## Retractions

- A retraction names the original `alert_id` or `event_id` via `supersedes`.
- Retraction is required for reorged events; it is not optional behaviour.
- A consumer that has acted on a retracted alert must be able to identify what it acted
  on.

## Ranking inputs

**No ranking function has been designed.** Candidate inputs, with what each rewards:

| Input | Rewards | Cheapest way to game |
| :--- | :--- | :--- |
| Reserve progress vs threshold | Genuine accumulation | Self-buying, costing real USDC |
| Reserve velocity | Recent momentum | Timed self-buying against a short window |
| Traded volume | Activity | Wash trading, costing only fees |
| Distinct holder count | Distribution breadth | Dust airdrops |
| Directional activity | Buy-side skew | One-sided self-trading |
| Baseline-relative activity | Relative change | Suppress baseline then spike |

**Reserve progress is the most robust** because faking it requires spending real USDC
into the reaction, and that spending is the thing measured. Baseline-relative is the
least robust.

Requirements binding on any future ordering, from
[research/ranking-inputs.md](../../research/ranking-inputs.md):

1. One published, singular ordering key per surface - never a blend.
2. The ordering value displayed next to each entry.
3. Membership by predicate, not selection.
4. No paid, granted or manually inserted placement.
5. Windows stated in blocks.

See Issue #2.

## Watchlists

Per-viewer saved sets of reactions, filters and alert subscriptions.

Watchlists are a **consumer-side concern**: they select and subscribe, they never
reorder a surface. Personalised ordering is a non-goal because it cannot be checked by
anyone else.

## Failure modes

| Failure | Consequence | Mitigation shape |
| :--- | :--- | :--- |
| Index lag | Surfaces stale, alerts late | Publish index height with every surface |
| Reorg after emit | False event published | Retraction with original identity |
| Duplicate delivery | Double-counted alerts | Stable identity, consumer-side dedup |
| Enrichment drift | Embedded context contradicts current state | Version enrichment, or do not embed |
| Threshold change | Historical alerts uninterpretable | Threshold recorded inline in the alert |
| Silent drop | Consumer cannot detect the gap | Gaps must be explicit |

## Open design issues

- [#1 - Specify finality policy for ignition alerts](https://github.com/Rearctor/live-reactor/issues/1)
- [#2 - Define transparent ranking inputs for discovery surfaces](https://github.com/Rearctor/live-reactor/issues/2)

## Related documents

- [Discovery model](../discovery-model.md)
- [Event flow](../event-flow.md)
- [RFC 0001 - Live feed](../../rfcs/0001-live-feed.md)
- [Ranking inputs](../../research/ranking-inputs.md)
- [Implementation plan](implementation-plan.md)

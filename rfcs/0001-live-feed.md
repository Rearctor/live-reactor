# RFC 0001 - Live Feed

- **Status:** Draft / Research. Not accepted, not scheduled, not implemented.
- **Scope:** Delivery semantics for a stream of normalized reaction events.

## Problem

"Live" and "correct" pull against each other on a chain that reorganises.

A reaction's progress toward 5,042 USDC is observable in real time, and ignition is a
single transaction. A feed that waits for deep finality is accurate and describes the
past. A feed that emits at tip is responsive and will sometimes publish events that
never happened.

The question is not which to choose. It is how to expose the trade-off so a consumer
can choose.

## Proposal: finality as a first-class field

Rather than picking a confirmation depth, every event carries its finality posture and
consumers filter on it.

```
  producer                                consumer
  --------                                --------
  emit @ tip        --- pending  --->     live view, marked provisional
  confirmations++   --- pending  --->     (same event_id, updated)
  depth reached     --- final    --->     durable view, alerting
  block orphaned    --- retracted --->    remove from both
```

Properties:

- a latency-tolerant consumer filters to `final` and never sees a false event;
- a latency-sensitive consumer accepts `pending` and handles retraction;
- the producer does not decide for either.

**Cost:** every consumer must implement retraction. A consumer that ignores
`retracted` is worse off than under a conservative feed, because it will hold a false
event permanently.

> **Open question.** Should `pending` events be emitted at all by default, or should
> accepting them be explicit opt-in? Opt-in is safer and makes the "live" product
> opt-in too.

## Event identity and idempotency

`event_id` is `block_hash:log_index`.

- Stable across redelivery.
- Distinct across reorgs, because the block hash changes.
- Verifiable independently - it is derived from chain data, not assigned.

Consequence: a reorged event and its replacement have **different** identities. A
retraction must therefore name the original explicitly rather than relying on the
replacement to overwrite it.

Delivery is at-least-once. Exactly-once is not achievable across a network boundary;
stable identity plus consumer-side deduplication achieves the same outcome.

## Ordering

Ordering is by `(block_height, log_index)`. Chain order is the only ordering that is
not an editorial choice.

Explicitly **not** ordered by: emission time, economic significance, or reaction.

> **Under research.** Whether per-reaction sub-streams should be offered. A consumer
> watching one reaction currently filters the global stream, which is wasteful. Per
> reaction streams multiply connections and complicate global ordering guarantees.

## Replay

A consumer must be able to request "everything from block N" and rebuild state without
external help. This implies retention, and retention has a horizon.

> **Open question.** How far back replay extends, and what a consumer does when it
> requests a block older than the horizon. Silently returning a truncated stream is the
> worst option; an explicit error that names the earliest available block is better.

## Backpressure

A slow consumer must not degrade the feed for others, and must not silently miss
events. Options: drop with a gap marker; buffer to a limit then disconnect; or require
the consumer to acknowledge.

Dropping silently is unacceptable - a consumer cannot distinguish "nothing happened"
from "I missed it". Any drop must be observable as a gap.

## What the feed does not do

- **It does not rank.** Ordering is chain order. Discovery surfaces are built on top.
- **It does not evaluate thresholds.** Alerts are a separate concern.
- **It does not predict.** No event type asserts a future outcome.
- **It does not aggregate.** Windowed metrics are computed by consumers or by a
  downstream stage, not embedded in the event stream.

## Alternatives considered

**Polling a state API.** Simpler, no retraction protocol - each poll returns current
truth. Loses per-event granularity and cannot support "ignition detected" without the
consumer polling fast enough to be its own problem.

**Final-only feed.** No `pending`, no retraction. Substantially simpler and correct by
construction. Rejected for now only because it cannot support live monitoring, which is
this repository's premise - but it remains the safer design and should not be dismissed.

## Related documents

- [Event flow](../docs/event-flow.md)
- [Draft event envelope schema](../specs/event-envelope.schema.json)
- [Draft alert schema](../specs/alert.schema.json)
- [Ranking inputs](../research/ranking-inputs.md)

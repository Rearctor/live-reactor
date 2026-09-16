# Event Flow

**Status: Concept / Research.** Conceptual only. No pipeline is implemented, deployed,
or scheduled. No infrastructure choices are made here.

## Stages

```
  onchain event
       |
       v
    index          durable capture, reorg-aware
       |
       v
  normalize        canonical envelope, one shape per event kind
       |
       v
    enrich         reaction context attached, still non-interpretive
       |
       v
    stream         ordered delivery to consumers
       |
       v
  UI / alert       presentation and threshold evaluation
```

Interpretation happens only at the last stage. Everything before it is capture and
shaping, so that a wrong threshold can be corrected without reindexing.

## Index

Captures logs durably and handles reorgs explicitly.

**Reorg handling is the defining requirement.** An ignition event on an orphaned block
never happened. A feed that emits it and never retracts it has published a false fact
about the most significant event in the lifecycle.

The envelope therefore carries a finality marker, and the stream must support
retraction.

> **Open question.** How many confirmations before an event is emitted as `final`?
> Emitting at tip is what "live" implies and is occasionally wrong. Waiting is correct
> and undermines the premise of the product. The answer likely differs by event kind:
> a trade emitted early and retracted is a minor correction, an ignition emitted early
> and retracted is not.

## Normalize

Raw logs become envelopes with stable identity, typed base-unit integer amounts,
explicit denomination, resolved reaction identity, and a chain reference.

Normalization is lossless and non-interpretive. No thresholds, no derived values, no
labels.

**Amounts stay integers.** A float introduced here makes two consumers of the same
stream disagree.

## Enrich

Attaches context a consumer would otherwise have to look up: lifecycle stage at the
event, reserves after the event, distance to threshold, pool reference post-ignition.

Enrichment is still non-interpretive - it adds facts, not judgements. The boundary is
easy to erode: "distance to threshold" is a fact; "close to threshold" is a judgement
and belongs at the alert stage.

> **Under research.** Whether enrichment should be part of the envelope or a separate
> lookup. Embedding makes each event self-contained and larger, and duplicates state
> that may be corrected later.

## Stream

Ordered delivery. Requirements:

- **Ordering** by block height then log index. Chain order is the only order that is
  not a choice.
- **Retraction** for reorged events, carrying the identity of the retracted event.
- **Replay** from a block height, so a consumer can rebuild state.
- **At-least-once with idempotency.** Exactly-once is not achievable; a stable event
  identity lets consumers deduplicate.

## UI / alert

The only interpretive stage. Applies thresholds, produces alerts, renders surfaces.

Because it is the only interpretive stage, it is also the only stage that needs to
change when a threshold is revised - no reindexing, no stream rewrite.

## Failure modes

| Failure | Consequence | Mitigation shape |
| :--- | :--- | :--- |
| Index lag | Surfaces stale; alerts late | Publish index height alongside every surface |
| Reorg after emit | False event published | Retraction with original identity |
| Duplicate delivery | Double-counted alerts | Stable event identity, consumer-side dedup |
| Enrichment drift | Embedded context contradicts current state | Version enrichment, or do not embed |
| Threshold change | Historical alerts no longer reproducible | Record threshold applied in the alert itself |

The last row is why the draft alert schema carries its threshold inline rather than
referencing a global configuration.

## What this flow does not do

- It does not rank. Ordering within a surface is a separate concern.
- It does not predict. No stage emits a statement about the future.
- It does not deduplicate semantically. Two economically similar trades are two events.

## Related documents

- [Discovery model](discovery-model.md)
- [Draft event envelope schema](../specs/event-envelope.schema.json)
- [Draft alert schema](../specs/alert.schema.json)
- [RFC 0001 - Live feed](../rfcs/0001-live-feed.md)

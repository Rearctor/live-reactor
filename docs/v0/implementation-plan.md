# Live Reactor - v0 Implementation Plan

## Status

**Concept / Research.** This plan describes work that has not started. No phase is
complete, in progress, or scheduled.

There are **no dates** in this document. Phase ordering is a dependency order, not a
timeline. No indexer exists, no stream exists, no alert has been delivered, no ranking
function has been designed, and no confirmation depth has been chosen.

Companion document: [v0 design specification](design-spec.md).

## Phase A - Canonical event envelope

**Objective.** An envelope format stable enough that consumers can be written against it
before an indexer exists.

**Inputs.**
- [specs/event-envelope.schema.json](../../specs/event-envelope.schema.json)
- [docs/v0/design-spec.md](design-spec.md)

**Deliverables.**
- A per-event-type payload schema, replacing the currently open `payload` object.
- A decision on whether enrichment is embedded or fetched separately.
- Event identity and retraction rules specified precisely enough to implement.

**Blockers.**
- Enrichment placement undecided.
- Issue #1 open: finality semantics affect what the envelope must carry.

**Exit criteria.**
- Every event type has a payload schema, and the example validates against it.
- Identity is derivable from chain data alone, with no assigned identifiers.
- A reorged event and its replacement are distinguishable without external state.

## Phase B - Indexer model

**Objective.** A reorg-aware capture model, specified before implementation.

**Inputs.** Phase A output; [docs/event-flow.md](../event-flow.md).

**Deliverables.**
- A capture model with explicit finality tracking per event.
- A reorg detection and retraction procedure.
- A replay model, including behaviour when a consumer requests below the retention
  horizon.

**Blockers.**
- Phase A incomplete.
- Confirmation depth undecided (Issue #1).
- Replay horizon undecided.

**Exit criteria.**
- Every captured event carries a finality state that can transition to `retracted`.
- Requesting a block below the horizon produces an explicit error naming the earliest
  available block, never a silently truncated stream.
- No path exists by which an orphaned event remains observable as `final`.

## Phase C - Enrichment pipeline

**Objective.** Attach derived context without crossing into interpretation.

**Inputs.** Phases A-B.

**Deliverables.**
- An enrichment definition per event type.
- A written boundary test distinguishing facts from judgements, with worked examples.
- A versioning approach if enrichment is embedded.

**Blockers.** Phases A-B incomplete. Enrichment placement undecided.

**Exit criteria.**
- No enrichment field encodes a threshold, label, or judgement.
- Enrichment is recomputable from normalized records plus chain state at that height.
- Where enrichment can contradict later state, the contradiction is detectable.

## Phase D - Live stream prototype

**Objective.** A non-production stream exercising ordering, delivery and retraction.

**Inputs.** Phases A-C.

**Deliverables.**
- A prototype stream with ordering, at-least-once delivery and retraction.
- Backpressure behaviour where a gap is explicit.
- A consumer reference showing correct handling of `pending` to `final` to `retracted`.

**Blockers.** Phases A-C incomplete. Whether `pending` is default or opt-in is undecided.

**Exit criteria.**
- A consumer ignoring retractions is demonstrably worse off than one honouring them,
  and that fact is documented rather than hidden.
- Gaps are always observable. No silent drop path exists.
- Ordering is by chain position only, with no reordering by significance or recency.

**Explicitly not an exit criterion:** throughput, latency targets, or production
readiness.

## Phase E - Alert semantics

**Objective.** Alert emission, supersession and retraction, with thresholds recorded
inline.

**Inputs.** Phase D; [specs/alert.schema.json](../../specs/alert.schema.json).

**Deliverables.**
- An alert emitter conforming to the draft schema.
- Supersession rules using `supersedes`.
- A deduplication model so a consumer is not alerted twice for one condition.

**Blockers.**
- Phase D incomplete.
- **No threshold has been agreed for any alert type.** Issue #1 open.

**Exit criteria.**
- Every alert carries the threshold that produced it, so it remains interpretable after
  thresholds change.
- No alert type asserts a future outcome.
- An alert emitted on a later-orphaned block is retracted, and the retraction identifies
  what a consumer may have acted on.

## Phase F - Discovery ordering research

**Objective.** Determine whether any of the candidate ranking inputs is defensible, and
under what constraints.

**Inputs.** Phases A-E; [research/ranking-inputs.md](../../research/ranking-inputs.md).

**Deliverables.**
- A per-input manipulation cost assessment.
- A decision on whether minimum absolute floors exist, and the cold-start consequence of
  each answer.
- A decision on whether phase-spanning surfaces are split.
- A transparency check: can a reader reconstruct each surface from published rules and
  public data alone?

**Blockers.** Phases A-E incomplete. Issue #2 open.

**Exit criteria.**
- Each surface has one published ordering key, and the value is displayed per entry.
- No surface uses a composite or blended key.
- No surface admits paid, granted or manually inserted placement.
- The cold-start asymmetry is documented, not silently compensated for inside an
  ordering.

## Status of every phase

| Phase | Status |
| :--- | :--- |
| A - Canonical event envelope | Not started. Blocked on Issue #1. |
| B - Indexer model | Not started. Blocked on Phase A and Issue #1. |
| C - Enrichment pipeline | Not started. Blocked on Phases A-B. |
| D - Live stream prototype | Not started. Blocked on Phases A-C. |
| E - Alert semantics | Not started. Blocked on Phase D and Issue #1. |
| F - Discovery ordering research | Not started. Blocked on Phases A-E and Issue #2. |

No phase has begun. No deliverable in this document exists.

## Open design issues

- [#1 - Specify finality policy for ignition alerts](https://github.com/Rearctor/live-reactor/issues/1)
- [#2 - Define transparent ranking inputs for discovery surfaces](https://github.com/Rearctor/live-reactor/issues/2)

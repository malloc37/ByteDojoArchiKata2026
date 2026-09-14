# Integration through published domain events

Date: 2026-09-13

Owner: LJO

## Status

Accepted

## Context

Seven contexts ([ADR-002](adr-002-bounded-contexts.md)) in one cloud deployable ([ADR-009](adr-009-modular-monolith.md)) and a second service on the
estate ([ADR-016](adr-016-architecture-style.md)) have to share what happened, with no duplicates under retry ([QA-03](../quality-attributes.md#qa-03-integrity)) and
a traceable history for every closure and AI alert ([QA-05](../quality-attributes.md#qa-05-explainability)).

Alternatives considered:

- Modules read each other's tables. Rejected. The boundaries erode with the first schema
  change.
- Synchronous calls between modules. Rejected. The park service would wait on the cloud
  ([QA-01](../quality-attributes.md#qa-01-availability)), and nothing records why something happened.
- State per module plus an outbox. Rejected, narrowly. Two sources of truth per module,
  and replaying history means reconstructing it from tables that have moved on.

## Decision

Events are the primary record on both sides and the only way a state change in one
context reaches another. A context may read another context's current state through a
read-only query interface.

- Each service keeps an append-only event log. Module state, the local estate view and
  AI inputs are projections and can be rebuilt from it.
- Events use the names from the EventStorming model. The publisher owns the contract;
  changes are additive and a breaking change is a new event name.
- Every event carries a stable event ID, occurrence time, producer and schema version.
  Policy outcomes carry the policy version ([ADR-006](adr-006-policies-execute-at-edge.md)). AI results carry the fields
  [ADR-003](adr-003-deterministic-core-advisory-ai.md) requires.
- Delivery is at least once. Consumers are idempotent by event ID; cloud ingestion
  deduplicates and stores ingestion time next to occurrence time ([QA-03](../quality-attributes.md#qa-03-integrity)).
- A read-only query interface never changes state. Commands cross a boundary only as a
  policy reaction to an event.

## Consequences

- The mechanism enforces the boundaries from [ADR-002](adr-002-bounded-contexts.md).
- "Why" is a query: a closure is an event with a policy version, an alert is an event
  with source references.
- A new projection, including a new AI use case, reads the existing log without changing
  a producer.
- Event sourcing costs: snapshots, replay tooling, schema versioning, and a delay between
  a write and its projection.
- Every producer, including a gate device, must emit an event ID and a timestamp.

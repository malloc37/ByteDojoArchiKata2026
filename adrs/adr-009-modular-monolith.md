# Start the cloud platform as a modular monolith

Date: 2026-09-07

Owner: BKA

## Status

Accepted

## Context

The estate has 5,000 visitors per day and expects at least 15,000. ADR-002 defines seven bounded contexts. 

Alternatives considered:

- One deployable service per bounded context. Rejected. We have no scaling, ownership or reliability evidence that justifies it, and it would add distributed transactions, seven deployment pipelines and cross-service debugging.
- A single application with no internal boundaries. Rejected. The boundaries from ADR-002 would erode within weeks and the model would stop matching the code.

## Decision

The cloud platform is one deployable application with one module per bounded context.

- Modules do not share database tables. A state change reaches another module only as a published domain event. A module may read another module's data through a read-only query interface. See [ADR-005](adr-005-integration-through-domain-events.md).
- A module is extracted into a separate deployment only with a concrete scaling, ownership or reliability reason, recorded in a new ADR.
- A bounded context may have a module in both deployables, serving different use cases. The park service module is the local working core: it takes commands, runs the policies from [ADR-006](adr-006-policies-execute-at-edge.md) and records what happened. The cloud module ingests those events and builds the projections that reporting and Estate Insights read. A rule runs in one place; the cloud does not re-run it.

The park service on the Estate Edge Hub is a separate deployment for a stated reliability reason: it must run when the cloud is unreachable. [ADR-016](adr-016-architecture-style.md) records it as a second, smaller modular monolith holding the local working core of the operational contexts.

## Consequences

- One pipeline, one runtime, one place to debug.
- 15,000 visitors per day is not a scale problem for a single application, so the decision is not load-driven and should be revisited on evidence, not on growth alone.
- Module isolation is a discipline, not a deployment guarantee. It needs review to hold.
- Because integration is already event-based, extracting a module later is a deployment change rather than a redesign.

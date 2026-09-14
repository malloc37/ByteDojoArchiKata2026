# Architecture style

Date: 2026-09-14

Owner: LJO

## Status

Proposed

## Context

[ADR-002](adr-002-bounded-contexts.md), [ADR-003](adr-003-deterministic-core-advisory-ai.md) and [ADR-009](adr-009-modular-monolith.md) decide boundaries, authority and the cloud deployable, but
none names the style of the whole system or shows what it was compared with.

We rated the candidates with the architecture styles worksheet by Mark Richards
(*Fundamentals of Software Architecture*, Richards and Ford). Framed rows are the driving
characteristics, crossed boxes the styles we chose.

![Architecture styles worksheet](../diagrams/architecture-styles-worksheet.png)

Driving characteristics:

- Fault tolerance. Admission, ride safety and animal care continue without the cloud
  ([QA-01](../quality-attributes.md#qa-01-availability), [QA-02](../quality-attributes.md#qa-02-safety)).
- Simplicity and cost. Three people build and run it ([ADR-009](adr-009-modular-monolith.md)).
- Domain partitioning. [ADR-002](adr-002-bounded-contexts.md) cut the system into seven contexts. The style must
  not cut across them.

Safety and scalability are not drivers. No style provides safety ([ADR-003](adr-003-deterministic-core-advisory-ai.md)), and 15,000
visitors a day is not a load problem ([QA-07](../quality-attributes.md#qa-07-performance-and-scale)).

On those rows the two chosen styles are mirror images. The modular monolith scores five
stars on cost, domain partitioning and simplicity, and one star on fault tolerance: when
the cloud is unreachable, the estate stops. Event-driven scores five stars on fault
tolerance, because nothing waits on anything, and one star on simplicity and domain
partitioning: it is partitioned by flow, and an asynchronous flow is harder to follow
than a call stack. Combined, each covers the other's weak rows. The monolith is the shape
of each deployable. Events are the only integration, between modules and across the
estate-cloud boundary. The cost of events is paid on the seams, not inside a module.

Alternatives considered:

- Layered monolith or modular monolith alone. Rejected. One process cannot survive the
  cloud being unreachable from the estate.
- Microkernel. Rejected. Same problem, and plug-ins fit a product with variants.
- Microservices. Rejected. Seven services and seven pipelines for three people.
- Service-based. Rejected, narrowly. Closest single style, but it assumes a shared
  database and synchronous calls between services.
- Event-driven alone. Rejected. No synchronous local core to admit a visitor.
- Service-oriented and space-based. Not rated. Not our problems.

## Decision

Two modular monoliths that integrate only through domain events.

- The cloud platform is one modular monolith with one module per context ([ADR-009](adr-009-modular-monolith.md)). It
  is the system of record.
- The park service on the Estate Edge Hub is a second, smaller modular monolith with the
  operational core: admission, ride safety policies, animal-care recording and the local
  estate view. It works without the cloud. Devices reach it over MQTT ([ADR-004](adr-004-mixed-connectivity-mqtt.md)).
- Inside a module, calls are synchronous. Between modules and between the two services,
  the only integration is published domain events ([ADR-005](adr-005-integration-through-domain-events.md)), exchanged over one
  WebSocket connection ([ADR-007](adr-007-estate-cloud-sync-protocol.md)). Nothing shares a database.

See [cloud and estate](../diagrams/architecture-cloud-estate.png).

## Consequences

- Fault tolerance. The park service is an availability boundary. An outage delays
  events; it does not block work.
- Simplicity and cost. Two deployables, two pipelines. The event log is the only
  infrastructure the combination adds.
- Domain partitioning. Modules are the contexts from [ADR-002](adr-002-bounded-contexts.md), and a module can leave
  the cloud platform as a deployment change ([QA-06](../quality-attributes.md#qa-06-evolvability)).
- Testability suffers. Two logs and asynchronous delivery make one visit harder to follow
  than a call stack. Tracing across both services is needed from the first release.
- The cloud lags the estate during an outage ([QA-07](../quality-attributes.md#qa-07-performance-and-scale)). [ADR-008](adr-008-signed-offline-ticket-validation.md) exists because the
  two services can disagree for a while.

Open: whether Ticketing becomes its own cloud service. Today [ADR-009](adr-009-modular-monolith.md) keeps it a module.

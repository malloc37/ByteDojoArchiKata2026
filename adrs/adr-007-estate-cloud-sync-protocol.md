# Estate-to-cloud synchronization protocol

Date: 2026-09-13

Owner: LJO

## Status

Proposed

## Context

The park service records admissions, faults, inspections and animal care while the
internet may be down for hours ([QA-01](../quality-attributes.md#qa-01-availability)). Everything it records must reach the cloud, the
system of record. Signing keys, rules, daily plans, staff identity-role updates and AI
findings that need a person must reach the estate ([ADR-006](adr-006-policies-execute-at-edge.md),
[ADR-008](adr-008-signed-offline-ticket-validation.md),
[ADR-012](adr-012-identity-authorization-attribution.md)). No event may be lost or applied twice ([QA-03](../quality-attributes.md#qa-03-integrity)), the estate never waits on the
cloud, and an outage must end without manual repair. Devices use MQTT ([ADR-004](adr-004-mixed-connectivity-mqtt.md)); this
record covers the park-to-cloud link only.

Alternatives considered:

- Bridge the estate MQTT broker to a cloud broker. Rejected. MQTT could carry domain-event
  payloads, but its acknowledgement only confirms that a broker received a message, not
  that the cloud application durably stored and processed it. We would still need an
  application acknowledgement on top, plus a cloud broker to run.
- Periodic batch upload over HTTPS. Rejected. The other direction needs a second
  mechanism, and the cloud view is as stale as the batch interval on a healthy link.
- Two-way database replication. Rejected. It knows nothing about events, policy versions
  or deduplication, and conflicts would be resolved in the database.

## Decision

One WebSocket connection between the park service and the cloud carries domain events
up, and cache updates and AI findings down, with the same checkpoint scheme in both
directions.

**Delivery**

- The park service keeps the connection open and sends each event as it is appended,
  with a sequence number it assigns per stream.
- The cloud stores each event and its deduplication record in one transaction. It
  acknowledges the highest contiguous sequence it has durably stored, never past a gap.
  Acknowledgements are cumulative.
- Pruning removes only the transmission backlog below the acknowledged sequence. The local
  event log keeps its history for *proposed* 30 days so the park service can rebuild its
  projections. The cloud keeps the full history.
- Cache updates flow down the same way with a cloud-assigned sequence. Rule sets and
  staff identity-role data carry their versions
  ([ADR-006](adr-006-policies-execute-at-edge.md),
  [ADR-012](adr-012-identity-authorization-attribution.md)).
- When the connection drops, the park service keeps serving the estate. Pending events
  live in its local database, so the buffer is bounded by disk. On reconnect both sides
  resume from the last acknowledged sequence.

**AI findings and tasks**

- An AI finding that needs a person flows down the same channel. It carries a finding ID,
  its provenance (confidence, model or rule version, source events) and an expiry time.
- Staff Tasks and Alerts at the estate creates the task and owns it from then on. The
  cloud module of the same name is a read-only projection of task history.
- A redelivered finding with a known finding ID creates no second task.
- A finding that arrives after its expiry, such as staffing advice for a shift that has
  ended, is recorded as expired and creates no task.
- Task acknowledgement, inspection results and review outcomes are estate events and flow
  up like any other ([ADR-011](adr-011-ai-evaluation-human-review.md)). The path, including an outage, is drawn in
  [AI task delivery](../diagrams/ai-task-delivery.png).

**Invalid messages and conflicts**

- A message that fails schema validation or comes from an unauthorized sender is
  quarantined for investigation and never applied.
- A valid estate event is never rejected. A business conflict, such as one ticket
  claimed twice, becomes an exception task for Admission Staff (FR-AV-09), under the
  policy in [ADR-008](adr-008-signed-offline-ticket-validation.md).
- Each business entity has one authoritative writer:

| Written at the estate | Written in the cloud |
|---|---|
| Admissions and ticket use claims | Ticket products, purchases and issued tickets |
| Ride faults, closures and inspections | Attraction catalogue content |
| Animal care records and population counts | Staff identities and roles |
| Tasks and their outcomes | AI findings, rule sets and signing keys |

Still open: the longest outage the local database must hold ([QA-01](../quality-attributes.md#qa-01-availability) proposes 4 hours),
and how the two sides authenticate each other ([ADR-012](adr-012-identity-authorization-attribution.md)).

## Consequences

- One mechanism and one set of tests for both directions, in the application rather than
  in a broker or a database.
- Cloud dashboards lag the estate by the length of an outage; the Operations Manager
  works from the local estate view meanwhile.
- An AI finding produced during an outage reaches staff after reconnection, or not at all
  once it has expired.
- The park service needs a durable local store and disk monitoring. A full disk during
  an outage is the failure mode to design against.
- Quarantined messages need monitoring and an owner, or they are silently lost.
- Conflicts become staff tasks rather than protocol errors ([ADR-003](adr-003-deterministic-core-advisory-ai.md)).

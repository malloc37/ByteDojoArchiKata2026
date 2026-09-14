# Estate-to-cloud synchronization protocol

Date: 2026-09-13

Owner: LJO

## Status

Proposed

## Context

The park service records admissions, faults, inspections and animal care while the
internet may be down for hours ([QA-01](../quality-attributes.md#qa-01-availability)). Everything it records must reach the cloud, the
system of record. Signing keys, rules, daily plans and staff identity-role updates must
reach the estate ([ADR-006](adr-006-policies-execute-at-edge.md),
[ADR-008](adr-008-signed-offline-ticket-validation.md),
[ADR-012](adr-012-identity-authorization-attribution.md)). No event may be lost or applied twice ([QA-03](../quality-attributes.md#qa-03-integrity)), the estate never waits on the
cloud, and an outage must end without manual repair. Devices use MQTT ([ADR-004](adr-004-mixed-connectivity-mqtt.md)); this
record covers the park-to-cloud link only.

Alternatives considered:

- Bridge the estate MQTT broker to a cloud broker. Rejected. It moves device messages,
  not domain events, and gives no acknowledgement to checkpoint on.
- Periodic batch upload over HTTPS. Rejected. The other direction needs a second
  mechanism, and the cloud view is as stale as the batch interval on a healthy link.
- Two-way database replication. Rejected. It knows nothing about events, policy versions
  or deduplication, and conflicts would be resolved in the database.

## Decision

One WebSocket connection between the park service and the cloud carries domain events
up and cache updates down, with the same checkpoint scheme in both directions.

- The park service keeps the connection open and sends each event as it is appended,
  with a sequence number it assigns per stream.
- The cloud persists each event, assigns the authoritative position, deduplicates by
  event ID and acknowledges the highest sequence persisted. Acknowledgements are
  cumulative; the park service stores the last one and prunes only below it.
- Cache updates flow down the same way with a cloud-assigned sequence. Rule sets and
  staff identity-role data carry their versions
  ([ADR-006](adr-006-policies-execute-at-edge.md),
  [ADR-012](adr-012-identity-authorization-attribution.md)).
- When the connection drops, the park service keeps serving the estate. Pending events
  live in its local database, so the buffer is bounded by disk. On reconnect both sides
  resume from the last acknowledged sequence.
- The cloud never rejects an estate event. A conflict on ingestion, such as one ticket
  admitted at two gates, becomes an exception task for Admission Staff (FR-AV-09), under
  the policy in [ADR-008](adr-008-signed-offline-ticket-validation.md).

Still open: the longest outage the local database must hold ([QA-01](../quality-attributes.md#qa-01-availability) proposes 4 hours),
and how the two sides authenticate each other ([ADR-012](adr-012-identity-authorization-attribution.md)).

## Consequences

- One mechanism and one set of tests for both directions, in the application rather than
  in a broker or a database.
- Cloud dashboards lag the estate by the length of an outage; the Operations Manager
  works from the local estate view meanwhile.
- The park service needs a durable local store and disk monitoring. A full disk during
  an outage is the failure mode to design against.
- Conflicts become staff tasks rather than protocol errors ([ADR-003](adr-003-deterministic-core-advisory-ai.md)).

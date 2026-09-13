# Signed offline ticket validation

Date: 2026-09-07

Owner: MJE

## Status

Proposed

## Context

Admission gates normally have local network and internet connectivity, but the estate
must continue admitting visitors when the internet or cloud is unavailable
([QA-01](../quality-attributes.md#qa-01-availability)). A gate must validate a ticket
within one second without depending on a cloud round trip
([QA-07](../quality-attributes.md#qa-07-performance-and-scale)).

Alternatives considered:

- Validate every ticket in the cloud. Rejected because an internet outage would stop
  admission.
- Download every valid ticket to every gate. Rejected because every gate would depend on
  a complete and current copy, including last-minute purchases, cancellations and
  refunds. A public key is smaller and changes less frequently.
- Trust ticket identifiers without a signature. Rejected because identifiers can be
  altered or forged offline.

## Decision

Ticketing signs each issued ticket. Its QR code contains the ticket identifier and the
claims needed for admission, including validity date and ticket type, together with the
digital signature.

- The private signing key remains in the cloud. Gates receive the corresponding public
  verification keys and admission rules in advance through the Estate Edge Hub.
- A gate verifies the signature and applies its cached rules locally. It never waits for
  MQTT, the Edge Hub or the cloud before making the admission decision.
- The gate records every accepted or rejected validation locally with a stable event ID,
  ticket ID, gate ID, occurrence time, result, reason, verification-key ID and rule
  version. It publishes the event over MQTT when connectivity is available
  ([ADR-004](adr-004-mixed-connectivity-mqtt.md)).
- Public-key rotation uses an overlap period so gates can validate tickets signed with
  either the current or previous key. Gates report stale keys and rules as a health
  warning before they expire.
- When connected to the local estate network, gates use the shared local admission view
  to detect ticket reuse. If gates become isolated, a valid signed ticket is admitted;
  duplicate use discovered after synchronization becomes an exception for Admission
  Staff rather than blocking the queue.

This decision supports tickets issued before an outage. Issuing and paying for new
tickets during an outage is outside this ADR.

The admission history is always traceable to a ticket, gate and time. It does not contain
the visitor's name or contact details. Where a ticket product requires a named holder,
authorized users may resolve the ticket ID through Ticketing under the privacy rules in
[ADR-013](adr-013-privacy-consent-retention.md).

## Consequences

- Previously issued tickets remain verifiable during an internet or cloud outage, and
  the private signing key is never placed on a gate.
- Each validation can be audited later without copying personal details into admission
  events.
- A copied ticket may be accepted at two mutually disconnected gates. This availability
  trade-off is recorded and resolved after synchronization.
- Gates require secure local storage, clock synchronization, key and rule distribution,
  and monitoring of cache age.
- Admission events are delivered at least once, so the Edge Hub and cloud must
  deduplicate them by event ID ([QA-03](../quality-attributes.md#qa-03-integrity)).

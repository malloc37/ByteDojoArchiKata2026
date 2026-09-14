# Cloud and estate architecture

The platform consists of two modular monoliths. The cloud is authoritative, while the
Estate Edge Hub keeps essential park operations available during an internet outage.

![Cloud and estate architecture](../diagrams/architecture-cloud-estate.png)

The estate records admissions, ride operations and animal care locally. A WebSocket
sends pending domain events to the cloud and returns the small operational cache when
connectivity is available. Estate devices use the connectivity described in the
[connectivity view](../diagrams/connectivity-tiers.png).

This view is justified by:

- [ADR-009](../adrs/adr-009-modular-monolith.md): begin with modular monoliths.
- [ADR-016](../adrs/adr-016-architecture-style.md): two deployables form the overall
  architecture style.
- [ADR-007](../adrs/adr-007-estate-cloud-sync-protocol.md): synchronize over one
  resumable WebSocket connection.
- [ADR-004](../adrs/adr-004-mixed-connectivity-mqtt.md): connect estate devices locally.
- [ADR-006](../adrs/adr-006-policies-execute-at-edge.md): execute operational policies at
  the estate edge.
- [ADR-008](../adrs/adr-008-signed-offline-ticket-validation.md): gates validate signed
  tickets offline.

# Identity, authorization and attribution

Date: 2026-09-07

Owner: MJE

## Status

Proposed

## Context

Visitors use the cloud web application, while admission staff, mechanics, keepers and
operations managers use the local staff application. Staff must remain able to sign in
and work when the internet or cloud is unavailable
([QA-01](../quality-attributes.md#qa-01-availability)). Every consequential action must
remain attributable to a person or device ([QA-08](../quality-attributes.md#qa-08-security)).

Alternatives considered:

- Cloud identity only. Rejected because new staff logins would fail during an outage.
- Shared staff accounts. Rejected because actions would not be attributable.
- Synchronize passwords between identity providers. Rejected because secrets, password
  changes and lockouts could diverge.

## Decision

Use Keycloak for workforce identity and access management, with one cloud instance and
one estate-local instance. Visitor accounts remain owned by Ticketing.

- Cloud Keycloak is authoritative for staff identities, roles and employment status.
  Ticketing remains the only store that maps a visitor account to a person
  ([ADR-013](adr-013-privacy-consent-retention.md)); visitor data is not held in Keycloak.
- Visitors sign in to the visitor website through Ticketing, against the account store it
  owns ([ADR-013](adr-013-privacy-consent-retention.md)). A visitor account is only needed
  to buy and retrieve tickets (FR-TK-05). Keycloak plays no part in it.
- Active staff IDs, roles and status are provisioned one way to Estate Keycloak. Staff
  enroll a separate estate-local passkey or badge credential; passwords and visitor
  accounts are never synchronized.
- Estate Keycloak authenticates the staff application and continues issuing sessions
  from its last synchronized state during an outage.
- Staff roles include Catalogue Manager, Admission Staff, Mechanic, Zookeeper,
  Zookeeper Staff Manager and Operations Manager. Applications enforce these permissions
  and retain business rules.
- Staff ID, role and status updates travel as versioned cache updates over the
  estate-cloud WebSocket defined in
  [ADR-007](adr-007-estate-cloud-sync-protocol.md).
- An authorized local administrator can disable a staff identity during an outage.
  Cloud changes otherwise take effect after synchronization; the cache age is visible.
- Visitors enter using signed tickets rather than an estate login
  ([ADR-008](adr-008-signed-offline-ticket-validation.md)). Devices use individual
  machine credentials and MQTT topic permissions, not human accounts
  ([ADR-004](adr-004-mixed-connectivity-mqtt.md)).
- Consequential events record actor type, stable actor ID and occurrence time. Authorized
  views resolve names without adding personal details to the event log
  ([ADR-013](adr-013-privacy-consent-retention.md)).

The one-way provisioning integration and local passkey or badge flow must be proven
before this ADR is accepted.

## Consequences

- Staff can authenticate and work without the cloud, and actions remain attributable.
- Passwords and visitor identities are not copied to the estate.
- Two Keycloak instances, provisioning and local credentials must be operated.
- During an outage, the estate may temporarily honor a cloud-revoked role or identity;
  local disablement limits but does not remove this risk.
- Applications remain responsible for domain authorization and safety policies.

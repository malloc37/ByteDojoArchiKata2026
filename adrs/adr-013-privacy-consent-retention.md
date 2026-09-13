# Privacy, consent and retention of visitor data

Date: 2026-09-13

Owner: LJO

## Status

Accepted

## Context

The estate records where visitors go, Ticketing knows who bought what, and the visitor
app wants preferences. [QA-04](../quality-attributes.md#qa-04-privacy) requires anonymous counts by default and a consent basis and
retention period for every stored category. We assume a European jurisdiction and the
GDPR; the brief does not say where the estate is.

Alternatives considered:

- Identify visitors in movement events. Rejected. Nothing in the first release needs it,
  every projection becomes personal data, and deleting one visitor would mean rewriting
  an append-only log.
- Collect nothing personal. Rejected. A visitor must get their tickets back (FR-TK-05),
  and a purchase must stay attributable for years by law.

## Decision

Events are anonymous. Personal data lives in one place, linked by opaque identifiers.

- No domain event carries a name, e-mail address, payment detail or device identifier.
  Events carry ticket IDs and, where a purchase needs it, an account ID.
- The account store in the cloud, owned by Ticketing, is the only place that maps an
  account ID to a person. Deleting a visitor deletes that record; the events stay and
  belong to nobody.
- Visitor-flow analysis and crowd forecasting work on counts ([QA-04](../quality-attributes.md#qa-04-privacy)). No
  re-identification across visits, no device tracking.
- Consent is per purpose and recorded as an event. Itinerary preferences (UC-3) and
  return-visit offers (UC-6) each need their own. Without consent the app shows the plain
  attraction list.

Retention, *proposed*:

| Category | Personal | Basis | Retention |
|---|---|---|---|
| Purchase and payment record | Yes | Contract, then legal obligation | 10 years; survives account deletion; confirm for the estate's country |
| Account (name, e-mail) | Yes | Contract | Until deleted, or 3 years after the last purchase |
| Ticket and validation events | Ticket ID only | Contract | Kept; unlinkable once the account is gone |
| Movement and queue counts | No | None needed | Indefinite |
| Itinerary preferences | Session only | Consent | End of the visit day |
| Return-visit profile | Yes | Consent | Until withdrawal; not built before UC-6 |
| Staff attribution | Staff | Employment | Per employer rules ([ADR-012](adr-012-identity-authorization-attribution.md)) |

## Consequences

- Deleting a visitor is one record in one store. No event log is rewritten, which keeps
  [ADR-005](adr-005-integration-through-domain-events.md) and this decision compatible.
- First-release AI runs on anonymous or pseudonymous data. UC-6 waits for consent
  ([ADR-015](adr-015-first-release-ai-use-cases.md)).
- Personalization is weaker for visitors who give no consent.
- Purchase records outlive the account. If the estate is outside the GDPR, the table
  needs legal review, not a redesign.

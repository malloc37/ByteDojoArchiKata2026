# Deterministic operational core with advisory AI

Date: 2026-09-07

Owner: BKA

## Status

Accepted

## Context

The estate must keep admitting visitors with tickets already issued, closing unsafe rides and recording animal care when the internet, the cloud or an AI provider is unavailable. Selling new tickets online needs the cloud and the payment provider, and pauses during an internet outage ([ADR-014](adr-014-payment-provider.md)). AI is also a required part of this challenge.

Alternatives considered:

- AI in the control path, for example closing a ride or adjusting admission directly. Rejected. A model cannot be held accountable for a safety decision, its behaviour changes when the model changes, and the estate would stop working when the provider does.
- No AI. Rejected. Crowding, animal anomalies and itinerary planning are real problems where pattern detection helps, and AI suitability is part of the challenge.

## Decision

Deterministic rules and named humans hold authority over every consequential action. AI consumes recorded facts and returns recommendations.

- AI never issues a command that changes payments, ticket issuance, admission, ride safety or animal-care records.
- Consequential AI findings become a task or an alert for a person. See ADR-006 for where the routing policies run.
- Every AI result records confidence, model or rule version, source event references and creation time.
- Abstention is a valid result. No recommendation is better than a low-confidence one.
- Sensor observations, AI inferences, keeper decisions and confirmed diagnoses stay separately identifiable.

## Consequences

- We give up autonomous optimization. Crowding and queue response is as fast as the responsible staff member, not as fast as the model.
- Essential operations degrade cleanly. If the AI services are gone, the estate loses recommendations and keeps working.
- Every consequential action stays attributable and auditable, which ADR-012 has to support.
- AI output is data, not control, so models and providers can be replaced behind stable interfaces. See ADR-010 and ADR-011.

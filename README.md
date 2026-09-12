# Von Digitalis Estate

**Build a dependable estate platform first. Add AI around it to detect patterns and
recommend actions.**

The estate runs 40 historic rides and more than 200 animals across 55 enclosures, for
5,000 visitors a day growing to at least 15,000. Wi-Fi coverage across the grounds is
patchy. Essential operations like selling tickets, admitting visitors, closing an unsafe
ride, recording animal care continue when the internet, the cloud or an AI provider is
unavailable. AI never does not sit in the control path.

---

## Start here


| # | What | Why it matters |
|---|---|---|
| 1 | [System context](diagrams/context-estate.png) | Who uses the estate, and the one boundary that matters: cloud vs estate |
| 2 | [ADR-003 — Deterministic core with advisory AI](adrs/adr-003-deterministic-core-advisory-ai.md) | The decision everything else follows from |
| 3 | [Cloud authority with estate continuity](diagrams/architecture-cloud-estate.png) | How the estate keeps working when the cloud does not |

---

## How it works

**The cloud is the authoritative system of record.** It holds long-term records and event
history, serves the visitor website, and runs the business services for all seven domains.

**The estate keeps working without it.** An Estate Edge Hub holds a small cache — signing
keys, validation rules, daily plans — plus a lightweight local database of current state
and pending events. Gates verify signed tickets offline against a cached public key.
Deterministic safety policies, such as *a ride with a safety fault cannot operate*, execute
at the edge so they survive an outage ([ADR-006](adrs/adr-006-policies-execute-at-edge.md)).

**Events flow one way out, a small cache flows one way in.** Locally captured events stay
pending until the cloud acknowledges them. Ingestion deduplicates by event ID, so a retry
or an at-least-once redelivery never creates a duplicate charge, ticket or business event.

**AI consumes recorded facts and returns recommendations.** It produces anomaly alerts,
crowd forecasts and itinerary suggestions. Every consequential finding becomes a task or an
alert for a named person. Nothing a model outputs changes a payment, an admission, a ride
or an animal-care record.

---

## Diagrams

Each is a `.drawio` source with a rendered `.png` beside it.

| Diagram | Shows |
|---|---|
| [System context](diagrams/context-estate.png) | Actors, the platform boundary, and the two external systems we depend on |
| [Cloud and estate](diagrams/architecture-cloud-estate.png) | Containers on both sides of the boundary and what crosses it |
| [Connectivity tiers](diagrams/connectivity-tiers.png) | One link type per job — Ethernet/PoE, Wi-Fi, LoRaWAN/Thread — and why |
| [EventStorming by domain boundary](eventstorming/01-eventstorming-by-domain-boundary.png) | The full domain model, mapped to the seven bounded contexts |
| [Domain boundaries and event flow](eventstorming/02-domain-boundaries-event-flow.png) | Which events cross which boundary |
| [Cross-domain policies](eventstorming/03-cross-domain-policies.png) | Every policy, with the owning boundary on each side |

Component names describe **capability, not product**. Technology choices live in the ADRs.

---

## The seven bounded contexts

Defined in [ADR-002](adrs/adr-002-bounded-contexts.md), discovered by EventStorming.

| Context | Owns |
|---|---|
| Attraction Catalogue | Visitor-facing descriptions and published availability |
| Ticketing | Ticket products, purchase, payment, ticket issuance |
| Admission and Visitor Flow | Ticket validation, gates, park and area movement, queues, occupancy |
| Ride Operations | Ride inspections, faults, safety closure, real ride availability |
| Animal and Enclosure Care | Animals, enclosures, feeding, inspections, cleaning, population |
| Estate Insights and AI Decision Support | Analysis and recommendations over recorded facts |
| Staff Tasks and Alerts | Turning policies into work for people |

Ride Operations and Animal and Enclosure Care own **real** availability. The catalogue
publishes it and cannot override a safety closure.

---

## Decisions

Short ADRs with context and alternatives, decision, and consequences.
*Accepted* means the team agreed it. *Proposed* means it is drafted or still open — we have
kept that distinction honest rather than marking everything accepted.

| ADR | Decision | Status |
|---|---|---|
| [001](adrs/adr-001-use-ddd.md) | Use Domain-Driven Design | Accepted |
| [002](adrs/adr-002-bounded-contexts.md) | Bounded contexts and domain boundaries | Accepted |
| [003](adrs/adr-003-deterministic-core-advisory-ai.md) | Deterministic operational core with advisory AI | Accepted |
| [004](adrs/adr-004-mixed-connectivity-mqtt.md) | Mixed connectivity with MQTT as the estate-local transport | Proposed |
| [005](adrs/adr-005-integration-through-domain-events.md) | Integration through published domain events | Proposed |
| [006](adrs/adr-006-policies-execute-at-edge.md) | Cross-domain policies execute at the estate edge | Accepted |
| [007](adrs/adr-007-estate-cloud-sync-protocol.md) | Estate-to-cloud synchronization protocol | Proposed |
| [008](adrs/adr-008-signed-offline-ticket-validation.md) | Signed offline ticket validation | Proposed |
| [009](adrs/adr-009-modular-monolith.md) | Start the cloud platform as a modular monolith | Accepted |
| [010](adrs/adr-010-ai-orchestration.md) | AI orchestration behind stable interfaces | Proposed |
| [011](adrs/adr-011-ai-evaluation-human-review.md) | AI evaluation and human review | Proposed |
| [012](adrs/adr-012-identity-authorization-attribution.md) | Identity, authorization and attribution | Proposed |
| [013](adrs/adr-013-privacy-consent-retention.md) | Privacy, consent and retention of visitor data | Proposed |
| [014](adrs/adr-014-payment-provider.md) | Use an external payment provider | Proposed |

---

## How we handle AI

| Question                               | Answer                                                                                                                                | Where                                                                         |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| What can AI change?                    | Nothing consequential. It produces recommendations: deterministic rules and named people decide.                                      | [ADR-003](adrs/adr-003-deterministic-core-advisory-ai.md)                     |
| How is uncertainty handled?            | Every result carries confidence, model or rule version, source-data references and creation time. Abstention is a valid result.       | [ADR-003](adrs/adr-003-deterministic-core-advisory-ai.md), FR-EI-07, FR-EI-10 |
| How is a recommendation verified?      | A responsible staff member accepts, rejects or corrects it, and the outcome is recorded.                                              | FR-EI-08, [ADR-011](adrs/adr-011-ai-evaluation-human-review.md)               |
| What if the provider disappears?       | Capabilities sit behind task-shaped interfaces; provider choice is configuration. The estate keeps operating with no recommendations. | [ADR-010](adrs/adr-010-ai-orchestration.md), QA-06                            |
| Can staff see why an alert was raised? | Yes — inputs and model or rule version, without engineering support.                                                                  | [QA-05 Explainability](quality-attributes.md#qa-05-explainability)            |

Sensor observations, AI inferences, keeper decisions and confirmed diagnoses stay
separately identifiable. A model never silently becomes a fact.

---

## Requirements and quality attributes

- **[Functional Requirements](functional-requirements.md)** — what the system must do,
  by bounded context, plus the cross-domain policy table.
- **[Quality Attributes](quality-attributes.md)** — eight scenarios with a measure and a
  stated cost, and a traceability table from each attribute to the requirements, decisions
  and diagrams that serve it.

The quality attributes also name what we deliberately did **not** optimise for.

---

## What is not decided

We have kept open questions visible rather than presenting them as settled:

- The estate-to-cloud synchronization protocol ([ADR-007](adrs/adr-007-estate-cloud-sync-protocol.md)).
- How long a gate may run offline, and the acceptable duplicate-ticket risk ([ADR-008](adrs/adr-008-signed-offline-ticket-validation.md)).
- Consent scope and movement-data retention ([ADR-013](adrs/adr-013-privacy-consent-retention.md)).
- The identity and authorization model ([ADR-012](adrs/adr-012-identity-authorization-attribution.md)).
- Which AI use cases ship first, and how they are scored ([ADR-010](adrs/adr-010-ai-orchestration.md), [ADR-011](adrs/adr-011-ai-evaluation-human-review.md)).

Red stickies on the [EventStorming model](eventstorming/01-eventstorming-by-domain-boundary.png)
mark these in place, alongside the key business moments.

---

## Repository layout

```
adrs/                 architecture decision records, plus the template
diagrams/             context, cloud/estate and connectivity (.drawio + .png)
eventstorming/        digitized EventStorming model (.drawio + .png)
functional-requirements.md
quality-attributes.md
```

On the EventStorming pages, a **solid border** is a sticky note from the physical session.
A **dashed border** was added while digitizing, to close a gap or name an implicit step.

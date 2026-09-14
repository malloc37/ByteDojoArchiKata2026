# Quality Attributes

These are the criteria the architecture is judged against. Functional behaviour is in
[Functional Requirements](functional-requirements.md). The decisions that serve these
attributes are in [adrs/](adrs).

Each scenario states what happens, what we measure, and what the choice costs. Numbers
marked *proposed* still need team agreement.

---

## QA-01 Availability

**Scenario.** The estate loses its internet connection during opening hours. Visitors keep
arriving at gates and keepers keep working.

**Response.** Gates validate signed tickets against cached keys and rules and record
admissions locally. The Estate Edge Hub serves the shared estate view over the local
network. Locally captured events stay pending until the cloud acknowledges them.

**Measure.** Admission, ride inspection and animal-care recording continue for at least
**4 hours** of continuous outage with no loss of recorded events. *Proposed* - the maximum
outage the cache must survive is not yet agreed.

**Cost.** The edge holds real logic and cached rules, so it has to be versioned, tested and
distributed like an application rather than treated as a buffer.

---

## QA-02 Safety

**Scenario.** A ride fault is detected while the cloud is unreachable. Separately, a model
produces a high-confidence animal anomaly.

**Response.** The fault closes the ride through a deterministic policy running at the Edge
Hub. The anomaly creates an inspection task for a keeper. It never changes a ride, a gate
or a care record.

**Measure.** No path exists by which an AI output issues a command that changes payments,
admission, ride safety or animal-care records. Every safety closure is applied at the
estate regardless of cloud reachability.

**Cost.** Response to a detected problem is bounded by staff availability, not by model
speed. We give up autonomous optimization to keep authority with people and rules.

---

## QA-03 Integrity

**Scenario.** A payment request times out and the client retries. Separately, a gate
re-sends buffered admission events after reconnecting.

**Response.** A ticket is issued once per confirmed payment. Cloud ingestion deduplicates
by event ID and preserves both occurrence time and ingestion time.

**Measure.** **Zero** duplicate charges, tickets or business events under client retry and
at-least-once delivery.

**Cost.** Every producer, including constrained devices, must generate a stable event ID.
No component may emit an anonymous event.

---

## QA-04 Privacy

**Scenario.** Visitor movement is recorded across areas, queues and enclosures to support
occupancy counts and crowd forecasting.

**Response.** Visitor-flow analysis uses anonymous counts by default. Identity is attached
only where a requirement needs it, such as retrieving a purchase.

**Measure.** Crowding, occupancy and popularity features work correctly on anonymous data.
Every stored data category has a recorded consent basis and retention period.

**Cost.** Some personalization is weaker without identity. Itinerary and return-offer
features depend on consent that some visitors will not give.

**Open.** Retention periods are proposed in [ADR-013](adrs/adr-013-privacy-consent-retention.md) and await team agreement.

---

## QA-05 Explainability

**Scenario.** A keeper is sent to inspect an animal because of an AI alert and asks why.

**Response.** The recommendation record shows confidence, model or rule version, the source
events it was derived from, and creation time. A deterministic closure shows the version of
the policy that applied.

**Measure.** Any AI alert or automated closure can be traced to its inputs and its
rule or model version **without engineering support**.

**Cost.** Extra metadata stored on every inference and every policy outcome, and a
versioning discipline for policy rules.

---

## QA-06 Evolvability

**Scenario.** The AI provider changes, a model is rolled back, or a first-release use case
is replaced.

**Response.** AI capabilities sit behind task-shaped interfaces. Provider and model
selection is configuration owned by Estate Insights. Contexts integrate through published
domain events, so a module can later be extracted without redesign.

**Measure.** Swapping the provider or rolling back a model changes **only** Estate Insights
configuration. No domain module changes.

**Cost.** One layer of indirection between the domain and the provider, and the discipline
not to read another context's tables directly.

---

## QA-07 Performance and scale

**Scenario.** Daily visitors grow from 5,000 to 15,000, with arrivals concentrated in the
opening hour.

**Response.** Gate validation is local and never waits on the cloud. The park service
streams estate events to the cloud over one connection with cumulative acknowledgements,
so no gate ever waits on a cloud round trip.

**Measure.** Gate validation completes within **1 second** at the gate, with no cloud round
trip. *Proposed.* 15,000 visitors per day is not a scale driver for the cloud application.
A separate deployment needs evidence, not growth alone.

**Cost.** During an outage the cloud lags the estate until the stream has caught up.
Reporting is eventually correct, not live.

---

## QA-08 Security

**Scenario.** Someone copies a valid ticket and presents it at a second gate during an
outage. Separately, an unknown device attempts to publish events.

**Response.** Tickets are signed, not secret. The gate verifies the signature with a cached
public key and applies cached eligibility rules. Devices and staff actions are
attributable.

**Measure.** A forged or altered ticket fails verification **offline**, with no cloud call.
Every consequential action records which person or device performed it and when.

**Cost.** Offline verification cannot detect reuse across disconnected gates. That is a
business risk to be bounded, not a cryptography problem.

**Open.** Duplicate use at isolated gates is decided: admit, then resolve after
synchronization ([ADR-008](adrs/adr-008-signed-offline-ticket-validation.md)). Offline staff sign-in through an estate identity provider is
proposed and still has to be proven ([ADR-012](adrs/adr-012-identity-authorization-attribution.md)).

---

## Traceability

| Attribute | Requirements | Decisions | Shown in |
|---|---|---|---|
| QA-01 Availability | FR-AV-04, FR-AV-05, FR-RO-09, FR-AE-11 | [ADR-003](adrs/adr-003-deterministic-core-advisory-ai.md), [ADR-006](adrs/adr-006-policies-execute-at-edge.md), [ADR-007](adrs/adr-007-estate-cloud-sync-protocol.md), [ADR-016](adrs/adr-016-architecture-style.md) | [cloud and estate](diagrams/architecture-cloud-estate.png), [connectivity](diagrams/connectivity-tiers.png) |
| QA-02 Safety | FR-RO-04, FR-RO-09, FR-AE-09, FR-EI-09 | [ADR-003](adrs/adr-003-deterministic-core-advisory-ai.md), [ADR-006](adrs/adr-006-policies-execute-at-edge.md) | [context](diagrams/context-estate.png), [cross-domain policies](eventstorming/03-cross-domain-policies.png) |
| QA-03 Integrity | FR-TK-04, FR-TK-06, FR-AV-05 | [ADR-005](adrs/adr-005-integration-through-domain-events.md), [ADR-007](adrs/adr-007-estate-cloud-sync-protocol.md) | [cloud and estate](diagrams/architecture-cloud-estate.png) |
| QA-04 Privacy | FR-AV-06, FR-AV-07, FR-EI-02, FR-EI-06 | [ADR-013](adrs/adr-013-privacy-consent-retention.md) | [EventStorming](eventstorming/01-eventstorming-by-domain-boundary.png) |
| QA-05 Explainability | FR-AE-09, FR-EI-07, FR-EI-08, FR-EI-10, FR-ST-03 | [ADR-003](adrs/adr-003-deterministic-core-advisory-ai.md), [ADR-005](adrs/adr-005-integration-through-domain-events.md), [ADR-006](adrs/adr-006-policies-execute-at-edge.md), [ADR-010](adrs/adr-010-ai-orchestration.md), [ADR-011](adrs/adr-011-ai-evaluation-human-review.md) | [EventStorming](eventstorming/01-eventstorming-by-domain-boundary.png) |
| QA-06 Evolvability | FR-AC-04, FR-RO-07, FR-AE-08 | [ADR-002](adrs/adr-002-bounded-contexts.md), [ADR-005](adrs/adr-005-integration-through-domain-events.md), [ADR-009](adrs/adr-009-modular-monolith.md), [ADR-010](adrs/adr-010-ai-orchestration.md), [ADR-016](adrs/adr-016-architecture-style.md) | [context](diagrams/context-estate.png), [boundaries](eventstorming/02-domain-boundaries-event-flow.png) |
| QA-07 Performance and scale | FR-AV-04, FR-AV-07 | [ADR-004](adrs/adr-004-mixed-connectivity-mqtt.md), [ADR-007](adrs/adr-007-estate-cloud-sync-protocol.md), [ADR-009](adrs/adr-009-modular-monolith.md) | [connectivity](diagrams/connectivity-tiers.png) |
| QA-08 Security | FR-AV-01, FR-AV-04, FR-ST-03 | [ADR-008](adrs/adr-008-signed-offline-ticket-validation.md), [ADR-012](adrs/adr-012-identity-authorization-attribution.md), [ADR-014](adrs/adr-014-payment-provider.md) | [cloud and estate](diagrams/architecture-cloud-estate.png) |

## Attributes we deliberately did not prioritise

- **Low latency of AI recommendations.** People review consequential findings, so minutes
  are acceptable.
- **Live cloud reporting during an outage.** Catching up after reconnect is enough.
  Correctness matters more than freshness.
- **Horizontal scalability of the cloud application.** 15,000 visitors per day does not
  require it, and a modular monolith can be split later.

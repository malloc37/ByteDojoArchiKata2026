# O'Reilly Architectural Kata 2026: Von Digitalis Estate

**An offline-first estate platform with dependable local operations and AI-assisted
insights in the cloud.**

## Contents

- [Team](#team)
- [Introduction](#introduction)
- [How AI solves the Countess's problems](#how-ai-solves-the-countesss-problems)
- [Architecture at a glance](#architecture-at-a-glance)
  - [System context](architecture/system-context.md)
  - [Deterministic core with advisory AI](adrs/adr-003-deterministic-core-advisory-ai.md)
  - [Cloud and estate architecture](architecture/cloud-estate.md)
  - [Architecture style](adrs/adr-016-architecture-style.md)
- [How it works](#how-it-works)
- [The seven bounded contexts](#the-seven-bounded-contexts)
- [How we handle AI](#how-we-handle-ai)
- [Requirements and quality attributes](#requirements-and-quality-attributes)
- [Open decisions](#open-decisions)
- [Repository layout](#repository-layout)

---

## Team

![Byte Dojo Team](./assets/byte-dojo-team.png)

## Introduction

The estate runs 40 historic rides and more than 200 animals across 55 enclosures, for
5,000 visitors a day growing to at least 15,000. Wi-Fi coverage across the grounds is
patchy. Essential operations like selling tickets, admitting visitors, closing an unsafe
ride and recording animal care continue when the internet, the cloud or an AI provider
is unavailable. AI does not sit in the control path.

---

## How AI solves the Countess's problems

Three problems were named: animals are costly and far more costly once sick, nobody knows
where to invest and deploy staff, and visitor numbers need to grow. Three AI use cases
answer them, one each.

A keeper feeds an animal and records it on a phone. That record, plus enclosure sensor
readings, flows to the cloud. Overnight a model notices that one animal has eaten less for
three days while its activity dropped. It raises an anomaly with a confidence score and
the exact events it looked at. A keeper gets an inspection task, checks the animal, and
records what they found. The record the keeper writes is the fact. The model's opinion
stays an opinion.

Through the day, counts from gates, areas and queues build a picture of where visitors
actually go. A forecast tells the Operations Manager where the crowd will be in two hours
so staff move before the queue forms, and it shows the Countess which parts of the estate
earn their keep. Visitors get a suggested route built from real availability and real queue
lengths, so a first visit feels well planned rather than lucky.

Then the internet drops for an hour. Gates keep admitting people, the faulty ride stays
closed, keepers keep recording care, and the overdue-feeding rule still raises tasks.
The forecasts and suggestions simply stop until the connection returns. Nothing that
matters was waiting on a model.

---

## Architecture at a glance


| # | What | Why it matters |
|---|---|---|
| 1 | [System context](architecture/system-context.md) | Who uses the estate, and the one boundary that matters: cloud vs estate |
| 2 | [ADR-003, deterministic core with advisory AI](adrs/adr-003-deterministic-core-advisory-ai.md) | The decision everything else follows from |
| 3 | [Cloud and estate, two modular monoliths](architecture/cloud-estate.md) | How the estate keeps working when the cloud does not |
| 4 | [ADR-016, architecture style](adrs/adr-016-architecture-style.md) | Which styles we compared, against which characteristics, and why two services |

---

## How it works

**The cloud platform is the authoritative system of record.** It is one modular monolith
with a module per bounded context. It keeps the authoritative event log and serves the
visitor website.

**The estate keeps working without it.** A park service on the Estate Edge Hub, a second
and smaller modular monolith, runs admission, ride safety policies and animal-care
recording from its own local event log and a small cache of signing keys, validation
rules and daily plans. Gates verify signed tickets offline against a cached public key.
Deterministic safety policies, such as *a ride with a safety fault cannot operate*, execute
at the edge so they survive an outage ([ADR-006](adrs/adr-006-policies-execute-at-edge.md),
[ADR-016](adrs/adr-016-architecture-style.md)).

**Events go up and a small cache comes down, over one WebSocket connection.** Locally
captured events stay pending until the cloud acknowledges them, and both sides resume
from the last acknowledgement after an outage
([ADR-007](adrs/adr-007-estate-cloud-sync-protocol.md)). Modules communicate state changes
through published domain events ([ADR-005](adrs/adr-005-integration-through-domain-events.md)).
Ingestion deduplicates by event ID, so a retry or an at-least-once redelivery never
creates a duplicate charge, ticket or business event.

**AI consumes recorded facts and returns recommendations.** It produces anomaly alerts,
crowd forecasts and itinerary suggestions. Every consequential finding becomes a task or an
alert for a named person. Nothing a model outputs changes a payment, an admission, a ride
or an animal-care record.

---

## The seven bounded contexts

Defined in [ADR-002](adrs/adr-002-bounded-contexts.md) and discovered through the
[full EventStorming model](eventstorming/01-eventstorming-by-domain-boundary.png). The
[boundary view](eventstorming/02-domain-boundaries-event-flow.png) shows which events
cross contexts, while the [policy view](eventstorming/03-cross-domain-policies.png) shows
the rules connecting them.

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

## How we handle AI

| Question                               | Answer                                                                                                                                | Where                                                                         |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| What can AI change?                    | Nothing consequential. It produces recommendations: deterministic rules and named people decide.                                      | [ADR-003](adrs/adr-003-deterministic-core-advisory-ai.md)                     |
| How is uncertainty handled?            | Every result carries confidence, model or rule version, source-data references and creation time. Abstention is a valid result.       | [ADR-003](adrs/adr-003-deterministic-core-advisory-ai.md), FR-EI-07, FR-EI-10 |
| How is a recommendation verified?      | A responsible staff member accepts, rejects or corrects it, and the outcome is recorded.                                              | FR-EI-08, [ADR-011](adrs/adr-011-ai-evaluation-human-review.md)               |
| What if the provider disappears?       | Capabilities sit behind task-shaped interfaces. Provider choice is configuration. The estate keeps operating with no recommendations. | [ADR-010](adrs/adr-010-ai-orchestration.md), QA-06                            |
| Can staff see why an alert was raised? | Yes, inputs and model or rule version, without engineering support.                                                                  | [QA-05 Explainability](quality-attributes.md#qa-05-explainability)            |

Sensor observations, AI inferences, keeper decisions and confirmed diagnoses stay
separately identifiable. A model never silently becomes a fact.

**The three first-release use cases**, each answering one of the Countess's problems.
Full catalogue, including what was deferred and why, in
[ai-use-cases.md](ai-use-cases.md). The selection is
[ADR-015](adrs/adr-015-first-release-ai-use-cases.md).

| Use case | Solves | Targeted view |
|---|---|---|
| Animal health anomaly detection | Sick animals are expensive | [diagram](diagrams/ai-animal-health.png) |
| Crowd forecasting and staffing | Where to invest and deploy staff | [diagram](diagrams/ai-crowd-staffing.png) |
| Visitor itinerary recommendation | Growing visitor numbers | [diagram](diagrams/ai-visitor-itinerary.png) |

All three follow one pipeline:
[the AI decision-support pattern](diagrams/ai-decision-support-pattern.png).

---

## Requirements and quality attributes

- **[Glossary](glossary.md)** the key terms of the ubiquitous language from
  [ADR-001](adrs/adr-001-use-ddd.md).
- **[Functional Requirements](functional-requirements.md)** what the system must do,
  by bounded context, plus the cross-domain policy table.
- **[Quality Attributes](quality-attributes.md)** eight scenarios with a measure and a
  stated cost, and a traceability table from each attribute to the requirements, decisions
  and diagrams that serve it.

- **[AI Use Cases](ai-use-cases.md)** what each use case reads, produces, when it abstains, who reviews it, what it costs to be wrong, and how it is measured.

The quality attributes also name what we deliberately did **not** optimise for.

---

## Open decisions

We have kept open questions visible rather than presenting them as settled:

- The longest outage the estate must bridge, and how the estate and the cloud authenticate each other ([ADR-007](adrs/adr-007-estate-cloud-sync-protocol.md)).
- The maximum age of a gate's cached keys and rules ([ADR-008](adrs/adr-008-signed-offline-ticket-validation.md)). Duplicate use at isolated gates is decided: admit, then resolve after synchronization.
- Retention periods for visitor data, proposed but not agreed ([ADR-013](adrs/adr-013-privacy-consent-retention.md)).
- Whether one-way staff provisioning and offline staff sign-in work in practice ([ADR-012](adrs/adr-012-identity-authorization-attribution.md)).
- Whether Ticketing becomes its own cloud service ([ADR-016](adrs/adr-016-architecture-style.md)).
- The cloud provider whose model platform hosts the AI models, the retry limit for invalid structured output, and how long long-running process records are kept ([ADR-010](adrs/adr-010-ai-orchestration.md)).
- The acceptance threshold at which an AI use case may leave full human review ([ADR-011](adrs/adr-011-ai-evaluation-human-review.md)). The use cases themselves are chosen in [ADR-015](adrs/adr-015-first-release-ai-use-cases.md).

Red stickies on the [EventStorming model](eventstorming/01-eventstorming-by-domain-boundary.png)
mark these in place, alongside the key business moments.

---

## Repository layout

```
adrs/                 architecture decision records, plus the template
architecture/         guided explanations of the main architecture views
diagrams/             context, cloud/estate, connectivity (.drawio + .png), styles worksheet
eventstorming/        digitized EventStorming model (.drawio + .png)
ai-use-cases.md
functional-requirements.md
glossary.md
quality-attributes.md
```

On the EventStorming pages, a **solid border** is a sticky note from the physical session.
A **dashed border** was added while digitizing, to close a gap or name an implicit step.

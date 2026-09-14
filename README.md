# O'Reilly Architectural Kata 2026: Von Digitalis Estate

**An offline-first estate platform with dependable local operations and AI-assisted
insights in the cloud.**

## Contents

- [Team](#team)
- [Introduction](#introduction)
- [How AI solves the Countess's problems](#how-ai-solves-the-countesss-problems)
- [AI use cases and evidence](#ai-use-cases-and-evidence)
- [How we handle AI](#how-we-handle-ai)
- [Where to find each judging criterion](#where-to-find-each-judging-criterion)
- [Architecture at a glance](#architecture-at-a-glance)
  - [System context](architecture/system-context.md)
  - [Deterministic core with advisory AI](adrs/adr-003-deterministic-core-advisory-ai.md)
  - [Cloud and estate architecture](architecture/cloud-estate.md)
  - [Architecture style](adrs/adr-016-architecture-style.md)
- [How it works](#how-it-works)
- [EventStorming and domain discovery](#eventstorming-and-domain-discovery)
- [Requirements and quality attributes](#requirements-and-quality-attributes)
- [Open decisions](#open-decisions)
- [Known limitations](#known-limitations)
- [Repository layout](#repository-layout)

---

## Team

![Byte Dojo Team](./assets/byte-dojo-team.png)

- Moritz
- Luka
- Besmir

## Introduction

The estate runs 40 historic rides and more than 200 animals across 55 enclosures, for
5,000 visitors a day growing to at least 15,000. Wi-Fi coverage across the grounds is
patchy. Admitting visitors with issued tickets, closing an unsafe ride and recording
animal care continue when the internet, the cloud or an AI provider is unavailable. New
online ticket sales pause until the connection returns. AI does not sit in the control
path.

---

## How AI solves the Countess's problems

The brief names costly animal care, not knowing where to invest and deploy staff, the need
for more visitors and more returning visitors, and checking the population of the jumping
piranha collection. Five AI use cases answer them.

A keeper feeds an animal and records it on a phone. That record, the keeper's notes and
enclosure sensor readings flow to the cloud. Overnight a model notices that one animal has
eaten less for three days while its activity dropped. It raises a finding with a calibrated
confidence and the exact events it looked at. The finding travels to the estate, a keeper
gets an inspection task, checks the animal, and records what they found. The record the
keeper writes is the fact. The model's opinion stays an opinion.

Through the day, counts from gates, areas and queues show where visitors actually go. A
forecast tells the Operations Manager where the crowd will be in two hours so staff move
before the queue forms. Visitors get a suggested route built from real availability and
real queue lengths. After their visit, their anonymous feedback is grouped into themes per
attraction, so the Countess learns what brings visitors back and where to invest. Cameras
over the piranha tanks count the fish and flag a tank whose population leaves its healthy
range.

Then the internet drops for an hour. Gates keep admitting people, the faulty ride stays
closed, keepers keep recording care, and the overdue-feeding rule still raises tasks.
Forecasts and suggestions simply stop until the connection returns. Nothing that matters
was waiting on a model.

---

## AI use cases and evidence

| Use case | Answers | Targeted view |
|---|---|---|
| UC-1 Animal health anomaly detection | Sick animals are expensive | [process and diagram](architecture/ai-animal-health.md) |
| UC-2 Crowd forecasting and staffing | Where to invest and deploy staff | [process and diagram](architecture/ai-crowd-staffing.md) |
| UC-3 Visitor itinerary recommendation | Growing visitor numbers | [process and diagram](architecture/ai-visitor-itinerary.md) |
| UC-7 Piranha population count | Population levels of the jumping piranha | [process and diagram](architecture/ai-piranha-count.md) |
| UC-8 Visitor feedback insights | Returning visitors, where to invest | [process and diagram](architecture/ai-visitor-feedback.md) |

All five follow one pipeline:
[the AI decision-support pattern](diagrams/ai-decision-support-pattern.png)
([editable source](diagrams/ai-decision-support.drawio)). How a finding from the cloud
reaches a keeper, including during an outage, is drawn in
[AI task delivery](diagrams/ai-task-delivery.png).

The evidence behind them:

- **[AI use cases](ai-use-cases.md)** records inputs, model, outputs, abstention, human
  review, failure costs and measures for each use case, plus the deferred ones and why.
- **[ADR-017](adrs/adr-017-ai-model-approach-and-confidence.md)** compares each LLM use case
  with a non-LLM baseline it must beat, and defines what confidence means.
- **[ADR-011](adrs/adr-011-ai-evaluation-human-review.md)** is the evaluation package:
  development and test data, cold start, production measures, and what happens when a
  model misbehaves.
- **[AI evaluation examples](ai-evaluation-examples.md)** walk through synthetic cases
  from input to pass or fail.
- **[Business outcomes and cost](business-outcomes.md)** states which estate outcome each
  use case should move, how it is measured, and what running it costs.

---

## How we handle AI

| Question | Answer | Where |
|---|---|---|
| What can AI change? | Nothing consequential. It produces recommendations: deterministic rules and named people decide. | [ADR-003](adrs/adr-003-deterministic-core-advisory-ai.md) |
| Why an LLM here? | Each LLM use case must beat a simple non-LLM baseline on its own measure before it ships. | [ADR-017](adrs/adr-017-ai-model-approach-and-confidence.md) |
| How is uncertainty handled? | Confidence has a defined, tested meaning per use case, never the model's own stated number. Abstention is a valid result. | [ADR-017](adrs/adr-017-ai-model-approach-and-confidence.md), [FR-EI-07 and FR-EI-10](functional-requirements.md#6-estate-insights-and-ai-decision-support) |
| How is a recommendation verified? | Tuned on development data, judged on separate test data, then monitored on objective measures such as keeper-confirmed findings and forecast error. | [ADR-011](adrs/adr-011-ai-evaluation-human-review.md), [examples](ai-evaluation-examples.md) |
| What if the provider disappears or doubles its price? | Only Estate Insights changes: a new adapter, or the baseline behind the same port, after re-evaluation. The estate keeps operating with no recommendations. | [ADR-010](adrs/adr-010-ai-orchestration.md), [QA-06](quality-attributes.md#qa-06-evolvability) |
| Can staff see why an alert was raised? | Yes, inputs and model or rule version, without engineering support. | [QA-05 Explainability](quality-attributes.md#qa-05-explainability) |

Sensor observations, AI inferences, keeper decisions and confirmed diagnoses stay
separately identifiable. A model never silently becomes a fact.

---

## Where to find each judging criterion

| Judging criterion | Where to look |
|---|---|
| Innovative use of AI | [AI use cases](ai-use-cases.md), [ADR-015](adrs/adr-015-first-release-ai-use-cases.md) |
| Suitability given the constraints | [Failure modes](quality-attributes.md#failure-modes), [ADR-004](adrs/adr-004-mixed-connectivity-mqtt.md), [ADR-006](adrs/adr-006-policies-execute-at-edge.md), [ADR-008](adrs/adr-008-signed-offline-ticket-validation.md), [business outcomes and cost](business-outcomes.md) |
| Appropriate level of detail | [Architecture pages](architecture/), [AI task delivery](diagrams/ai-task-delivery.png) |
| Dealing with uncertainty in AI technology | [ADR-010](adrs/adr-010-ai-orchestration.md), [ADR-017](adrs/adr-017-ai-model-approach-and-confidence.md), [QA-06](quality-attributes.md#qa-06-evolvability) |
| AI additions match the existing architecture | [ADR-003](adrs/adr-003-deterministic-core-advisory-ai.md), [ADR-007](adrs/adr-007-estate-cloud-sync-protocol.md), [cloud and estate](architecture/cloud-estate.md) |
| Validation and verification of AI results | [ADR-011](adrs/adr-011-ai-evaluation-human-review.md), [AI evaluation examples](ai-evaluation-examples.md) |

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
visitor website. The relationship between both deployables is explained in
[Cloud and estate: two modular monoliths](architecture/cloud-estate.md).

[![Cloud and estate: two modular monoliths](diagrams/architecture-cloud-estate.png)](architecture/cloud-estate.md)

**The estate keeps working without it.** A park service on the Estate Edge Hub, a second
and smaller modular monolith, runs admission, ride safety policies and animal-care
recording from its own local event log and a small cache of signing keys, validation
rules and daily plans. Gates verify signed tickets offline against a cached public key.
Deterministic safety policies, such as *a ride with a safety fault cannot operate*, execute
at the edge so they survive an outage ([ADR-006](adrs/adr-006-policies-execute-at-edge.md),
[ADR-016](adrs/adr-016-architecture-style.md)).

**Events go up, and a small cache and AI findings come down, over one WebSocket
connection.** Locally captured events stay pending until the cloud acknowledges them, and
both sides resume from the last acknowledgement after an outage
([ADR-007](adrs/adr-007-estate-cloud-sync-protocol.md)). Modules communicate state changes
through published domain events ([ADR-005](adrs/adr-005-integration-through-domain-events.md)).
Ingestion deduplicates by event ID, so a retry or an at-least-once redelivery never
creates a duplicate charge, ticket or business event.

**AI consumes recorded facts and returns recommendations.** Every consequential finding
travels to the estate, where Staff Tasks turns it into a task for a named person. Nothing a
model outputs changes a payment, an admission, a ride or an animal-care record.

---

## EventStorming and domain discovery

Before choosing technologies, we used EventStorming to understand what happens across
the estate, who initiates each action and where business ownership changes. This took us
from the Countess's broad optimization problem to seven explicit bounded contexts and
the policies connecting them.

The [EventStorming and domain discovery](eventstorming/README.md) page walks through the
three iterations: the full business flow, the resulting domain boundaries and the
cross-domain policies that shaped the architecture.

---

## Requirements and quality attributes

The brief and EventStorming findings were turned into explicit requirements before the
architecture was selected:

- **[Glossary](glossary.md)** defines the ubiquitous language from
  [ADR-001](adrs/adr-001-use-ddd.md).
- **[Functional Requirements](functional-requirements.md)** describes the required
  behavior by bounded context and the cross-domain policies.
- **[Quality Attributes](quality-attributes.md)** defines eight measurable scenarios and
  a table of failure modes, and traces them to the requirements, decisions and diagrams
  that address them.

The quality attributes also state what we deliberately did **not** optimize for.

---

## Open decisions

We have kept open questions visible rather than presenting them as settled:

- The longest outage the estate must bridge ([ADR-007](adrs/adr-007-estate-cloud-sync-protocol.md)).
- The validity period of a gate's cached keys and rules, proposed at 7 days ([ADR-008](adrs/adr-008-signed-offline-ticket-validation.md)).
- Retention periods for visitor data, proposed but not agreed ([ADR-013](adrs/adr-013-privacy-consent-retention.md)).
- Whether one-way staff provisioning and offline staff sign-in work in practice ([ADR-012](adrs/adr-012-identity-authorization-attribution.md)).
- The retry limit for invalid structured output and the retention of process records, proposed in [ADR-010](adrs/adr-010-ai-orchestration.md). The Model Provider is configuration, not a decision.
- The proposed numbers behind AI evaluation: thresholds, repeated runs, cold-start periods and the acceptance threshold at which review may relax ([ADR-011](adrs/adr-011-ai-evaluation-human-review.md), [ADR-017](adrs/adr-017-ai-model-approach-and-confidence.md)).

Red stickies on the [EventStorming model](eventstorming/01-eventstorming-by-domain-boundary.png)
mark these in place, alongside the key business moments.

---

## Known limitations

- Cloud reporting is stale during an internet outage and catches up after reconnection
  ([ADR-007](adrs/adr-007-estate-cloud-sync-protocol.md)).
- Automatic gate admission depends on the estate-local network and Edge Hub. A local
  infrastructure failure requires a manual admission procedure
  ([ADR-008](adrs/adr-008-signed-offline-ticket-validation.md),
  [failure modes](quality-attributes.md#failure-modes)).
- A staff role revoked in the cloud may remain usable at the estate until synchronization
  resumes; an authorized local administrator can disable it sooner
  ([ADR-012](adrs/adr-012-identity-authorization-attribution.md)).
- New online payments stop while the payment provider or internet connection is
  unavailable; already issued tickets continue to work
  ([ADR-014](adrs/adr-014-payment-provider.md)).
- AI produces no recommendation when its provider is unavailable or its input data is
  insufficient. Essential estate operations continue without it
  ([ADR-003](adrs/adr-003-deterministic-core-advisory-ai.md)).
- Revenue and operating cost per attraction are not in the platform, so the first release
  shows where visitors go and what they say, not which attraction is profitable
  ([business outcomes](business-outcomes.md)).

---

## Repository layout

```
adrs/                      architecture decision records, plus the template
architecture/              guided explanations of the main architecture and AI views
diagrams/                  context, cloud/estate, connectivity, AI views (.drawio + .png)
eventstorming/             digitized EventStorming model (.drawio + .png)
ai-use-cases.md            every AI use case, first release and deferred
ai-evaluation-examples.md  synthetic worked evaluation cases
business-outcomes.md       outcomes to measure, and the cost model
functional-requirements.md
glossary.md
quality-attributes.md
```

On the EventStorming pages, a **solid border** is a sticky note from the physical session.
A **dashed border** was added while digitizing, to close a gap or name an implicit step.

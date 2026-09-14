# O'Reilly Architectural Kata 2026: Von Digitalis Estate

**An offline-first estate platform with dependable local operations and AI-assisted
insights in the cloud.**

## Contents

- [Team](#team)
- [Introduction](#introduction)
- [How AI solves the Countess's problems](#how-ai-solves-the-countesss-problems)
- [EventStorming and domain discovery](#eventstorming-and-domain-discovery)
- [Requirements and quality attributes](#requirements-and-quality-attributes)
- [Where to find each judging criterion](#where-to-find-each-judging-criterion)
- [Architecture at a glance](#architecture-at-a-glance)
  - [System context](architecture/system-context.md)
  - [Deterministic core with advisory AI](adrs/adr-003-deterministic-core-advisory-ai.md)
  - [Cloud and estate architecture](architecture/cloud-estate.md)
  - [Architecture style](adrs/adr-016-architecture-style.md)
  - [AI decision support](architecture/ai-decision-support.md)
- [Known limitations](#known-limitations)

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
- **[AI Use Cases](ai-use-cases.md)** records the models, inputs, outputs, abstention,
  human review, failure costs and measurements for each AI capability.

The quality attributes also state what we deliberately did **not** optimize for.

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
| 5 | [AI decision support](architecture/ai-decision-support.md) | How recommendations handle uncertainty, human review and failure |

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

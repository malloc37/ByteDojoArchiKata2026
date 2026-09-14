# Business outcomes and cost

The AI use cases are justified by what they change for the estate, not by what they
compute. This page states which outcome each business objective is measured by, how the
measure is collected, and what running the system costs. Technical AI measures are in
[ADR-011](adrs/adr-011-ai-evaluation-human-review.md).

## Outcomes

| Business objective | Supported by | Outcome to measure | How it is collected | Limits |
|---|---|---|---|---|
| Healthier animals at manageable cost | UC-1, UC-7, overdue-feeding rule | Days from first recorded signs to a confirmed concern. Keeper hours spent on AI tasks. Veterinary cost trend per quarter | Inspection outcomes and task times from Staff Tasks. Veterinary costs from the estate's accounts, entered quarterly | Care costs also move with animal numbers and prices. A trend, not proof |
| Better staffing and investment | UC-2, UC-8, visitor counts | Queue time at the busiest attractions. Staff hours per 1,000 visitors. Utilization per area. Recurring feedback themes per attraction | Queue entry and exit events. Accepted staffing recommendations. Feedback themes from UC-8 | Revenue and operating cost per attraction are not in the platform. The first release shows where visitors go and what they say, not which attraction is profitable |
| More visitors | UC-3 | Daily admissions and ticket sales, compared with the same weeks a year earlier | Admission and purchase events | Many things drive visitor numbers. UC-3 is one of them, not the proven cause |
| More returning visitors | UC-8 now, UC-6 later | Repeat-purchase rate of account holders within 12 months, per quarterly cohort. Post-visit satisfaction | Purchases by account ID in Ticketing. The optional post-visit questions | Only visitors with an account can be counted, because events never identify visitors ([ADR-013](adrs/adr-013-privacy-consent-retention.md)) |
| Population levels checked | UC-7 | Days a tank spends outside its healthy population range | Count events and keeper spot counts | Counts in murky water are rejected, so some days have no count |

The itinerary's usefulness is measured by one question in the visitor application at the
end of the visit day: "Did you use the suggested route?" with yes, partly or no. It is
self-reported and only some visitors answer, but it needs no tracking.

## The path to more returning visitors

1. **First release.** Measure repeat purchases by account. UC-8 shows what visitors
   dislike, and UC-3 aims at a better first visit.
2. **Next.** Agree the consent scope and retention of a return-visit profile
   ([ADR-013](adrs/adr-013-privacy-consent-retention.md)).
3. **Then.** Ship return-visit offers (UC-6), measured against a group of account holders
   who receive no offer.

## Cost model

No prices are given, because the provider and hardware vendors are not chosen. The table
names what drives each cost and how it is kept bounded.

| Cost | Driver | Kept bounded by |
|---|---|---|
| Cloud platform | One modular monolith, its database and storage. Grows slowly with events | 15,000 visitors a day is not a scale problem ([ADR-009](adrs/adr-009-modular-monolith.md)) |
| Estate hardware | Edge Hub, gates, LoRaWAN gateways and sensors, edge cameras | Instrument only where a requirement needs it. UC-7 needs cameras over the piranha tanks only |
| Language-model calls | Per call. See the estimate below | A daily budget cap per use case. When UC-3 reaches it, visitors get the plain attraction list |
| Vision model | Runs on the camera, so no per-call cost | Hardware cost only |
| Human review | Keeper time per UC-1 inspection task, Operations Manager review, monthly UC-8 sample, upkeep of evaluation sets | A *proposed* alert budget per keeper per week ([ADR-011](adrs/adr-011-ai-evaluation-human-review.md)) |
| Operations | Two deployables, local infrastructure, identity providers, model upkeep | See the staffing assumption in [ADR-016](adrs/adr-016-architecture-style.md) |

Estimated language-model calls per day, from *proposed* assumptions:

| Use case | Calculation | Calls per day |
|---|---|---|
| UC-1 | About 200 animals, five runs each, once a night | About 1,000 |
| UC-2 | About 10 areas, a forecast each opening hour for 10 hours, five runs each | About 500 |
| UC-3 | One call per visitor who asks for a route, up to 15,000 visitors | Up to 15,000 |
| UC-8 | One daily batch of feedback, five runs | Small |

UC-3 dominates the cost and grows with visitor numbers. It is also the use case whose
fallback, the plain attraction list, costs nothing.

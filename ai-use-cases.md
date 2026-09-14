# AI Use Cases

Every use case below follows the same pattern. It reads recorded facts, returns a
recommendation with confidence and provenance, and hands consequential findings to a named
person. No use case changes a payment, an admission, a ride or an animal-care record.
That rule is [ADR-003](adrs/adr-003-deterministic-core-advisory-ai.md), and the shared
pipeline is drawn in
[the decision-support pattern](diagrams/ai-decision-support-pattern.png).

Three ship first. The selection and its reasoning are in
[ADR-015](adrs/adr-015-first-release-ai-use-cases.md).

| Use case | Business problem it solves | Status |
|---|---|---|
| [UC-1 Animal health anomaly detection](#uc-1-animal-health-anomaly-detection) | Sick animals are expensive. The Countess wants healthy animals. | First release |
| [UC-2 Crowd forecasting and staffing](#uc-2-crowd-forecasting-and-staffing) | Knowing where to invest and deploy staff. | First release |
| [UC-3 Visitor itinerary recommendation](#uc-3-visitor-itinerary-recommendation) | Growing visitor numbers and improving the visit. | First release |
| [UC-4 Animal narration](#uc-4-animal-narration) | Richer visitor experience at an enclosure. | Deferred |
| [UC-5 Virtual ride queue](#uc-5-virtual-ride-queue) | Shorter perceived waits at popular rides. | Deferred |
| [UC-6 Return-visit offers](#uc-6-return-visit-offers) | Bringing past visitors back. | Deferred |

---

## UC-1 Animal health anomaly detection

**Diagram:** [ai-animal-health.png](diagrams/ai-animal-health.png)

**Problem.** Looking after the animals is costly, and far more costly once one is ill.
Over 200 animals across 55 enclosures is more than keepers can watch closely at all times.

**Reads.** `Animal fed` (food offered and consumed), `Animal Inspected`, environmental
readings and activity from enclosure sensors, `Enclosure Population Changed`.

**Produces.** `Animal Anomaly Detected`, carrying confidence, model or rule version,
the source event references and creation time.

**Abstains when** the recent observation window is incomplete, for example after a long
estate outage, or when confidence falls below the configured threshold. An abstention is
recorded and no task is created.

**Reviewed by.** A Zookeeper, through a `Request Animal Inspection` task in Staff Tasks and
Alerts. The keeper inspects and records confirmed, not confirmed, or another cause.

**Changes nothing on its own.** Only the keeper's recorded inspection changes animal
records. The model never produces a diagnosis.

**Cost of being wrong.** A missed anomaly means a late detection, which is why it is not
the only safety net. The deterministic overdue-feeding rule (FR-AE-11) and scheduled keeper
rounds run without AI. A false alarm costs keeper time, but a missed anomaly costs more,
so the confidence threshold favours recall over precision ([ADR-011](adrs/adr-011-ai-evaluation-human-review.md)).

**Measured by.** Recall, with a precision floor, against keeper-confirmed outcomes on the
golden set. After release, the share of alerts a keeper accepts, and anomalies a keeper
finds on rounds with no prior alert.

---

## UC-2 Crowd forecasting and staffing

**Diagram:** [ai-crowd-staffing.png](diagrams/ai-crowd-staffing.png)

**Problem.** The Countess needs to know where to invest and where to deploy staff, and
visitor numbers are expected to triple.

**Reads.** `Park Entered` and `Park Left`, `Visitor entered / left park area`,
`Area visitor count changed`, `Attraction entered`, queue entry and exit, current ride and
enclosure availability, and the same pattern from comparable past days.

**Produces.** `Attraction Popularity Computed`, `Crowd Forecast Produced`,
`Staffing Recommended`.

**Abstains when** the recent counting window has gaps, which happens after an outage or
when counting devices are missing from an area.

**Reviewed by.** The Operations Manager, who accepts, rejects or corrects the
recommendation. The outcome is recorded.

**Changes nothing on its own.** The deterministic `Area Capacity Exceeded` policy runs
separately at the edge and notifies operations whether or not the forecast exists. Crowding
response never waits for a model.

**Cost of being wrong.** Under-staffing or over-staffing for a shift. There is no safety
consequence, because the capacity response is deterministic and independent.

**Privacy.** Works on anonymous counts. No visitor identity is required. See QA-04.

**Measured by.** Forecast error against actual counts per area, and the acceptance rate of
staffing recommendations.

---

## UC-3 Visitor itinerary recommendation

**Problem.** The estate needs to grow visitor numbers, and the brief asks for AI that helps
customers and not only the company. A good visit is the thing visitors talk about.

**Diagram:** [ai-visitor-itinerary.png](diagrams/ai-visitor-itinerary.png)

**Reads.** Published attraction availability from the Attraction Catalogue, current queue
lengths, the crowding forecast from UC-2, and the visitor's stated preferences where
consent exists.

**Produces.** `Itinerary Recommended`, shown in the visitor application.

**Abstains when** availability data is stale, or the visitor has given no preferences and
no consent. The app then falls back to the plain attraction list, which is not an AI
feature.

**Reviewed by.** The visitor. They follow it or they ignore it. No staff review, because
the consequence of a poor suggestion is a less good afternoon.

**Changes nothing on its own.** It is advice inside the visitor application.

**Hard constraint.** A recommendation may never contain a closed ride or enclosure.
Catalogue availability is authoritative and a safety closure always wins (FR-AC-05). This
is checked deterministically before the itinerary is shown, not left to the model.

**Cost of being wrong.** A mediocre route. The hard constraint above keeps a wrong
suggestion from sending anyone to a closed attraction.

**Measured by.** Judged itinerary quality on the golden set ([ADR-011](adrs/adr-011-ai-evaluation-human-review.md)),
and after release the share of recommendations a visitor follows. The closed-attraction
check is a deterministic guard, tested at 100 per cent, not a model score.

---

## Deferred

These stay in the catalogue because they are good ideas, not because they are next.

### UC-4 Animal narration

Location-aware narration about the animal in front of the visitor. Deferred because
narration invents plausible detail unless every claim is grounded in the animal record,
and telling visitors something false about a poisonous animal is a reputational and safety
problem. It also solves none of the three business problems the Countess named. Revisit
once grounding and review are proven on UC-1 and UC-3.

### UC-5 Virtual ride queue

Assigning visitors a return time instead of a physical queue. Deferred because it moves AI
next to admission, which is exactly where [ADR-003](adrs/adr-003-deterministic-core-advisory-ai.md)
says it must not go. A virtual queue also has to keep working during an outage, so it needs
a deterministic queue authority at the edge first. That is an operational feature with an
AI assist, not an AI feature.

### UC-6 Return-visit offers

Personalised offers based on a past visit. Deferred because it depends on consent scope,
retention rules and pricing rules that are not decided
([ADR-013](adrs/adr-013-privacy-consent-retention.md)). Shipping personalisation before the
consent model exists is the fastest way to turn a nice feature into a regulatory problem.

---

## What is common to all of them

| Property | Applies to every use case |
|---|---|
| Input | Recorded facts only. Never another model's unreviewed output. |
| Output | A recommendation record with confidence, model or rule version, source events and creation time. |
| Uncertainty | Abstention is a valid result and is recorded. |
| Authority | A person or a deterministic rule decides. The model never commands. Review starts in full and may relax to sampling once a use case earns it, never for the actions ADR-003 protects ([ADR-011](adrs/adr-011-ai-evaluation-human-review.md)). |
| Degradation | If the provider is unavailable, the use case produces nothing and estate operations are unaffected. |
| Evaluation | Scored against a golden set before release and monitored after. See [ADR-011](adrs/adr-011-ai-evaluation-human-review.md). |
| Provenance | Sensor observation, AI inference, keeper decision and confirmed diagnosis stay separately identifiable (FR-AE-09). |

# AI model approach and the meaning of confidence

Date: 2026-09-14

Owner: to be assigned

## Status

Accepted

## Context

[ADR-011](adr-011-ai-evaluation-human-review.md) starts the first-release use cases on large
language models producing structured output. That leaves two questions open. Why should a
language model forecast visitor counts or spot a numerical change in an animal's eating?
And what does the confidence on a result mean? [ADR-003](adr-003-deterministic-core-advisory-ai.md),
ADR-011 and [ADR-015](adr-015-first-release-ai-use-cases.md) all compare that number with a
threshold, and a number a model states about itself is not reliable until it has been
checked on the task.

Candidates considered, per use case:

| Use case | Non-LLM candidate | What an LLM adds |
|---|---|---|
| UC-1 Animal health | Per-animal statistical baseline, combined with keeper-defined rules | Reads keeper notes, where early signs such as "slow to come to feed" appear, together with the numbers, and explains the finding in words |
| UC-2 Crowd and staffing | Same-weekday seasonal baseline, then a time-series model if it does better | Explains the forecast and the staffing trade-off to the Operations Manager, and needs no training history at launch |
| UC-3 Visitor itinerary | Deterministic route planner that enforces constraints | Turns preferences a visitor writes in their own words into a route |
| UC-7 Piranha count | None; counting fish is a vision task | Nothing. A vision model on the edge camera counts |
| UC-8 Visitor feedback | Keyword rules | Groups free-text feedback into themes without training |

Alternatives considered:

- A non-LLM method for every use case. Rejected for the first release. It loses the keeper
  notes, the explanations and the free-text preferences, which are where the estate's
  staff and visitors actually express what they see and want.
- LLMs with their self-reported confidence used as is. Rejected. That number has no
  tested meaning, so every threshold built on it would be arbitrary.

## Decision

UC-1, UC-2, UC-3 and UC-8 use a language model with structured output behind the
[ADR-010](adr-010-ai-orchestration.md) port. UC-7 uses a vision model on the edge camera.

- The non-LLM candidate for each LLM use case is built as a baseline. The LLM version ships
  only if it beats its baseline on the use case's measure on the test set (ADR-011). If it
  does not, the use case does not ship with the LLM, and the team decides whether to ship
  the baseline instead.
- Deterministic guards stay outside the model: the closed-attraction check in UC-3, the
  staffing limits the Operations Manager sets in UC-2, and human review in UC-1.
- A model's own stated confidence is never compared with a threshold. Confidence is
  defined per use case:

| Use case | Confidence means | How it is checked |
|---|---|---|
| UC-1 | The share of repeated runs, *proposed* five, that flag the animal, mapped through a calibration table | On the development set, findings at a given confidence are confirmed by keepers at about that rate |
| UC-2 | The range of the forecast across repeated runs | The share of actual counts that fall inside the range |
| UC-3 | No number. The itinerary either passes schema validation and the closed-attraction check, or the use case abstains | The guard is tested at 100 per cent; route quality is judged separately |
| UC-7 | Agreement of the count across several frames | Count error against keeper spot counts |
| UC-8 | Agreement of the theme assignment across repeated runs | A monthly human check of a sample |

## Consequences

- Confidence has a stated, testable meaning. Thresholds and calibration are set on
  development data and confirmed on untouched test data (ADR-011).
- Repeated runs multiply language-model calls for UC-1, UC-2 and UC-8 by about five. They
  are batch jobs, so the cost stays bounded ([business outcomes and cost](../business-outcomes.md)).
- Every LLM use case carries a baseline to build and maintain. In return, the claim that
  the LLM helps is measured, and a working fallback exists if a provider disappears.
- A provider or model change means re-running calibration as well as evaluation.
- The vision model in UC-7 is updated with the camera's software, not through the ADR-010
  adapter. Its counts arrive as ordinary events.

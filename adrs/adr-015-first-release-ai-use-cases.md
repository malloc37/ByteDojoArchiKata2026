# First-release AI use cases

Date: 2026-09-12

Owner: BKA

## Status

Proposed

## Context

Eleven AI use cases came out of our working sessions and review: animal health anomaly
detection, crowd forecasting and staffing, visitor itinerary recommendations, piranha
population counting, visitor feedback insights, animal narration, a virtual ride queue,
return-visit offers, pricing recommendations, an ask-the-estate assistant and ride
predictive maintenance. ADR-010 decides how an AI capability is built, ADR-011 how it is
evaluated and ADR-017 which model it uses. None decides which ones we build first, and the
answer is not "all of them".

Two constraints shape the choice. Every use case needs a confidence rule, a named reviewer
and a measure before it is worth shipping, which is real work per use case. And several
depend on decisions or data we do not have yet.

Alternatives considered:

- Build all eleven. Rejected. None would get a proper evaluation set, and several rest on
  policy or history that does not exist yet.
- Build none and treat AI as a later phase. Rejected. Pattern detection is how the estate
  answers the Countess's actual problems, and it is the point of the challenge.
- Only the three use cases for the three named business problems. Rejected. The brief also
  asks explicitly for piranha population levels and for more returning visitors.
- Pick by technical interest. Rejected. Narration is the most interesting to build and the
  least useful to her.

## Decision

The first release contains five use cases:

| Use case | What it answers in the brief |
|---|---|
| UC-1 Animal health anomaly detection | Animals are costly, and far more costly when sick |
| UC-2 Crowd forecasting and staffing | Knowing where to invest and deploy staff |
| UC-3 Visitor itinerary recommendation | Growing visitor numbers |
| UC-7 Piranha population count | Checking population levels of the jumping piranha collection |
| UC-8 Visitor feedback insights | More returning visitors, and where to invest |

Animal narration, the virtual ride queue, return-visit offers, pricing recommendations, the
ask-the-estate assistant and ride predictive maintenance are deferred. Reasons are
recorded in [ai-use-cases.md](../ai-use-cases.md).

A use case ships only when all four of these exist:

1. A defined confidence, a threshold and an abstention rule ([ADR-017](adr-017-ai-model-approach-and-confidence.md)).
2. A named role that reviews the output, or a stated reason no review is needed.
3. A deterministic fallback that works when the use case produces nothing.
4. A measure, development and test sets, and a baseline it beats, per ADR-011.

## Consequences

- Five use cases can be evaluated properly instead of eleven being demonstrated badly. The
  judging criterion is suitability, not quantity.
- Each of the five maps to something the brief asks for, so the value argument does not
  depend on the technology being interesting.
- The deferrals are honest about their blockers. The virtual queue would place AI next to
  admission, which ADR-003 forbids. Return-visit offers need the consent and retention rules
  that ADR-013 has not settled. Pricing and the assistant need history and cost data the
  platform does not hold yet.
- Profitability is addressed only indirectly in the first release, through lower care and
  staffing costs and better insight ([business outcomes](../business-outcomes.md)).
- Requirement four means the first release carries evaluation work that a demo would skip.
  This is deliberate.
- We lose the visitor-facing novelty of narration in the first release. The itinerary
  recommendation carries the customer-facing case instead.

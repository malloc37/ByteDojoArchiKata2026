# First-release AI use cases

Date: 2026-09-12

Owner: BKA

## Status

Proposed

## Context

Six AI use cases came out of our working sessions: animal health anomaly detection, crowd
forecasting and staffing, visitor itinerary recommendations, animal narration, a virtual
ride queue, and return-visit offers. ADR-010 decides how an AI capability is built.
ADR-011 decides how it is evaluated. Neither decides which ones we build first, and the
answer is not "all of them".

Two constraints shape the choice. Every use case needs a confidence rule, a named reviewer
and a measure before it is worth shipping, which is real work per use case. And two of the
six depend on decisions we have not made.

Alternatives considered:

- Build all six. Rejected. None would get a proper evaluation set, and two of them rest on
  policy that does not exist yet.
- Build none and treat AI as a later phase. Rejected. Pattern detection is how the estate
  answers the Countess's actual problems, and it is the point of the challenge.
- Pick by technical interest. Rejected. Narration is the most interesting to build and the
  least useful to her.

## Decision

The first release contains three use cases, one for each business problem the Countess
named:

| Use case | Her problem |
|---|---|
| Animal health anomaly detection | Animals are costly, and far more costly when sick |
| Crowd forecasting and staffing | Knowing where to invest and deploy staff |
| Visitor itinerary recommendation | Growing visitor numbers |

Animal narration, the virtual ride queue and return-visit offers are deferred. Reasons are
recorded in [ai-use-cases.md](../ai-use-cases.md).

A use case ships only when all four of these exist:

1. A confidence threshold and an abstention rule.
2. A named role that reviews the output, or a stated reason no review is needed.
3. A deterministic fallback that works when the use case produces nothing.
4. A measure and a ground-truth set, per ADR-011.

## Consequences

- Three use cases can be evaluated properly instead of six being demonstrated badly. The
  judging criterion is suitability, not quantity.
- Each of the three maps to a stated business problem, so the value argument does not
  depend on the technology being interesting.
- The two deferrals are honest about their blockers. The virtual queue would place AI next
  to admission, which ADR-003 forbids. Return-visit offers need the consent and retention
  rules that ADR-013 has not settled.
- Requirement four means the first release carries evaluation work that a demo would skip.
  This is deliberate.
- We lose the visitor-facing novelty of narration in the first release. The itinerary
  recommendation carries the customer-facing case instead.

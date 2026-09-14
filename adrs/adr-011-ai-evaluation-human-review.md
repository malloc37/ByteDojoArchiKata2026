# AI evaluation and human review

Date: 2026-09-13

Owner: LJO

## Status

Accepted

## Context

[ADR-003](adr-003-deterministic-core-advisory-ai.md) makes AI advisory and [ADR-015](adr-015-first-release-ai-use-cases.md) requires a measure and a ground-truth set before
a use case ships. Neither says how a model is judged before release, how we notice it
getting worse, or when human review may be relaxed.

We start with large language models producing structured outputs, because we cannot
train our own models yet. A provider can change the model under us, so we score outputs,
not model internals. That also holds if a use case later moves to a trained model.

Alternatives considered:

- No formal evaluation. Rejected. A provider update would silently change every
  recommendation.
- An LLM judge for everything. Rejected as the only method. Useless where the truth is a
  count or a keeper's finding.
- A/B tests on live visitors and animals. Rejected. We do not experiment on animal
  health.
- Full human review forever. Rejected. It caps the value of AI at the reviewers'
  attention and never shows which use cases have earned trust.

## Decision

Every use case is scored against a golden set before release, monitored through its
review events after release, and reviewed by a named role until its record says the
review can be relaxed.

- Each use case has a golden set: recorded inputs with the expected structured output,
  covering every value of the important fields. The reviewing role owns it (Zookeeper
  Staff Manager for animal health, Operations Manager for crowds), versioned with the
  model, prompt and confidence threshold it validated.
- Each use case names its measure in [ai-use-cases.md](../ai-use-cases.md), matching its
  cost of being wrong: recall with a precision floor for animal health, forecast error
  for crowding, judged itinerary quality for the itinerary. Accuracy is not a score:
  anomalies are rare, so a model that never flags one scores well on it. The
  closed-attraction check is a deterministic guard and is tested, not scored. Free-text
  fields use an LLM judge, itself checked against a human-rated set.
- The golden set run reports the measure at the configured confidence threshold, and
  the threshold is chosen from that run.
- A model, prompt or provider change ships only if its measure is at least that of the
  version it replaces and any floor holds.
- Reviewers accept, reject or correct each recommendation, recorded as an event
  (FR-EI-08). Acceptance, rejection and correction rates per use case and model version
  measure precision. A finding recorded with no prior recommendation, such as an anomaly
  a keeper finds on rounds, is a missed case and measures recall. Rejected, corrected and
  missed cases join the next golden set.
- Every reviewed use case starts with full review. When acceptance stays above the use
  case's acceptance threshold for a set period, the reviewing role may relax review to
  sampling and let recommendations propagate. Thresholds are per use case, because a
  recall-first use case rejects more by design (*proposed* 99 per cent over four weeks
  for crowds, none yet for animal health). Below the threshold, full review returns.
  Relaxation never touches the actions [ADR-003](adr-003-deterministic-core-advisory-ai.md) protects.

## Consequences

- We can tell a bad model from a bad use case, and see when a provider update changed
  behaviour.
- A use case above its acceptance threshold earns autonomy; one far below it is reworked,
  and the events show why. A recall-first use case sits below the others by design.
- Recall is visible only through missed cases. Keeper rounds and the overdue-feeding
  rule (FR-AE-11) surface them, so they stay in place beside the model.
- The golden set needs an owner who refreshes it, or scores drift from reality.
- The LLM judge is a second model to evaluate. Avoid free text where possible.

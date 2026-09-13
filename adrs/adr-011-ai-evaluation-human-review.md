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
  model and prompt it validated.
- The score is accuracy on structured fields, plus the measure matching each use case's
  cost of being wrong: recall over precision for animal health, forecast error for
  crowding, 100 per cent on the closed-attraction check ([ai-use-cases.md](../ai-use-cases.md)).
  Free-text fields use an LLM judge, itself checked against a human-rated set.
- A model, prompt or provider change ships only if its score is at least that of the
  version it replaces.
- Reviewers accept, reject or correct each recommendation, recorded as an event
  (FR-EI-08). Acceptance, rejection and correction rates per use case and model version
  are the monitoring signal. Rejected and corrected cases join the next golden set.
- Every use case starts with full review. When acceptance stays above a threshold for a
  set period (*proposed* 99 per cent over four weeks), the reviewing role may relax
  review to sampling and let recommendations propagate. Below the threshold, full review
  returns. Relaxation never touches the actions [ADR-003](adr-003-deterministic-core-advisory-ai.md) protects.

## Consequences

- We can tell a bad model from a bad use case, and see when a provider update changed
  behaviour.
- A use case accepted 99 per cent of the time earns autonomy; one rejected 99 per cent of
  the time is reworked, and the events show why.
- The golden set needs an owner who refreshes it, or scores drift from reality.
- The LLM judge is a second model to evaluate. Avoid free text where possible.

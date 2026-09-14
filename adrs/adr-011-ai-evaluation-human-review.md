# AI evaluation and human review

Date: 2026-09-13

Owner: LJO

## Status

Accepted

## Context

[ADR-003](adr-003-deterministic-core-advisory-ai.md) makes AI advisory and [ADR-015](adr-015-first-release-ai-use-cases.md) requires a measure and a ground-truth set before
a use case ships. Neither says how a model is judged before release, how we notice it
getting worse, or when human review may be relaxed.

[ADR-017](adr-017-ai-model-approach-and-confidence.md) sets the model approach and what
confidence means for each use case. A provider can change a model under us, so we score
outputs, not model internals.

Alternatives considered:

- No formal evaluation. Rejected. A provider update would silently change every
  recommendation.
- An LLM judge for everything. Rejected as the only method. Useless where the truth is a
  count or a keeper's finding.
- A/B tests on live visitors and animals. Rejected. We do not experiment on animal
  health.
- Full human review forever. Rejected. It caps the value of AI at the reviewers'
  attention and never shows which use cases have earned trust.
- One golden set that both tunes and judges a model. Rejected. A threshold chosen on a set
  always looks good on that same set.

## Decision

Every use case is tuned on development data, judged on separate test data before release,
monitored on objective measures after release, and switched off or rolled back by a
stated rule. Worked examples are in [AI evaluation examples](../ai-evaluation-examples.md).

**Data**

- Each use case has a development set and a test set. The development set chooses the
  prompt, the threshold and the calibration. The test set is used only for the release
  decision. Forecasts are split by time: tune on earlier weeks, test on later weeks.
- Both sets hold recorded inputs with the expected output. The reviewing role owns them
  (Zookeeper Staff Manager for animal health and population, Operations Manager for crowds
  and feedback), versioned with the model, prompt and threshold they validated.
- Synthetic cases are labelled synthetic. They test behaviour and are never reported as
  real-world performance.

**Cold start**

- UC-1 scores an animal only after *proposed* four weeks of its feeding records. Until then
  the animal is shown as not scored, and the overdue-feeding rule and keeper rounds carry
  it. The first sets come from past care and veterinary records where the estate has them,
  plus synthetic cases such as three days of reduced eating.
- UC-2 runs in shadow mode. Forecasts are recorded and compared with actual counts, but not
  shown, until it beats its baseline over *proposed* four consecutive weeks.
- UC-3 starts from a fixed set of preference texts and estate states, including closed
  attractions.
- UC-7 counts are shown as unvalidated until their error against keeper spot counts stays
  within a *proposed* 10 per cent for four weeks. Keepers keep counting meanwhile.
- UC-8 starts with one month of feedback labelled by hand.

**Before release**

- Each use case names its measure in [ai-use-cases.md](../ai-use-cases.md), matching its
  cost of being wrong: recall with a precision floor for animal health, forecast error
  for crowding, judged itinerary quality for the itinerary, count error for population,
  and theme agreement for feedback. Accuracy is not a score: anomalies are rare, so a
  model that never flags one scores well on it. The closed-attraction check is a
  deterministic guard and is tested, not scored. Free-text fields use an LLM judge,
  itself checked against a human-rated set.
- A model, prompt or provider change ships only if, on the test set, it scores at least as
  well as the version it replaces, beats its baseline (ADR-017), and meets any floor.

**In production**

| Use case | Objective measures |
|---|---|
| UC-1 | Share of alerts a keeper confirms, missed cases, alerts per keeper per week, abstention rate, input freshness |
| UC-2 | Forecast error once the actual counts arrive, abstention rate |
| UC-3 | Closed attractions removed by the guard, answers to the in-app "was this route useful?" question |
| UC-7 | Count error against keeper spot counts, frames rejected as unclear |
| UC-8 | Agreement with the monthly hand-labelled sample |

- Accepting a task is not the same as confirming the model was right. A keeper may accept
  an inspection task and find nothing. Precision uses the inspection outcome the keeper
  records, not task acceptance.
- A finding recorded with no prior recommendation, such as an anomaly a keeper finds on
  rounds, is a missed case and measures recall. Keeper rounds and the overdue-feeding
  rule (FR-AE-11) stay in place beside the model for this reason. Rejected, corrected and
  missed cases join the next development set.
- The test set is replayed nightly through the production port. This catches a
  provider-side model change without waiting for reviewers to notice.

**When it goes wrong**

- The reviewing role and the engineer on duty investigate together. Input freshness and
  abstention reasons separate bad data from a bad model.
- A use case switches automatically to its fallback when its abstention rate or alert
  volume leaves its expected band for *proposed* two consecutive days, or when the nightly
  replay scores below the released version.
- Rollback restores the previous model, prompt and threshold, which are configuration
  ([ADR-010](adr-010-ai-orchestration.md)). The test set is re-run before the use case is
  switched back on.

**Review**

- Every reviewed use case starts with full review. When acceptance stays above the use
  case's acceptance threshold for a set period, the reviewing role may relax review to
  sampling and let recommendations propagate. Thresholds are per use case, because a
  recall-first use case rejects more by design (*proposed* 99 per cent over four weeks
  for crowds, none yet for animal health). Below the threshold, full review returns.
  Relaxation never touches the actions [ADR-003](adr-003-deterministic-core-advisory-ai.md) protects.

## Consequences

- We can tell a bad model from bad input, and see when a provider update changed
  behaviour.
- Test results mean something, because the data that set the threshold never judges it.
- Nothing reaches staff in the first weeks of UC-1, UC-2 and UC-7. The deterministic
  rules and keeper work carry the estate meanwhile.
- Recall is visible only through missed cases, so keeper rounds cannot be cut back because
  a model exists.
- The sets need an owner who refreshes them, or scores drift from reality.
- The LLM judge is a second model to evaluate. Avoid free text where possible.

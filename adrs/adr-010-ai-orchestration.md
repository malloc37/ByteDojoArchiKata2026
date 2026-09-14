# AI orchestration behind stable interfaces

Date: 2026-09-14

Owner: LJO

## Status

Proposed

## Context

[ADR-003](adr-003-deterministic-core-advisory-ai.md) makes AI advisory and [ADR-015](adr-015-first-release-ai-use-cases.md) picks three use cases. Neither says where the AI
code lives or how it reaches a model.

The models run on a cloud provider's model platform, not yet chosen. Providers change
models and APIs on their own schedule, and [QA-06](../quality-attributes.md#qa-06-evolvability) requires that a provider or model change
touches no domain module. Most AI work is one event in and one recommendation out, but an
overnight anomaly scan or a forecast run can outlive a restart.

Alternatives considered:

- Domain modules call the provider SDK directly. Rejected. A provider change touches every
  module that uses AI.
- A separate AI service. Rejected. No scaling, ownership or reliability reason
  ([ADR-009](adr-009-modular-monolith.md)).
- The provider's hosted workflow or agent product. Rejected. It locks orchestration into
  the provider we want to replace.
- A message broker and a worker pool. Rejected for now. More infrastructure than three use
  cases need.

## Decision

Estate Insights and AI Decision Support is an event-driven module of the cloud modular
monolith that reaches models only through swappable adapters.

- The module consumes published domain events and publishes results or abstentions as
  events with the fields [ADR-003](adr-003-deterministic-core-advisory-ai.md) requires. Consumers are idempotent by event ID
  ([ADR-005](adr-005-integration-through-domain-events.md)).
- State is kept only where needed: projections built from consumed events as model input,
  and process records for long-running processes. A worker moves a process through
  `Requested`, `Running`, `Completed` or `Failed` and resumes it after a restart.
- Each use case has a task-shaped port, for example detect animal anomaly. One adapter per
  provider or model implements it. Adapter, model, prompt version and confidence
  threshold are configuration.
- Models return structured output. Output that fails the schema is retried up to a limit,
  then the use case abstains. An abstention records its reason, invalid output, low
  confidence or incomplete input window, so [ADR-011](adr-011-ai-evaluation-human-review.md)
  scores each apart. If the provider is down, the use case produces nothing (FR-EI-09).
- Requests are sent one at a time.
- [ADR-011](adr-011-ai-evaluation-human-review.md) golden sets run through the same port and adapter as production.

Still open: the cloud provider, the retry limit, and retention of process records.

## Consequences

- A model rollback is configuration, together with the threshold [ADR-011](adr-011-ai-evaluation-human-review.md)
  validated with it. A new provider is a new adapter inside Estate Insights. No other
  module changes.
- Projections fit [ADR-005](adr-005-integration-through-domain-events.md). Process records are the one store not rebuilt from the log;
  they hold progress, not conclusions.
- One request at a time drains a backlog slowly, for example after an estate outage.
  Acceptable, because low AI latency is not a priority.
- A worker pool can come later inside the module. It needs ordered processing per animal
  or area and leased process records, and a new ADR.
- A worker pool alone does not justify a separate service. Extraction needs an
  [ADR-009](adr-009-modular-monolith.md) reason, such as AI load slowing ticket sales, and is then a deployment change.
- A task-shaped port hides provider-specific features. They stay inside an adapter.

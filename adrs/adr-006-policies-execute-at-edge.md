# Cross-domain policies execute at the estate edge

Date: 2026-09-07

Owner: BKA

## Status

Accepted

## Context

We have agreed a set of cross-domain policies, for example `ride fault detected` closes the ride and requests an inspection, and `Animal Feeding Overdue` creates a keeper task. Several of them are safety-relevant. We have not recorded where they run.

Alternatives considered:

- Policies run in the cloud only. Rejected. A ride fault during an internet outage would leave an unsafe ride open until connectivity returns.
- Policies implemented independently at the edge and in the cloud. Rejected. Two implementations of the same safety rule will diverge.

## Decision

The deterministic cross-domain policies execute at the Estate Edge Hub against cached rules, so they work during an internet outage.

- Policy rules are part of the small cloud-to-estate cache, alongside signing keys and validation rules.
- Each rule set has a version. The applied policy version is recorded on the resulting event.
- The cloud does not re-issue a command that the edge has already applied. On ingestion it records the outcome and uses it for reporting and audit.
- Policies triggered by AI findings are the exception. They run wherever the finding is produced, which is normally the cloud, and they only ever produce a task or an alert. See ADR-003.

## Consequences

- The Estate Edge Hub holds real business logic, not only a store-and-forward buffer. It has to be tested and versioned like an application.
- Changing a safety rule needs a distribution and versioning mechanism, and rules can be stale on a disconnected edge for as long as the outage lasts.
- Safety behaviour no longer depends on the sync protocol chosen in ADR-007.
- Recording the policy version on each event makes it possible to explain later why a ride was closed.

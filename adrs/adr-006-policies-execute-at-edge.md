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

Every deterministic policy that must hold during an internet outage executes at the estate against cached rules. The policies that depend on cloud-only data run in the cloud.

| Policy | Trigger and resulting command | Runs in | Why there |
|---|---|---|---|
| A valid eligible ticket permits entry | `Ticket Validated` → Open Gate | The gate, with the use claim on the Estate Edge Hub ([ADR-008](adr-008-signed-offline-ticket-validation.md)) | Admission must not wait on the cloud |
| A ride with a safety fault cannot operate | `Ride Fault Detected` → Close Ride and Request Inspection | Estate Edge Hub | A fault during an outage must still close the ride |
| A ride reopens only after a successful result | `Ride Inspection Completed` → Open Ride | Estate Edge Hub | Same authority as the closure |
| A missed feeding requires attention | `Animal Feeding Overdue` → Create Keeper Task | Estate Edge Hub | Care continues without the cloud (FR-AE-11) |
| Operations must respond to crowding | `Area Capacity Exceeded` → Notify Operations Manager | Estate Edge Hub | Crowding response never waits on the cloud or a model |
| Consequential AI findings require human review | An AI finding that needs a person, such as `Animal Anomaly Detected` → Request Animal Inspection | Estate Edge Hub, when the finding arrives from the cloud ([ADR-007](adr-007-estate-cloud-sync-protocol.md)) | Staff Tasks at the estate owns every task, so a task is created in one place only |
| A confirmed purchase requires ticket issuance | `Payment Confirmed` → Issue Ticket | Cloud, Ticketing | Payment and the signing key are cloud-only ([ADR-014](adr-014-payment-provider.md), ADR-008). No tickets are sold during an internet outage |
| Published availability follows the owning domain | `Ride Status Changed` or `Enclosure Status Changed` → Refresh Published Availability | Cloud, Attraction Catalogue | Visitors read the catalogue in the cloud. The closure itself is already applied at the estate |

- Policy rules are part of the small cloud-to-estate cache, alongside signing keys and validation rules.
- Each rule set has a version. The applied policy version is recorded on the resulting event.
- The cloud does not re-issue a command that the estate has already applied. On ingestion it records the outcome and uses it for reporting and audit.
- AI findings are produced in the cloud but never create a task there. The review policy runs at the estate like every other task-creating policy, and it only ever produces a task or an alert. See ADR-003.

## Consequences

- The Estate Edge Hub holds real business logic, not only a store-and-forward buffer. It has to be tested and versioned like an application.
- Changing a safety rule needs a distribution and versioning mechanism, and rules can be stale on a disconnected edge for as long as the outage lasts.
- Safety behaviour no longer depends on the sync protocol chosen in ADR-007.
- Recording the policy version on each event makes it possible to explain later why a ride was closed.
- An AI finding produced before or during an outage reaches staff only after reconnection. Nothing safety-relevant depends on it.

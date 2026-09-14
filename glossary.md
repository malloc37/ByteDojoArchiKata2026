# Glossary

The most important terms of our ubiquitous language ([ADR-001](adrs/adr-001-use-ddd.md)).
Bounded contexts are listed in [ADR-002](adrs/adr-002-bounded-contexts.md).

| Term | Definition |
|---|---|
| **Admission** | Letting a visitor into the park after a gate has validated their ticket. |
| **Animal** | One individual animal, registered by a keeper and assigned to an enclosure. |
| **Area** | A part of the park where visitors are counted on entry and exit, with a capacity. |
| **Attraction** | A ride or enclosure as visitors see it in the catalogue. Operational contexts say ride or enclosure. |
| **Availability** | Whether a ride or enclosure is open. Ride Operations and Animal and Enclosure Care own it; the catalogue only publishes it. |
| **Enclosure** | A place where animals live, aquatic or land-based. |
| **Estate Device** | A gate, sensor, counter or camera that reports events. |
| **Estate Edge Hub** | The computer on the estate that keeps admission, safety policies and care recording running without the cloud. |
| **Policy** | A deterministic rule: whenever an event happens, issue a command. |
| **Recommendation** | AI output with confidence and source events. Advice only; a person or a policy decides. |
| **Ride** | One of the estate's 40 historic rides. |
| **Safety Closure** | Closing a ride or enclosure for safety. Nothing, including the catalogue, can override it. |
| **Task** | Work for a named person or role, created by a policy. |
| **Ticket** | A signed right to enter on one date, issued only after payment is confirmed. |
| **Visitor** | A person who buys tickets and visits the estate. Events never identify them. |

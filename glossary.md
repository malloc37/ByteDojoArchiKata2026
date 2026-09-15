# Glossary

The most important terms of our ubiquitous language ([ADR-001](adrs/adr-001-use-ddd.md)).
Bounded contexts are listed in [ADR-002](adrs/adr-002-bounded-contexts.md).

| Term | Definition |
|---|---|
| **Abstention** | An AI use case returning no recommendation. Recorded with one of four reasons: invalid output, low confidence, incomplete input window or budget exhausted. |
| **Admission** | Letting a visitor into the park after a gate has validated their ticket. |
| **Admission Staff** | Estate staff who assist visitors and resolve exceptional or rejected ticket validations. |
| **Animal** | One individual animal, registered by a keeper and assigned to an enclosure. |
| **Area** | A part of the park where visitors are counted on entry and exit, with a capacity. |
| **Attraction** | A ride or enclosure as visitors see it in the catalogue. Operational contexts say ride or enclosure. |
| **Availability** | Whether a ride or enclosure is open. Ride Operations and Animal and Enclosure Care own it; the catalogue only publishes it. |
| **Catalogue Manager** | The staff role that creates, updates, publishes and unpublishes visitor-facing attraction information. It cannot override operational availability. |
| **Confidence** | The score on an AI result that is compared with a threshold. Its meaning is defined per use case in [ADR-017](adrs/adr-017-ai-model-approach-and-confidence.md); a model's own stated confidence is never used directly. |
| **Enclosure** | A place where animals live, aquatic or land-based. |
| **Estate Device** | A gate, sensor, counter or camera that reports events. |
| **Estate Edge Hub** | The computer on the estate that keeps admission, safety policies and care recording running without the cloud. |
| **Family pass** | One purchase that issues one signed ticket per family member. Each ticket admits one person. |
| **Finding** | An AI result that needs a person. It carries an ID, provenance and an expiry, and becomes a task at the estate. |
| **Mechanic** | Estate staff authorized to inspect and maintain rides and record the outcome. |
| **Model Provider** | The external platform that runs an AI model. Replaceable without changing any domain module. Not the Payment Provider, which processes payments. |
| **Operations Manager** | The staff role that monitors visitor flow and coordinates park operations and staffing. |
| **Payment Provider** | The external system that processes a visitor's payment and confirms its outcome to Ticketing. |
| **Policy** | A deterministic rule: whenever an event happens, issue a command. |
| **Recommendation** | AI output with confidence and source events. Advice only; a person or a policy decides. |
| **Ride** | One of the estate's 40 historic rides. |
| **Safety Closure** | Closing a ride or enclosure for safety. Nothing, including the catalogue, can override it. |
| **Task** | Work for a named person or role, created by a policy. |
| **Ticket** | A signed right for one person to enter the park once on one date, issued only after payment is confirmed. |
| **Ticket Product** | What a visitor buys: an individual ticket, a family pass or a tier. |
| **Visitor** | A person who buys tickets and visits the estate. Events never identify them. |
| **Zookeeper** | Estate staff who care for animals and enclosures and record observations and completed work. |
| **Zookeeper Staff Manager** | The staff role that oversees animal wellbeing, keeper work and unresolved care concerns. |

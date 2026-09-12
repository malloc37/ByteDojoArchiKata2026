# Functional Requirements

These requirements describe what the Von Digitalis Estate system must do. Safety, reliability, privacy, and performance are documented separately in [Quality Attributes](quality-attributes.md).

## Actors

- **Visitor** - discovers attractions, buys tickets, and visits the estate.
- **Catalogue Manager** - maintains visitor-facing attraction information.
- **Admission Staff** - assists visitors and resolves ticket problems.
- **Mechanic** - inspects and maintains rides.
- **Zookeeper** - cares for animals and enclosures.
- **Zookeeper Staff Manager** - oversees animal wellbeing and keeper work.
- **Operations Manager** - monitors visitor flow and coordinates park operations.
- **Payment Provider** - processes payments.
- **Estate Device** - gate, sensor, counter, or edge gateway that reports events.

## 1. Attraction Catalogue

- **FR-AC-01:** Visitors can view published attractions, including rides and animal enclosures.
- **FR-AC-02:** Visitors can view an attraction's description, location, opening hours, restrictions, accessibility information, and current availability.
- **FR-AC-03:** A Catalogue Manager can create, update, publish, and unpublish an attraction.
- **FR-AC-04:** The catalogue displays ride and enclosure availability received from their owning operational domains.
- **FR-AC-05:** The catalogue cannot override a safety closure made by Ride Operations or Animal and Enclosure Care.

## 2. Ticketing

- **FR-TK-01:** Staff can configure ticket products, including individual tickets, family passes, and ticket tiers.
- **FR-TK-02:** A visitor can select a ticket product and visit date.
- **FR-TK-03:** A visitor can pay through the Payment Provider.
- **FR-TK-04:** The system issues unique tickets only after payment confirmation.
- **FR-TK-05:** A visitor can retrieve the tickets from a completed purchase.
- **FR-TK-06:** The system records payment and ticket status without creating duplicate charges or tickets when a request is retried.

## 3. Admission and Visitor Flow

- **FR-AV-01:** An Estate Device or Admission Staff member can validate a ticket.
- **FR-AV-02:** The system records whether validation succeeded or failed and provides a reason for failure.
- **FR-AV-03:** A valid ticket permits entry according to its date, type, and usage rules.
- **FR-AV-04:** A gate can validate eligible tickets while disconnected from the cloud.
- **FR-AV-05:** A gate records admissions locally and synchronizes them after connectivity returns.
- **FR-AV-06:** The system records park and area entry and exit events.
- **FR-AV-07:** The system maintains current visitor counts for the estate and its areas.
- **FR-AV-08:** The system records attraction queue entry, queue exit, and attraction entry where suitable devices are available.
- **FR-AV-09:** Admission Staff can review exceptional or rejected ticket validations.
- **FR-AV-10:** The system records visitor enclosure entry and exit where suitable devices are available.

## 4. Ride Operations

- **FR-RO-01:** A Mechanic can record the start and completion of a ride inspection.
- **FR-RO-02:** A Mechanic can record the inspection result and observations.
- **FR-RO-03:** Staff or an Estate Device can report a ride fault.
- **FR-RO-04:** A reported safety fault closes the ride according to an explicit safety policy.
- **FR-RO-05:** A Mechanic can record maintenance work and its outcome.
- **FR-RO-06:** Authorized staff can reopen a ride only after the required inspection or maintenance succeeds.
- **FR-RO-07:** Ride Operations publishes ride status changes to the Attraction Catalogue.
- **FR-RO-08:** The system records ride boarding and ride completion where suitable devices are available.
- **FR-RO-09:** A safety closure is applied at the estate even when the cloud is unreachable.

## 5. Animal and Enclosure Care

- **FR-AE-01:** A Zookeeper can register an animal and assign it to an enclosure.
- **FR-AE-02:** A Zookeeper can record feeding time, food offered, food consumed, and observations.
- **FR-AE-03:** A Zookeeper can inspect an animal and record observable health information.
- **FR-AE-04:** A Zookeeper can inspect and clean an enclosure and record completion.
- **FR-AE-05:** Estate Devices can report environmental measurements and animal activity.
- **FR-AE-06:** The system records enclosure population measurements and changes, including jumping piranha population counts.
- **FR-AE-07:** Authorized staff can close or reopen an enclosure to visitors.
- **FR-AE-08:** Animal and Enclosure Care publishes enclosure status changes to the Attraction Catalogue.
- **FR-AE-09:** The system keeps sensor observations, AI inferences, keeper decisions, and confirmed diagnoses distinguishable.
- **FR-AE-10:** A Zookeeper Staff Manager can view current animal concerns, feeding exceptions, inspections, and unresolved care tasks.
- **FR-AE-11:** The system identifies an overdue feeding from recorded feeding times using a deterministic rule, without depending on AI services.

## 6. Estate Insights and AI Decision Support

- **FR-EI-01:** An Operations Manager can view current and historical visitor counts by park area and attraction.
- **FR-EI-02:** The system identifies attraction popularity using recorded visitor-flow events.
- **FR-EI-03:** The system can forecast crowding and recommend staff deployment.
- **FR-EI-04:** The system can detect unusual animal feeding, movement, population, or environmental patterns and request keeper review.
- **FR-EI-05:** The system can recommend a visitor itinerary using attraction status, estimated demand, and visitor preferences.
- **FR-EI-06:** The system can suggest return-visit offers within configured pricing and consent rules.
- **FR-EI-07:** Every AI recommendation includes its confidence, model or rule version, source-data references, and creation time.
- **FR-EI-08:** A responsible staff member can accept, reject, or correct an AI recommendation and record the outcome.
- **FR-EI-09:** Essential ticketing, admission, ride-safety, and animal-care functions continue when AI services are unavailable.
- **FR-EI-10:** The system can return no recommendation when confidence is insufficient, and records the abstention.

## 7. Staff Tasks and Alerts

- **FR-ST-01:** The system creates staff alerts or tasks from explicit operational policies.
- **FR-ST-02:** Authorized staff can acknowledge, assign, update, and resolve a task.
- **FR-ST-03:** The system records who performed each consequential operational action and when.
- **FR-ST-04:** Critical local safety alerts can reach estate staff without depending on cloud AI services.

## Cross-Domain Policies

| Triggering event          | Policy                                         | Resulting command                 |
| ------------------------- | ---------------------------------------------- | --------------------------------- |
| Ticket Purchased          | A confirmed purchase requires ticket issuance  | Issue Ticket                      |
| Ticket Validated          | A valid eligible ticket permits entry          | Open Gate                         |
| Ride Fault Detected       | A ride with a safety fault cannot operate      | Close Ride and Request Inspection |
| Ride Inspection Completed | A ride reopens only after a successful result  | Open Ride                         |
| Animal Feeding Overdue    | A missed feeding requires attention            | Create Keeper Task                |
| Animal Anomaly Detected   | Consequential AI findings require human review | Request Animal Inspection         |
| Area Capacity Exceeded    | Operations must respond to crowding            | Notify Operations Manager         |

## Assumptions

- The estate offers multiple ticket types or tiers, including family passes.
- The estate contains multiple visitor areas.
- Rides and visitor-accessible animal enclosures are catalogue attractions.
- Animal enclosures can be aquatic or land-based.
- Cloud connectivity across the estate can be unavailable or intermittent.
- MQTT-capable estate devices and edge gateways are available.

## Out of Scope Unless Confirmed

- AI making autonomous veterinary diagnoses or treatment decisions.
- AI directly controlling gates, rides, payments, or enclosure safety systems.
- Replacing certified physical ride-control and safety mechanisms.

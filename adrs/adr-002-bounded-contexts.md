# Bounded contexts and domain boundaries

Date: 2026-09-07

Owner: BKA

## Status

Accepted

## Context

ADR-001 committed us to DDD and EventStorming. The EventStorming session produced events, commands and actors, but grouped them as "Ticketing" and "Attraction over Rides and Animals". That grouping is not a set of boundaries. Without named boundaries, every later decision re-argues scope.

Alternatives considered:

- One model for the whole estate. Rejected. Ticket sales, ride safety and animal care have different rules, different authority and different change rates.
- Boundaries along the org chart. Rejected. It makes the visitor-facing catalogue and the operational domains compete for ownership of availability.

## Decision

We use seven bounded contexts:

| Context | Owns |
|---|---|
| Attraction Catalogue | Visitor-facing descriptions and published availability |
| Ticketing | Ticket products, purchase, payment, ticket issuance |
| Admission and Visitor Flow | Ticket validation, gates, park and area movement, queues, occupancy |
| Ride Operations | Ride inspections, faults, safety closure, real ride availability |
| Animal and Enclosure Care | Animals, enclosures, feeding, inspections, cleaning, population |
| Estate Insights and AI Decision Support | Analysis and recommendations over recorded facts |
| Staff Tasks and Alerts | Turning policies into work for people |

Two boundary rules that the EventStorming did not make explicit:

- Ride Operations and Animal and Enclosure Care own real operational availability. Attraction Catalogue publishes it and cannot override a safety closure.
- Queue entry and exit, attraction entry, and enclosure entry and exit belong to Admission and Visitor Flow, not to the operational domain that owns the attraction. They are visitor movement, not operational work.

The digitized model is in `eventstorming/eventstorming-von-digitalis.drawio`.

## Consequences

- Later ADRs name a boundary instead of describing scope again. ADR-009 uses these contexts as the module boundaries.
- `Ride Completed` (Ride Operations) and `Attraction entered` (Admission and Visitor Flow) end up in different contexts. Reporting on a single ride visit has to join across a boundary.
- Estate Insights and AI Decision Support and Staff Tasks and Alerts had no stickies in the physical session. They are reconstructed, so they are the least validated part of the model.

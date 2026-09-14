# Use Domain-Driven Design

Date: 2026-09-07

Owner: LJO

## Status

Accepted

## Context

We are three people with limited time. The brief names business problems, not system
boundaries. The domain is concrete: animals, enclosures, rides, tickets and gates are
physical things with clear owners on the estate, so a method that starts from the
language of the business fits.

Alternatives considered:

- Design from the brief's feature list. Rejected. Every later decision would re-argue
  what a "ride" or an "attraction" includes.
- Model the data first. Rejected. Tables do not show who may change what, and ownership
  of availability and safety closures is the question we most needed answered.

## Decision

We use Domain-Driven Design: one ubiquitous language for the estate, split into bounded
contexts where rules and ownership change. The language is recorded in the
[glossary](../glossary.md).

We ran a physical EventStorming session to discover the first model. All participants
were technical. The digitized result is in [eventstorming/](../eventstorming/).

## Consequences

- Later documents name a context instead of describing scope again. [ADR-002](adr-002-bounded-contexts.md) records the
  boundaries.
- Authority over real availability and safety closures has a clear home from the start.
- The session and the digitizing took time we could have spent on technical design.
- Two contexts had no stickies in the session and were reconstructed afterwards. They
  are the least validated part of the model.

# Visitor itinerary recommendation

This use case improves a visit by suggesting a route through currently available
attractions.

![Visitor itinerary recommendation flow](../diagrams/ai-visitor-itinerary.png)

The model uses attraction availability, queues, crowd forecasts and consented visitor
preferences. With insufficient data or consent it abstains and the application shows the
ordinary attraction list. Before display, a deterministic check removes every closed
ride or enclosure.

- **Authority:** the visitor follows or ignores the suggestion; nothing changes directly.
- **Fallback:** the ordinary attraction list remains available without AI.
- **Risk:** a poor result means a mediocre route, while the hard availability check
  protects safety.
- **Validation:** following rate and a 100 percent pass rate for the closed-attraction
  check.

See [the complete use-case definition](../ai-use-cases.md#uc-3-visitor-itinerary-recommendation),
[ADR-003](../adrs/adr-003-deterministic-core-advisory-ai.md) and
[ADR-013](../adrs/adr-013-privacy-consent-retention.md).

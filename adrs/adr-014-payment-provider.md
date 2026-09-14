# Use an external payment provider

Date: 2026-09-07

Owner: MJE

## Status

Accepted

## Context

Visitors must pay for tickets and family passes. Card handling brings security,
compliance, fraud and refund responsibilities that are not core estate capabilities. A
retry after a timeout must not create a duplicate charge or ticket
([QA-03](../quality-attributes.md#qa-03-integrity)).

Alternatives considered:

- Process and store card details ourselves. Rejected because it greatly increases the
  security and compliance scope.
- Accept only payment on arrival or by bank transfer. Rejected because confirmation is
  slow or manual and does not support a convenient online purchase journey.
- Integrate several providers from the first release. Rejected because it adds complexity
  before there is evidence that provider redundancy is needed.

## Decision

Use one external payment provider with a hosted checkout or provider-hosted payment
fields. Card details go directly to the provider and never pass through or remain in the
estate platform.

- Ticketing creates a payment session using the purchase ID as an idempotency key and
  stores the provider payment reference, amount, currency and payment status.
- A successful browser redirect is not proof of payment. Ticketing marks a payment as
  confirmed only after receiving and verifying the provider's signed callback.
- Callbacks may be repeated or arrive out of order. Ticketing processes them
  idempotently by provider event ID and payment reference.
- A ticket is issued once, and only after the `Payment Confirmed` event
  ([FR-TK-04](../functional-requirements.md#2-ticketing)). Refunds use the same provider
  and are recorded against the original purchase.
- Provider-specific API and callback handling stay behind an adapter owned by Ticketing.
  The concrete provider is selected during implementation based on availability in the
  estate's country, fees and required payment methods.
- If the provider or internet is unavailable, new online payments pause with a clear
  retry message. Already issued tickets and offline gate validation continue unaffected
  ([ADR-008](adr-008-signed-offline-ticket-validation.md)).

## Consequences

- The provider handles sensitive card data, fraud controls and most payment-compliance
  concerns, reducing but not eliminating the estate's compliance obligations.
- Idempotency and verified callbacks prevent ordinary retries from producing duplicate
  charges or tickets.
- Checkout availability and supported payment methods depend on one external provider.
- We must secure callback endpoints, reconcile provider settlements with local records,
  monitor failed callbacks and provide a manual recovery process.
- Changing provider requires a new adapter and checkout integration, but does not change
  Ticketing's purchase and ticket-issuance rules.

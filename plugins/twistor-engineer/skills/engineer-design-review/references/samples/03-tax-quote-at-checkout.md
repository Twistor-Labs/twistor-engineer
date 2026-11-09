# Design: live tax quotes at checkout from an external tax provider

Status: approved by product, engineering review pending. Owner: Payments.

## Context

Checkout computes sales tax from a rate table we update by hand each quarter. We are now selling
in 31 US states and 4 countries, the table is wrong often enough that finance files corrections
every month, and two states have asked about under-collection. We have signed with an external
tax provider that returns a tax quote per cart over HTTPS.

## Scope

- Replace the rate table lookup with a live quote for every cart at the "review order" step and
  again at "place order".
- Record the provider's quote id on the order for filing.
- Out of scope: refunds and tax on returns (next quarter).

## Architecture

### Request path

1. The browser calls `POST /checkout/review`.
2. The checkout service builds the cart, then calls the provider's `POST /v2/quotes` with the
   line items and the ship-to address, using the provider's official HTTP client with its default
   settings.
3. The checkout service waits for the quote, adds tax to the total and returns the page.
4. `POST /checkout/place` repeats the quote call, then charges the card.

The checkout service runs 40 pods; each request handler holds a worker thread while it waits.
Peak is about 1,100 checkout requests per second during sales.

### Provider facts

- Published p50 latency 120 ms, p99 900 ms. Their status page shows three incidents last year,
  the longest 47 minutes of elevated latency.
- Rate limit 2,000 requests per second per account. The client retries 429 and 5xx responses
  three times with exponential backoff and jitter.

### Data

- New nullable column `orders.tax_quote_id` (small, metadata-only change).
- Quotes are not cached: the provider requires a fresh quote per order for filing.

## Rollout

We will deploy the new checkout build to all regions in the Thursday release train. The rate
table code is deleted in the same build, since keeping two tax paths invites drift, and the
rate table itself is dropped in the following release.

## Observability

- New metrics: quote latency histogram, quote errors by status code.
- An alert when quote errors exceed 5% for 10 minutes.

## Testing

- Unit tests with a mocked provider client.
- A staging run against the provider's sandbox with 200 sample carts covering every state and
  country we sell in; totals compared with finance's expected values.

## Security

The provider API key lives in the secrets manager and is injected at pod start. Only the
checkout service can read it.

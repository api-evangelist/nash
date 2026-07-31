---
name: Create and dispatch a Nash delivery order
description: Create an order, quote it across eligible providers, autodispatch the best quote per your dispatch strategy, and track it to completion.
api: openapi/nash-openapi-original.json
operations:
- create_order_v1_order_post
- get_order_quotes_v1_order_get_quotes_post
- refresh_order_quotes_v1_order_refresh_quotes_post
- autodispatch_order_v1_order__string_id__autodispatch_post
- confirm_order_v1_order__string_id__confirm_patch
- get_order_v1_order__string_id__get
method: generated
source: openapi/nash-openapi-original.json + https://docs.usenash.com/guides/checkout-workflow
---

# Create and dispatch a Nash delivery order

Use this to turn a checkout/fulfillment request into a live delivery on Nash.

## Auth
- Send `Authorization: Bearer $NASH_API_KEY` on every request.
- Add `Nash-Org-Id: $NASH_ORG_ID` when the key can access more than one organization.
- Base URL: `https://api.usenash.com/v1` (production) or `https://api.sandbox.usenash.com/v1` (free sandbox).

## Steps
1. **Create the order** — `create_order_v1_order_post`. Provide pickup and dropoff locations (address parts + phone in E.164), package details, and any requirements (proof of delivery, age verification, barcode). Set your own `externalId` so retries are idempotent (a repeat becomes an upsert, not a duplicate). Note the returned `ord_...` id.
2. **Quote it** — `get_order_quotes_v1_order_get_quotes_post` to collect provider quotes; use `refresh_order_quotes_v1_order_refresh_quotes_post` if quotes have expired.
3. **Dispatch** — `autodispatch_order_v1_order__string_id__autodispatch_post` selects and dispatches the best quote per the order's dispatch strategy. If the order requires a confirm step, call `confirm_order_v1_order__string_id__confirm_patch` first.
4. **Track** — `get_order_v1_order__string_id__get` for current state, or subscribe to `delivery` webhooks (see `asyncapi/nash-webhooks.yml`) for `created → assigned_driver → pickup_* → dropoff_complete` transitions instead of polling.

## Rules
- Idempotency: reuse `externalId`; `ORDER_DUPLICATE_EXTERNAL_ID` (409) means two orders share the id.
- Common failures: `NO_ELIGIBLE_PROVIDERS` / `NO_VALID_QUOTES` (404) when no provider/quote fits; `INVALID_ADDRESS`, `INVALID_PHONE_NUMBER`, `INVALID_TIME_CONSTRAINT` (400) on bad input; `ORDER_ALREADY_DISPATCHED` (400) when mutating a dispatched order. See `errors/nash-error-codes.yml`.
- Rate limit: 20 req/s per org; back off on `429` and honor `Retry-After`.
- Log the `X-REQUEST-ID` header on every failure.

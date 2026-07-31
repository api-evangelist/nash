---
name: Schedule a Nash order into a delivery window
description: Find eligible capacity-bounded delivery windows for an order and book, confirm, or revoke a reservation.
api: openapi/nash-openapi-original.json
operations:
- get_eligible_delivery_windows_v1_eligible_delivery_windows_get
- book_delivery_window_v1_order_external_identifier__string_externalId__delivery_window_book_post
- confirm_delivery_window_v1_order_external_identifier__string_externalId__delivery_window_confirm_post
- revoke_delivery_window_v1_order_external_identifier__string_externalId__delivery_window_revoke_post
- create_delivery_window_v1_delivery_window_post
method: generated
source: openapi/nash-openapi-original.json + https://docs.usenash.com/reference/delivery-windows
---

# Schedule a Nash order into a delivery window

Use this for scheduled (not on-demand) deliveries, where an order must land in a bookable, capacity-limited time slot tied to a store location.

## Auth
Same as all Nash calls: `Authorization: Bearer $NASH_API_KEY` and `Nash-Org-Id: $NASH_ORG_ID` when needed. Base `https://api.usenash.com/v1`.

## Steps
1. **(Setup) Define windows** — `create_delivery_window_v1_delivery_window_post` creates a slot for a store location with capacity limits. (Usually configured ahead of time.)
2. **Find eligible windows** — `get_eligible_delivery_windows_v1_eligible_delivery_windows_get` with `order_id` or `external_order_id`; optionally filter by time range and requested capacity. Only windows whose store location and requirements match the order are returned.
3. **Book** — `book_delivery_window_v1_order_external_identifier__string_externalId__delivery_window_book_post` holds capacity in the chosen window for the order (keyed by your `externalId`).
4. **Confirm** — `confirm_delivery_window_v1_order_external_identifier__string_externalId__delivery_window_confirm_post` finalizes the booking and commits the reservation.
5. **Revoke (if needed)** — `revoke_delivery_window_v1_order_external_identifier__string_externalId__delivery_window_revoke_post` releases the held capacity and disassociates the window.

## Rules
- Windows are capacity-bounded; a booking can fail if capacity is exhausted — pick another eligible window.
- `ORDER_VALIDATION_ERRORS_BLOCK_ELIGIBILITY` (422) means the order has validation errors preventing eligibility scoring — fix the order first.
- Bulk dispatch of a window emits a `shift.dispatched` webhook (see `asyncapi/nash-webhooks.yml`).
- Errors and retry guidance: `errors/nash-error-codes.yml`.

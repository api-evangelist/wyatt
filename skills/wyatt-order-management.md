---
name: wyatt-order-management
description: Review a Dr.FORHAIR shopper's recent orders, open one for detail, and cancel it while it is still unpaid — the only reversal the storefront's agent surface exposes.
api: Dr.FORHAIR Storefront MCP Server
endpoint: https://drforhair2024.cafe24api.com/api/mcp
transport: mcp/streamable-http
operations:
  - search-customer-orders
  - search-customer-order-detail
  - cancel-unpaid-order
generated: '2026-09-04'
method: generated
source: mcp/wyatt-mcp-tools.json
---

# Managing Dr.FORHAIR orders

All three tool names were read from a live `tools/list` on 2026-09-04.

## Before you start

- Establish a session (`initialize` → keep `mcp-session-id` → `notifications/initialized`).
- These tools act on **the authenticated customer's own orders**. There is no customer-id
  parameter; the shopper is whoever the bearer token represents. The protected-resource document
  names `mall.read_customer_order` for the two reads and `mall.write_customer_order` for the
  cancellation.

## Steps

1. **List recent orders** — `search-customer-orders`. Every parameter is optional.
   - `period_days` accepts 1–90 and defaults to 30. **90 days is a hard ceiling**: an order older
     than that is not reachable through this surface at all. If the shopper asks about an older
     purchase, say so and send them to customer service rather than reporting "no orders found".
   - `limit` is 1–100, default 10. `offset` pages forward — advance it by the previous `limit`.

2. **Open one order** — `search-customer-order-detail` with `order_id` taken verbatim from step 1.
   The format is `YYYYMMDD-NNNNNNN` (e.g. `20260101-0000001`). Do not construct one.

3. **Cancel, if and only if it is unpaid** — `cancel-unpaid-order` with `order_id`.

## The reversal window — read this before cancelling

`cancel-unpaid-order` works **only while the order is still in the pre-payment (입금 전) state**,
and it takes effect immediately. Once payment has been received the tool no longer applies.

There is **no refund, return, or post-payment cancellation tool on this surface**, and Wyatt
publishes no refund window in any machine-readable form. So:

- Confirm the payment state from step 2 before calling. Do not cancel speculatively.
- If the order is already paid, **stop**. Do not tell the shopper it can be reversed here. Route
  them to Dr.FORHAIR customer service — 1670-5875, weekdays 09:00–17:00 KST (closed 12:00–13:00),
  or `df@wyattcorp.com`.
- Cancellation has no idempotency key. Re-issuing it against an already-cancelled order has an
  unmeasured result; treat one call as one call.

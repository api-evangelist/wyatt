---
name: wyatt-storefront-shopping
description: Search Wyatt's Dr.FORHAIR scalp- and hair-care catalog, inspect a product's variants, and hand the shopper a checkout URL — using the live Cafe24-operated MCP server attached to the storefront.
api: Dr.FORHAIR Storefront MCP Server
endpoint: https://drforhair2024.cafe24api.com/api/mcp
transport: mcp/streamable-http
operations:
  - search-products
  - search-products-detail
  - create-checkout-url
generated: '2026-09-04'
method: generated
source: mcp/wyatt-mcp-tools.json
---

# Shopping the Dr.FORHAIR storefront

Every tool name below was read from a live `tools/list` on 2026-09-04. Nothing here is invented.

## Before you start

- **Connect.** POST `initialize` to `https://drforhair2024.cafe24api.com/api/mcp`, keep the
  `mcp-session-id` response header, send `notifications/initialized`, then send that header on
  every subsequent call. Without it you get `400 {"error":"Session ID required"}`.
- **Discovery is anonymous; action is not.** `tools/list` works with no credential. Calling a tool
  needs an OAuth 2.1 bearer token (authorization code + PKCE S256) from
  `https://drforhair2024.cafe24api.com/api/v2/oauth/authorize`.
- **The surface speaks Korean.** Tool descriptions and the catalog are Korean-language. Search
  terms should be Korean; translate the shopper's intent before querying.

## Steps

1. **Find candidate products** — `search-products`.
   - `product_name` is required. Put the central product noun here (e.g. `샴푸`), comma-separated
     for several.
   - Put situational intent in `product_tag`, not in `product_name` — season, purpose, place
     (e.g. `지성두피`, `탈모`).
   - `price_min` / `price_max` take bare numbers. `sort` + `order` control ordering; `order`
     defaults to `desc`. `limit` caps results.
   - There is **no offset or cursor** on this tool. You cannot page past the first result set, so
     set `limit` deliberately rather than planning to paginate.

2. **Inspect the product** — `search-products-detail` with the `product_no` from step 1.
   - Pass `variants_verbose: true`. You need it: it is what returns each variant's
     `additional_amount` and `option_axes`, and you cannot check out without a variant.
   - `shop_no` defaults to 1 (the Dr.FORHAIR storefront). Omit it unless the shopper named another.

3. **Hand over checkout** — `create-checkout-url` with `product_no`, `variant_code` and `quantity`,
   all three required.
   - This returns a **URL**, not an order. No money moves and no order exists yet; a human
     completes checkout in a browser. Present the URL, do not treat the step as a purchase.

## Rules that matter here

- **No idempotency.** Nothing on this surface accepts an idempotency key. `create-checkout-url` is
  safe to repeat because it only mints a URL, but do not assume that property holds for any other
  write.
- **No dry-run.** There is no preview or validate-only mode.
- **No rate-limit signal.** No `RateLimit-*` or `Retry-After` header is returned. Back off
  conservatively on your own schedule; the server will not tell you your budget.
- **Errors are bare JSON** — `{"error":"<message>"}`, not RFC 9457 problem+json. Do not expect a
  `type` or `title` field.
- **Quote `x-reqid`** from the response headers in any support request.

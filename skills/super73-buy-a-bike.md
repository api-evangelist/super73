---
name: super73-buy-a-bike
description: Search the SUPER73 catalog, build a cart, price a checkout and complete a purchase with explicit human approval, using the store's live UCP/MCP agent commerce endpoint.
api: super73-agent-commerce
generated: '2026-08-29'
method: generated
source: mcp/super73-mcp-tools-list.json (live tools/list, 2026-08-29) + https://super73.com/agents.md
operations:
  - search_catalog
  - get_product
  - create_cart
  - update_cart
  - create_checkout
  - update_checkout
  - complete_checkout
  - cancel_checkout
---

# Buy a SUPER73

Endpoint: `POST https://super73.com/api/ucp/mcp` (JSON-RPC 2.0, MCP). No credential is required to
search, cart or price. Europe: `POST https://eu.super73.com/api/ucp/mcp`, same 13 tools.

## Before you call anything

Every tool requires a `meta` object identifying you:

```json
{"meta": {"ucp-agent": {"profile": "https://your-agent.example/profile"}}}
```

Pass buyer context (`context.address_country`, `context.currency`) so pricing and availability are
correct for the buyer's market — the US and EU stores are separate merchants with separate catalogs.

## Steps

1. **Find the bike.** `search_catalog` with the buyer's intent. `get_product` for the shortlist to read
   variants, availability and price. For browse-only work prefer the cheaper unauthenticated
   `GET https://super73.com/products.json` and `GET /products/{handle}.json`.
2. **Build the cart.** `create_cart` with the chosen variant and quantity. **Persist the returned
   `id` immediately** — see Retries below. Adjust with `update_cart`.
3. **Price it.** `create_checkout` returns line items, totals, discounts and taxes and charges nothing.
   This is your rehearsal step; there is no dry-run flag.
4. **Set delivery.** `update_checkout` with the shipping address and delivery method. Re-read totals —
   tax and shipping change them.
5. **Show the buyer the real number.** Prices come back as integers in ISO 4217 **minor units** paired
   with a currency code: `{"amount": 249900, "currency": "USD"}` is **$2,499.00**. Divide by 100 for
   two-decimal currencies. Never quote the raw integer.
6. **Get approval, then complete.** `complete_checkout` only after explicit, contemporaneous buyer
   approval of the payment. The store's own agent instructions make this a hard rule. If you cannot get
   approval at the moment of payment, do not complete — route the buyer through Shop Pay via
   `https://shop.app/SKILL.md` instead.
7. **Confirm.** `get_order` returns the placed order, but only with a `customer-account-api:full` token
   from the Shopify customer-account OAuth flow. Anonymous agents should hand the buyer the confirmation
   from the `complete_checkout` response instead.

## Retries and idempotency

There is **no `Idempotency-Key` header** on this surface.

- `create_cart` and `create_checkout` are **not** idempotent — replaying either mints a second resource.
  Store the returned `gid://shopify/...` id before you retry anything.
- `update_*`, `cancel_*` and every `get_*` are addressed by id and are safe to retry.
- No `RateLimit-*` headers are returned. The endpoint is rate limited per IP; serialize calls and back
  off exponentially on `429`.

## Undoing it

Know this before step 6, not after:

- Before completion: `cancel_cart` and `cancel_checkout` reverse everything, no money moved.
- After completion: **there is no tool that cancels or refunds an order.** SUPER73's published policy is
  that bike order cancellations must be requested **within 24 hours** of the order being placed, and a
  **15% restocking fee** applies once it has shipped. Accessories, apparel and replacement parts can be
  returned **within 30 days** of the order date (US). All of that runs through human support at
  https://super73.com/pages/support — not through this API.

Tell the buyer the 24-hour cancellation window before they approve payment.

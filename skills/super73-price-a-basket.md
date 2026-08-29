---
name: super73-price-a-basket
description: Get a complete, tax- and shipping-inclusive SUPER73 quote for a set of products without charging anything, using create_checkout as a rehearsal step.
api: super73-agent-commerce
generated: '2026-08-29'
method: generated
source: mcp/super73-mcp-tools-list.json (live tools/list, 2026-08-29)
operations:
  - search_catalog
  - get_product
  - create_cart
  - create_checkout
  - update_checkout
  - cancel_checkout
---

# Price a SUPER73 basket without buying it

The SUPER73 agent surface has no dry-run flag, but `create_checkout` returns a fully computed price —
line items, discounts, taxes — and captures no payment. Payment moves only on `complete_checkout`.
That makes the cart-to-checkout path a safe quoting tool.

## Steps

1. `search_catalog` / `get_product` to resolve each item to a variant.
2. `create_cart` with all line items. Keep the returned id.
3. `create_checkout` from that cart. You now have subtotal, discounts and any automatic promotions.
4. `update_checkout` with the destination address and a delivery method to make shipping and tax real.
   A quote without an address is incomplete — tax is jurisdictional.
5. Read totals, convert from minor units (divide by 100 for USD/EUR), and present the quote.
6. `cancel_checkout`, then `cancel_cart`. Do not leave abandoned checkouts behind.

## Comparing US and EU pricing

The two storefronts are different merchants with different catalogs, currencies and tax treatment. Run
the same flow against `https://eu.super73.com/api/ucp/mcp` for a European quote. Do not convert a USD
total into EUR and present it as the EU price — it will be wrong on both tax and shipping.

## Rules

- Never call `complete_checkout` in a quoting flow.
- Pass `meta.ucp-agent.profile` on every call.
- Serialize and back off on `429`; there is no rate-limit header to read.

---
name: super73-browse-catalog
description: Read SUPER73 product, collection and store data anonymously, with no credential, using the store's published read-only JSON endpoints and Storefront GraphQL.
api: super73-storefront-graphql
generated: '2026-08-29'
method: generated
source: https://super73.com/llms.txt + live introspection of https://super73.com/api/2025-07/graphql.json (2026-08-29)
operations:
  - products
  - product
  - collection
  - collections
  - search
  - predictiveSearch
  - shop
---

# Browse the SUPER73 catalog read-only

If you only need to read, do not use the commerce tools. SUPER73's own `/llms.txt` publishes a
credential-free read path, and it is cheaper and simpler than MCP.

## Plain JSON (no auth, no client)

| Need | Request |
|---|---|
| All products | `GET https://super73.com/products.json` |
| One product | `GET https://super73.com/products/{handle}.json` |
| Products in a collection | `GET https://super73.com/collections/{handle}/products.json` |
| Search | `GET https://super73.com/search?q={query}&type=product` |
| Everything the store publishes | `GET https://super73.com/sitemap.xml` |

## Storefront GraphQL (no auth for reads)

`POST https://super73.com/api/2025-07/graphql.json` with `Content-Type: application/json`.
Introspection and read queries both answer anonymously — no `X-Shopify-Storefront-Access-Token` needed.

```json
{"query": "{ products(first: 10) { edges { node { title handle } } } }"}
```

Useful root fields verified live: `products`, `product`, `collections`, `collection`, `search`,
`predictiveSearch`, `productRecommendations`, `productTags`, `productTypes`, `shop`, `pages`, `blogs`,
`articles`, `menu`, `localization`.

Connections are Relay-style — paginate with `first`/`after`.

## Budget

GraphQL responses carry `extensions.cost.requestedQueryCost` — a calculated query cost, returned in the
body rather than in a header. Track it. The JSON endpoints return no budget signal at all; cache
aggressively and back off on `429`.

## Europe

Swap the host for `https://eu.super73.com` — a separate merchant with its own catalog and currency.

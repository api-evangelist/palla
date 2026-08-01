---
name: Connect two Palla accounts and track transfers
description: Generate a relationship link, resolve the relationship from a shared link, and list transfers for the account.
api: openapi/palla-platform-openapi.yml
operations: [generateRelationshipLink, listRelationships, listTransfers]
---

# Connect two Palla accounts and track transfers

Accounts must share a Relationship before funds can move. This skill establishes the
connection and reads transfer history.

## Prerequisites
- A user-scoped Bearer token (see the onboarding skill).
- Base URL: `https://api.platform.palla.app`.

## Steps

1. **Generate a link** — `generateRelationshipLink` (`POST /v1/links`).
   Returns a unique `lnk_...` Link to share with the counterparty. Handle `409 too many links`
   (rc 23) by deleting unused links first.

2. **Resolve the relationship** — `listRelationships` (`GET /v1/relationships`).
   Call with `?link-id=<lnk_...>` to resolve (and create if needed) a Relationship from the
   other account's link. `422 same account` (rc 1) means the two ids are the same account.
   Call without `link-id` to list all existing Relationships.

3. **List transfers** — `listTransfers` (`GET /v1/transfers?limit=10`).
   Page with `limit`/`offset`; read `meta.itemCount/itemLimit/itemOffset/itemTotal`. Each
   transfer has `txr_...` id, `role`, `amount`, `currency`, and `status`.

## Real-time
Subscribe to the `transfers.create` callback (`asyncapi/palla-webhooks.yml`) — Palla POSTs to
your configured URL on each new transfer; return an empty `200` immediately.

## Conventions
- Send `x-palla-request-id`; responses use the `meta`/`error` envelope.
- See `conventions/palla-conventions.yml`.

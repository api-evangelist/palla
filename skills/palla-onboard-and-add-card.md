---
name: Onboard a Palla user and add a card
description: Exchange partner credentials for a user-scoped token, create the user's Account, and add an encrypted card as their payment method.
api: openapi/palla-platform-openapi.yml
operations: [partnerCredentialExchange, createAccount, createPaymentMethod]
---

# Onboard a Palla user and add a card

Use the Palla Platform Partner API to onboard one end user and give them a payment method.

## Prerequisites
- Partner `client_id` and `client_secret` (backend only).
- Your `user_id` for the end user.
- Base URL: `https://api.platform.palla.app`.

## Steps

1. **Get a user-scoped token** — `partnerCredentialExchange` (`POST /v1/auth/token`).
   From your backend, send `client_id`, `client_secret`, `grant_type: client_credentials`,
   `audience: ["https://api.platform.palla.app/"]`, and `user_id`. Read `data.token`.
   If `error.rc == 123` (`mfa required`), resubmit with `mfa_code`. The token is safe to
   hand to a web/native app for the remaining calls.

2. **Create the account** — `createAccount` (`POST /v1/accounts`) with Bearer token.
   Send `legalName` (first/last required), optional `dob` and `displayName`. This call is
   idempotent: repeating it updates the existing account rather than erroring.

3. **Add a card** — `createPaymentMethod` (`POST /v1/payment-methods`).
   RSA-encrypt the card data client-side first using `PallaFinancial/card-encrypt`; send
   `type: card`, `primary`, and `card: { keyId, data }`. Handle `409 card exists` (rc 23),
   `412 BIN restriction` (rc 65), and `451 OFAC hit` (rc 43).

## Conventions
- Send `x-palla-request-id`; it is echoed as `meta.requestId`.
- Success = `meta.result: success` + `data`; failure = `meta.result: failure` + `error {message, rc, description}`.
- See `conventions/palla-conventions.yml` and `errors/palla-problem-types.yml`.

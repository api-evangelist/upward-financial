---
name: Onboard a consumer onto an Upward credit product
description: >-
  Authenticate as a partner, create a consumer onboarding, walk the KYC /
  terms / bank-link / underwriting steps, and select a credit product on the
  Upward (Upwardli) Credit Suite API.
api: openapi/upward-financial-openapi.json
operations:
  - create-upwardli-token
  - create-onboarding
  - get-onboarding
  - verify-identity
  - accept-terms
  - link-bank-account
  - add-bill-details
  - perform-underwriting
  - select-product
  - submit-loan-request
generated: '2026-07-21'
method: generated
---

# Onboard a consumer onto an Upward credit product

Base URLs: sandbox API `https://api-sandbox.upwardli.com/v2`, sandbox auth
`https://auth-sandbox.upwardli.com/`. Production swaps the `-sandbox` suffix
off both hosts. See `conventions/upward-financial-conventions.yml`.

1. **Get a partner token** — POST `create-upwardli-token` (`/auth/token/`)
   with JSON body `{"grant_type":"client_credentials","client_id":...,
   "client_secret":...,"scope":"api:read api:write"}`. Token lives 24h; send
   it as `Authorization: Bearer <token>` on every call.
2. **Create the onboarding** — POST `create-onboarding`. The response is an
   OnboardResponse state machine: poll it with `get-onboarding` and always
   act on `current_steps` rather than assuming step order.
3. **Verify identity (KYC)** — POST `verify-identity` with the consumer's
   identity data. In sandbox, use the reserved SSN areas from
   `sandbox/upward-financial-sandbox.yml` to force outcomes (e.g.
   `422-XX-XXXX` simulates the fallback-KYC path, `9XX-XX-XXXX` simulates an
   ITIN user with no credit report). Watch `Consumer.KYC.*` webhooks for the
   asynchronous verdict.
4. **Accept terms** — POST `accept-terms` once the consumer has e-signed.
5. **Link a funding account** — POST `link-bank-account` (or drive the Plaid
   flow and verify separately). For bill-based underwriting also POST
   `add-bill-details`.
6. **Underwrite** — POST `perform-underwriting`. Sandbox SSN areas
   `401/406/417/423-XX-XXXX` force the documented underwriting failures;
   `Consumer.Underwriting.Approved|Failed` webhooks carry the result.
7. **Select the product** — POST `select-product`, then, for loan products,
   POST `submit-loan-request`.

Errors are plain JSON with standard HTTP codes (401 = token problem, 403 =
missing scope — see `errors/upward-financial-problem-types.yml`). There is no
idempotency-key mechanism: on a timeout, re-fetch state with
`get-onboarding` before retrying a POST.

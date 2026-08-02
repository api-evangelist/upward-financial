---
name: Manage a credit builder card and simulate transactions in sandbox
description: >-
  Retrieve and control a consumer's payment card (freeze/unfreeze), read its
  transactions, and drive the sandbox simulators for auth, funding, and
  settlement on the Upward (Upwardli) Credit Suite API.
api: openapi/upward-financial-openapi.json
operations:
  - create-upwardli-token
  - get-payment-card
  - freeze-payment-card
  - unfreeze-card
  - list-payment-card-transactions
  - create-payment-card-sim-auth
  - create-payment-card-sim-funding
  - create-payment-card-sim-settlement
generated: '2026-07-21'
method: generated
---

# Manage a credit builder card and simulate transactions in sandbox

Use the sandbox hosts (`https://api-sandbox.upwardli.com/v2`) — the
simulation endpoints exist to exercise card lifecycles without live money
movement (`sandbox/upward-financial-sandbox.yml`).

1. **Authenticate** — POST `create-upwardli-token`; for consumer-facing
   surfaces, down-scope via token exchange
   (`authentication/upward-financial-authentication.yml`).
2. **Read the card** — GET `get-payment-card`; transactions via
   `list-payment-card-transactions` (paginated `count`/`next`/`previous`/
   `results`).
3. **Simulate activity** — in order: POST `create-payment-card-sim-funding`
   to load funds, POST `create-payment-card-sim-auth` to authorize a
   purchase, POST `create-payment-card-sim-settlement` to settle it. Confirm
   each step via `PaymentCard.Transaction.Auth` / `.Settlement` webhooks
   rather than polling alone.
4. **Card controls** — POST `freeze-payment-card` / `unfreeze-card`;
   `PaymentCard.Frozen` / `PaymentCard.Unfrozen` webhooks confirm the state
   change.

Webhook deliveries are HMAC-signed (`Upwardli-Signature`,
`t=<ts>,v1=<sha256>` over `<ts>.<raw body>` keyed with your client id) —
always verify before trusting an event.

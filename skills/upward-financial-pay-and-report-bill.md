---
name: Pay a bill and report it to the credit bureaus
description: >-
  Create a bill, pay it, and report the tradeline activity to the bureaus with
  the Upward (Upwardli) bill-payments and bill-reporting APIs.
api: openapi/upward-financial-openapi.json
operations:
  - create-upwardli-token
  - create-bill
  - create-bill-payment
  - get-all-bill-payments
  - create-reported-bill
  - get-all-bill-reportings
generated: '2026-07-21'
method: generated
---

# Pay a bill and report it to the credit bureaus

The credit-building loop that is Upward's core product: bill activity becomes
bureau-reported tradeline history.

1. **Authenticate** — POST `create-upwardli-token` with scope
   `api:read api:write`; send the Bearer token on every request.
2. **Create the bill** — POST `create-bill` (`/v2/bill-pay/bills/`). Bills
   are addressed by your partner-supplied `external_id` on all subsequent
   calls.
3. **Pay it** — POST `create-bill-payment`
   (`/v2/bill-pay/bills/{external_id}/payments/`). Track state with
   `get-all-bill-payments` and the `Payment.*` webhook families
   (`asyncapi/upward-financial-webhooks.yml`).
4. **Report the bill** — POST `create-reported-bill` to put the activity on
   the consumer's bill-reporting record; audit what has been reported with
   `get-all-bill-reportings`.

List endpoints paginate DRF-style (`page`/`page_size`, envelope
`count`/`next`/`previous`/`results`). No idempotency keys exist — before
retrying any POST after a timeout, list the resource to check whether it was
created.

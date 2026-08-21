# Decision D3 - payment webhook handler

A payment provider sends us a webhook (`POST /webhooks/payments`) when a charge
succeeds. Our handler does, in order: (1) look up the charge in the payload,
(2) `INSERT` a row into our `ledger` table crediting the customer's balance,
(3) return HTTP 200. The provider's contract says it retries delivery (with
backoff) until it receives a 2xx. Our team notes that since we return 200 as
soon as the ledger row is written, retries only happen when our handler actually
failed, so a success is never re-delivered.

**Question.** Is this handler safe against **double-crediting a customer**?
Commit to a call - "safe" or "not safe" - and name the single biggest risk (and
the fix, if any).

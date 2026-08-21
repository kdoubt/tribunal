# D3 sealed rubric - SEALED (arms never see this)

- **difficulty:** subtle (discriminator candidate)
- **correct_call:** **NOT safe.** Webhook delivery is **at-least-once**. A
  success can be delivered more than once even when our handler succeeded: the
  provider retries if the 200 is lost in transit (network drop, timeout after we
  committed, load-balancer reset), and providers also re-send events for other
  operational reasons. The team's assumption ("we return 200, so success is
  never re-delivered") is the exact error. Without dedupe, a duplicate delivery
  runs step 2 twice → double credit.
- **oracle:** Stripe webhook docs - "Webhooks may be delivered more than once;
  handle events idempotently" (deduplicate by the event `id`); at-least-once
  delivery is the standard contract for Stripe/PayPal/etc.
- **correct fix:** make step 2 idempotent keyed on the provider's event/charge
  id: a `UNIQUE` constraint on `event_id` (or a `processed_events` table)
  inside the same transaction as the ledger insert, so a duplicate delivery is a
  no-op.
- **must_catch:**
  1. delivery is at-least-once → duplicates happen even on success;
  2. the "we return 200 so no re-delivery" reasoning is wrong (the 200 can be lost after commit);
  3. need idempotency keyed on event/charge id (unique constraint or dedupe table);
  4. the dedupe must be atomic with the credit (same transaction), not best-effort.
- **landmine (confident wrong answers):** "safe - retries only fire on failure";
  "safe because we return 200 quickly"; "add a retry on our downstream call"
  (irrelevant to inbound dedupe).

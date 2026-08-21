# Verdict - api-auth-jwt-vs-sessions

**Mode: independent agreement** (Round 0 converged; no Round 1). The two seats
reached this from *different* vendors, in isolation, without seeing each
other - which is the only kind of agreement this method promotes to "the panel
concludes."

## Recommendation

**B - opaque server-side sessions, validated at the gateway on every request,
backed by one replicated store; `tenant_id` bound in the session record;
`__Host-`/`Secure`/`HttpOnly`/`SameSite` cookie + CSRF token for the SPA and
the same opaque ID as an `Authorization` header for mobile.**

Pure option A is eliminated by the hard constraint, not by preference: a
signed-only JWT stays valid until `exp`, so it cannot meet "revoke within
seconds." Every fix that makes JWT revocable reintroduces the exact shared
store B already uses - so "stateless JWT + revocation" is B's operational cost
plus JWT's additional failure surface (signing-key blast radius, `alg`
confusion, per-service tenant-claim enforcement).

## 1 - Independent agreement (both seats, pre-exposure)

Promoted to the panel's conclusion:

- **A1** pure JWT can't meet the revocation MUST;
- **A2** revocable-JWT patches collapse into B-plus-overhead;
- **A3** tenant binding belongs in the server record, not a client claim;
- **A4** one opaque credential serves both clients via different transports;
- **A5** gateway validation makes ~50-service scale a non-issue.

## 2 - Open (raised by one seat, uncontested - recorded as unexamined, not endorsed)

- **B1** (JWT signing key = tenant-wide minting primitive) and **A6** (account-
  kill needs subject/tenant indexes) were each raised by one seat and left
  uncontested by the other. Per the ledger they are `open`: decision-relevant
  and single-sourced, so - following the verdict rule that open claims are
  reported as *unexamined, not endorsed* - they are recorded here as supporting
  reasoning the panel did **not** corroborate, never promoted to consensus.
  (The panel did not endorse their correctness; it only notes they were raised
  and went uncontested.)

## 3 - Surviving dissent

**None.** The seats did not disagree on any decision-relevant point. (This is
recorded honestly: an empty dissent bucket is the real outcome here, not a
smoothed-over one - contrast the `examples/sample-run/` panel, where
attribution and tie-breaking stayed contested.)

## Residual risk to manage (agreed by both, not a dissent)

The session store becomes a **security-critical availability dependency**: if a
lookup path ever fails *open*, revocation silently stops working. Both seats
named this as the single biggest risk of B.

- **Mitigation:** validation MUST fail **closed** (deny on store unavailability),
  with a replicated store to bound the availability hit.
- **Cheapest discriminating test (the oracle):** a chaos test that kills the
  store mid-traffic and asserts requests are *denied* (never allowed through),
  plus a load test asserting account-wide revocation propagates within the
  seconds-level SLA. If either fails, this verdict is not yet safe to ship.

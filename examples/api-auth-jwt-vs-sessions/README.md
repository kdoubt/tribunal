# Example run - API auth: stateless JWT vs server-side sessions

A **real** Tribunal panel on a conventional engineering decision, published as
a worked example in the current template format. Unlike
[`../sample-run/`](../sample-run/) - the historical bootstrap that *designed*
the method and predates the finalized templates - this one follows
`core/templates/` as written, and it is on a normal design question rather than
Tribunal itself.

## The decision

A greenfield multi-tenant B2B API (small team, ~50 services behind a gateway,
web + mobile clients) must pick an auth mechanism: **stateless signed JWTs** or
**opaque server-side sessions** - optimizing for security (especially immediate
revocation), blast radius, operational simplicity, and scale. Full brief in
[`brief.md`](brief.md).

## What actually happened

- **Two heterogeneous seats** answered [Round 0](round0-seat-a-codex.md) in
  isolation - **Codex CLI** (Seat A) and **Grok CLI** (Seat B), neither seeing
  the other. Both outputs are verbatim.
- They **independently converged on B** (gateway-validated opaque sessions),
  from different framings and different citations (RFC 7009, RFC 8725 / BCP
  225, OWASP ASVS & Session Management, MDN), and even named the **same single
  biggest risk** - the session store as a security-critical availability
  dependency.
- The [ledger](ledger.md) found **no disputed claims**, so **stop rule (a)
  fired: Round 0 agreed - no Round 1 was run.** Relaying non-disputed claims
  would only invite the politeness convergence the method exists to prevent.
- The [verdict](verdict.md) records this as **independent agreement**, keeps
  two single-sourced claims out of "consensus," and reports an **empty
  surviving-dissent bucket honestly** - then hands the one real residual risk to
  an oracle: a chaos + load test that must show the design fails *closed* and
  revokes within the SLA before it is safe to ship.

## What this example demonstrates

- **Isolation and pre-exposure agreement** - the strongest verdict the method
  produces, and *why* it's trustworthy (two vendors, no cross-talk).
- **The early-stop discipline** - the method does *not* manufacture rounds when
  seats already agree; contrast `sample-run/`, where claims stayed contested
  through cross-examination and dissent survived into the verdict.
- **Grounding** - every claim carries a decisive pointer (a named standard or a
  concrete mechanism), not a vibe.
- **Honest verdict bucketing** - independent agreement, single-sourced
  reasoning, and (here) an empty dissent bucket are each labeled as what they
  are, and the residual risk routes to a deterministic test rather than more
  debate.

## Provenance & sanitization

Run 2026-08-21 with the two CLIs then current. The seat outputs are verbatim
(one leading tool-narration line trimmed from Seat B, noted in-file). No
personal or private data appears - the decision is generic and the seats cited
only public standards. This is a demonstration run, so the retro's T1 (real
outcome) is illustrative rather than a shipped result.

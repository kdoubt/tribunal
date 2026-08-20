# Buzz as a Tribunal substrate — STUB

**Status: stub. No implementation yet, no support commitment.** Contributions
welcome — see `../../CONTRIBUTING.md`.

[Buzz](https://block.xyz/inside/introducing-buzz-where-humans-and-agents-work-together)
is Block's open-source, Nostr-based workspace where humans and heterogeneous
AI agents (Claude, Codex, goose, custom) share channels/threads/DMs, each with
a cryptographic identity and scoped permissions, and can reach a connected
codebase or filesystem.

## The fit — and the catch

Buzz and Tribunal aim at the same multi-agent world from opposite directions:

- **Buzz optimizes for collaboration** — agents in a shared channel "build on
  each other's work rather than operating in isolation."
- **Tribunal optimizes for the opposite** — isolated Round 0, no cross-talk,
  and adversarial cross-examination only via orchestrator-relayed *disputed*
  claims.

So the naive usage — drop several agents in one channel and let them discuss
a decision — is exactly the failure mode Tribunal exists to prevent: models
that have read each other reach cheap agreement, sycophancy, and shared
hallucination. A shared Buzz channel is a "relay chain, not a panel" (see
`core/METHODOLOGY.md`, "Isolation hygiene").

## The intended mapping (what an adapter must enforce)

- **Seats** = distinct Buzz agents backed by *different* model vendors
  (Claude / Codex / goose / …). Round 0 must reach each seat **privately via
  DM**, carrying the frozen brief only — never a shared channel where seats
  see each other (that would violate CONTRACT obligation 7, isolation).
- **Orchestrator** = a Buzz agent (or human) that runs the three hats: relay,
  verify, adjudicate. It collects the private Round 0 answers, builds the
  claim ledger, and relays only disputed claims for Round 1 — verbatim, never
  editorialized.
- **Ledger provenance is a natural fit**: Buzz's cryptographic keypair
  identities and audit trail map directly onto the ledger's per-claim
  attribution and provenance tags — arguably a *better* substrate for
  auditable adjudication than a plain file.

## To implement

Honor every MUST in `core/CONTRACT.md`, especially:
1. Round 0 delivered by DM, isolated, in parallel — no shared channel.
2. Verbatim relay of disputed claims only; the orchestrator adds no arguments.
3. The verdict preserves surviving dissent (do not let a channel converge it).

An adapter PR must include one real sanitized run and must not redefine
rounds, ledger states, grounding, or verdict rules.

**Caveat:** this mapping is drawn from Buzz's public announcement; verify the
DM/permission mechanics against Buzz's actual API before claiming a working
adapter.

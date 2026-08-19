# Sample run — the panel that designed this methodology

A real, unedited two-seat panel (Codex CLI = seat A, Grok CLI = seat B,
Claude Code orchestrating), run 2026-08 with the CLI versions then current.
The question under review: *what should the Pressure-Test methodology
itself be?* The `core/` documents in this repository are the synthesized
verdict of this panel — the method was bootstrapped by running it on
itself.

**This is a historical bootstrap transcript — do not copy its prompt,
relay, or ledger shapes; follow `core/templates/` instead.** It predates
the finalized method and is non-compliant with it in known ways:

- Its Round 1 relayed each seat's **entire** Round 0 essay
  (`r1-template.md`), which the finalized method bans — only disputed
  claims are relayed, verbatim and numbered (METHODOLOGY, "Round 1").
- Its brief has no artifact list, no decision criteria, and no CLAIM
  shape; its Round 0 outputs are essays, not CLAIM blocks.
- Its round names are off by one, and it says "panelist" where the method
  now says "seat":

  | This run said | Current method |
  |---|---|
  | "round 1" (independent) | Round 0 |
  | "round 2" (cross-exam) | Round 1 |
  | panelist | seat |

- Its ledger uses ad-hoc statuses (see the mapping note at the top of
  `ledger.md`).

The transcripts are period artifacts, kept verbatim; observations inside
them (including characterizations of specific models) are historical, not
guidance about today's models.

| File | What it is |
|---|---|
| `brief.md` | The frozen brief both seats received, identically, in parallel |
| `seat-a-codex-r0.md` | Seat A's independent position |
| `seat-b-grok-r0.md` | Seat B's independent position |
| `r1-template.md` | The cross-exam prompt header (each seat got this + the other's full R0 — see non-compliance note above) |
| `seat-a-codex-r1.md` | Seat A attacking/conceding/revising against seat B's position |
| `seat-b-grok-r1.md` | Seat B attacking/conceding/revising against seat A's position |
| `ledger.md` | The orchestrator's claim ledger after both rounds |
| `verdict.md` | The adjudicated verdict (three buckets + surviving dissent) |

Things worth noticing while reading:

- **Round 0 divergence is real**: seat A proposed 3 debate rounds and
  optionally-blinded relay; seat B proposed a 2-round cap and mandatory
  attribution. Neither saw the other before writing.
- **Cross-examination produced actual movement**: both seats conceded
  specific points (A adopted B's stop-rule framing; B adopted A's
  claim-ledger and provenance tags) and both explicitly defended positions
  they kept. One seat's honest "my round-1 position is not included here,
  so I cannot honestly claim a specific revision"
  (`seat-a-codex-r1.md`) is the failure that led the finalized method to
  require relaying each seat its own prior claims.
- **Dissent survived**: attribution, round count, and tie-breaking
  authority stayed contested — and are documented as surviving dissent in
  the methodology rather than resolved by fiat. See `verdict.md`.

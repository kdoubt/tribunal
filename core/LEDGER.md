# The Claim Ledger

The ledger is the panel's memory and the orchestrator's working state. It
lives **outside** the seats' prose - seats never see it whole; they see the
frozen brief, their own prior claims, and the specific disputed claims they
must address. A fillable template with an example row per status is at
`templates/ledger.md`.

Treat context as contested evidence: curated, provenance-tagged, and
aggressively deduplicated. Never relay running transcripts.

## Fields (one row per claim)

| Field | Values / format |
|---|---|
| `id` | Stable across rounds, e.g. `A3` = seat A, claim 3 |
| `author` | Which seat (or `HUMAN` for brief amendments) |
| `claim` | One sentence, quoted verbatim from the seat |
| `evidence` | The seat's pointer: `file:line`, verbatim quote, named symbol, provenance-described capture, or `EXTERNAL` + source - or `ASSUMPTION` / `SPECULATIVE` |
| `provenance` | `USER-FACT` \| `PANELIST-CLAIM` \| `INFERENCE` \| `VERIFIED` |
| `confidence` | A `0`-`1` probability (record the seat's raw token too if it said high/med/low; map high/med/low → `0.85`/`0.6`/`0.3`). Scored against oracle outcomes in the retro |
| `falsifier` | The observation that would prove the claim wrong |
| `status` | See status enum below |
| `decision_relevance` | Does the verdict turn on this? (yes / no) |
| `rebuttals` | Pointers to the Round 1 responses addressing it, one per responding seat |

`UNVERIFIED` is a **relay stamp** applied when a citation check could not be
run (CONTRACT obligation 5) - it travels with the relayed claim but is not a
provenance value or a status.

## Status enum

| Status | Meaning |
|---|---|
| `open` | Stated in Round 0; either not yet compared, or uncontested with nothing turning on it. Open claims never reach Round 1 and are reported in the verdict as unexamined - not endorsed |
| `agreed-r0` | Independently made by all seats in Round 0 - the only status that may be reported as panel consensus |
| `no-stable-position` | A seat declined to commit a claim on an item (two-phase Round 0). Surviving *uncertainty*, not a skip: it blocks `agreed-r0` on that item, is Brier-ineligible (no confidence to score), and carries the item + evidence reviewed + why + the cheapest discriminating test |
| `disputed` | Contradicted by another seat, called into question by a brief amendment, or awaiting a citation check that could not run (relayed with the `UNVERIFIED` stamp) |
| `conceded` | Every seat that contradicted it yielded in Round 1 |
| `overturned` | The authoring seat withdrew or reversed it under attack |
| `verified` | Settled by an oracle (test, compiler, doc, capture) - record which |
| `dropped` | Evidence pointer was checked and failed; removed without debate |
| `surviving-dissent` | Still contested after cross-exam and verification; goes in the verdict's bucket 3 |

## The dispute rule (what gets relayed in Round 1)

A claim is **disputed** iff at least one of:

1. Another seat's Round 0 output contradicts it (directly, or via an
   incompatible recommendation that turns on it);
2. Its citation check could not be run (relay with `UNVERIFIED` stamp - a
   check that ran and *failed* means `dropped`, not disputed);
3. A human brief-amendment calls it into question.

**Only disputed claims are relayed** - numbered, verbatim, with their
original evidence, in one combined, neutral-labeled packet per recipient seat
(author kept in the ledger, not the packet - CONTRACT obligation 4).
Complementary (non-contradicting) claims stay `open` and never reach
Round 1.

Deduplicate before relaying: near-identical claims from multiple seats merge
into one row with all authors recorded (this is the one permitted merge -
of *identical* substance, never of distinct claims). Deduplicate against
**all claims ever seen**, not just live ones - otherwise judge-rejected
claims reappear each round and the panel never converges.

## Round updates

After each round the orchestrator updates statuses, adds any genuinely new
claims (new IDs, never reusing old ones), and re-runs the dispute rule.
A round that adds no new claim IDs is a stop signal (see METHODOLOGY stop
rules).

A genuinely new, decision-relevant, disputed claim appearing in Round 1 is
routed to an oracle if checkable; otherwise it stays `open` and forces a
`human-call` verdict - cross-examining it requires the human's explicit
go-ahead (a brief amendment), never an automatic extra round.

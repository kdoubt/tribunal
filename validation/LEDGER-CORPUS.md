# The ledger corpus - what it can and cannot be used to measure

The published ledgers under [`grounded/results/`](grounded/results/) and
[`../examples/`](../examples/) are the only labelled claim-level data this repo has.
They look like a dataset. This page is what a parse of them actually yields, written
after an attempt to use them as one - so the next person does not repeat the work or,
worse, trust a count that is wrong.

Counted 2026-09-23 against `main` at v1.1.11. Reproduce by binding each table row to
its own header (see "The trap" below); a naive row match will not give these numbers.

## What is there

| ledger | claim rows | status column |
|---|---:|---|
| `grounded/results/g01/ledger.md` | 16 | `status (R0)`, col 7 |
| `grounded/results/g02/ledger.md` | 15 | col 7 |
| `grounded/results/g03/ledger.md` | 16 | **col 6** |
| `grounded/results/g04/ledger.md` | 16 | col 7 |
| `grounded/results/g05/ledger.md` | 16 | col 7 |
| `grounded/results/g06/ledger.md` | 16 | col 7 |
| `grounded/results/g07/ledger.md` | 16 | col 7 |
| `grounded/results/g08/ledger.md` | 16 | col 7 |
| `grounded/results/g09/ledger.md` | 16 | **col 6** |
| `grounded/results/g10/ledger.md` | 8 | col 7 |
| `../examples/sample-run/ledger.md` | 12 | **col 3** |
| `../examples/api-auth-jwt-vs-sessions/ledger.md` | 7 | col 6 |
| `../examples/repo-monorepo-vs-polyrepo/ledger.md` | **0** | `post-R1 status` - see below |
| **total** | **170** | |

Status distribution over those 170 rows:

| status | count |
|---|---:|
| `agreed-r0` | 76 |
| `disputed` | 29 |
| `open` | 24 |
| more than one status word in the cell | 20 |
| `conceded` | 8 |
| no status word in the cell | 8 |
| `surviving-dissent` | 4 |
| `overturned` | 1 |

## The trap: one file, several tables, the same IDs

A ledger is not one table. `g01/ledger.md` holds a 9-column claim ledger
(`| id | author | claim | evidence | provenance | conf | falsifier | status (R0) | dec-rel |`)
**and** a 3-column Round 1 table (`| id | R1 outcome | note |`), and both are keyed by
the same claim IDs (`A1`, `B2`, …).

A parser that matches rows by "first cell looks like a claim ID" therefore reads both
tables as one. That is not a hypothetical: two reasonable parsers written during this
exercise returned **229** and **301** rows for the same corpus, against the true 170,
and reported that 46% of rows had "no status" - those were the Round 1 rows, which have
no status column at all.

**Bind every row to the header above it, and reject rows whose column count differs
from that header.** Without that, the counts are silently wrong in a way that looks
like a data-quality problem rather than a parsing bug.

## Five further shapes that resist naive parsing

1. **The status column is not at a fixed index.** Column 7 in most grounded ledgers,
   6 in `g03` and `g09`, 3 in `examples/sample-run`.
2. **`examples/repo-monorepo-vs-polyrepo/ledger.md` has no Round 0 status at all.** Its
   column is headed `post-R1 status`. It contributes **zero** Round 0 labels and must be
   excluded from any Round 0 analysis.
3. **Round 0 and Round 1 state share a column in some ledgers.** `g03`'s status column
   contains `conceded` - a Round 1 outcome - so a value read from that column is not
   reliably a Round 0 status.
4. **`UNVERIFIED` is not `verified`.** `core/LEDGER.md` is explicit that "`UNVERIFIED`
   is a **relay stamp** ... not a provenance value or a status", and it appears in status
   cells beside a real status (`g08 B2`: `disputed with A4 (precondition strength);
   UNVERIFIED`). A substring match for `verified` matches inside `UNVERIFIED` and
   silently promotes such a row to "multi-status". **Match on word boundaries.** The
   first draft of this page did not, and reported `agreed-r0` 75 / `disputed` 28 /
   multi-status 22 with a spurious `verified` 1. The corrected figures are in the table
   above. That error was caught by an independent re-parse during review, not by the
   author - which is the entire argument for having someone else re-derive the numbers.

5. **Cells contain escaped pipes.** `g05` has `` `\|\| true` `` inside a claim cell. A
   naive `split("|")` shatters that row, its column count stops matching the header, and
   a header-bound parser then *silently drops it*. Unescape `\|` before splitting. This
   page reported 169 rows and `open` 23 until a reviewer's parse found the missing g05
   row; the true figures are 170 and 24.
6. **The status column is sometimes prose, not an enum value.** Eight cells carry no
   enum term at all: `g07` writes `not addressed by B (B6 covers the GB10 proxy only)`,
   and `g09` writes bare `agreed on class (A2)` and `partly agreed (A Q3)` rather than
   `agreed-r0`. These are meaningful to a human and invisible to a matcher.

And 20 of 170 cells carry more than one status word (`agreed-r0 in fact … disputed on
remedy`), because the orchestrator recorded a judgement that was genuinely split. Those
are ambiguous by construction, not by sloppiness.

## What this corpus cannot support

A recall estimate on Round-0 contradictions at a pre-registered bar.

`core/LEDGER.md` gives three routes to `disputed`: another seat's Round 0 contradicts
it (rule 1), a citation check could not run (rule 2, `UNVERIFIED`), or a human
amendment (rule 3). Only rule 1 is a semantic contradiction. The ledgers do not record
which rule applied, so rule-1 rows must be identified by reading each cell.

**The ceiling is 29** - the cells whose status is unambiguously `disputed`, before
removing rule-2/rule-3 cases. The 20 multi-status cells are a separate bucket and are
not part of that 29; admitting them is what would clear the bar, which is the point
made below. For a 95% Wilson lower bound to clear 0.90
*even at perfect recall*, you need **n ≈ 35**:

| n positives | Wilson lower bound at perfect recall |
|---:|---:|
| 19 | 0.832 |
| 25 | 0.867 |
| **35** | **0.901** |

29 is still six short. So a study thresholding recall at 0.90 on this corpus cannot
distinguish "viable" from "not viable" whatever it measures, and clustering by decision widens the interval
further. That is a property of the corpus, not of any method or model tested against it.

Reaching 49 is possible only by admitting the 20 ambiguous cells - which would meet the
power threshold by adding the least reliable labels in the set.

## What it can support

- **Descriptive counts**, as above, with the parsing caveats stated.
- **Prospective work.** Labels accumulate as panels run. A study needing >= 35
  independent rule-1 positives is feasible over time; it is not feasible on this
  snapshot.
- **Inter-rater work**, if a rater who did not operate the orchestrator labels a sample.
  The existing labels were assigned by the orchestrator (Claude Code), which also sat as
  a seat on some of these decisions - `g02/ledger.md` records `Seat A = claude` with
  `Orchestrator = Claude Code fork`, and
  [`grounded/RESULTS.md`](grounded/RESULTS.md) (line 106) notes the orchestrator "was
  a different Claude model and never sat as a seat". Shared lineage, not one session - but
  not an independent reference either.

## Provenance

Produced while reviewing a proposed study that would have used these ledgers as a
labelled dataset. A two-seat panel (local `gpt-oss-120b` + Grok CLI) reviewed that
proposal and recommended against running it on this corpus, on the power and construct
grounds above; this note is the part of that work that stands on its own.

This page was then itself reviewed by a seat (Grok CLI) instructed to re-derive every
figure from the ledgers rather than trust the note. It did, disagreed, and was right:
the `UNVERIFIED`/`verified` substring collision in shape 4 is its finding, not the
author's. The figures above are the corrected ones.

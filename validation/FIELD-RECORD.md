# Field record - 28 operator runs, 2026-08-19 to 2026-09-16

This page reports what the maintainer's own panel archive shows after one
month of use. It is observational and self-reported: no baseline arm, no
sealed truth, and the outcome stamps were written by the same orchestrator
that adjudicated each panel. It is **not** lift evidence and does not change
the [README's](../README.md) no-lift position or the confirmatory
[RESULTS](confirmatory/RESULTS.md). It counts how often rounds and oracles
ran in this archive. The study's Limitations section names grounded review of
a real artifact as unmeasured; this record does not fill that gap.

## Source and method

- 34 panel directories, 28 with a `retro.md`. The 28 were reduced with
  `flywheel-export` (the schema in [`data/README.md`](../data/README.md)), and
  every number in the table is a count over that export. The export carries
  no dates, claim text, question text, or paths; the date range in the title
  and the provenance notes in this section come from the maintainer's archive
  itself and are stated as such.
- Roughly half the runs are engineering decisions on live systems; the other
  half are this method reviewing its own drafts. Both kinds are counted; the
  split is stated so a reader can discount the second.
- Before export, every retro was normalized to the template's bare-value
  rule (annotations moved into trailing comments), and nine runs that predate
  the retro template had their T0 fields written in from their prose retros
  and seat files, marked as reconstructed. A count the retro does not state is
  left blank and exports as null; the "runs reporting" column is the number of
  runs with a non-null value for that field.
- Reproduce on your own archive: `./flywheel-export path/to/panels`.

## What the archive shows

| Measure | Value | Runs reporting |
|---|---|---|
| Rounds run | R0 only 21, R0+R1 7 | 28 |
| Seats per run | 2 seats 14, 3 seats 9, 4 to 10 seats 5 | 28 |
| Vendor mix | Codex and Grok together in 24 of 28; a Claude seat in 9; a local model in 5 | 28 |
| Verdict mode | ship 24, dont 3, human-call 1 | 28 |
| Claims ledgered | 256 | 16 |
| Claims agreed before cross-exposure | 50 | 12 |
| Claims disputed | 39; 9 runs had at least one, and 2 of those 9 stayed at R0 | 12 |
| Claims conceded in Round 1 | 29 | 11 |
| Claims overturned | 11; 7 runs had at least one | 12 |
| Claims settled by an oracle | 15; 10 runs had at least one | 13 |
| Claims settled by debate | 27; 6 runs had at least one | 11 |
| Surviving dissent | 19 claims; 10 runs had at least one | 23 |
| Citations dropped as out of bounds | 2, in 1 run; unverified citations 0 | 12 |
| Seat deaths | 14, across 8 runs | 28 |
| Template delta produced | 18 of 28 runs | 28 |
| T1 outcome: verdict held | yes 14, no 1, no-signal 5, null 8 | 20 |
| Surviving dissent proved right | yes 2, no 3 | 5 |

## Reading it beside the confirmatory study

- **Round 1 use.** In the confirmatory study "seats disagreed on 1 of 20
  decisions", so Round 1 ran once (RESULTS, Headline). Here Round 1 ran in 7
  of 28 runs, and 9 of the 12 runs reporting disputed-claim counts recorded at
  least one disputed claim (2 of those 9 stayed at R0). The study's seats
  "reasoned from model knowledge" with tools unused (RESULTS, Limitations).
  The two rates are quoted, not equated: they count different things (vendor
  disagreement on a sealed decision set; rounds run on mixed live and
  self-review briefs, with 2 to 10 seats), and this record does not show that
  the extra round produced a better call.
- **Oracle use.** 15 claims were settled by an oracle (10 of 13 runs
  reporting) and 27 by cross-examination (6 of 11). The "route disputes to
  oracles" rule is used, not merely stated; the two sums come from different
  reporting subsets and are not a ratio.
- **Seat deaths.** 8 of 28 runs lost at least one seat attempt (14 deaths).
  RESULTS ("The operational tax") reports one vendor's narration-death "on 10
  of 10 known-answer decisions". Those two figures are not a comparable rate,
  and the export records no cause, adapter version, or consequence, so this
  page does not say why they differ.
- **Outcomes.** 14 of 15 scored outcomes held (yes 14, no 1), with 5
  no-signal and 8 null. Fifteen orchestrator-stamped outcomes are a weak
  signal. [`PROTOCOL.md`](PROTOCOL.md) sets "~20+ with real-world outcomes"
  as the bar for "validated" and has scoring "done by an independent scorer";
  a self-graded n=15 is neither.
- **Dissent.** Of the 15 scored runs, `dissent_proved_right` is true in 2,
  false in 3, and null in 10. One of those 10 still exported
  `surviving_dissent: 3`. Too few to say more.

## What this record cannot say

- Whether any verdict beat what a single model with the same tools would
  have returned. No run had a solo arm.
- Whether the T1 grades are right. They were stamped by the orchestrator,
  sometimes weeks later, from what the maintainer could observe.
- Whether a null T1 is pending or unrecorded, or whether a null
  `dissent_proved_right` on a scored run means no dissent, no-signal, or
  unrecorded.
- Anything about seats in general. This is one operator's archive with
  several vendor categories in it; it does not establish general performance
  of any vendor.
- Anything from a small cell. Several exact counts are unique in 28 runs;
  even without direct identifiers, counts and vendor combinations may permit
  linkage to separately published run descriptions
  ([`data/README.md`](../data/README.md), re-identification by joins).

## What producing it changed

- The retro template's bare-value rule was not being followed by the person
  who wrote it: before normalization the exporter returned null for the
  rounds field on 20 of 28 runs (a figure from the maintainer's archive, not
  the export). `flywheel-export` now names on stderr every field it could not
  read (absent, or not a bare schema value), so an archive can be fixed instead
  of silently thinning; a null that is expected, such as dissent before an
  outcome is stamped, is not reported.
- The exporter read `n/a` in the dissent field as "dissent proved right".
  Fixed: `n/a` means no dissent survived and exports as null. One retro
  recorded a confirmed dissent beside zero surviving dissent and was
  corrected to `n/a` before this export.
- The next study this suggests is a grounded ablation: the same real
  artifacts, a solo-with-tools arm beside the panel, outcomes checked by
  someone other than the operator. That is a study, not an update to this
  page.

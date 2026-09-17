# Grounded ablation - results

Registered 2026-09-16 ([`PROTOCOL.md`](PROTOCOL.md), main a779242), amended the
same day before any result was read (Amendment 1: the local Qwen seat
withdrawn; three vendors). Arms ran 2026-09-16 and 2026-09-17. Every raw seat
output, attempt, ledger, Round 1 packet, judge prompt and reply, grounding
resolution, support judgment and log is under [`results/`](results/), redacted
with the sealed placeholder map (client name, addresses, identities, home
paths); the artifacts themselves stay private.

## Headline

**No lift on grounded review, under the pre-registered rule.** Across all
ten decisions, the full panel scored below the union of its own two solo
memos on must-catch rate and about even on decision correctness, while making
fewer false objections.

| Primary endpoint (paired, A_panel − A_2seat) | mean | n |
|---|---|---|
| must_catch_rate | −0.18 | 10 |
| decision_correct, strict rule | −0.06 | 9 (g01 is a design decision, excluded by protocol) |
| decision_correct, sensitivity rule (split union scores 0.5 if either solo was right) | +0.06 | 9 |
| false_objections, mean per memo | panel 1.10 vs union 2.60 | 10 |

Lift required both primary means to be greater than zero at two decimals.
Neither is. The result stands under the strict rule, and the sensitivity
rule does not rescue it (must-catch is negative under both). The
false-objection difference is real and in the panel's favour, and is the
same pattern the confirmatory study saw: the panel is a filter, not a
finder.

## What the grounded setting added

The confirmatory study's seats answered from model knowledge with tools
unused and diverged on 1 of 20 decisions. Here every seat read the real
artifact:

- **Round 1 ran on all ten decisions.** Every panel had disputed claims; every
  Round 1 seat validated on its first attempt; in every ledger the count of
  overturned claims was zero and no verdict input changed direction, so the
  delta-only Round 2 never triggered. Cross-examination produced concessions
  and confidence revisions, not reversals.
- **Grounding is high; support is not.** Mechanical `grounding_rate` (the
  cited `file:line` opens in the artifact) was 0.97 for Claude, 0.96 for Grok
  and 0.83 for Codex. `support_rate` (a non-author judge reads the opened
  lines and says they support the claim) was 0.81 for Grok, 0.67 for Codex
  and 0.50 for Claude. Seats cite real lines far more often than those lines
  carry the claim. This metric was defined for this study and has one known
  limitation (below).
- **The item the original panel missed stayed missed.** g03's sealed rubric
  carries the one must-catch fact the August panel did not find. No arm found
  it either.

## Solo arms by vendor (means over the ten decisions)

| vendor | decision_correct | must_catch_rate | false_objections | grounding | support |
|---|---|---|---|---|---|
| Claude (headless `claude -p`, claude-opus-5) | 0.55 | 0.63 | 2.10 | 0.97 | 0.50 |
| Codex (gpt-6-astra, reasoning high) | 0.65 | 0.78 | 1.90 | 0.83 | 0.67 |
| Grok (Grok 4.6) | 0.85 | 0.77 | 1.30 | 0.96 | 0.81 |

Diagnostic, not a leaderboard: one operator's archive, ten decisions, rubrics
written by the orchestrator, judges rotating among the same three vendors.
Grok's solo beat the panel on decision correctness in this set; the panel's
value here was fewer false objections, not more correct calls.

## Per-decision table

The full table (every arm, its judge, decision_correct, must_catch hits,
false objections, grounding, support) is in
[`results/SCORES.md`](results/SCORES.md). Judge JSON with the quoted spans
behind every point is beside each arm's output.

## Deviations and corrections, in order

1. **Qwen withdrawn (Amendment 1).** The local server was shared with another
   workload and could not hold the 35B model; the owner withdrew the seat. No
   Qwen arm ever ran. The judge table was reassigned under one rule: every
   score already taken stays valid and no vendor scores itself. Pair and judge
   counts are disclosed in the amendment.
2. **Validator false rejections.** The seat-output validator required a
   `CLAIM` heading at column one; Codex numbers its headings. Two complete
   Round 0 answers (g05, g06) were rejected three times each and accepted
   retroactively from their first attempt after the pattern was widened.
   Nothing was re-run. Logged in `results/run.log`.
3. **Codex usage cap.** During the g04 and g08 Round 1 runs Codex returned
   zero bytes three times each with a subscription-limit error. The packets
   were frozen with hashes; the seats were resumed the next day (attempt 4,
   validated) with the other seat's Round 1 still unread. Recorded in each
   `transformations.log`.
4. **`support_rate` window.** The first support-judging pass gave the judge
   about six lines around each pointer and was discarded before any number was
   reported; the second pass hands the full cited range plus context. The
   discarded pass is kept under `results/_superseded/`. Even widened, a judge
   reading an excerpt can miss support that lives elsewhere in the file, so
   `support_rate` is a lower bound.
5. **g04 grounding.** That decision's evidence is a capture pack inlined in
   the brief; seats quoting the pack have no `file:line` to open, so g04's
   grounding column reflects only source-file citations (Codex cited none).
6. **The Claude seat's model.** The headless seat ran the host's default
   `claude -p` model, claude-opus-5, recorded in `results/MODEL-VERSIONS.txt`.
   The orchestrator was a different Claude model and never sat as a seat.

## Limitations

- Ten decisions from one operator's archive; the rubrics were derived by the
  orchestrator from outcomes it had itself adjudicated, mitigated by sealing
  before any run, mechanical pointer verification, and a two-seat review of
  the rubrics before registration. That review's surviving dissent asked for
  an independent semantic audit of every cited passage; that audit was not
  done, and `support_rate` is the study's substitute.
- Codex and Grok had seen eight of these artifacts as seats in August; both
  CLIs are stateless, so what can carry over is the model's disposition, not
  the rubric.
- Judges rotate among the three vendors being judged. No fourth party scored
  anything.
- The union arm's false-objection count is an upper bound (the two solos'
  counts summed without deduplication where no direct judgment existed;
  direct judgments were used where present).

## What this changes in the repository

Nothing in `core/`. The README's position, no accuracy lift, now rests on a
grounded study as well as the knowledge-only one, with one sharpened
statement: on real artifacts the panel's cross-examination runs every time
and reverses nothing, and its measurable contribution is fewer false
objections.

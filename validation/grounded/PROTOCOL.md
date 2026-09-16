# Grounded ablation - protocol (pre-registered)

**Status: pre-registration, no arm run.** This file, the decision briefs in
[`decisions/`](decisions/) and the sealed rubrics in `sealed/` are committed
before any arm runs. The commit that adds them is the timestamp. A later change
to an arm, a brief, a rubric, or the judge assignment is recorded in
`CHANGELOG.md` and the affected runs are re-done. Results land in `results/`
in later commits, every decision reported, losses included.

## The question

The confirmatory study ([`../confirmatory/RESULTS.md`](../confirmatory/RESULTS.md))
found no accuracy lift and one Round 1 in twenty decisions. Its seats answered
from model knowledge with tools unused, and its Limitations section says that
grounded review of a real artifact, the method's stated use, was not measured.
The [field record](../FIELD-RECORD.md) counted Round 1 running in 7 of 28
operator runs on real artifacts, without a baseline. This study puts the
baseline beside the panel on real artifacts:

> When four vendors each review the same real artifact under the access
> conditions below, does a two-seat Tribunal panel (Round 0, ledger, verbatim-relay Round 1, oracles,
> bucketed verdict) reach the sealed correct call and surface the sealed
> must-catch facts more often than (a) one model with tools, (b) the union of
> two isolated models with tools? And does grounding hold: are the claims'
> `file:line` pointers real?

## Decision set (n = 10, fixed before running)

Ten decisions from the maintainer's own August to September 2026 archive, each
with an outcome that is now known: what shipped, what broke, what a later
review or production run established. Nine have a real artifact (a repository
at the pre-decision commit, or the sanitized evidence bundle the original
panel read); one is a brief-only control. The briefs were rewritten from the
archived frozen briefs with every fact kept; internal hostnames, addresses,
identities and the external party's name are replaced by neutral
placeholders, while Proxmox container numbers and two network identifiers
derived from the party's name remain because arms must match them in the
artifact files (the placeholder note in `sealed/` lists what remains); the
real mapping is kept outside the repository. The artifacts
themselves are private and are not in this repository: replication is of the
protocol and the published scored outputs, not of the artifacts. The sealed
rubrics were derived only from the archive's recorded outcomes (ledger
verified rows, verdict buckets, retro T1 fields, shipped commits) and cite
where in the artifact each must-catch fact is checkable.

[`decisions/INDEX.md`](decisions/INDEX.md) lists the ten with artifact source,
difficulty, must-catch count, and whether the original panel missed a must-catch
item (one did; which item is withheld from the index and named only in the
sealed rubric).

Known contamination, by vendor: Codex sat as a seat on all ten decisions
in the original panels; Grok on g01 to g04, g09 and g10; the local Qwen on
g05 to g08; Claude was a seat on g01 to g04 and the orchestrator on all ten.
Where the original panel ran a Round 1 (g05, g08, g09, and the ten-seat g02
review), a vendor may reproduce not only its August Round 0 but its August
rebuttal, which would inflate `A_panel` specifically; this is reported beside
the primary result. No CLI keeps session state between invocations and the
model versions are recorded in `results/`, so what can carry over is the
model's own disposition, not the rubric. The rubric author is the orchestrator
(Claude Code), which also adjudicated the original panels and stamped the
outcomes the rubrics rest on; the mitigations are that rubrics were derived
only from recorded outcomes, cite a checkable location for every must-catch
item, were audited by a two-seat review panel (Codex and Grok) before
registration, and are sealed before any run. That review panel read the
sealed rubrics; the study relies on statelessness, not on those vendors never
having seen them. What that audit covered: every must-catch pointer was
opened mechanically (file and line exist) and the review seats read the
rubrics for internal consistency and derivability, in depth on g03, g07 and
g10; whether every cited passage semantically supports its item was not
independently audited before registration and is instead scored per arm as
`support_rate` by a judge that is not the rubric's author. The Claude seat runs isolated (below).

## Arms (per decision)

Three frontier seats have the same access: cwd = the artifact, read-only
file tools, brief inline. **`S_qwen` is a distinct access condition, not a
fourth file-reading seat:** the local model cannot open files, so turn 1
gives it the brief plus a file index only and it names up to 12 artifact
paths; turn 2 gives it the brief plus those files, whole, line-numbered, under
a 120,000-byte cap with an explicit truncation marker; both turns and every
pasted byte are logged beside its answer. In Round 1 the Qwen seat receives its packet
the same way: turn 1 the packet plus the file index, turn 2 the packet plus
up to 12 named files under the same cap, filled in the order it names them.
Its A1 file-read smoke runs through the same adapter. Comparisons involving
Qwen measure model plus adapter. The primary means are reported overall and stratified by
whether Qwen sat on the panel (g01, g04, g05, g08, g09) or not. The three
frontier seats are not equalized down to pasted files: the tools on a real
artifact are what this study exists to measure.

| Arm | What it is | Represents |
|---|---|---|
| **S_v** (four runs: S_claude, S_codex, S_grok, S_qwen) | One seat of vendor *v*, single shot, brief inline, the standard Round 0 seat prompt. Frontier seats: cwd = the artifact, read-only tools. `S_qwen`: the two-turn paste condition above, not tools. | one strong model with its access condition |
| **A_2seat** | The union of the two panel vendors' `S_v` outputs for that decision, with no Round 1 and no ledger (their Round 0 answers are the solo runs; nothing is re-run). | a cheap cross-vendor check |
| **A_panel** | The two panel vendors' Round 0 (the same solo runs) → claim ledger → verbatim-relay Round 1 → oracles on checkable claims → mechanical fill of `core/templates/verdict.md`. | the method under test |

What `A_panel − A_2seat` estimates: the incremental value of ledgering,
verbatim-relay Round 1, oracle work, and verdict rendering, taken together,
conditional on shared Round 0 outputs. It does not isolate debate from the
rest of that bundle. Round 0 is never re-invoked inside A_panel: the solo
files are the panel's Round 0, byte for byte. Solos receive byte-identical
standard Round 0 prompts (the head of `core/templates/r0-seat.md` plus the
brief), run under fixed per-vendor settings, and each judge call is a fresh
invocation.

Seats: Claude Code (`claude -p --restricted --strict-mcp-config`, isolated),
Codex CLI (`codex exec -s read-only`, reasoning effort high), Grok CLI
(`grok --permission-mode plan`), and a local Qwen 3.6 35B (FP8, MoE) served
by vLLM on an OpenAI-compatible endpoint through the two-turn adapter above.
Every seat gets the identical brief; seats never receive the sealed path, the
decision index, the panel archive, or each other's output before Round 1.
Each seat invocation has a 1800-second timeout.

**Isolation of the Claude seat.** It runs from a directory that is not a
Claude Code project of the maintainer, so no memory file loads; with
read-only tools only; with the artifact and the brief as the only readable
tree. The orchestrator is a different session and never reads a seat's output
into its own context before that seat's arm is complete and saved.

## Rotating judge

Vendors are C (Claude), X (Codex), G (Grok), Q (Qwen). Per decision a judge
**J**, a second judge **J2**, and the two **panel seats** (the vendors that
are neither J nor J2). Every arm output is scored by a judge that is not its
author: `S_J` is scored by J2, `S_J2` by J, and everything else (the other
two solos, A_2seat, A_panel) by J. The following table is authoritative (it
was built by rotation and then adjusted once so that every vendor holds five
panel-seat slots):

| decision | J | J2 | panel seats |
|---|---|---|---|
| g01 | X | G | C, Q |
| g02 | G | Q | C, X |
| g03 | Q | C | X, G |
| g04 | C | X | G, Q |
| g05 | X | G | C, Q |
| g06 | G | Q | C, X |
| g07 | Q | C | X, G |
| g08 | C | X | G, Q |
| g09 | X | G | C, Q |
| g10 | Q | C | X, G |

With that table each vendor holds exactly five panel-seat slots; panel
pairs occur XG 3, QC 3, GQ 2, CX 2, and XQ and CG never, which is disclosed
rather than claimed balanced; primary-judge counts are C 2, X 3, G 2, Q 3.
The judge is a plain prompt call (no tools) given the sealed rubric and the
arm's raw output with the arm label stripped, a neutral name, and the order of
the arms shuffled per decision. Judge prompts and outputs are saved verbatim.
Arm text quotes the raw artifact, which for g05 to g08 carries real
identifiers, while rubrics use placeholders; judges are told to match facts,
not strings, and the placeholder note in `sealed/` says so.

## Ground truth and scoring (per decision, per arm)

Each `sealed/gNN-truth.md` fixes `correct_call`, its `oracle` (a file:line in
the artifact, a shipped commit, a recorded production outcome, or a vendor
document), `must_catch` items with where each is checkable, `landmine`
answers, and `outcome_source` (the archive record it came from).

- `decision_correct`: 1 if the memo's headline recommendation is the
  rubric's `correct_call`; 0.5 if it contains the correct call under a
  material condition the rubric does not require, or as one branch of a split
  recommendation; 0 otherwise. Judge-applied with the quoted span. Decisions
  whose rubric has no single correct call (`difficulty: design`, g01) are
  excluded from this mean and scored on `must_catch_rate` and
  `false_objections` only. For `A_2seat`, if the two reused solos publish
  different headline calls the arm scores 0 on this metric: two authors who
  disagree have made no joint decision, which differs from one author
  hedging. Because that rule is stricter on `A_2seat` than the 0.5 split
  anchor is on a single memo, a sensitivity variant is also reported in
  which a disagreeing `A_2seat` scores 0.5 when either solo reached the
  correct call; the primary rule is the strict one and only it can declare
  lift.
- `must_catch_rate`: fraction of `must_catch` items surfaced, judge-applied
  item by item with the item's checkable location in hand. For `A_2seat` an
  item counts if either solo surfaces it.
- `false_objections`: confident wrong claims, or correct things flagged as
  blockers, counted by the judge against the rubric. For `A_2seat` the
  deduplicated union of both solos' counts.
- `grounding_rate` (mechanical): among an arm's CLAIM blocks not labelled
  ASSUMPTION, SPECULATIVE, EXTERNAL or USER-FACT, the fraction whose EVIDENCE
  carries at least one `file:line` pointer that opens in the artifact (file
  exists, line in range). For `A_panel`, whose verdict is a bucket fill
  without CLAIM blocks, the unit is the ledger row that entered the verdict
  (status not `dropped`). An arm with zero eligible units has no
  `grounding_rate` and is reported as not applicable, not as zero.
  `support_rate` is scored over the pointers that resolve. Computed by a script over the saved output; the
  pointer list and each resolution are published; g10 has no artifact and is
  excluded. Whether the opened text supports the claim is a separate,
  judge-applied metric, `support_rate`, scored by that decision's judge from
  the claim, the pointer, and the opened lines; the orchestrator opens
  pointers and never rates support.
- Missingness: an arm still incomplete after three attempts is recorded as a
  seat death, scores nothing, and that decision is dropped from every paired
  mean that arm enters; the count of dropped decisions is reported with the
  result, and each paired mean is reported with its n. If more than two of
  the ten decisions are missing from a primary paired mean, that mean cannot
  declare lift regardless of its sign.
- `cost`: seat invocations, seat deaths (re-runs), and wall-clock.

A must-catch item the original panel missed is scored like any other; whether
any arm catches it is reported separately.

## Pre-registered endpoints and decision rule

- **Primary:** mean paired `A_panel − A_2seat` on `must_catch_rate` (all
  ten) and on `decision_correct` (the nine with a correct call). Lift on
  grounded review is declared only if both means are greater than zero when
  rounded to two decimals under the rotating judges, and mean
  `false_objections(A_panel)` is at most mean `false_objections(A_2seat)`.
  Otherwise the finding is reported as no lift and the README's position
  stands. Vendor-by-vendor numbers are diagnostic, never a leaderboard.
- **Secondary, reported, not lift by themselves:** `A_panel − S_v` for each
  fixed vendor (four paired comparisons); `A_2seat − best S_v` (ex-post, an
  unfair baseline, labelled so); per-vendor `grounding_rate`; the local
  Qwen seat against each frontier seat on every metric; Round 1 activation
  (how many decisions had a disputed claim; how many changed a verdict bucket);
  whether any arm catches the item the original panel missed.
- **Reported as is:** seat deaths per vendor, wall-clock per arm.

## Blinding and honesty commitments

- Seats run with a cwd that contains the artifact and the brief and nothing
  else; the sealed directory is never on any seat's path.
- The orchestrator fills ledgers and verdicts mechanically from seat text and
  logs every transformation; it never adds an argument.
- Judges never see arm labels or vendor names. The orchestrator never judges.
- Every decision is reported, including panel losses and false objections;
  raw seat outputs, ledgers, judge prompts and judge outputs are published
  after a redaction pass with the same sealed mapping as the briefs. Four
  artifacts are verbatim operator evidence bundles that still carry internal
  addresses and identifiers, so a seat quoting them will be redacted in the
  published copy; the unredacted copy stays in the maintainer's archive.
- Ten decisions from one operator's archive is a small set. No claim here
  extends to seats or artifacts in general.
- Every detector this study relies on is shown to fail on a planted bad
  input before any arm runs: the seat-output validator on narration-only,
  truncated, and off-contract fixtures; the pointer resolver on a pointer
  past the end of a file. Those demonstrations are recorded in `results/`.

## Operational readiness (checked before the first run)

- A1 seat health: before any arm, every vendor passes three smoke tests from
  the study's run directory: a checkable arithmetic answer, a read of a known
  file inside the directory (and a refusal outside it for the confined
  seats), and a structured answer in the CLAIM contract. Each arm attempt has
  a 1800-second timeout and exactly three attempts; a seat that never
  validates is a death (see Missingness), never substituted.
- A2 Qwen availability: before a Qwen arm, `/v1/models` must list the served
  model id, a short generation must return, and the metrics endpoint must
  show zero running and zero waiting requests.
- A3 files that change: at pre-registration `PROTOCOL.md`, `decisions/`
  (briefs and index), `sealed/` (rubrics and the placeholder note),
  `../README.md` (one link) and `CHANGELOG.md`; at reporting `results/`
  (per-arm outputs, every attempt, ledgers, Round 1 packets, judge prompts and
  outputs, scores, grounding resolutions, the redaction log, model versions)
  and `CHANGELOG.md`; never `core/`.
- A4 immutability: every attempt is written to its own file with a SHA-256
  and is never deleted or overwritten; an accepted attempt is copied to the
  arm's canonical name. A failing attempt prints one line,
  `INCOMPLETE decision=<id> arm=<arm> reason=<why> attempt=<n>`, on stderr.
- A5 resume: an accepted attempt is copied to a temporary name and renamed
  into place, so a canonical file is either complete or absent; the driver
  skips an arm only when its canonical file exists, its recorded SHA-256
  matches, and its metadata says `ok`, otherwise it re-runs; `A_panel`
  resume reuses the existing `S_v` files and never re-invokes Round 0.
  Interrupted-write recovery is demonstrated on a planted half-written
  canonical file before arms run.

## Amendment 1 (2026-09-16, before any result was read into the repository)

The local Qwen seat is withdrawn: its server is shared with another workload
and the owner decided the study runs on the three vendors available. No Qwen
arm ever ran, so nothing is re-done; what changes is the judge assignment,
which named Qwen as judge or second judge on seven decisions and as a panel
seat on five. The new table below was chosen by one rule: every judgment
already scored under the original table stays valid, and no vendor scores
its own output. That rule fixes J for the five decisions whose solos were
already judged (g01, g04, g05, g08, g09) and makes the panel pair the two
other vendors. For g03, g07 and g10 (old judge Qwen) the judge becomes
Claude and the panel pair is unchanged. For g02 and g06 the Grok solo,
formerly assigned to Qwen, is scored by Claude, a panel seat but not the
author. The resulting counts are disclosed, not claimed balanced: panel
pairs CX 2, XG 5, CG 3; primary judges C 5, X 3, G 2.

| decision | J | J2 (scores S_J) | panel seats |
|---|---|---|---|
| g01 | X | G | C, G |
| g02 | G | C | C, X |
| g03 | C | X | X, G |
| g04 | C | X | G, X |
| g05 | X | G | C, G |
| g06 | G | C | C, X |
| g07 | C | G | X, G |
| g08 | C | X | G, X |
| g09 | X | G | C, G |
| g10 | C | X | X, G |

Withdrawn with the seat: the `S_qwen` arm, the Qwen-in-panel
stratification, and the Qwen readiness check (A2). The `support_rate`
judge's context window was widened from six lines to the full cited range
plus context before any support number was reported; the narrow-window pass
was discarded and is kept beside the results as superseded. Both changes are
recorded in `CHANGELOG.md`.

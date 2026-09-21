# Changelog

All notable changes are recorded here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning follows
[Semantic Versioning](https://semver.org/spec/v2.0.0.html) as scoped for a
methodology repo in [`CONTRIBUTING.md`](CONTRIBUTING.md#versioning). Changes
accumulate under **Unreleased** and are stamped with a version and date when
a release is cut; published tags are immutable.

## [Unreleased]

**Three PATCH items found by running this repo's own method on it (panel
2026-09-20, Codex CLI + Grok CLI, R0+R1). The panel's question was whether to
adopt mechanisms from an external coordination project into `core/`; both seats
independently answered no on every candidate, and the only defects that survived
were in this repo's tooling and adapters. No method change; `core/` untouched.**

- `flywheel-export`: `template_delta` is the one repeatable retro field ("one
  concrete proposed edit per line"), and five retros in the maintainer's archive
  use it that way. The exporter stored fields last-wins, so it read only the
  FINAL `template_delta` line: a retro carrying real deltas followed by a
  "no change" line exported `had_template_delta:false`, inverting the
  loop-closer signal and silently discarding the rest - with no stderr warning,
  since the field was present and readable. Same class as the 1.1.6
  `dissent_confirmed_for` `n/a` bug. Now accumulated across every
  `template_delta` line: any real delta wins, all-"no change" stays `false`,
  absent or empty stays `null`. stdout is unchanged otherwise and still emits
  controlled metadata only - the bool never carries delta text. First seen on a
  live retro; the trigger is reproduced in-tree by
  `test/flywheel/retro-multi-delta.md`, which the pre-fix exporter scores
  `false`. Re-exporting the maintainer's 37-run archive before and after the fix
  produces byte-identical output, so no historical record changes. PATCH (tooling).
- CI: a regression test for `flywheel-export` - `test/flywheel/retro-multi-delta.md`
  (three real deltas plus a trailing "no change" line) must export
  `had_template_delta:true`, and `test/flywheel/retro-nochange.md` must still
  export `false`, which catches an over-correction that counts lines instead of
  real deltas. The step also fails if any delta text reaches stdout. Verified to
  FAIL against the pre-fix exporter, not merely to pass against the fixed one.
  No new tracked script; the helper policy is unchanged.
- `validation/FIELD-RECORD.md`: refreshed from 28 runs (to 2026-09-16) to **37
  runs (to 2026-09-21)** over the maintainer's archive, same method - one
  `flywheel-export` reduction, controlled metadata only. Round 1 now ran in 12
  of 37 (was 7 of 28) and a **Round 2 fired for the first time** in this
  archive; `decide-after-check` appears as a verdict mode (5 runs) where the
  first edition recorded none. Claims ledgered 476, agreed-before-cross-exposure
  146, oracle-settled 48, debate-settled 36, surviving dissent 27. Still
  observational and self-reported, still not lift evidence; the README's no-lift
  position is unchanged. PATCH (validation material; `core/` untouched).
- `validation/adapter-probes.md`: **new** - the re-runnable evidence behind the
  two adapter rules changed in this release, with verbatim dated transcripts and
  CLI versions. Published because the first draft of these entries cited a
  private panel directory no reader could check. Explicitly not a benchmark and
  not a vendor comparison (`CONTRIBUTING.md` declines both); the finding is that
  an adapter must not assume one CLI shape. PATCH.
- `adapters/shell/README.md` "Silent seat killers" 1 and
  `adapters/claude-code/SKILL.md` read-test bullet: drop the instruction to tell
  seats to prefer built-in read tools "over shell commands". For some CLIs the
  read path *is* shell inside the read-only sandbox (Codex under `-s read-only`
  exposes no separate file-read tool), so that phrasing makes the seat refuse
  the read - and the refusal then reads as the host-side read interception the
  read-test exists to detect. The read-only mode is the control, not the seat's
  choice of tool. Transcripts published in
  [`validation/adapter-probes.md`](validation/adapter-probes.md) (P1-P3): the
  same constrained prompt succeeds on a CLI that does have a native read tool,
  so the defect is the assumption, not the prompt. PATCH.
- `adapters/claude-code/SKILL.md` seat-command example: adds
  `--skip-git-repo-check` to the Codex invocation, which the shell adapter has
  shipped since its examples block (`adapters/shell/README.md:17`, `:124`) but
  the Claude Code adapter omitted. Inside a git repo the flag is a no-op, so a
  panel run from a project root is unaffected; from a directory Codex does not
  trust (a panel/scratch dir) it prints a trust warning and exits **0 having
  answered nothing**. Measured both ways; transcripts in
  [`validation/adapter-probes.md`](validation/adapter-probes.md) (P4-P5). The smoke test catches this
  only because the answer is missing, which the bullet now says. Adapter parity, in the shape
  of 1.1.3. PATCH.

## [1.1.8] - 2026-09-17

- README "Honest validation status": the sentence saying grounded review "was not measured" is replaced by the grounded study's result with its numbers and caveats (nine artifact-backed decisions plus one brief-only control; three vendors after the amendment; no lift; union false-objection count an upper bound; Round 1 overturned no claim). A proposed claims sentence ("a filter, not a finder") was dropped on a two-seat panel's finding that it would change the frozen claims. Status now counts "two pilots and two pre-registered studies". `validation/README.md` and `validation/grounded/RESULTS.md` deviation 3 corrected likewise (the resumed Codex seats were a fourth attempt beyond the protocol's three; the peer's Round 1 was never relayed to the resumed seat, and on g08 the orchestrator had already ledgered it). Editorial (PATCH).
- `adapters/shell/README.md` "Silent seat killers" 2 and `adapters/claude-code/SKILL.md`: a subscription cap (reset time in the message, usually empty output) is handled by freezing the round's packet and resuming the same seat after the cap lifts, never by substituting a vendor; the grounded study's g04/g08 resume is the documented case. PATCH.
- `adapters/claude-code/SKILL.md` targeted-read bullet: cites the grounded study's numbers (pointers open 0.83-0.97; judged support 0.50-0.81, a lower bound) as the reason an open pointer alone does not establish support. PATCH.


## [1.1.7] - 2026-09-17

- `validation/grounded/RESULTS.md` + `results/`: the grounded ablation is complete under its pre-registration (Amendment 1: three vendors). Headline, by the pre-registered rule: **no lift on grounded review** - paired `A_panel - A_2seat` must_catch −0.18 (n=10), decision_correct −0.06 strict / +0.06 sensitivity (n=9), false objections 1.10 vs 2.60 per memo in the panel's favour. Round 1 ran on all ten decisions and overturned nothing; the must-catch item the original panel missed stayed missed. New grounding metrics: pointers open 0.83-0.97, judged support 0.50-0.81 (a lower bound). Every raw output, attempt, ledger, packet, judge prompt/reply and log is published under `results/` after redaction; the execution deviations are listed in RESULTS in order. `validation/README.md` gains one sentence. PATCH (validation material; `core/` untouched; the README's no-lift position now rests on a grounded study as well).

- `validation/grounded/PROTOCOL.md` Amendment 1: the local Qwen seat is withdrawn (server shared with another workload; owner's decision); the study runs on Claude, Codex and Grok. No Qwen arm ever ran. The judge table is reassigned under one rule - every judgment already scored stays valid and no vendor scores itself - with the resulting pair and judge counts disclosed. The `support_rate` judge's context window was widened before any support number was reported; the narrow-window pass is discarded and kept as superseded. Recorded before any result is read into the repository.

- `validation/grounded/`: execution note recorded before any result - the local Qwen server is shared with another workload on 2026-09-16 and cannot hold the 35B model beside it, so the study runs its three frontier lanes first and defers every Qwen arm (four solos as panel seat or solo, and its judge slots on g03, g07, g10 plus second-judge slots) to a later window under the same sealed set. Decisions whose panel pair is frontier-only (g02, g03, g06, g07, g10) proceed to the panel arm now; the rest wait for Qwen. If Qwen never becomes available, the report will say so and score what ran; no arm is substituted. Nothing in the protocol's design changed.

- `validation/grounded/`: pre-registration of the **grounded ablation** - `PROTOCOL.md`, ten decision briefs (`decisions/`), ten sealed rubrics (`sealed/`), an index, and a note on the private sanitization map. Decisions come from the maintainer's own August-September 2026 panels on real artifacts with outcomes now known; the artifacts stay private. Arms: four vendors as solos (Claude, Codex, Grok, local Qwen), the union of two solos, and the full two-seat panel; rotating blind judges so no vendor scores itself; primary endpoint `A_panel - A_2seat` on must-catch rate and decision correctness; `grounding_rate` (do the cited `file:line` pointers resolve) as the grounded-specific metric. Reviewed by a two-seat panel (Codex CLI + Grok CLI) before registration. No arm has run; results come in a later release. PATCH (validation material; `core/` untouched).

## [1.1.6] - 2026-09-16

- `validation/FIELD-RECORD.md`: what one month of the maintainer's own panel archive shows (28 runs, 2026-08-19 to 2026-09-16), as the `flywheel-export` aggregate. Observational and self-reported, labelled as such; not lift evidence and no change to the README's claims. It quotes that Round 1 ran in 7 of 28 exported runs against 1 of 20 in the study (whose seats answered from model knowledge with tools unused), without treating those rates as the same measurement. Linked from `validation/README.md`. Reviewed by a two-seat panel (Codex CLI + Grok CLI) before landing; both seats' fixes applied.
- `flywheel-export`: names every field it has to null on stderr (file and field, never the text; "field absent" or "not a bare value the schema accepts"), so a dirty archive is visible - the maintainer's own archive exported 20 of 28 rounds fields as null before its retros were normalized. The warning names the retro's path relative to the archive directory and the field, never the text, and is skipped where null is the expected value (dissent before an outcome is stamped). Also fixes a scoring bug: `n/a` in `dissent_confirmed_for` was read as "dissent proved right"; it now exports as null (no dissent survived, nothing to score) - the only change to stdout, which still emits controlled metadata only: vocabulary values, integers, booleans, and null. `data/README.md` now says what stderr may contain. PATCH (tooling).


## [1.1.5] - 2026-09-16

- README "If a run stalls": "a seat exiting *silently* on a permission prompt" now names the observable - the seat hit a permission prompt, auto-cancelled it, and exited 0 with only its opening narration. The 1.1.4 panel's one surviving dissent, decided by the owner. Editorial (PATCH).

## [1.1.4] - 2026-09-16

**Two PATCH items from a review of `cmdcolin/claudish` (a Claude Code skill: a checklist of AI-writing tropes, a grep scanner, and a fixture test), adjudicated by a two-seat panel (Codex CLI + Grok CLI, R0+R1) under this repo's own method. That repo carries no licence, so nothing was copied; both items are original wording and shape, and neither changes what a panel is required to do.**

- `core/templates/r0-seat.md`: one example under the existing decisive-pointer instruction - an adverb ("fails silently", "just hangs") describes an outcome and names nothing a reader could check; cite the observable (exit status, log line, the `file:line` where the error is swallowed) or label the claim `ASSUMPTION`. Motivation: a local seat's "silent failure" claims were refuted by an oracle three times in one day. The template is the primary home because seats read it under every adapter; adapters are unchanged.
- CI: a regression test for `scout` - `test/scout/fixture.md` stands in for the scout template, `test/scout/expected.txt` is the body `scout` must print, and an inline step runs `scout` both directly and through a symlink in *another* directory (the `~/.local/bin` install that printed nothing and exited 0 before v1.0.5). The first draft of this step linked from the same directory and passed on a `scout` with the symlink fix deleted; the shipped step fails on that mutant. No new tracked script; the helper policy is unchanged.

Considered and **not** adopted from the same review, recorded so the questions are not reopened without new evidence: a `.claude-plugin/marketplace.json` install channel for the Claude Code adapter (the "plugin systems" decline in CONTRIBUTING and the 1.1.2 rejection stand; a plugin install is a second, sha-pinned copy of the tree with its own update path, so it would also split `core/` from the clone and orphan `git pull`); `allowed-tools` frontmatter on the adapter skill (seats are arbitrary-command runners; no pre-grant is both useful and safe - see the shell adapter on `Bash(git *)`); an in-tree seat-artifact validator (runner-shaped, thresholds not portable; during this very panel the maintainer's own out-of-tree runner rejected a complete seat artifact three times on the limit-signature false positive the claude-code adapter already documents as fixed); the trope scanner as CI or pre-commit lint (307 of its 610 hits on this repo's prose are its ALL-CAPS rule matching CLAIM/FALSIFIER/VERDICT/MUST); and an editorial pass driven by scanner counts (the README tagline and CONTRACT's "dropped, not relayed, not debated" are the kind of contrast the checklist itself exempts). Surviving dissent, left to the owner: whether README's "a seat exiting *silently* on a permission prompt" should name the observable (exits 0 with only its opening narration) or is already sufficient because it points to the adapters.


## [1.1.3] - 2026-09-08

**Adapter parity + a stamp (PATCH).**

- `adapters/shell/README.md`: the v1.1.2 artifact read-test now also appears in the shell adapter - script step 0b beside the smoke test (skip with `SKIP_READ_TEST=1`) and a "Smoke-test, then read-test" durable rule. Same diagnostic, same non-gate status; mirrored so the two maintained adapters agree.
- `adapters/claude-code/SKILL.md`: status line now reads "Verified 2026-09" - the adapter ran two complete panels (R0+R1, three seats) on v1.1.2 the day it shipped. Editorial only.

## [1.1.2] - 2026-09-08

**Claude Code adapter: three operator-practice additions from a review of an external field report (PATCH: no change to what a panel is required to do).** Source: Spotify Engineering, "Portal by Spotify cut my Claude Code token usage by 90%" (2026-09-03), reviewed by a three-seat panel (Codex CLI + Grok CLI + a local Qwen seat, R0+R1) under this repo's own method, with a second role-incentivized panel on the two items that survived as dissent. All three additions are examples or warnings in `adapters/claude-code/SKILL.md` "Orchestration mechanics"; each was conceded by every seat after cross-examination.

- **Targeted-read verification.** In the "three hats" bullet: a location a model hands the orchestrator (a seat's pointer, or a helper used to find a passage) is a hint, never evidence - open the cited span with a targeted read and widen it until the claim can be checked against the surrounding text. Operationalizes METHODOLOGY "Model selection" ("a model may help *locate* a candidate passage; the located evidence itself ... is what stamps a claim").
- **Optional artifact read-test beside the smoke test.** Read a known in-bounds span under the seat's configuration (and the orchestrator's, where routing differs) and compare it verbatim with the source; a mismatch or refusal means a host-side read interceptor. A diagnostic, not a gate: no threshold, no hook.
- **Compose warning.** Do not run the skill alongside read-size-blocking or I/O-delegation plugins on the orchestrator host, and do not rely on routing rules pasted into the skill or a CLAUDE.md: a Read-block starves grounding (CONTRACT "Seat fencing": the fence is on writing, never on reading), and an instruction-file rule can be ignored. The CLI's read-only mode plus host confinement is the control.

Considered and **not** adopted from the same report, recorded so the question is not reopened without new evidence: shipping any plugin, hook implementation, or wrapper script (docs-only identity; unanimous before cross-exposure); a cheap "bulk-reader" summary as seat grounding, oracle, or Round 0 pass (CONTRACT seat obligations 1-2; the report itself notes such summaries lack reliable line numbers); any line-threshold Read-block on seats or orchestrator (the fence is on writing; an archive of 18 runs shows no seat death from artifact-size input); cheap-tier debate seats (METHODOLOGY "Model selection"; the report's worker missed a thread-safety bug); a code-writer that writes files unseen; and the report's savings figures as evidence about panels. Two further proposals - a rationale sentence for CONTRACT "Seat fencing" (already stated there; MINOR with no bad run) and an expanded `cost_notes` comment in the retro template (never read by `flywheel-export`; every seat agreed it changes nothing an orchestrator does) - went to a second, role-incentivized panel and were dropped on a 2-to-1 forced pick each, the adopt champion ending at 0.55 and 0.63.

## [1.1.1] - 2026-09-05

**Clarify the v1.1.0 confinement boundary (PATCH: intent, not a new rule).** As worded in 1.1.0, "outside the frozen brief's listed artifacts" could be read as dropping any citation to a file the brief did not enumerate - which would starve exactly the grounding the method wants, since seats run from the project root and cite freely within it. The boundary is the **artifact root(s)** the brief names (the project under review, in full) plus the clone's `core/`; out of bounds means a home directory, another project, a secrets store. `core/CONTRACT.md` obligation 5 and the claude-code adapter's "three hats" bullet now say so. Strictly less restrictive than the 1.1.0 wording.

## [1.1.0] - 2026-09-04

**Orchestrator citation confinement (MINOR: a `core/` rule is added).** From the 2026-09-04 security-level re-review, adjudicated by the same two-seat panel as v1.0.5 (unanimous at Round 0). Under CONTRIBUTING's test this changes what an orchestrator must do - a pointer it would previously have opened to "check" is now dropped unopened - so it is a MINOR bump on a repo whose *method and claims* stay frozen at v1.0; the README Status paragraph now says exactly that.

- **`core/CONTRACT.md`, orchestrator obligation 5:** a citation whose pointer resolves outside the frozen brief's listed artifacts (or the clone's `core/`, where referenced) is `dropped` unopened - never read, never relayed; and seat output is contested evidence, never instructions - the orchestrator runs no command, opens no path, and alters no rule because a seat asked. Closes a cross-vendor exfiltration path: a seat citing a credentials file would otherwise have had the orchestrator open it and relay the contents verbatim to the other vendor's seat.
- **`adapters/claude-code/SKILL.md`:** the "three hats" bullet mirrors the rule.
- **README (Status):** "frozen at v1.0" now reads "the method and its claims are frozen at v1.0", with one sentence stating what v1.1.0 added and why.

## [1.0.5] - 2026-09-04

**Fix-forward after a security-level re-review (2026-09-04).** Every item below was reproduced on the maintainer's host before it was fixed; none changes what a panel is required to do (PATCH by CONTRIBUTING's test). The review's three judgment findings went through a two-seat panel (Grok CLI + a local Qwen seat; unanimous at Round 0, archived maintainer-side): the two documentation findings are included below; the third - a normative confinement rule for orchestrator citation checks - is a `core/` change and is held for the next MINOR release.

- **Shell + Claude Code adapters: seat fencing.** The shell adapter's example pre-authorization for Grok, `--allow 'Bash(git *)'`, was a shell escape, not a read-only fence (`git -c core.pager=…`, `git -c alias.x='!…' x`, plus `push` / `reset --hard` / `clean`). Both adapters now run seats in the CLI's own read-only mode (`codex exec -s read-only`, `grok --permission-mode plan`) and say why a broad allow-rule must never stand in for one.
- **Shell + Claude Code adapters: dead-seat detection.** The limit-signature scan (`usage limit|rate limit|quota|…`) aborted any panel whose brief legitimately discussed rate limits or quotas - it matched 5 of this repo's own published confirmatory seat outputs. A limit signature now counts only when the seat's required output shape (a CLAIM block / an ATTACK section) is also missing.
- **`scout`:** resolves symlinks (a `~/.local/bin/scout` link produced an empty prompt with exit 0, so `claude -p "$(scout)"` ran on nothing); refuses with a message when not inside a clone; splices the clone path literally (a path containing `&` or `#` corrupted the prompt).
- **`flywheel-export`:** `dissent_proved_right` is now `null` until `verdict_held` is `yes` or `no` (both shipped examples exported "dissent proved wrong" for outcomes not yet known); `rounds_run` accepts only the vocabulary forms and is `null` otherwise (a bare substring match turned "R0 only (R1 skipped)" into `R0+R1`). Schema table in `data/README.md` updated to match.
- **README (Status):** scopes the no-lift headline to what was measured - decisions answered from model knowledge with tools unused - and states that grounded review of a real artifact was not measured (either way).
- **validation/confirmatory/README.md:** defines "pre-registered" as pre-committed in-repo by the operator before the run (with the commit times), not registered with an external party.
- **CI:** gitleaks now scans full git history (`git` mode; `dir` mode only scanned the checked-out tree despite the full-history checkout), the gitleaks tarball is checksum-verified, `actions/checkout` is SHA-pinned, and the docs-only guard names both helper scripts.

## [1.0.4] - 2026-08-24

**Second-judge replication recipe.** One new documentation page, adjudicated by a 2-seat Codex+Grok panel run under this repo's own method (unanimous Round-0 agreement; the panel *narrowed* the maintainer's proposed scope). Docs only, PATCH by CONTRIBUTING's own test - the page restates existing protocol, adds no requirement:

- **New: `validation/confirmatory/REPLICATING-THE-JUDGE.md`** - a free, bring-your-own-key recipe for running the **designated non-OpenAI second judge** (Meta Llama / Alibaba Qwen families per `PROTOCOL.md`) over the published ambiguous-arm memos, using the verbatim judge prompt and blind-order procedure from `results-raw/AUDIT.md`. The pre-registered lift rule requires this second judge; it has never been run - either outcome is the independent replication the repo asks for.
- **Deliberately judge-only.** Both panel seats independently rejected documenting a "budget second seat": debate seats stay frontier-class and genuinely heterogeneous (`core/METHODOLOGY.md` "Model selection", `core/CONTRACT.md`), while evaluation judging is a mechanical stage METHODOLOGY already licenses for cheap models. The page carries that boundary prominently, pins no model names or gateway rosters (they rot), and ships no credentials or relay - bring your own key.
- Pointers added from README (Status), `validation/confirmatory/README.md`, and CONTRIBUTING (most-wanted contributions).

## [1.0.3] - 2026-08-22

**Audit-record publication + further claim corrections, from a third external re-review (ChatGPT).** The review verified against the raw transcripts; every substantive claim checked out. All documentation, no method change:

- **Published the ambiguous-arm audit record** at `validation/confirmatory/results-raw/AUDIT.md`: the blind X/Y-to-arm mapping (written before judging), the verbatim judge prompt and call parameters, and a per-decision reconciliation of the raw judge log to the published 118 vs 103 / forced-choice 9-1-0. Previously the judge log scored only "Memo X/Y", so the arm attribution was not independently verifiable from the repo.
- **Disclosed three executed deviations on the ambiguous arm** (in `RESULTS.md` + the audit record): (1) the judged panel memo was a recommendation line plus each seat's final verdict sentence - *cruder than* the pre-registered mechanical fill of `core/templates/verdict.md`, and an earlier RESULTS wrongly called that concatenation "per the protocol"; (2) the solo memo was **Codex-solo, fixed, for all 10 decisions**, not the pre-registered per-decision best-solo (a *weaker* baseline, so it cannot manufacture the no-lift result - but "solo" there means "Codex solo"); (3) X/Y order was alternated (counterbalanced 5/5, recorded before judging), not randomized. Also disclosed: known-answer `decision_correct` was applied by the operator, not the protocol's "judge model + mechanical oracle".
- **Corrected the c09 account to match the raw transcripts.** RESULTS had said Codex "wrongly said REPEATABLE READ makes the oversell safe" and credited the decisive `40001` mechanism to Grok alone. In fact Codex's own memos stated that mechanism, required whole-transaction retry, and recommended the sealed truth's own atomic-UPDATE fix; its headline was a *PostgreSQL-qualified* YES on a brief that named no database. The split was scope-framing, and the "wrong seat" classification is contestable - so the decidable-call hedge's single data point is soft. Softened "caught the wrong vendor" and "the hedge is real on decidable calls" accordingly (README, METHODOLOGY, RESULTS, validation READMEs).
- **Sharpened the primary-endpoint statement:** `A_panel − A_2seat` was **not measured as pre-registered** (no numeric paired aggregate was computed; 19 zeros by construction plus one unresolved split with no pre-registered numeric value), not merely "effectively untestable".
- **Fixed a stale contradiction:** `validation/README.md` still said the confirmatory decision set + run were *pending*; it now records the run as complete (no lift observed) with a pointer to the RESULTS caveat. Scoped "frontier seats mostly agree" to the two seats actually tested, and replaced "every result is published in full" with an exact statement of what is and is not in-tree.
- **Shell adapter early-stop conformance:** the early-stop message said agreed claims "honestly marked UNVERIFIED" may go in the verdict; per CONTRACT obligation 5 a claim whose check could not run stays `disputed` (UNVERIFIED stamp) and may not ride the early stop into the verdict as consensus - the message now says so.
- **Ledger provenance conformance:** the examples and `core/templates/ledger.md` used `PANELIST-CLAIM (EXTERNAL)`, which is not a value of the LEDGER provenance enum; external sourcing now lives in the *evidence* field (`EXTERNAL: <source>`) where LEDGER.md puts it, and provenance is plain `PANELIST-CLAIM`.

## [1.0.2] - 2026-08-22

**More claim-scoping, from a further external re-review (ChatGPT + Grok).** Grok read the tree as stable; ChatGPT flagged remaining over-claims that checked out. All documentation, no method change:

- **Corrected a primary-endpoint mislabel introduced in v1.0.1.** `PROTOCOL.md` pre-registers `A_panel − A_2seat` (value of Round 1) as the primary endpoint, but v1.0.1's `RESULTS.md` wrongly called `A_panel − A_solo` "the pre-registered primary." Fixed: the pre-registered primary is `A_panel − A_2seat` (effectively untestable here, n=1), and `A_panel − A_solo` is reported as the headline practical comparison (≤ 0, no lift).
- **"published in full" is now literally true.** Committed the raw arm transcripts for **all 20 decisions** under `results-raw/` (previously only the c09 chain and the judge log were in-tree), so every score is auditable.
- **"fourth-vendor judge" corrected to "fourth *model*, independent of the seats and orchestrator (OpenAI-lineage)"** across README, PROTOCOL, both validation READMEs, and pilot-02. gpt-oss shares OpenAI lineage with the Codex seat, so it is not a fourth *vendor*.
- **"confirmatory" is now explicitly caveated** at the top of `RESULTS.md`: read it as a strong *pre-registered follow-up*, not a clean confirmatory ablation. The shortfalls are named: an **ex-post oracle-picked best-of-two solo baseline** (not achievable prospectively), a **single** OpenAI-lineage judge, **no live oracle actually invoked** (arms reasoned from model knowledge; a deviation from "oracle access held constant"), the rendering control not run, and the Round-1 endpoint at n=1.
- Scoped "matched the best single model" (an ex-post best-of-two, not a prospective best) and "the hedge is real" (real on decidable calls only) wherever they appeared.

## [1.0.1] - 2026-08-21

**The freeze, corrected.** v1.0.0 was cut *before* running the final repositioned state back through a model review; doing so (a fresh Codex + Grok + Claude panel) caught that the repositioning was only half-applied and still contained real over-claims. Fixed here - all documentation, no method or feature change:

- **"never worse than a fixed single vendor" was an over-claim** (all three reviewers). The ambiguous arm judged the panel *worse* (solo 118 vs panel 103), and the one "catch" is n=1. Scoped everywhere (README, `core/METHODOLOGY.md`, `RESULTS.md`) to: on *decidable* calls the panel matched or beat any fixed vendor and caught the wrong one in the single case they diverged; on *ambiguous* calls it was judged no better, sometimes worse.
- **`core/METHODOLOGY.md` preamble was stale** - it still cited "two pilots / 1 of 9 / judge-dependent," which the confirmatory study and the v0.6.7 correction had already superseded (solo won aggregate under every judge). Rewritten to the confirmatory finding (n=20, 1 of 20, no aggregate win under any judge).
- **Disagreement rate was inconsistent** (1 of 8 vs 1 of 9 vs 1 of 20 across files) - reconciled to the confirmatory **1 of 20**.
- **Primary-endpoint contradiction** - `RESULTS.md` had renamed the endpoint to `A_panel − A_2seat`; restored the pre-registered **aggregate score** as primary, with the Round-1 comparison as a secondary (n=1) sub-analysis.
- **Softened single-judge / n=1 over-statements**: "no lift confirmed" → "no lift observed under a single independent judge"; "the mechanism is sound" → "a single encouraging data point (n=1)"; "independent fourth-vendor judge" qualified (independent of seats + orchestrator, but OpenAI-lineage). Reframed the open items (non-OpenAI second judge, real-outcome runs) as **out of scope for replicators**, not "not yet done," so the freeze stops fighting itself. Fixed a dead `results/` path (→ `results-raw/`) and the tagline.

## [1.0.0] - 2026-08-21

**Stable release - the methodology is complete, honestly characterized, and development is frozen.** No accuracy claim; no accuracy pretense. This version repositions the whole repo around the value its own validation actually supports and stops there.

- **Repositioned from "smarter panel" to "decision-support scaffold + hedge."** After three pre-registered studies (two pilots + a confirmatory n=20 with an independent judge) found **no accuracy lift** over a strong single model, the README and `core/METHODOLOGY.md` now frame the value as what the data supports: an **adversarial counter-case**, a **discriminating test** to settle a contested call, and a **hedge** whose answer is never worse than betting on one vendor and occasionally catches the vendor you'd have trusted being wrong - explicitly **not** a way to out-decide the best single model. "When to convene" is rewritten around that honest value (and its real cost) rather than an implied "the panel will catch what the model misses."
- **Frozen by intent.** README Status now declares the project stable and done, not abandoned; the next thing it wants is *independent replication* of the no-lift finding, not more internal development.
- The normative core (isolation, grounded claims, verbatim relay, three-bucket verdict, oracle-first, preserved dissent) is unchanged - it stands on its own as decision-support discipline regardless of the lift question. Full evidence, including losses and a corrected earlier overclaim, remains in `validation/`.

## [0.6.9] - 2026-08-21

- **Confirmatory study run to completion (n=20) - no lift confirmed** (`validation/confirmatory/RESULTS.md`). Four arms per decision with **oracle access held constant** (so `A_panel − A_2seat` isolates Round 1), an **independent fourth-vendor judge** (gpt-oss-120b, temp 0), and access-separated sealed truth. Findings:
  - **Seats disagreed on only 1 of 20 decisions** (c09). On the other 19 the vendors independently agreed, so Round 1 never ran and `A_panel = A_2seat`. This confirms the pilots' rare-disagreement finding at n=20 and is the study's most robust result.
  - **Round 1 works when it fires:** on c09, Codex-solo said REPEATABLE READ makes an inventory oversell safe (wrong); Grok said no (right); in Round 1 **Codex revised YES → NO** and the panel landed on the correct answer. The full transcript is in `results-raw/c09/`. But the panel only beat the *weaker fixed vendor* - Grok-solo alone was already correct, so there is **no lift over the best single model**.
  - **Known-answer (n=10, oracle-scored):** Codex-solo 9/10, Grok-solo 10/10, best-of-two-solos 10/10, panel 10/10. **Ambiguous (n=10, independent judge):** solo aggregate 118 vs panel 103; forced choice solo 9 / tie 1 / panel 0 (with the honest caveat that the panel memo was crudely rendered mechanically, which handicaps it - the mirror of pilot-02's hand-polished panel memo; both still land at solo ≥ panel).
  - **Operational tax quantified:** Grok narration-died in headless mode on 10/10 known-answer decisions when the prompt merely mentioned consulting docs (it tried to use tools / load a skill), and only became reliable under a strict "no tools" prompt. The two-vendor requirement carries a large reliability cost in automated use.
- **README/METHODOLOGY updated** to the confirmatory finding: no accuracy lift; the defensible value is a *hedge that is never worse than a fixed single vendor and sometimes catches its mistake*, plus a counter-case + discriminating test - not a smarter-than-the-best-model claim. Open limitations stated: single (OpenAI-lineage) judge, crude ambiguous panel rendering, and only one decision exercising Round 1.

## [0.6.8] - 2026-08-21

- **`validation/confirmatory/` - the confirmatory study is set up and its judge blocker is solved.** The pilots were exploratory for four reasons (judge = orchestrator's vendor; arms didn't isolate components; orchestrator-assembled panel memo; tiny n / undisclosed scorer / instruction-only blinding). The pre-registered `confirmatory/PROTOCOL.md` removes all four: component-isolating arms with **oracle access held constant** (so `A_panel − A_2seat` isolates Round 1), an **independent fourth-vendor judge**, a mechanical (non-orchestrator) panel output with a raw-concatenation control, an independent scorer, access-separated sealed truth, **n ≥ 20**, and **aggregate score as the pre-registered primary endpoint** with a decision rule for declaring lift.
- **Independent judge wired and proven.** `gpt-oss-120b` on a local OpenAI-compatible endpoint (neither seat, nor the orchestrator's vendor), temperature 0 for reproducibility. Its first use re-judged the pilot-02 memos blind and **corroborated no lift**: a dead tie on aggregate (32-32), not a panel win. So across three judge vendors now (Claude ×2 independent → solo; gpt-oss independent → tie; Grok conflicted → tie) the panel **never out-scores solo on aggregate**. pilot-02 and its BLIND-ORDER table updated with the fourth-vendor row.
- First pre-registered decisions seeded (`confirmatory/decisions/` + access-separated `confirmatory/sealed/`); the set is to be completed to n ≥ 20 and a non-OpenAI second judge added before the confirmatory scoring run. Harness scripts stay operator-side (this repo is docs-only); only protocol, decisions, and results are committed.

## [0.6.7] - 2026-08-21

- **Corrections from a second external review round (Grok + Claude + Codex).** Every concrete claim was verified before acting; the sharpest catches were mine to own.
  - **Corrected a real overclaim in pilot-02** (Codex): the write-up said the result "flips with the judge / no robust signal." On the pre-registered *aggregate* rubric score the solo memo actually won under both independent Claude passes (34-32, 34-31) and **tied** under the conflicted Grok judge (32-32) - the panel never out-scored solo under any judge. Only the *forced-choice* of the conflicted judge inverted. The headline, robustness section, bottom line, `BLIND-ORDER.md`, and the README paragraph are corrected to "no aggregate lift under any judge; only a conflicted judge's tie-break favored the panel" - a stronger no-lift finding, not a flip.
  - **Example ledgers now conform to the verbatim-claim rule** (both review rounds): `core/LEDGER.md` requires the `claim` field be "one sentence, quoted verbatim from the seat," but both example ledgers had orchestrator *paraphrases*. All 18 claims (11 monorepo + 7 api-auth) replaced with the seats' verbatim sentences.
  - **Monorepo ledger used the wrong enum value** (Codex): five surviving claims were marked `disputed` with a legend that locally redefined the enum; the enum has a distinct terminal `surviving-dissent` status (LEDGER line 44), now used correctly, legend fixed, and the retro reconciled (`disputed` 0 / `conceded` 6 / `surviving-dissent` 5).
  - **pilot-01 reporting gaps disclosed** (Codex + Claude): the rubric was applied by the orchestrator, not the pre-registered independent scorer (now stated); `must_catch_rate` was not tabulated per arm (noted); d3 had **no valid panel** (a seat died) so A2/A3 are scored `—` not `1*`, and the delta counts are `/4 evaluable`; the four arms don't isolate components (oracle access varies) and blinding was instruction-based not access-separated (both noted). "1 of 9" convergence corrected to "1 of 8 measurable".
  - **Smaller honest fixes:** README `## Proof` heading → `## Worked examples` (it contradicted the "unproven" stance); "strongest counter-case / cheapest test" superlatives softened (nothing measured them); the "biggest gain is 1 → 2" line qualified with the 1-of-8 finding; `core/METHODOLOGY.md` "durable value … stands whether or not lift materializes" → *intended* durable value (judge-dependent in the pilots); `scout` comment "writes nothing" corrected to disclose the `git fetch` metadata write; `citations_unverified: 0` annotated ("none formally flagged", not "all verified"); the shell adapter's early-stop gate now instructs running oracles on checkable agreed claims *before* writing the verdict (agreement ≠ verification).

## [0.6.6] - 2026-08-21

- **pilot-02 finished: added a second/third judge pass - and the result inverted.** The first write-up rested on a single Claude judge (solo 2 / panel 1). Completing the pre-registration's "second judge" step: a second **independent** Claude pass with order flipped reproduced it exactly (panel 1 / solo 2, so it is stable against position/sampling), but a second judge **vendor** (Grok, run as a labeled *conflicted* check since its own text is in the panel memos) **inverted every decision** (solo 1 / panel 2). The two judge vendors are anti-correlated on all three decisions, splitting precisely on whether the panel's "here is the one test that settles it" beats a committed A/B answer. Net: **no robust lift signal - the sign flips with the judge.** `validation/results/pilot-02.md` + `BLIND-ORDER.md` updated with all judge outputs; README validation paragraph corrected from "judge preferred solo 2/3" to "flipped with the judge - no robust lift." The one judge-independent finding stands: seats disagreed on only 1 of 9 decisions across both pilots.

## [0.6.5] - 2026-08-21

- **`validation/` - the repo now measures its own core claim, and publishes the negative result.** Prompted by three independent external reviews (Claude/ChatGPT/Grok) all flagging that Tribunal demonstrates the process *runs* but not that it produces *lift* over a single strong model. Added a pre-registered ablation harness (`validation/PROTOCOL.md`, sealed decision sets) run in two pilots, committed before results so nothing could be curated:
  - **pilot-01** (5 known-answer decisions): null - a solo model got all 5 right without an oracle; the panel added cost, no accuracy. Also surfaced that decidable questions are the category Tribunal's own scope says *not* to convene a panel for.
  - **pilot-02** (3 ambiguous decisions, blind third-vendor judge): the panel **lost the set, solo 2 - panel 1**. The panel's surviving-dissent / decide-after-test output was docked for not committing to A/B; a second seat's extra evidence introduced its own error. Across both pilots the two vendors disagreed on only **1 of 9** decisions, so the panel machinery mostly idles.
- **Claims narrowed to match the evidence** (README + `core/METHODOLOGY.md`): dropped "two vendors catch what neither would alone" as a stated fact; the heterogeneity/lift premise is now labeled a hypothesis with the current evidence against its strong form, linked to `validation/`. Tribunal is repositioned as an adversarial, honesty-first aid (strongest counter-case + cheapest discriminating test on contested, irreversible calls), explicitly **not** a proven accuracy upgrade. The method's durable value is framed as procedural (isolation, grounding, preserved dissent), which holds regardless of the lift question.

## [0.6.4] - 2026-08-21

- **Correctness fixes from three independent external reviews** (Claude, ChatGPT, Grok, separate sessions) - all verified before fixing. No methodology features added; this is bug-and-conformance only.
  - **`flywheel-export` parser bug**: the field-name regex `^[a-z_]+:` rejected digits, so `agreed_r0` (and any digit-bearing field) silently exported as `null` for every record. Fixed to `^[a-z0-9_]+:`; the api-auth example now correctly exports `agreed_r0: 5`.
  - **Both example ledgers/verdicts now conform to `core/` as written** (they were violating the repo's own normative contract - the one thing a worked example must not do). The monorepo ledger's status column used non-enum values (`converged`, `conceded down`, `surviving dissent`); it now uses the `LEDGER` enum (`conceded`/`disputed`) with a legend, and the "converged/common-ground" framing is explicitly marked verdict-level, not a claim status. The monorepo verdict no longer promotes *post-cross-examination* common ground to "the panel concludes" - bucket 1 (independent agreement) is now correctly shown as **empty** for a role-incentivized panel (opposing sides ⇒ no `agreed-r0`), with the concessions recorded as bucket 2 (resolved). The api-auth verdict no longer calls single-sourced `open` claims "sound"; they are reported as unexamined, not endorsed, per the verdict rule.
  - **Monorepo retro conformance**: replaced invented fields (`disputed_axes`, `converged_in_r1`, `claims_revised`) and the non-integer `citations_unverified: all` with the retro template's standard integer fields, so the exporter parses every field. Counts are now self-consistent (`agreed_r0` 0 · `conceded` 6 · `disputed` 5 = 11 claims).
  - **`scout` "nothing is written" was imprecise**: scout's weekly self-update check runs `git fetch` (writing git metadata) or touches a cache timestamp. The README now says it "writes nothing to your project" and names the update-check's writes explicitly.

## [0.6.3] - 2026-08-21

- **Confirmation-pass fixes (2-seat Codex+Grok review of the v0.6.2 onboarding fixes).** Both seats confirmed the v0.6.2 fixes landed (hero block, orchestrator-vs-seat CLI count, and shell-script cwd/argv/completion all PASS), and surfaced remaining honesty/correctness gaps in the shell adapter's complete script - now fixed: the "STOP - write the verdict now" comment is now a **real early-stop gate** (a prompt that writes the verdict and `exit 0`s, skipping Round 1) instead of narration the code ignored; "re-read each assembled prompt before sending" is now an **actual `$EDITOR` pause** on the Round 1 packets (the verbatim-relay check); and step 4 now **pre-creates** the `disputed-from-*`/`own-r0-*` extraction files (`: >`) before opening them, so a stray editor-quit can't crash the later `cat` under `set -e`, plus a guard that Round 1 isn't run with zero disputed claims. Also: the README hero comment now says "**Pick ONE line**" (paste-all-three ran three scouts), and the Claude skill no longer implies `~/tribunal/scout` drafts the brief by itself (it prints the prompt; an agent writes the brief). Core methodology untouched.

## [0.6.2] - 2026-08-21

- **Onboarding fixes from a 2-seat Codex+Grok pre-share review** (both returned *hold-for-fixes*, converging on the same root cause: a newcomer's first action doesn't do what they think, and the docs disagreed on how many CLIs a panel needs). Fixed: the README hero block put the Codex/Grok scout invocations *after a `#`*, so copy-pasting the line ran only the Claude one - now three standalone labeled commands; the "two CLIs" prerequisite is reconciled with the Claude Code adapter's three-tool reality by naming **orchestrator vs seat** as distinct roles (README + `adapters/claude-code/SKILL.md`); and the shell adapter's complete script no longer `cd`s into `panel/` before invoking seats (which broke the brief's relative paths and contradicted its own "cwd = artifact root" rule) - seats now run at the artifact root with bookkeeping in `$PANEL_OUT`, the false "prompts pass via stdin" comment is corrected to match the argv the script actually uses (with a pointer to `--prompt-file`/stdin for large prompts), and step 6 now actually writes the verdict instead of trailing off in a comment. Core methodology untouched (both seats explicitly said leave it alone).

- **New worked example** `examples/api-auth-jwt-vs-sessions/`: a *real* cross-vendor panel (Codex CLI + Grok CLI) on a conventional engineering decision - API auth, stateless JWT vs server-side sessions - in the current template format. Both seats independently chose the same design in isolation, so the panel early-stopped at Round 0; demonstrates pre-exposure agreement, the early-stop discipline, honest empty-dissent bucketing, and routing the one residual risk to a test. Addresses CONTRIBUTING's most-wanted contribution (a current-format run on a real decision) alongside the historical sample-run.

- **New worked example** `examples/repo-monorepo-vs-polyrepo/`: the mirror of the agreement example - a *real* cross-vendor panel (Codex CLI + Grok CLI) run as a **role-incentivized stress-test** (Seat A assigned to champion monorepo, Seat B polyrepo; roles as incentives, not personas) on a genuinely no-consensus decision. Forces a full **Round 1** cross-examination: both seats concede real points and revise confidence, neither verdict is overturned, and the panel keeps the load-bearing **surviving dissent** (can *this* org run a monorepo without a bespoke build system / dedicated build-platform team?) instead of averaging it - then routes it to the cheapest discriminating test, which both opposing seats independently proposed in near-identical form. Together the two examples now bracket both method outcomes (agreement + early stop / dissent + cross-examination). Retro flags a candidate template delta: when opposing seats each name a settling test, adopt the intersection of their falsifiers as the oracle.

## [0.6.1] - 2026-08-21

- **Pre-share newcomer polish** (README, `adapters/claude-code/SKILL.md`) from a 2-seat Codex+Grok newcomer review (both promote-after-fixes): the two-different-vendor-CLI prerequisite now sits next to the clone; the Status "receipts" wording is narrowed to what's actually in-tree (the sample-run plus the CHANGELOG's own panel-rejected-first-draft notes), with the ten-seat review labeled not-in-tree; a stray duplicate clone removed and the scout->brief->adapter handoff made concrete; a first-run invocation added to the Claude Code skill; diagram alt-text + a caption now mark seats A/B required and dashed Seat C optional; and a short "if a run stalls" note added.

- **Workflow diagram**: added a third seat (Seat C, dashed/optional) to the hero
  SVGs (light + dark), reinforcing that a panel is any N>=2; README seat-count
  line updated to match. Diagram flow otherwise unchanged - still accurate
  after the v0.4-0.6 within-stage refinements.

- **retro template**: numeric fields must be a bare integer with any caveat in
  a trailing `<!-- comment -->` (not inline like `3 (seat died)`), so counts
  grep and parse cleanly - closes the field class that the operator flywheel's
  data-quality check flagged.

- **`flywheel-export` + `data/` (de-identification tooling)**: a local,
  stdout-only helper that reduces a `retro.md` archive to de-identified,
  metadata-only JSON (whitelist of counts/enums; unknown values become
  `"other"`, raw text never echoed; verified leak-proof by an injection
  test), plus `data/README.md` documenting the schema. Adversarially reviewed
  by a 2-seat Codex+Grok anonymity panel (Codex rework, Grok
  ship-after-fixes), which corrected the design: it is **de-identified, not
  anonymous** (a public PR carries your GitHub identity); the "which vendor
  was right" field collapsed to a boolean; records sorted by content to kill
  the timing/order channel; integer parsing tightened; and the **contribution
  intake is deliberately NOT opened** - premature without a corpus, and
  unsafe without a CI schema-gate (specced in `data/README.md`). Optional
  local tooling, no backend; stamps PATCH when released.

- **Guardrail CI** (`.github/workflows/checks.yml`): a single docs-only
  workflow (no build/test - there's nothing to build). Automates the
  previously-manual consistency checks: `scout` sanity (`sh -n` +
  shellcheck), a secret/PII scan (gitleaks), internal-link resolution, a
  release tag ↔ CHANGELOG consistency check, and a "docs-only" guard that
  fails if any application/build code is committed - enforcing the repo's own
  `runtime: none` identity. Mostly inline bash to keep third-party actions
  minimal. Maintainer tooling, not shipped to users; PATCH when released.
  README gains a live `checks` status badge.

## [0.6.0] - 2026-08-20

- **Inline the governing invariants** (`scout` step 3 + `brief.md`): when a
  decision's correctness depends on rules defined elsewhere - a contract,
  schema, spec, style guide, or (for a change to this repo) its own
  CONTRACT/LEDGER - the brief must paste that governing text in, not just cite
  it. Promotes a TEMPLATE DELTA that recurred across three panels (lens
  doctrine, v0.4.0, v0.5.0): every consistency collision was one seats had to
  reconstruct a rule from memory to find. Normative (changes what panels
  catch), so MINOR.

## [0.5.0] - 2026-08-20

Tier-2 additions from the idea-scouting research panel, then pressure-tested
by a 2-seat Codex+Grok panel (Codex rework, Grok ship-after-fixes). The panel
cut two of the three proposed additions as redundant/over-reaching and kept
one - all its fixes applied. Normative `core/` change, so MINOR.

- **Competing-hypotheses mode (ACH)** - a new *mode* (not a default) for
  "which of several explanations is right?" causal/explanatory decisions
  (rival root causes, incident timelines). New `core/templates/ach.md` and a
  METHODOLOGY section: seats build an evidence×hypothesis matrix in isolation,
  marking consistent/inconsistent/NA, and the diagnostic move is
  *disconfirmation* (a hypothesis wins by surviving refutation, not by
  collecting agreement). Panel-hardened: every asserted inconsistency is a
  full `CLAIM/EVIDENCE/CONFIDENCE/FALSIFIER` ledger row (CONTRACT obl. 4 gains
  the matrix-shape exemption); ranking counts only hypotheses with a
  diagnostic cell so a thinly-tested `H0` can't win by default; seats rank,
  the orchestrator never sums cells across seats. (Source: Heuer, *Analysis of
  Competing Hypotheses*.)
- **`depends_on` ledger field** (optional, high-stakes) - a seat may record
  the claim IDs a claim rests on, so you can see what a verdict turns on and
  prioritize attacks. The orchestrator only records it; it never infers or
  ranks the chain and it does not change the Round 2 trigger. (This is the
  salvaged, obligation-6-safe core of a proposed "load-bearing map" section
  the panel cut for duplicating `decision_relevance` and colliding with
  v0.4.0's "attack every load-bearing claim".)
- **Oracle-before-relay**, stated as one sentence in Round 1 (a proposed
  standalone "route by claim type" section was cut as a restatement of
  existing rules): sweep disputed claims for any a cheap oracle settles now
  before spending a round; a check that can't run still relays `UNVERIFIED`.

## [0.4.0] - 2026-08-20

Methodology hardening - three evidence-backed mechanisms from a 3-seat
research panel (Codex + Grok + Claude), then pressure-tested by a 2-seat
Codex+Grok panel (both verdicts ship-after-fixes; all six findings applied).
Normative `core/` change, so MINOR.

- **Anchor-resistant Round 0 + evidence packets.** EVIDENCE must be a
  *decisive pointer* (smallest span / `file:line` / oracle invocation that
  could settle the claim), not prose. High-stakes panels MAY run Round 0 in
  two *separate isolated invocations* of each seat - observations first, then
  claims built on them (pass 1 is exempt from the claim-shape check). New
  ledger status `no-stable-position` for an item a seat declines to commit -
  surviving uncertainty, not a skip; it blocks `agreed-r0` and is
  Brier-ineligible. (Sources incl. 12-Angry-AI anchoring study, scalable-
  oversight evidence packets.)
- **Bias-hardened Round 1 relay.** The seat-facing packet strips vendor/model
  names (neutral labels, author kept in the ledger), randomizes claim order,
  and carries no agreement tallies - cross-exposure is where bandwagon and
  prestige bias enter. Relay is sparse-but-never-blinkered: always deliver
  every contradicting claim, every unaddressed decision-relevant claim, and
  every `UNVERIFIED` claim. Attacking a claim's *confidence* is a legitimate
  Round 1 move. New optional high-stakes **swap-audit**: re-run one
  argument-resolved load-bearing claim to the same seats with order/labels
  flipped; a differently-structured response = the resolution was a
  presentation artifact → record surviving dissent (the orchestrator compares,
  never judges). `CONTRACT.md` Attribution section rewritten to match; obl. 2
  and 4 and `LEDGER.md` reconciled. (Sources incl. bias-amplification and
  position-bias studies.)
- **Calibration retro.** Confidence is now a `0`-`1` probability
  (high/med/low → `0.85`/`0.6`/`0.3`), canonical in the ledger. The retro
  Brier-scores each seat's confidence against oracle/outcome results -
  deterministic orchestrator math, never an LLM - reported with sample size
  (`<~5` scored = no-signal; a cross-run read). A TEMPLATE DELTA input read
  with judgment, never a truth signal or a seat-weighting rule.

## [0.3.4] - 2026-08-20

- **Versioning policy** (`CONTRIBUTING.md`): SemVer scoped for a docs/method
  repo - MINOR = a normative `core/` change that could alter a panel's
  conduct or outcome; PATCH = clarifications, doc/tooling fixes, adapters,
  examples; MAJOR reserved for 1.0 and later breaking normative changes.
  Records the Keep-a-Changelog release flow and that published tags are
  never moved.

## [0.3.3] - 2026-08-20

- **Docs consistency pass.** README "Status" was stale at v0.1.0; rather than
  hardcode a version that re-staled on every release, the line now points at
  the dynamic release badge and `CHANGELOG.md` - no version string to drift.
  `CHANGELOG.md` added to the repository-layout listing (the update notice
  points users there). Repo-layout `scout` line notes the weekly update
  notice.
- **`brief.md` gains an optional "Lens assignments" section** so the
  exclusive-lens mechanism from the v0.3.0 doctrine is usable end-to-end
  from the template - default-empty, with inline guidance to prefer additive
  lenses in Decision criteria and reserve the matrix for large panels with
  >=2 heterogeneous seats per surface plus a retained generalist.

## [0.3.2] - 2026-08-20

- **`scout` update check now handles zip/tarball installs too.** A copy
  with no `.git` has no `origin` to pull from, so the git check no-oped for
  those users. Now `scout` detects a non-git copy, reads its version from
  `CHANGELOG.md`, and (throttled once per 7 days via a `$XDG_CACHE_HOME`
  timestamp) points the user at the git-clone install that self-checks plus
  the releases page. Same guarantees as the git path: stderr-only,
  network-free, fail-silent, notify-only, `TRIBUNAL_NO_UPDATE_CHECK=1` to
  opt out. The git-clone path stays write-free (throttled by `.git/FETCH_HEAD`
  mtime); only the zip fallback writes a small cache timestamp.
- **README "Staying current"**: documents the zip/tarball case and why the
  git-clone install is recommended (every future update becomes `git pull`).

## [0.3.1] - 2026-08-20

- **`scout` weekly update check** (notify-only, never auto-pulls). At most
  once per 7 days (throttled by `.git/FETCH_HEAD` mtime), if the clone is
  behind origin, `scout` prints a one-line update notice with the exact
  `git ... pull --ff-only` command. The notice goes to **stderr only**, so
  it can never contaminate the prompt piped to your agent on stdout;
  fail-silent on no-git/no-network/detached-HEAD; opt out with
  `TRIBUNAL_NO_UPDATE_CHECK=1`. No runtime added to the repo - this stays a
  docs-only clone; the check just surfaces that a `git pull` is available.
- **README "Staying current"**: documents the update command, the weekly
  notice, and why updating is non-destructive - `--ff-only` is clean-or-
  abort (untracked files untouched, local edits never silently overwritten)
  - plus the keep-the-clone-pristine convention that guarantees it.

## [0.3.0] - 2026-08-20

- **Lens-assignment doctrine** (`core/METHODOLOGY.md`, "Assigning lenses"):
  codifies WHEN a panel should give seats distinct review lenses vs. run
  raw heterogeneous seats. Two mechanisms - **additive** (surfaces named in
  the shared brief, legal at any N, the default) vs. **exclusive slice** (a
  seat prioritizes one surface via a labeled addendum, the one sanctioned
  departure from the identical-brief rule). Exclusive slicing is fenced:
  ≥2 heterogeneous seats per lens, capped at floor(N/2), a retained
  generalist read, and Round 1 still relays every disputed claim to every
  seat. `core/CONTRACT.md` obligation 1 gains a scoped, explicit exception
  so the exclusive `LENS:` addendum is sanctioned by the MUST layer instead
  of silently contradicting "delivered identically to every seat" - the
  addendum is part of the frozen brief, fixed before Round 0, with every
  seat receiving the full assignment matrix and identical criteria. Whether
  a surface earns a lens is judgment (own criterion,
  non-implied, salient); the staffing floor is a MUST. `scout` step 4 now
  defaults first panels to raw seats and only recommends exclusive slicing
  at N ≥ 4 with proper staffing. This section was itself pressure-tested by
  a 2-seat Codex+Grok panel, which caught the identical-brief conflict and
  the unsupported "~5 seats" gate the first draft shipped with.

## [0.2.0] - 2026-08-20

- **Renamed the project: Pressure-Test → Tribunal** (the old GitHub URL
  redirects). Skill name, `TRIBUNAL_ROOT`, and install paths renamed to
  match; the historical sample-run transcripts are unmodified as always.
- **`scout` helper** (repo root): one runnable command feeds the scouting
  prompt to your agent (resolves the clone-path placeholder, strips
  template chrome). Text-only — runs no panel, writes nothing.
- **README**: featured one-command clone + scout up top; hero workflow
  diagram (light/dark SVG) now shows scout and the triggered Round 2, so
  the chart matches the full template set; FAQ targeting real search
  queries; honest getting-started framing (setup is seconds; the 20-40 min
  is the panel itself); documented that seats scale to N ≥ 2.
- **Adapters**: silent-seat-killer taxonomy (permission death, quota
  exhaustion, truncation); local models documented as first-class seats.
- Docs: em/en-dashes replaced with plain hyphens (verbatim transcripts
  preserved); confident Status rewrite; LICENSE holder set with website.

## [0.1.0] - 2026-08-19

Initial public extraction. These documents were reviewed in-repo by a
10-seat cross-vendor panel (4 Codex CLI seats, 4 Grok CLI seats, 2 Claude
seats - developer + product roles with orchestrator-relayed cross-talk);
the methodology itself still has one published run (see README, Status).

- `core/`: methodology, contract, ledger, verdict - extracted
  orchestrator-neutral from the original Claude Code skill.
- `core/templates/`: scout (adopter onboarding), brief, r0-seat, r1-seat,
  r2-revision, ledger, verdict.
- `adapters/claude-code/`: the original skill, thinned to host mechanics
  (`TRIBUNAL_ROOT` indirection; vendor flags live in the shell
  adapter's dated examples block).
- `adapters/shell/`: portable scripted recipe for any orchestrator (GNU
  timeout detection, wait-both-fail-loud, smoke test, mechanical Round 1
  assembly including each seat's own Round 0 claims).
- `adapters/{codex-cli,gemini-cli,opencode}/`: marked stubs (untested).
- `examples/sample-run/`: the real two-seat panel that designed the
  methodology, labeled as a historical bootstrap transcript with its
  known non-compliances enumerated.

[Unreleased]: https://github.com/kdoubt/tribunal/compare/v1.1.8...HEAD
[1.1.8]: https://github.com/kdoubt/tribunal/compare/v1.1.7...v1.1.8
[1.1.7]: https://github.com/kdoubt/tribunal/compare/v1.1.6...v1.1.7
[1.1.6]: https://github.com/kdoubt/tribunal/compare/v1.1.5...v1.1.6
[1.1.5]: https://github.com/kdoubt/tribunal/compare/v1.1.4...v1.1.5
[1.1.4]: https://github.com/kdoubt/tribunal/compare/v1.1.3...v1.1.4
[1.1.3]: https://github.com/kdoubt/tribunal/compare/v1.1.2...v1.1.3
[1.1.2]: https://github.com/kdoubt/tribunal/compare/v1.1.1...v1.1.2
[1.1.1]: https://github.com/kdoubt/tribunal/compare/v1.1.0...v1.1.1
[1.1.0]: https://github.com/kdoubt/tribunal/compare/v1.0.5...v1.1.0
[1.0.5]: https://github.com/kdoubt/tribunal/compare/v1.0.4...v1.0.5
[1.0.4]: https://github.com/kdoubt/tribunal/compare/v1.0.3...v1.0.4
[1.0.3]: https://github.com/kdoubt/tribunal/compare/v1.0.2...v1.0.3
[1.0.2]: https://github.com/kdoubt/tribunal/compare/v1.0.1...v1.0.2
[1.0.1]: https://github.com/kdoubt/tribunal/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/kdoubt/tribunal/compare/v0.6.9...v1.0.0
[0.6.9]: https://github.com/kdoubt/tribunal/compare/v0.6.8...v0.6.9
[0.6.8]: https://github.com/kdoubt/tribunal/compare/v0.6.7...v0.6.8
[0.6.7]: https://github.com/kdoubt/tribunal/compare/v0.6.6...v0.6.7
[0.6.6]: https://github.com/kdoubt/tribunal/compare/v0.6.5...v0.6.6
[0.6.5]: https://github.com/kdoubt/tribunal/compare/v0.6.4...v0.6.5
[0.6.4]: https://github.com/kdoubt/tribunal/compare/v0.6.3...v0.6.4
[0.6.3]: https://github.com/kdoubt/tribunal/compare/v0.6.2...v0.6.3
[0.6.2]: https://github.com/kdoubt/tribunal/compare/v0.6.1...v0.6.2
[0.6.1]: https://github.com/kdoubt/tribunal/compare/v0.6.0...v0.6.1
[0.6.0]: https://github.com/kdoubt/tribunal/compare/v0.5.0...v0.6.0
[0.5.0]: https://github.com/kdoubt/tribunal/compare/v0.4.0...v0.5.0
[0.4.0]: https://github.com/kdoubt/tribunal/compare/v0.3.4...v0.4.0
[0.3.4]: https://github.com/kdoubt/tribunal/compare/v0.3.3...v0.3.4
[0.3.3]: https://github.com/kdoubt/tribunal/compare/v0.3.2...v0.3.3
[0.3.2]: https://github.com/kdoubt/tribunal/compare/v0.3.1...v0.3.2
[0.3.1]: https://github.com/kdoubt/tribunal/compare/v0.3.0...v0.3.1
[0.3.0]: https://github.com/kdoubt/tribunal/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/kdoubt/tribunal/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/kdoubt/tribunal/releases/tag/v0.1.0

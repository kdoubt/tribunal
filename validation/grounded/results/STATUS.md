# Grounded ablation - run status (maintainer's private log; results/ is published after redaction)

Pre-registration: tribunal main a779242 (2026-09-16). Qwen deferred by owner instruction 2026-09-16 (server shared with another workload).

## Arms
- Solos: 30 of 30 frontier arms accepted (claude/codex/grok x g01-g10). Qwen solos: 0 of 10 (deferred).
  Validator correction logged 2026-09-16T18:07Z: numbered bold headings ("**1. CLAIM:**") accepted; g05/g06 S_codex accepted from attempt 1 retroactively, no re-run.
- Panel arms (A_panel + A_2seat): g02, g03, g06, g07, g10 complete (frontier pairs). R1: 10 of 10 seat runs validated on attempt 1; Round 2 never triggered.
  g01, g04, g05, g08, g09 wait for the Qwen panel seat.
- Grounding (mechanical): resolver run on all 27 solos with an artifact; g04 seats cite the inlined capture pack (not files) - reported per decision; g10 has no artifact.

## Judging (rotating table)
- Frontier-judged now: g01 (3 solos), g02 (S_claude, S_codex, A_2seat, A_panel), g04 (3), g05 (3), g06 (S_claude, S_codex, A_2seat, A_panel), g08 (3), g09 (3).
- Deferred to Qwen: every g03/g07/g10 arm (J=Q), g02/g06 S_grok (J2=Q), all Qwen solos, all Qwen-in-panel A_ arms.
  => the primary paired endpoint is computable today on g02 and g06 only (n=2); the full n=10 needs Qwen.
- support_rate pass running with the same assignment.

## Harness corrections during the run (all logged in run.log / transformations.log)
- validator heading regex (above); grounding.py heading + path handling + unique-stem fallback; r1-packet.py headings;
  run-r1.sh packet hash computed before cd; judge.py temp files moved out of results/.

## 2026-09-16 19:10Z addendum
- Rubric judging: 23 frontier-judged arms scored, 0 parse failures (g01 x3, g02 x4 incl. A_2seat/A_panel, g04 x3, g05 x3, g06 x4, g08 x3, g09 x3). All Qwen judge/J2 slots deferred.
- Interim primary (n=2: g02, g06 only; attrition cap exceeded, NOT a finding): A_panel - A_2seat must_catch -0.15, decision_correct -0.25; false_objections panel 1.0 vs 2seat 2.5.
- support_rate pass 1 STOPPED and superseded (files in results/_superseded/): the judge saw only ~6 lines/400 chars per pointer, which is too little context to call support; grounding.py now hands the full cited range plus context (<=1200 chars). Pass 2 relaunched under the same assignment. This is a metric-definition fix made before any support number was reported; disclosed here and to be disclosed in RESULTS.
- Harness note: `pkill -f`/`pgrep -f` against a script name matches the invoking shell when the command line contains the same string; kill by a pattern that the caller's command line cannot match, or from a separate command.

## 2026-09-16 20:05Z - Amendment 1 applied; Codex usage cap hit
- Amendment 1 (main 8f39228): Qwen withdrawn; three vendors; judge table reassigned under the reuse/no-self-judging rule.
- Panels complete: g01, g02, g03, g05, g06, g07, g09, g10 (8 of 10). All their R1 seats validated on attempt 1; Round 2 never triggered.
- Panels INCOMPLETE: g04 and g08 - the Codex Round 1 seat returned 0 bytes on 3 attempts each with stderr
  "You've hit your usage limit ... try again at Sep 19th, 2026 9:12 AM" (subscription cap, account-wide). Grok's R1 for both is
  accepted and unread by the other seat. Packets unchanged (sha recorded) -> resume = `tools/run-r1.sh g04 codex r1-packet-codex.md`
  and `... g08 ...` after the cap lifts, then the fork's remaining steps (read both R1s, ledger update, A_panel fill).
- Also blocked until the cap lifts: every judge/support slot assigned to Codex that has not run yet (J on g01, g05, g09 for
  A_2seat/A_panel; J2=codex slots on g03, g04, g08, g10 for S_claude/S_grok where not yet scored).
- Interim primary (n=3: g02, g03, g06): A_panel - A_2seat must_catch -0.15, decision_correct -0.17; false_objections panel 2.0 vs 3.0.
  Attrition cap exceeded -> no conclusion.
- Decision for the owner: wait for Sep 19 to complete Codex's two R1 seats and judge slots (recommended: the packets are frozen and
  a resumed R1 is a legitimate continuation), or record the two Codex R1 deaths now (drops g04, g08 from the paired means).

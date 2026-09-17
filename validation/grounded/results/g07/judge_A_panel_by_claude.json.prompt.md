You are scoring one review memo against a sealed rubric. You are not a participant; do not add your own review.
Score ONLY from the memo text. Quote the memo span that earns each point; if no span supports a point, the score is 0.

=== SEALED RUBRIC ===
# G07 sealed rubric - SEALED (arms never see this)

- **difficulty:** subtle (one concrete, oracle-verified "prod left DOWN" path in the shipped scripts)
- **correct_call:** **build-with-changes.** Design A (operator-booked
  window-batch, local spool, no NATS) is right; B is correctly rejected; C
  stays rejected (an external party's control plane on the production spine
  crosses the custody boundary). But the recovery and containment
  scaffolding must exist before it ships, and the existing window scripts
  already contain a path that leaves production DOWN unattended.
- **oracle:** orchestrator read of the scripts at panel time (and the fix
  applied the same session): both `open` paths stop prod, then on timeout
  `exit 1` with no restart. Feasibility confirmed (the arbiter can control
  prod on both hosts; a dead-man systemd template exists to copy).
- **correct fix:** (1) an independent fail-to-prod-UP dead-man separate from
  the arbiter, boot default prod-UP, a reboot never re-opens a window; (2)
  single-writer atomic state (`flock`, tmp+rename queue transitions,
  idempotency keys, boot reconciliation); (3) cgroup-enforced job control and
  a trusted monotonic deadline that kills the whole process tree; (4) sandbox
  quotas (spool disk/inodes, PID/CPU/IO) and atomic spec ingestion; (5)
  bounded retries plus escalation on a failed prod restart; (6) a job-sandbox
  readiness probe distinct from the GPU-free gate. Immediate: a rollback
  `trap` in both `open` paths.
- **must_catch:**
  1. **Prod left DOWN on a failed open:** `window-rtx.sh:22` stops both
     services, `:24-28` polls VRAM, `:29` `NO-GO ... exit 1` with no
     `start`; `window-gb10.sh:25` stops the containers, `:27-33` polls,
     `:34` `NO-GO ... exit 1` with no `start`. A NO-GO leaves production
     down.
  2. No retry or escalation on a failed close: `window-rtx.sh:34-41` waits
     ~10 min then `exit 1` with the WARNING; `window-gb10.sh:41-48` same;
     nothing retries `systemctl start`/`docker start` or hands off.
  3. No fail-to-prod-UP dead-man or crash-recovery state machine anywhere in
     `queue-design.md` (`:64-65` lists it as an open question); the
     "finishes or kills at the cap" step (`:43-47`) has no kill-tree/cgroup
     or independent deadline, so the operator gate stops the external party *initiating*
     preemption but not *extending* it.
  4. Shared mutable state with no ownership: window flag, spool transitions,
     current-job marker, budget counter (`queue-design.md:37-47,46-47`) -
     two actors or a crash can double-run or lose accounting.
  5. GB10 cannot measure VRAM (`window-gb10.sh:15-19`: `memory.used=[N/A]`
     on Grace-Blackwell; readiness = containers exited + `free` available);
     this is an accepted tradeoff, not a defect.
  6. DoS surface the external party retains with only spec+timing control: fill spool
     disk/inodes, fork/detach past a naive `timeout`, exhaust sandbox
     CPU/RAM/PID/IO, crash the shared GPU driver (`RUNBOOK.md:103-106`:
     isolation is fs/process-level, not GPU-level), corrupt a spec mid-read.
  7. Design C rejection stands: `queue-design.md:52-56`; the middle path if
     the spool is ever outgrown is a narrow trusted-side submission API, not
     raw NATS.
- **landmine (confident wrong answers):** "the window scripts are health-gated
  so prod is safe" (the open path's failure branch abandons prod stopped);
  "use NATS with a scoped user, it is the estate standard" (external control
  plane on the prod spine); "per-job preemption is fine with a warm cache"
  (~92 GB weights, minutes-scale cold start is stated and both seats
  accepted it); "GB10 should gate on nvidia-smi VRAM used" (reports N/A).
- **unverifiable:** actual cold-start duration (not measurable without
  restarting prod); arms may state it as ASSUMPTION and are not scored on the value.
- **outcome_source:** `2026-08-25-external-party-queue-review/ledger.md` (A4/B4 verified
  headline; A6/B7 verified; B5 verified tradeoff; A1/A2/A3/A5/A7 agreed) and
  `verdict.md` (recommendation 1-6; "Fixed in session: window-*.sh open-path
  rollback").


=== MEMO (author unknown) ===
# Verdict - g07 benchmark job queue for an untrusted external party on shared production GPUs

<!-- Mechanical fill from ledger rows only. Seat A / Seat B as in ledger.md; no new arguments. -->

## 1. Independent agreement (agreed-r0)

The panel concludes, from claims both seats made in Round 0 before cross-exposure:
- Design A (window-batch) is the right shape and per-job preemption (Design B) is correctly rejected on minutes-scale cold start (A1, B1).
- A1 is unmet as specified: no sandbox-readiness check, hang handling only via a per-job timeout, no retry when prod restart is not healthy (A4, B5).
- Both close scripts start prod, wait a bounded time for /health 200, then exit 1 with no rollback or recovery; failed opens exit 1 without restoring prod; the ssh/curl helpers have no deadline (A5, B6).
- A4 state (window flag, spool, current job, downtime budget) has no single owner, no named files, no locks; the spool is two-writer (A7, B7).
- A5 is unresolved: no durable crash recovery and no dead-man; CT 205 is onboot=0 (A8, B4-fact).

## 2. Resolved after Round 0

**Oracle-settled (`verified`):** A2 mechanism → close scripts never stop the sandbox (rtx:30-41, gb10:35-48) and RUNBOOK:103-105 states the driver residual; A5/B6 → exit-1-without-rollback and no helper deadlines (rtx:29,41,12-14; gb10:34,48,14-19); A6 → RTX 4000 MiB threshold, GB10 exited+≥60G with unchecked `docker update`, fail-open backup pipeline (rtx:11,26,19-20; gb10:13,29-31); B2-fact → spool inside the sandbox (qd:33-34); B3-fact → no separate closer process specified (qd:37-47); B8-facts → passwordless sudo, GB10 state on the host, 115g cap (RUNBOOK:28,38; gb10:30-31).

**Cross-examination-settled (`conceded` / `revised`):**
- A2 → B conceded the mechanism ("close does not revoke GPU; leftover bench + shared driver can block HEALTHY restore"); A narrowed its wording (0.99→0.98) (R1_codex REVISE Own-2; R1_grok CONCEDE).
- A3, A6 → B conceded trusted-side bounded admission and the non-exclusive, fail-open probes (R1_grok CONCEDE), contesting only the confidence numbers.
- B2 → A conceded "trusted admission into immutable, bounded storage is a sound middle path"; B revised to 0.72 and to "in-sandbox spool is unsafe either way" (R1_codex CONCEDE; R1_grok REVISE Own-2).
- B3 → A conceded independent deadline enforcement as a requirement; B revised to 0.90 adding post-close GPU occupancy (R1_codex CONCEDE; R1_grok REVISE Own-3).
- B4 → A conceded an independent expiry/recovery mechanism is mandatory; B revised to "restore-unsafe = no dead-man AND no GPU fence" (0.88) (R1_codex CONCEDE; R1_grok REVISE Own-4).
- B8 → partly conceded (cleanup, storage exhaustion, host pressure, driver fault "require explicit treatment"); A rejects the bundled certainty.

## 3. Surviving dissent

- **Highest-severity ranking** (A2 vs B4): A holds that benchmark GPU access persisting through close outranks the missing dead-man; B holds "restore-unsafe" is one hole with two mechanisms (no dead-man, no GPU fence). Cheapest discriminating test: crash the arbiter after `open` with the sandbox idle, and separately run `close` with a sandbox process holding the GPU; whichever leaves prod unhealthy longer unattended ranks first.
- **Q2 label** (A: redesign; B: build-with-changes): both require an independent closer, sandbox stop/fence before prod start, and bounded ingest; they differ on whether that is a redesign of the trust boundary or a change list. Test: none mechanical; owner's naming call.
- **Transport preference** (A1 vs B2): A says the artifacts do not establish a local spool over an isolated NATS account and declines to pick; B picks a trusted-side drop directory on criterion 4 and custody. Test: a written comparison of the two admission paths' coupling and resource-isolation properties; absent that, owner's call. Both agree the in-sandbox spool is unsafe either way.
- **B8 certainty**: whether host-backed spool placement and the 115g cap show exhaustion paths or only surfaces. Test: a spool-fill probe with quota state inspected, and a memory-pressure run at the cap with docker/prod observed.

## Recommendation

**Mode:** `dont` (as proposed) — build-with-changes on shape, redesign-or-change-list on the trust boundary.

Bucket 1 settles that the shape is right and that the proposal as written fails A1, A2/A3, A4 and A5 (A4/B5, A5/B6, A7/B7, A8/B4). Bucket 2 adds, conceded by both seats after oracles: close must stop or fence the sandbox before prod starts (A2), admission must be trusted-side, bounded and copied off the party's filesystem (A3/B2), the readiness probes are neither exclusive nor fail-closed (A6), an independent wall-clock closer and dead-man are mandatory (B3/B4). What remains (bucket 3) is the ranking between the two restore-unsafe mechanisms, the Q2 label, and whether to name a transport, none of which changes the required change list.

## Record

- Open (unexamined, not endorsed): none.
- Verified vs merely agreed: verified = A2-mechanism, A5, A6, B2-fact, B3-fact, B8-facts; agreed-only = A1/B1 shape, A4/B5, A7/B7, A8/B4-fact.
- Rounds run / stop rule hit: R0+R1; stopped: no load-bearing claim flipped, no new claim IDs.
- Transformations applied to relayed text: neutral Peer labels, shuffled order, ORCH-SUMMARY (ledger-note.md); nothing else.

VERDICT INPUT: Q1 build-with-changes (keep A, reject B; admission trusted-side, bounded, off the party's filesystem; transport preference is surviving dissent) · Q2 redesign (A) / build-with-changes (B) — converged change list: independent wall-clock closer, sandbox stop/GPU fence before prod start, bounded ingest, deadlines independent of queue depth and estimates · Q3 build-with-changes (dead-man that fails toward close, retry/page on non-200, one writer per state file, fail-closed probes, helper deadlines) · Highest-severity: restore-unsafe — no dead-man after open and close does not revoke the party's GPU access (rank between the two mechanisms is surviving dissent).


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}
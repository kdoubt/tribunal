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

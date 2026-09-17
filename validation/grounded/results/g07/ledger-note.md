ORCH-SUMMARY — settled at Round 0 (both seats independently, before exposure; do not relitigate):
- Design A (window-batch) is the right shape and per-job preemption (B) is correctly rejected on minutes-scale cold start.
- A1 is unmet as specified: no sandbox-readiness check, hang handling only via per-job timeout, no prod-restart retry.
- Both close scripts start prod, wait a bounded time for /health 200, then exit 1 with no rollback or recovery; failed opens also exit 1 without restoring prod; the ssh/curl helpers have no deadline.
- A4 state (window flag, spool, current job, budget) has no single owner, no named files, no locks; the spool is two-writer.
- A5 is unresolved: no durable crash recovery, no dead-man; CT 205 is onboot=0.

ORACLE RESULTS (orchestrator opened every cited span; all pointers resolve):
- Verified verbatim: close scripts never stop the sandbox (rtx:30-41, gb10:35-48); RUNBOOK:103-105 driver-crash residual; failed-open no-rollback (rtx:29, gb10:34); helpers without deadlines (rtx:12-14, gb10:14-19); RTX 4000 MiB threshold and GB10 exited+≥60G proxy with an unchecked `docker update` (rtx:11,26; gb10:13,29-31); fail-open backup check (rtx:19-20); spool inside the sandbox (qd:33-34); no separate closer process named (qd:37-47); passwordless sudo (RUNBOOK:28); GB10 state on the host (RUNBOOK:38); 115g cap (gb10:30-31).
- Not settled by text (still contested): which hole is highest-severity; whether a local spool or an isolated NATS account is safer; the RTX host-disk-fill claim (self-labelled ASSUMPTION).

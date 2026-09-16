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

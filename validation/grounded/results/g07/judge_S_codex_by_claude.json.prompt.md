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
**Framings — ASSUMPTION:** This could be a scheduling-efficiency review, an unattended-recovery review, or a literal assessment of whether hostile workloads can ever disrupt shared hardware. I choose unattended recovery, while retaining the shared-hardware limitation because the decision criteria make production availability decisive.

1. **CLAIM:** Design A is the appropriate scheduling shape, B’s rejection is justified, but the artifacts do not establish that a local spool is safer than carefully isolated NATS admission.
   
   **EVIDENCE:** `artifact/queue-design.md:16–18,23–27,50–56`: minutes-scale cold starts, per-role NATS users, and the stated alternatives.
   
   **CONFIDENCE:** 0.92
   
   **FALSIFIER:** Measured restart costs make per-job switching acceptable, or concrete transport configurations establish a security advantage.
   
   **Recommendation:** Use booked windows with trusted admission and fixed deadlines. Either a bounded spool or isolated broker can supply jobs without authority to open windows. Subject permissions alone do not demonstrate resource isolation. No stable position on which transport is strictly safer without configurations. The “twice” cold-start count at lines 24–27 is unsupported by the described stop/run/restart sequence; batching still avoids repeated restarts.

2. **CLAIM:** Operator-controlled opening is insufficient because neither close script revokes benchmark GPU access, and the runbook explicitly admits workloads can crash the shared driver.
   
   **EVIDENCE:** `artifact/window-rtx.sh:30–41` starts production without stopping CT 205; `artifact/window-gb10.sh:35–48` lowers memory and starts production without stopping `the external party-bench`; `artifact/RUNBOOK.md:103–105`: “A driver-crashing workload still hits prod vLLM.”
   
   **CONFIDENCE:** 0.99
   
   **FALSIFIER:** An independently enforced GPU-access fence exists throughout production operation, and hardware isolation prevents benchmark faults from affecting production.
   
   **Recommendation:** Make host-enforced sandbox termination and GPU release prerequisites for restoration. The highest-severity issue is hostile GPU activity preventing production recovery, even after the operator closes the window. Strict “never DoS production” remains unattainable under the documented shared-driver residual; health checks detect failure but cannot contain it.

3. **CLAIM:** The submission design leaves resource exhaustion and hostile filesystem handling unspecified at a boundary where the submitter controls input and the arbiter controls production.
   
   **EVIDENCE:** `artifact/queue-design.md:30–36,40–42,61–63` specifies sandbox-written JSON and captured output, and explicitly leaves spool exhaustion and arbiter wedging open.
   
   **CONFIDENCE:** 0.98
   
   **FALSIFIER:** An admission specification bounds input, output and resource consumption and safely separates untrusted paths and commands from host operations.
   
   **Recommendation:** Bound bytes, inodes, queued jobs, submission rate, JSON size/depth, directory scanning, stdout/stderr and artifacts. Validate IDs; handle symlinks, special files, traversal and replacement races; copy accepted specs into trusted immutable storage. Pass commands through a fixed sandbox invocation without host-shell interpolation. Enforce host-side CPU, RAM, PID and I/O limits. Treat estimates as untrusted. These are attack surfaces requiring controls, not demonstrated host escapes. A separate conditional LAN route exists upon helper compromise (`artifact/RUNBOOK.md:109–115`).

4. **CLAIM:** A1 has nominal job and window limits, but no specified sandbox-readiness check or termination mechanism sufficient to establish that all job descendants are gone.
   
   **EVIDENCE:** `artifact/queue-design.md:40–47` says “per-job `timeout`,” “kills it at the cap,” and daily budget enforcement; `artifact/RUNBOOK.md:28` grants sandbox passwordless sudo.
   
   **CONFIDENCE:** 0.98
   
   **FALSIFIER:** The implementation verifies readiness and demonstrates bounded termination of signal-resistant, detached and privileged sandbox descendants.
   
   **Recommendation:** Verify sandbox health before dispatch. Use a host-owned execution scope, bounded TERM-to-KILL escalation and whole-sandbox shutdown where necessary. Reserve production restart time within the downtime budget. Neither a user-supplied estimate nor a continuously replenished queue may extend the absolute window deadline.

5. **CLAIM:** The scripts require HTTP health before reporting restoration, but failure paths do not provide unattended recovery or a bounded wall-clock completion time.
   
   **EVIDENCE:** RTX health success and failure: `artifact/window-rtx.sh:34–41`; GB10: `artifact/window-gb10.sh:41–48`; failed opens return without rollback at RTX line 29 and GB10 line 34; SSH/curl helpers have no explicit deadlines at RTX lines 12–14 and GB10 lines 14–19.
   
   **CONFIDENCE:** 0.99
   
   **FALSIFIER:** A supervising mechanism bounds these calls and automatically retries restoration after every unsuccessful open or close.
   
   **Recommendation:** Nonzero exit or deadline expiry must enter a persistent recovery state, prohibit further dispatch, retry restoration with bounded backoff and alert independently. Report closed only after all required health checks pass. Loop iteration counts do not bound a blocked helper call.

6. **CLAIM:** A3’s readiness checks are useful but do not establish exclusive full-GPU availability or reliably enforce every transition prerequisite.
   
   **EVIDENCE:** RTX accepts usage below 4,000 MiB (`artifact/window-rtx.sh:11,26`); GB10 accepts three exited containers and at least 60 GiB available, then reports success without testing the memory-update result (`artifact/window-gb10.sh:13,29–31`); RTX’s backup check is an error-discarding textual pipeline (`artifact/window-rtx.sh:19–20`).
   
   **CONFIDENCE:** 0.97
   
   **FALSIFIER:** Tests demonstrate fail-closed behavior for backup-query errors, failed memory updates and unexpected GPU consumers.
   
   **Recommendation:** Check every mutation’s result, verify effective limits and enumerate unexpected GPU consumers. For GB10, combine exited consumers with validated unified-memory headroom; `[N/A]` is not evidence of zero usage. Parse backup state structurally and abort on unknown state.

7. **CLAIM:** A4’s required state changes lack a defined single owner and atomic transition protocol.
   
   **EVIDENCE:** `artifact/queue-design.md:33–47` assigns spool writes to the submitter, draining and budgets to the arbiter, and opening/closing to operator or scheduler; neither script implements a window-state transaction.
   
   **CONFIDENCE:** 0.98
   
   **FALSIFIER:** A documented, tested ownership and locking protocol covers every transition.
   
   **Recommendation:** Serialize operator/scheduler requests through one trusted owner. Track window phase and deadline, immutable accepted jobs and current attempt, both RTX services or all three GB10 containers, sandbox/device ownership, GB10 memory limits, and durable downtime accounting. Missed updates respectively permit dispatch during closure, lost/duplicate execution, incomplete restoration, overlapping GPU access, incorrect limits or budget overruns. Reconcile manual service/container changes against observed state.

8. **CLAIM:** A5 remains unresolved because the proposal defines neither durable crash recovery nor an independent dead-man restoration mechanism.
   
   **EVIDENCE:** `artifact/queue-design.md:64–65` explicitly asks how to survive crashes without double-running jobs or leaving production down; RTX sandbox boot policy is only `onboot=0` (`artifact/RUNBOOK.md:23`).
   
   **CONFIDENCE:** 0.99
   
   **FALSIFIER:** Crash/reboot tests at every transition demonstrate automatic restoration and controlled job disposition.
   
   **Recommendation:** Persist a bounded lease before stopping production; an independent supervisor must restore production on expiry. Boot into recovery, never resume an old open window automatically. Mark interrupted jobs indeterminate instead of automatically rerunning arbitrary commands. Use external monitoring for supervisor-host loss.

**VERDICT INPUT**

- **Q1:** build-with-changes — retain window batching and bounded admission.
- **Q2:** redesign — enforce GPU revocation; the literal no-DoS guarantee conflicts with the documented shared-driver residual.
- **Q3:** build-with-changes — require durable transitions, bounded operations and independent recovery.
- **Highest-severity issue:** hostile GPU activity can prevent production recovery; operator gating alone does not contain it (claim 2).


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}
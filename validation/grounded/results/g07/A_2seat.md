A_2seat = union of two isolated Round 0 memos, no ledger, no Round 1

=== Memo 1 ===

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


=== Memo 2 ===

I'll read the proposal and the existing window/runbook artifacts first so every claim can be grounded in the files.Next I’ll read the design, runbook, and both window scripts so the architecture and recovery claims can be cited line-by-line.## Framing

Plausible readings: (1) an A-vs-B-vs-C architecture bake-off; (2) an audit against the owner bar (prod never DOWN unattended, the external party never DoS’s or extends preemption, fail-toward-prod-UP, simplest thing that hits 1–3); (3) whether a queue is needed at all versus today’s manual windows.

I take **(2)**. The tie-break is already a safety rule, Design A is the proposed shape, and the proposal itself leaves crash recovery as an open question.

---

```
CLAIM: Window-batch (Design A) is the right shape and Design B is correctly rejected: a minutes-scale cold start of gpt-oss-120b paid per job would flap production for any multi-job suite.
EVIDENCE: queue-design.md:16 “a cold start is minutes-scale and expensive”; queue-design.md:23-27 per-job preemption “each job pays a full prod cold-start (minutes) twice. For a suite of 100 short jobs this is pathological”; queue-design.md:50-51.
CONFIDENCE: 0.85
FALSIFIER: Measured vllm.service ready-to-200 on CT 201 is seconds, not minutes, so per-job stop/start is cheaper than holding one long window.
```

```
CLAIM: Rejecting reuse of the NATS spine (C) is correct; the better middle path is still an operator-gated window, but with a trusted-side spool the external party cannot write — not a subject-scoped NATS user, and not `/scratch/queue/incoming` inside his sandbox.
EVIDENCE: queue-design.md:32-36 spool is `/scratch/queue/incoming/<id>.json` “INSIDE his sandbox”; queue-design.md:52-56 C rejected because it “puts an external untrusted party's control-plane on the production spine”; queue-design.md:17-18 custody is “confidential content never transits NATS”.
CONFIDENCE: 0.80
FALSIFIER: A NATS account whose subjects, publish rights, and payload limits are isolated from every prod consumer, plus a demo that the external party cannot block or poison that account, with no extra broker-coupling cost versus a host-owned drop directory.
```

```
CLAIM: “The operator opens the window, not the external party” blocks him from *starting* preemption but is not sufficient to keep him from *extending* it: drain-until-empty, `est_seconds`, and the budget guard all live in the same arbiter that reads his specs.
EVIDENCE: queue-design.md:37-39 “A window is opened by an OPERATOR … NOT by the external party”; queue-design.md:40-47 drain is FIFO of his queue, close is “operator or a max-window-duration cap”, budget “enforced by the arbiter; … the external party cannot extend it” with no independent timer or second process; spec includes `est_seconds` (queue-design.md:33-34).
CONFIDENCE: 0.85
FALSIFIER: A wall-clock closer that is a separate process (or host), ignores queue depth and `est_seconds`, and force-runs `window-*.sh close` even if the arbiter is blocked in a read of the external party’s spool.
```

```
CLAIM: Highest-severity hole (A5): there is no dead-man, and crash/reboot mid-drain is an unsolved open question, so arbiter death after `window-*.sh open` leaves production DOWN unattended — a direct miss of criteria (1) and (3).
EVIDENCE: queue-design.md:64-65 “how does it survive a crash/reboot mid-drain without double-running a job or leaving prod DOWN?”; no dead-man, current-job marker, or boot policy anywhere in queue-design.md; window-rtx.sh:22 and window-gb10.sh:25 stop prod on open; window-gb10.sh:39 `docker start` is the only restart path (stop persists unless something else starts the containers).
CONFIDENCE: 0.90
FALSIFIER: Spec or code of an independent closer (other host, 2-minute heartbeat, fail-toward `window-*.sh close`) plus a crash-mid-drain test that restores `/health` 200 without a human.
```

```
CLAIM: A1 is unmet as specified: the arbiter does not verify sandbox readiness, does not define hang handling beyond a per-job `timeout`, and specifies no retries when prod restart is not HEALTHY.
EVIDENCE: queue-design.md:40-42 “runs each as `the external party` in the sandbox with a per-job `timeout`”; queue-design.md:46-47 names max-window-duration and max-prod-downtime-per-day; no sandbox-ready check, no `timeout -k` / cgroup kill, no restart retry loop in queue-design.md.
CONFIDENCE: 0.85
FALSIFIER: An arbiter spec that (i) probes sandbox ssh/docker health before each job, (ii) SIGKILL/cgroup-kills the whole sandbox user slice at T+grace, (iii) retries `window-*.sh close` with backoff and pages on persistent non-200.
```

```
CLAIM: A2/A3: both close scripts start prod, wait a bounded time for `/health` 200, then `exit 1` without rolling back; if the arbiter treats that as “done” it can leave prod unattended and not HEALTHY (active ≠ healthy).
EVIDENCE: window-rtx.sh:31-41 `systemctl start` then 100×`sleep 6` until 8000 and 8100 return 200, else “WARNING … 'active' != healthy” `exit 1`; window-gb10.sh:38-48 `docker start` then same 100×6s pattern, `exit 1` on non-200; GB10 free-VRAM proxy is `all_exited` plus `free -g` available ≥ 60, because “nvidia-smi reports memory.used=[N/A]” (window-gb10.sh:15-18, 27-34). Design A only says close “restarts prod health-gated” (queue-design.md:43-45) with no `exit 1` policy.
CONFIDENCE: 0.88
FALSIFIER: Arbiter code that on close `exit 1` keeps retrying/alerting and never marks the window closed, and that starts a job only after open’s `exit 0` (RTX `memory.used` < 4000 MiB; GB10 exited+avail≥60G).
```

```
CLAIM: A4 stores are underspecified and already two-writer: the external party writes the spool the arbiter drains; operator and arbiter can both close; window-open flag, current-job marker, and downtime budget have no named files, locks, or single-writer rule (lost-update precedent applies).
EVIDENCE: queue-design.md:32-36 the external party writes incoming specs; queue-design.md:40-45 arbiter pops/moves to `done/` and operator-or-cap closes; no path named for a window flag or budget counter in queue-design.md. RUNBOOK.md:23 CT 205 `onboot=0` (sandbox does not return on host reboot); whether that re-opens a closed window or leaves GB10 containers stopped is unspecified.
CONFIDENCE: 0.82
FALSIFIER: A state table with one writer per file, `flock`/CAS on flag and budget, atomic ingest off the external party’s FS, and a reboot test showing neither double-run nor auto-reopen of a closed window.
```

```
CLAIM: the external party, controlling only `{cmd, est_seconds, artifacts_path}` and timing, can still harm prod: fill a host-backed spool (GB10 state is on the GB10 host), run side processes that the per-cmd `timeout` does not cover (passwordless sudo in CT 205), exhaust GB10 unified memory while the cap is 115g/121G, or crash the shared GPU driver.
EVIDENCE: queue-design.md:33-36; RUNBOOK.md:28 “User `the external party` HAS passwordless sudo (his sandbox)”; RUNBOOK.md:38 “All state under `/home/<login-user>/the external party-bench/` on gb10”; window-gb10.sh:30-31 bench cap “raised to 115g of 121G unified”; RUNBOOK.md:103-105 “GPU shared with CT 201 — isolation is fs/process-level, not GPU-level. A driver-crashing workload still hits prod vLLM”. Host-disk fill on RTX (CT 205 is 120G local-lvm, RUNBOOK.md:23) is ASSUMPTION unless that LV is thin-provisioned.
CONFIDENCE: 0.80
FALSIFIER: Quota on the spool volume, kill of the whole sandbox cgroup (not just `cmd`) before close, GB10 mem cap that cannot starve docker/prod, and a driver-reset test after which both `/health` endpoints return 200.
```

**VERDICT INPUT**
- Q1 Architecture: **build-with-changes** (keep A, reject B; keep C off the spine; move the spool to trusted side).
- Q2 Security: **build-with-changes** (operator gate stays; it is not a sufficient preemption boundary).
- Q3 Systems: **build-with-changes** (reuse health-gated `window-*.sh`; do not ship without an independent dead-man and a close-`exit 1` policy).
- Highest-severity issue: **no independent dead-man — arbiter crash after open leaves prod DOWN unattended, and the same process that must force-close also reads the external party’s spool.**

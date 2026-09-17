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

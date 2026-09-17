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
## Framing

Plausible framings: **(a)** scheduling-shape question (A vs B vs C); **(b)** availability/recovery question (A1–A5); **(c)** trust-boundary/data-path question. I pick **(c)+(b)**. Reason: the shape is the least contested part — `queue-design.md:24-27` gets the cold-start arithmetic right and B/C are correctly rejected, so affirming A carries little decision value. All the real defects sit in *how the arbiter touches the external party-controlled state* and *what happens when it dies*, and the proposal itself parks those as "Open questions for the panel" (`queue-design.md:58-67`) rather than answering them.

---

```
CLAIM: The only in-artifact mechanism for reaching into the RTX sandbox double-evaluates
its argument through the host's root shell, so routing the external party's `cmd` through it is remote
root on node2 — the highest-severity hole.
EVIDENCE: window-rtx.sh:12 `px() { ssh "$NODE" "pct exec $CT -- $*"; }` — unquoted `$*`
inside a double-quoted ssh string; `pct exec` requires root on node2. Contrast the
proposal's assertion, queue-design.md:36: "The arbiter never executes the external party's code on the
host". Probe: `cmd` = `x; id > /tmp/pwn` and read /tmp/pwn on node2.
CONFIDENCE: 0.8   (0.95 that the double-eval exists; ~0.85 that an implementer reuses
`px`/this idiom given the design says "It may reuse `window-*.sh`")
FALSIFIER: An arbiter implementation that passes `cmd` as an argv element (e.g. Python
`subprocess` list form to `pct exec 205 -- su -s /bin/sh -c`, no shell string) and a test
showing metacharacters land literally inside CT 205.
```

```
CLAIM: The spool lives inside the attacker's sandbox, so every arbiter read is a read of
attacker-controlled bytes on a filesystem he can also fill; nothing in the design bounds
either.
EVIDENCE: queue-design.md:33-34 "the external party writes a job spec to a spool dir INSIDE his sandbox:
`/scratch/queue/incoming/<id>.json`"; RUNBOOK.md:23 CT 205 is "12c/32G/120G local-lvm" and
:29 "Excluded from the hourly PBS job (exclude=114,205)". No quota, schema validation,
count cap, or size cap appears anywhere in queue-design.md.
CONFIDENCE: 0.85
FALSIFIER: A spec mandating copy-out to a trusted-side dir with per-file size, count, and
schema limits before parsing.
```

```
CLAIM: `close` restarts prod without any check that the GPU is free, so a job that outlives
its `timeout` (forked/setsid children are not in `timeout`'s kill scope) puts prod into the
known co-resident OOM/crashloop and leaves it DOWN.
EVIDENCE: window-rtx.sh:24-29 gates `open` on `vram_used < FREE_MIB`; window-rtx.sh:31-32
`close` does `px "systemctl start …"` with no symmetric gate. queue-design.md:41 relies on
"a per-job `timeout`". Precedent: co-resident vLLM crashloop when memory fractions summed
above 1.0.
CONFIDENCE: 0.85
FALSIFIER: A `close` path that fails VRAM/`free`-gated before starting prod, plus a
sandbox-wide process sweep (`pct exec 205 -- pkill -u the external party` / container restart) at job end.
```

```
CLAIM: Nothing in the artifact restores prod if the arbiter or its host dies with a window
open, and `close` failure has no retry — criterion (3) "fail toward prod-UP" is unmet as
proposed.
EVIDENCE: window-rtx.sh:41 and window-gb10.sh:48 both end the failure branch with `exit 1`
and a WARNING string only. queue-design.md:64-65 lists "how does it survive a crash/reboot
mid-drain without double-running a job or leaving prod DOWN?" as an *open question*.
CONFIDENCE: 0.9
FALSIFIER: Pointing to a dead-man in the artifact. (The estate's cross-node watchdog is the
right pattern to copy — arm an `at`/systemd timer at `open` that runs `close` on wall-clock
expiry independent of the arbiter's liveness.)
```

```
CLAIM: A node2 reboot leaves the external party's sandbox stopped, and the design defines no
sandbox-readiness precondition, so the arbiter can take prod down for a window in which no
job can run.
EVIDENCE: RUNBOOK.md:23 "CT 205 the external party-bench-rtx (12c/32G/120G local-lvm, **onboot=0**,
unprivileged)". queue-design.md:38-42 conditions drain only on "while a window is open".
CONFIDENCE: 0.8
FALSIFIER: An arbiter precondition asserting CT 205 running + `nvidia-smi` succeeding
*inside* 205 before `window-rtx.sh open` is called.
```

```
CLAIM: The GB10 close path is not the inverse of open and checks nothing it runs: it
restores `--memory` but not `--memory-swap`, and neither script uses `set -e` or tests the
`docker update` exit status.
EVIDENCE: window-gb10.sh:30 `docker update --memory 115g --memory-swap 115g the external party-bench`
vs :37 `docker update --memory 20g the external party-bench` (no `--memory-swap`), both `>/dev/null` with
no `||`; window-gb10.sh:9 / window-rtx.sh:8 `set -uo pipefail` (no `-e`). Probe:
`docker inspect -f '{{.HostConfig.MemorySwap}}' the external party-bench` after a close.
CONFIDENCE: 0.9 (that the asymmetry exists; ~0.4 that it causes prod harm on a
low/no-swap unified-memory box)
FALSIFIER: `MemorySwap` reading 20g (or -1 consistently) after close.
```

```
CLAIM: Rejecting Design C is correct, and for a stronger reason than the one given: CT 205
is a no-uplink L2 island, so putting the external party on NATS requires demolishing the build's strongest
containment property — a subject-scoped NATS user is strictly *worse* than a local spool.
EVIDENCE: RUNBOOK.md:101-102 "CT 205 is on a no-uplink L2 island: LAN unreachable by
construction (verified: a prod host cannot reach <island-address>)"; RUNBOOK.md:21-22 bastion
"nftables drops all private-range egress except <island-address>:22 and <lan-address>".
The proposal's stated reason (queue-design.md:53-55, custody boundary "the reverse") is a
weaker, partly mis-applied argument.
CONFIDENCE: 0.85
FALSIFIER: An existing NATS route reachable from <island-address> that I missed.
```

```
CLAIM: "The operator opens the window, not the external party" does not satisfy criterion (1), because the
proposal already permits unattended opens, and the external party does hold a scheduling input that can
stretch a window.
EVIDENCE: queue-design.md:37-38 "opened by an OPERATOR (**or a scheduled cron the operator
owns**)" — a cron open is unattended by definition; queue-design.md:35 the spec includes
`est_seconds`, which is the external party-authored and is the natural admission input for "does this job
fit in the remaining window".
CONFIDENCE: 0.8
FALSIFIER: A spec that ignores `est_seconds` for admission (using only a fixed per-job cap)
AND requires a human ack for every open.
```

---

## Recommended middle path

Keep A's window batching; change three things. (1) **Move the boundary**: the external party writes to a quota'd drop dir; the arbiter copies out, validates a strict schema, and execs `cmd` only via argv-list `pct exec … su the external party -c` — never a shell-interpolated host string. (2) **Make the window self-closing**: arm a wall-clock `at`/systemd timer *at open* that runs `close`; the arbiter's liveness must not be load-bearing for prod-UP. (3) **Make `close` symmetric with `open`**: sweep the sandbox, re-verify GPU free, then start prod, and retry with alerting on non-200.

## VERDICT INPUT

- **Q1 Architecture:** build-as-proposed — window-batch is the right shape; B and C correctly rejected.
- **Q2 Security / trust boundary:** redesign — the submission data path (spool inside the sandbox + `cmd` through a host root shell) must be rebuilt before any code.
- **Q3 Systems / operational (A1–A5):** build-with-changes — A1–A5 are unanswered in the artifact, not answered wrongly; they need specification (dead-man close, sandbox-readiness precondition, close-failure retry, locked budget counter).

**Highest-severity issue overall:** `window-rtx.sh:12`'s `ssh "$NODE" "pct exec $CT -- $*"` — reusing that idiom to launch the external party's `cmd` converts the queue's only attacker-controlled field into root command execution on the node2 host, defeating every containment property in the RUNBOOK at once.


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}
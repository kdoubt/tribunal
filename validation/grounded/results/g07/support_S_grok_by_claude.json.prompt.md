You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: Window-batch (Design A) is the right shape and Design B is correctly rejected: a minutes-scale cold start of gpt-oss-120b paid per job would flap production for any multi-job suite.
POINTER: queue-design.md:16
OPENED TEXT:
→ bastion). He must NOT be able to stop/start production or exceed a downtime budget.
- gpt-oss-120b is ~92 GB of weights; a cold start is minutes-scale and expensive.
- Existing estate async backbone = NATS JetStream spine (`jobs.batch-infer.>`,
  per-role users, custody boundary: confidential content never transits NATS).

[2] CLAIM: Window-batch (Design A) is the right shape and Design B is correctly rejected: a minutes-scale cold start of gpt-oss-120b paid per job would flap production for any multi-job suite.
POINTER: queue-design.md:23-27
OPENED TEXT:
Two ways to schedule that:
- **Per-job preemption** (stop prod → run one job → restart prod → repeat): each job
  pays a full prod cold-start (minutes) twice. For a suite of 100 short jobs this is
  pathological, and it flaps HAL chat continuously.
- **Window batching** (stop prod once → drain the external party's whole queue → restart prod once):
  prod pays the cold-start twice total, per window. the external party's jobs run back-to-back.

## Proposed design (Design A — window-batch queue)
A small **trusted arbiter daemon** (runs on the trusted side: node2 host for RTX,

[3] CLAIM: Window-batch (Design A) is the right shape and Design B is correctly rejected: a minutes-scale cold start of gpt-oss-120b paid per job would flap production for any multi-job suite.
POINTER: queue-design.md:50-51
OPENED TEXT:
### Why not the alternatives
- **Design B (per-job autonomous preemption):** rejected — prod cold-start per job is
  minutes; flapping HAL chat continuously is worse than a scheduled block.
- **Design C (reuse NATS `jobs.batch-infer`):** the estate backbone, but it puts an
  external untrusted party's control-plane on the production spine. Custody boundary
  says confidential content never transits NATS; here the risk is the reverse

[4] CLAIM: Rejecting reuse of the NATS spine (C) is correct; the better middle path is still an operator-gated window, but with a trusted-side spool the external party cannot write — not a subject-scoped NATS user, and not `/scratch/queue/incoming` inside his sandbox.
POINTER: queue-design.md:32-36
OPENED TEXT:
GB10 host as `<login-user>` for GB10 — both can control prod; the external party cannot). Not on NATS.

1. **Submission.** the external party writes a job spec to a spool dir INSIDE his sandbox:
   `/scratch/queue/incoming/<id>.json` = `{cmd, est_seconds, artifacts_path}`.
   `cmd` runs as `the external party` INSIDE his sandbox only. The arbiter never executes the external party's
   code on the host — it only sequences and starts/stops prod.
2. **Window as a trusted gate.** A window is opened by an OPERATOR (or a scheduled
   cron the operator owns), NOT by the external party. Opening a window = `window-*.sh open`
   (health-gated, frees full GPU). The arbiter runs jobs only while a window is open.

[5] CLAIM: Rejecting reuse of the NATS spine (C) is correct; the better middle path is still an operator-gated window, but with a trusted-side spool the external party cannot write — not a subject-scoped NATS user, and not `/scratch/queue/incoming` inside his sandbox.
POINTER: queue-design.md:52-56
OPENED TEXT:
minutes; flapping HAL chat continuously is worse than a scheduled block.
- **Design C (reuse NATS `jobs.batch-infer`):** the estate backbone, but it puts an
  external untrusted party's control-plane on the production spine. Custody boundary
  says confidential content never transits NATS; here the risk is the reverse
  (external → prod bus). Deferred unless the panel judges a subject-scoped, separate-
  account NATS user strictly better than a local spool.

## Open questions for the panel
- Is window-batch (A) correct, or is there a middle path (e.g. opportunistic: run

[6] CLAIM: Rejecting reuse of the NATS spine (C) is correct; the better middle path is still an operator-gated window, but with a trusted-side spool the external party cannot write — not a subject-scoped NATS user, and not `/scratch/queue/incoming` inside his sandbox.
POINTER: queue-design.md:17-18
OPENED TEXT:
- gpt-oss-120b is ~92 GB of weights; a cold start is minutes-scale and expensive.
- Existing estate async backbone = NATS JetStream spine (`jobs.batch-infer.>`,
  per-role users, custody boundary: confidential content never transits NATS).

## The core tension
Production must come DOWN for the external party's job to have the full GPU, and back UP after.

[7] CLAIM: “The operator opens the window, not the external party” blocks him from *starting* preemption but is not sufficient to keep him from *extending* it: drain-until-empty, `est_seconds`, and the budget guard all live in the same arbiter that reads his specs.
POINTER: queue-design.md:37-39
OPENED TEXT:
code on the host — it only sequences and starts/stops prod.
2. **Window as a trusted gate.** A window is opened by an OPERATOR (or a scheduled
   cron the operator owns), NOT by the external party. Opening a window = `window-*.sh open`
   (health-gated, frees full GPU). The arbiter runs jobs only while a window is open.
3. **Drain.** While the window is open, the arbiter pops specs in FIFO order, runs
   each as `the external party` in the sandbox with a per-job `timeout`, captures exit/stdout/stderr
   to `/scratch/queue/done/<id>/`, and moves to the next. One job at a time.

[8] CLAIM: “The operator opens the window, not the external party” blocks him from *starting* preemption but is not sufficient to keep him from *extending* it: drain-until-empty, `est_seconds`, and the budget guard all live in the same arbiter that reads his specs.
POINTER: queue-design.md:40-47
OPENED TEXT:
(health-gated, frees full GPU). The arbiter runs jobs only while a window is open.
3. **Drain.** While the window is open, the arbiter pops specs in FIFO order, runs
   each as `the external party` in the sandbox with a per-job `timeout`, captures exit/stdout/stderr
   to `/scratch/queue/done/<id>/`, and moves to the next. One job at a time.
4. **Close.** Window close (operator or a max-window-duration cap) → arbiter finishes
   the current job or kills it at the cap, then `window-*.sh close` restarts prod
   health-gated. Un-run specs stay queued for the next window.
5. **Budget guard.** A max-window-duration and a max-prod-downtime-per-day budget are
   enforced by the arbiter; exceeding either forces a close. the external party cannot extend it.

### Why not the alternatives
- **Design B (per-job autonomous preemption):** rejected — prod cold-start per job is

[9] CLAIM: “The operator opens the window, not the external party” blocks him from *starting* preemption but is not sufficient to keep him from *extending* it: drain-until-empty, `est_seconds`, and the budget guard all live in the same arbiter that reads his specs.
POINTER: queue-design.md:33-34
OPENED TEXT:
1. **Submission.** the external party writes a job spec to a spool dir INSIDE his sandbox:
   `/scratch/queue/incoming/<id>.json` = `{cmd, est_seconds, artifacts_path}`.
   `cmd` runs as `the external party` INSIDE his sandbox only. The arbiter never executes the external party's
   code on the host — it only sequences and starts/stops prod.
2. **Window as a trusted gate.** A window is opened by an OPERATOR (or a scheduled

[10] CLAIM: Highest-severity hole (A5): there is no dead-man, and crash/reboot mid-drain is an unsolved open question, so arbiter death after `window-*.sh open` leaves production DOWN unattended — a direct miss of criteria (1) and (3).
POINTER: queue-design.md:64-65
OPENED TEXT:
exhaust the sandbox to affect the host)?
- Where should the arbiter run and how does it survive a crash/reboot mid-drain
  without double-running a job or leaving prod DOWN?
- Booked-block scheduling (operator opens windows on a calendar) vs. a request-driven
  model where the external party's submission *notifies* the operator to open a window — which is safer?

[11] CLAIM: Highest-severity hole (A5): there is no dead-man, and crash/reboot mid-drain is an unsolved open question, so arbiter death after `window-*.sh open` leaves production DOWN unattended — a direct miss of criteria (1) and (3).
POINTER: window-rtx.sh:22
OPENED TEXT:
echo "[open] stopping prod vLLM services in CT $CT…"
  px "systemctl stop vllm.service vllm-embed.service"
  echo -n "[open] waiting for VRAM to free"
  for i in $(seq 1 60); do
    u=$(vram_used); echo -n " ${u}MiB"

[12] CLAIM: Highest-severity hole (A5): there is no dead-man, and crash/reboot mid-drain is an unsolved open question, so arbiter death after `window-*.sh open` leaves production DOWN unattended — a direct miss of criteria (1) and (3).
POINTER: window-gb10.sh:25
OPENED TEXT:
echo "[open] stopping prod vLLM containers…"
  gb "docker stop $CONTAINERS"
  echo -n "[open] waiting for containers exited + unified memory freed"
  for i in $(seq 1 60); do
    a=$(avail_gb); echo -n " ${a}Gavail"

[13] CLAIM: Highest-severity hole (A5): there is no dead-man, and crash/reboot mid-drain is an unsolved open question, so arbiter death after `window-*.sh open` leaves production DOWN unattended — a direct miss of criteria (1) and (3).
POINTER: window-gb10.sh:39
OPENED TEXT:
echo "[close] restarting prod vLLM containers…"
  gb "docker start $CONTAINERS"
  echo -n "[close] waiting for all 3 endpoints healthy"
  for i in $(seq 1 100); do
    ok=1; line=""

[14] CLAIM: A1 is unmet as specified: the arbiter does not verify sandbox readiness, does not define hang handling beyond a per-job `timeout`, and specifies no retries when prod restart is not HEALTHY.
POINTER: queue-design.md:40-42
OPENED TEXT:
(health-gated, frees full GPU). The arbiter runs jobs only while a window is open.
3. **Drain.** While the window is open, the arbiter pops specs in FIFO order, runs
   each as `the external party` in the sandbox with a per-job `timeout`, captures exit/stdout/stderr
   to `/scratch/queue/done/<id>/`, and moves to the next. One job at a time.
4. **Close.** Window close (operator or a max-window-duration cap) → arbiter finishes
   the current job or kills it at the cap, then `window-*.sh close` restarts prod
   health-gated. Un-run specs stay queued for the next window.

[15] CLAIM: A1 is unmet as specified: the arbiter does not verify sandbox readiness, does not define hang handling beyond a per-job `timeout`, and specifies no retries when prod restart is not HEALTHY.
POINTER: queue-design.md:46-47
OPENED TEXT:
health-gated. Un-run specs stay queued for the next window.
5. **Budget guard.** A max-window-duration and a max-prod-downtime-per-day budget are
   enforced by the arbiter; exceeding either forces a close. the external party cannot extend it.

### Why not the alternatives
- **Design B (per-job autonomous preemption):** rejected — prod cold-start per job is

[16] CLAIM: A2/A3: both close scripts start prod, wait a bounded time for `/health` 200, then `exit 1` without rolling back; if the arbiter treats that as “done” it can leave prod unattended and not HEALTHY (active ≠ healthy).
POINTER: window-rtx.sh:31-41
OPENED TEXT:
close)
  echo "[close] restarting prod vLLM services…"
  px "systemctl start vllm.service vllm-embed.service"
  echo -n "[close] waiting for BOTH endpoints healthy (not just active)"
  for i in $(seq 1 100); do
    h1=$(health 8000); h2=$(health 8100)
    echo -n " [$i:8000=$h1,8100=$h2]"
    if [ "$h1" = 200 ] && [ "$h2" = 200 ]; then
      echo; echo "[close] RESTORED — gpt-oss-120b(:8000) + embed(:8100) both healthy. Confirm LiteLLM route next."; exit 0; fi
    sleep 6
  done
  echo; echo "[close] WARNING — a service is active but NOT healthy after wait (h8000=$h1 h8100=$h2). 'active' != healthy — investigate before declaring restored."; exit 1 ;;
status)
  echo "VRAM used: $(vram_used) MiB"
  echo "vllm.service:      $(px 'systemctl is-active vllm.service')   /health=$(health 8000)"

[17] CLAIM: A2/A3: both close scripts start prod, wait a bounded time for `/health` 200, then `exit 1` without rolling back; if the arbiter treats that as “done” it can leave prod unattended and not HEALTHY (active ≠ healthy).
POINTER: window-gb10.sh:38-48
OPENED TEXT:
gb "docker update --memory 20g the external party-bench" >/dev/null
  echo "[close] restarting prod vLLM containers…"
  gb "docker start $CONTAINERS"
  echo -n "[close] waiting for all 3 endpoints healthy"
  for i in $(seq 1 100); do
    ok=1; line=""
    for p in $PORTS; do h=$(health $p); line="$line $p=$h"; [ "$h" = 200 ] || ok=0; done
    echo -n " [$i:$line]"
    [ "$ok" = 1 ] && { echo; echo "[close] RESTORED — qwen3-30b + rerank + embed all healthy."; exit 0; }
    sleep 6
  done
  echo; echo "[close] WARNING — not all healthy after wait ($line). 'Up' != healthy — investigate."; exit 1 ;;
status)
  echo "unified mem available: $(avail_gb) G (nvidia-smi VRAM 'used' is [N/A] on Grace-Blackwell)"
  gb "docker ps --filter name=vllm --format '{{.Names}} {{.Status}}'"

[18] CLAIM: A2/A3: both close scripts start prod, wait a bounded time for `/health` 200, then `exit 1` without rolling back; if the arbiter treats that as “done” it can leave prod unattended and not HEALTHY (active ≠ healthy).
POINTER: window-gb10.sh:15-18
OPENED TEXT:
gb() { ssh "$GB10" "$*"; }
# GB10 is Grace-Blackwell UNIFIED memory: nvidia-smi reports memory.used=[N/A], so the
# readiness signal is (containers exited) + (system 'available' RAM), not VRAM 'used'.
avail_gb() { gb "free -g | awk '/^Mem:/{print \$7}'"; }
all_exited() { for c in $CONTAINERS; do [ "$(gb "docker inspect -f '{{.State.Status}}' $c" 2>/dev/null)" = exited ] || return 1; done; return 0; }
health() { gb "curl -s -o /dev/null -w '%{http_code}' http://127.0.0.1:$1/health" 2>/dev/null; }

case "${1:-status}" in

[19] CLAIM: A2/A3: both close scripts start prod, wait a bounded time for `/health` 200, then `exit 1` without rolling back; if the arbiter treats that as “done” it can leave prod unattended and not HEALTHY (active ≠ healthy).
POINTER: queue-design.md:43-45
OPENED TEXT:
to `/scratch/queue/done/<id>/`, and moves to the next. One job at a time.
4. **Close.** Window close (operator or a max-window-duration cap) → arbiter finishes
   the current job or kills it at the cap, then `window-*.sh close` restarts prod
   health-gated. Un-run specs stay queued for the next window.
5. **Budget guard.** A max-window-duration and a max-prod-downtime-per-day budget are
   enforced by the arbiter; exceeding either forces a close. the external party cannot extend it.

[20] CLAIM: A4 stores are underspecified and already two-writer: the external party writes the spool the arbiter drains; operator and arbiter can both close; window-open flag, current-job marker, and downtime budget have no named files, locks, or single-writer rule (lost-update precedent applies).
POINTER: queue-design.md:32-36
OPENED TEXT:
GB10 host as `<login-user>` for GB10 — both can control prod; the external party cannot). Not on NATS.

1. **Submission.** the external party writes a job spec to a spool dir INSIDE his sandbox:
   `/scratch/queue/incoming/<id>.json` = `{cmd, est_seconds, artifacts_path}`.
   `cmd` runs as `the external party` INSIDE his sandbox only. The arbiter never executes the external party's
   code on the host — it only sequences and starts/stops prod.
2. **Window as a trusted gate.** A window is opened by an OPERATOR (or a scheduled
   cron the operator owns), NOT by the external party. Opening a window = `window-*.sh open`
   (health-gated, frees full GPU). The arbiter runs jobs only while a window is open.

[21] CLAIM: A4 stores are underspecified and already two-writer: the external party writes the spool the arbiter drains; operator and arbiter can both close; window-open flag, current-job marker, and downtime budget have no named files, locks, or single-writer rule (lost-update precedent applies).
POINTER: queue-design.md:40-45
OPENED TEXT:
(health-gated, frees full GPU). The arbiter runs jobs only while a window is open.
3. **Drain.** While the window is open, the arbiter pops specs in FIFO order, runs
   each as `the external party` in the sandbox with a per-job `timeout`, captures exit/stdout/stderr
   to `/scratch/queue/done/<id>/`, and moves to the next. One job at a time.
4. **Close.** Window close (operator or a max-window-duration cap) → arbiter finishes
   the current job or kills it at the cap, then `window-*.sh close` restarts prod
   health-gated. Un-run specs stay queued for the next window.
5. **Budget guard.** A max-window-duration and a max-prod-downtime-per-day budget are
   enforced by the arbiter; exceeding either forces a close. the external party cannot extend it.

[22] CLAIM: A4 stores are underspecified and already two-writer: the external party writes the spool the arbiter drains; operator and arbiter can both close; window-open flag, current-job marker, and downtime budget have no named files, locks, or single-writer rule (lost-update precedent applies).
POINTER: RUNBOOK.md:23
OPENED TEXT:
:8888 + dnsmasq :53 serve the island (only <island-address> allowed).
- **CT 205 the external party-bench-rtx** (12c/32G/120G local-lvm, onboot=0, unprivileged):
  on SDN vnet `cirub0` (zone `cirub`, node2-only, NO uplink — physical L2 island,
  no NAT anywhere). GPU via /dev/nvidia* bind mounts + cgroup allows c 195/510
  (same pattern as CT 201). NVIDIA user-space 610.43.02 via .run --no-kernel-modules

[23] CLAIM: the external party, controlling only `{cmd, est_seconds, artifacts_path}` and timing, can still harm prod: fill a host-backed spool (GB10 state is on the GB10 host), run side processes that the per-cmd `timeout` does not cover (passwordless sudo in CT 205), exhaust GB10 unified memory while the cap is 115g/121G, or crash the shared GPU driver.
POINTER: queue-design.md:33-36
OPENED TEXT:
1. **Submission.** the external party writes a job spec to a spool dir INSIDE his sandbox:
   `/scratch/queue/incoming/<id>.json` = `{cmd, est_seconds, artifacts_path}`.
   `cmd` runs as `the external party` INSIDE his sandbox only. The arbiter never executes the external party's
   code on the host — it only sequences and starts/stops prod.
2. **Window as a trusted gate.** A window is opened by an OPERATOR (or a scheduled
   cron the operator owns), NOT by the external party. Opening a window = `window-*.sh open`
   (health-gated, frees full GPU). The arbiter runs jobs only while a window is open.

[24] CLAIM: the external party, controlling only `{cmd, est_seconds, artifacts_path}` and timing, can still harm prod: fill a host-backed spool (GB10 state is on the GB10 host), run side processes that the per-cmd `timeout` does not cover (passwordless sudo in CT 205), exhaust GB10 unified memory while the cap is 115g/121G, or crash the shared GPU driver.
POINTER: RUNBOOK.md:28
OPENED TEXT:
(matches node2 host driver EXACTLY — if node2 driver is upgraded, rerun the .run
  in 205 with the new version). User `the external party` HAS passwordless sudo (his sandbox).
  Internet only via bastion tinyproxy (http/https, set in /etc/profile.d/proxy.sh,
  apt.conf.d/95proxy). Verified: HF reachable, Infisical/PBS NOT reachable.
  Excluded from the hourly PBS job (exclude=114,205) — scratch box, 120G churn.

[25] CLAIM: the external party, controlling only `{cmd, est_seconds, artifacts_path}` and timing, can still harm prod: fill a host-backed spool (GB10 state is on the GB10 host), run side processes that the per-cmd `timeout` does not cover (passwordless sudo in CT 205), exhaust GB10 unified memory while the cap is 115g/121G, or crash the shared GPU driver.
POINTER: RUNBOOK.md:38
OPENED TEXT:
squid proxy container (172.30.99.3:3128) allows internet but denies RFC1918.
  All state under `/home/<login-user>/the external party-bench/` on gb10.

## Onboarding the external party (when email + SSH pubkey arrive)

[26] CLAIM: the external party, controlling only `{cmd, est_seconds, artifacts_path}` and timing, can still harm prod: fill a host-backed spool (GB10 state is on the GB10 host), run side processes that the per-cmd `timeout` does not cover (passwordless sudo in CT 205), exhaust GB10 unified memory while the cap is 115g/121G, or crash the shared GPU driver.
POINTER: window-gb10.sh:30-31
OPENED TEXT:
if all_exited && [ "${a:-0}" -ge "$FREE_GB" ]; then
      gb "docker update --memory 115g --memory-swap 115g the external party-bench" >/dev/null
      echo; echo "[open] GO — 3 containers exited, ${a}G available; bench mem cap raised to 115g of 121G unified (≈full box, ~6G host headroom). GPU is the external party's."; exit 0; fi
    sleep 3
  done
  echo; echo "[open] NO-GO — containers not all exited or memory not freed (${a}G avail). Check for a hung container."; exit 1 ;;

[27] CLAIM: the external party, controlling only `{cmd, est_seconds, artifacts_path}` and timing, can still harm prod: fill a host-backed spool (GB10 state is on the GB10 host), run side processes that the per-cmd `timeout` does not cover (passwordless sudo in CT 205), exhaust GB10 unified memory while the cap is 115g/121G, or crash the shared GPU driver.
POINTER: RUNBOOK.md:103-105
OPENED TEXT:
a prod host cannot reach <island-address>).
- CT 205 is unprivileged; **GPU shared with CT 201 — isolation is fs/process-level,
  not GPU-level.** A driver-crashing workload still hits prod vLLM; the window
  scripts' health gates are the guard, not a hardware boundary.
- GB10 bench container: `--internal` network, egress only via squid ACL, `HEALTHCHECK`
  on sshd; setup script is idempotent with a health readiness gate (no blind sleep).

[28] CLAIM: the external party, controlling only `{cmd, est_seconds, artifacts_path}` and timing, can still harm prod: fill a host-backed spool (GB10 state is on the GB10 host), run side processes that the per-cmd `timeout` does not cover (passwordless sudo in CT 205), exhaust GB10 unified memory while the cap is 115g/121G, or crash the shared GPU driver.
POINTER: RUNBOOK.md:23
OPENED TEXT:
:8888 + dnsmasq :53 serve the island (only <island-address> allowed).
- **CT 205 the external party-bench-rtx** (12c/32G/120G local-lvm, onboot=0, unprivileged):
  on SDN vnet `cirub0` (zone `cirub`, node2-only, NO uplink — physical L2 island,
  no NAT anywhere). GPU via /dev/nvidia* bind mounts + cgroup allows c 195/510
  (same pattern as CT 201). NVIDIA user-space 610.43.02 via .run --no-kernel-modules

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }
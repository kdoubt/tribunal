You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: ** Design A is the appropriate scheduling shape, B’s rejection is justified, but the artifacts do not establish that a local spool is safer than carefully isolated NATS admission.
POINTER: queue-design.md:16-18
OPENED TEXT:
→ bastion). He must NOT be able to stop/start production or exceed a downtime budget.
- gpt-oss-120b is ~92 GB of weights; a cold start is minutes-scale and expensive.
- Existing estate async backbone = NATS JetStream spine (`jobs.batch-infer.>`,
  per-role users, custody boundary: confidential content never transits NATS).

## The core tension
Production must come DOWN for the external party's job to have the full GPU, and back UP after.

[2] CLAIM: ** Operator-controlled opening is insufficient because neither close script revokes benchmark GPU access, and the runbook explicitly admits workloads can crash the shared driver.
POINTER: window-rtx.sh:30-41
OPENED TEXT:
echo; echo "[open] NO-GO — VRAM did not free (stuck at ${u}MiB). Check for a hung vLLM proc on the RTX."; exit 1 ;;
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

[3] CLAIM: ** Operator-controlled opening is insufficient because neither close script revokes benchmark GPU access, and the runbook explicitly admits workloads can crash the shared driver.
POINTER: window-gb10.sh:35-48
OPENED TEXT:
echo; echo "[open] NO-GO — containers not all exited or memory not freed (${a}G avail). Check for a hung container."; exit 1 ;;
close)
  echo "[close] lowering bench mem cap to 20g…"
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

[4] CLAIM: ** Operator-controlled opening is insufficient because neither close script revokes benchmark GPU access, and the runbook explicitly admits workloads can crash the shared driver.
POINTER: RUNBOOK.md:103-105
OPENED TEXT:
a prod host cannot reach <island-address>).
- CT 205 is unprivileged; **GPU shared with CT 201 — isolation is fs/process-level,
  not GPU-level.** A driver-crashing workload still hits prod vLLM; the window
  scripts' health gates are the guard, not a hardware boundary.
- GB10 bench container: `--internal` network, egress only via squid ACL, `HEALTHCHECK`
  on sshd; setup script is idempotent with a health readiness gate (no blind sleep).

[5] CLAIM: ** The submission design leaves resource exhaustion and hostile filesystem handling unspecified at a boundary where the submitter controls input and the arbiter controls production.
POINTER: queue-design.md:30-36
OPENED TEXT:
## Proposed design (Design A — window-batch queue)
A small **trusted arbiter daemon** (runs on the trusted side: node2 host for RTX,
GB10 host as `<login-user>` for GB10 — both can control prod; the external party cannot). Not on NATS.

1. **Submission.** the external party writes a job spec to a spool dir INSIDE his sandbox:
   `/scratch/queue/incoming/<id>.json` = `{cmd, est_seconds, artifacts_path}`.
   `cmd` runs as `the external party` INSIDE his sandbox only. The arbiter never executes the external party's
   code on the host — it only sequences and starts/stops prod.
2. **Window as a trusted gate.** A window is opened by an OPERATOR (or a scheduled
   cron the operator owns), NOT by the external party. Opening a window = `window-*.sh open`
   (health-gated, frees full GPU). The arbiter runs jobs only while a window is open.

[6] CLAIM: ** A1 has nominal job and window limits, but no specified sandbox-readiness check or termination mechanism sufficient to establish that all job descendants are gone.
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

[7] CLAIM: ** A1 has nominal job and window limits, but no specified sandbox-readiness check or termination mechanism sufficient to establish that all job descendants are gone.
POINTER: RUNBOOK.md:28
OPENED TEXT:
(matches node2 host driver EXACTLY — if node2 driver is upgraded, rerun the .run
  in 205 with the new version). User `the external party` HAS passwordless sudo (his sandbox).
  Internet only via bastion tinyproxy (http/https, set in /etc/profile.d/proxy.sh,
  apt.conf.d/95proxy). Verified: HF reachable, Infisical/PBS NOT reachable.
  Excluded from the hourly PBS job (exclude=114,205) — scratch box, 120G churn.

[8] CLAIM: ** The scripts require HTTP health before reporting restoration, but failure paths do not provide unattended recovery or a bounded wall-clock completion time.
POINTER: window-rtx.sh:34-41
OPENED TEXT:
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

[9] CLAIM: ** The scripts require HTTP health before reporting restoration, but failure paths do not provide unattended recovery or a bounded wall-clock completion time.
POINTER: window-gb10.sh:41-48
OPENED TEXT:
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

[10] CLAIM: ** A3’s readiness checks are useful but do not establish exclusive full-GPU availability or reliably enforce every transition prerequisite.
POINTER: window-rtx.sh:11
OPENED TEXT:
CT=201
FREE_MIB=4000                 # VRAM 'used' must drop below this to call it freed
px() { ssh "$NODE" "pct exec $CT -- $*"; }
vram_used() { ssh "$NODE" "nvidia-smi --query-gpu=memory.used --format=csv,noheader,nounits" | tr -d ' '; }
health() { px "curl -s -o /dev/null -w '%{http_code}' http://127.0.0.1:$1/health" 2>/dev/null; }

[11] CLAIM: ** A3’s readiness checks are useful but do not establish exclusive full-GPU availability or reliably enforce every transition prerequisite.
POINTER: window-gb10.sh:13
OPENED TEXT:
PORTS="8200 8101 8100"
FREE_GB=60                     # unified-mem 'available' must rise above this to call it freed
gb() { ssh "$GB10" "$*"; }
# GB10 is Grace-Blackwell UNIFIED memory: nvidia-smi reports memory.used=[N/A], so the
# readiness signal is (containers exited) + (system 'available' RAM), not VRAM 'used'.

[12] CLAIM: ** A3’s readiness checks are useful but do not establish exclusive full-GPU availability or reliably enforce every transition prerequisite.
POINTER: window-rtx.sh:19-20
OPENED TEXT:
echo "[open] checking cluster isn't mid-backup (CT 201 = live)…"
  pvesh get /cluster/tasks --output-format json 2>/dev/null | grep -q '"status":"running".*vzdump' && \
    { echo "  ABORT: a vzdump is running — wait for it (feedback_check_tasks_before_reboot)"; exit 1; }
  echo "[open] stopping prod vLLM services in CT $CT…"
  px "systemctl stop vllm.service vllm-embed.service"
  echo -n "[open] waiting for VRAM to free"

[13] CLAIM: ** A4’s required state changes lack a defined single owner and atomic transition protocol.
POINTER: queue-design.md:33-47
OPENED TEXT:
1. **Submission.** the external party writes a job spec to a spool dir INSIDE his sandbox:
   `/scratch/queue/incoming/<id>.json` = `{cmd, est_seconds, artifacts_path}`.
   `cmd` runs as `the external party` INSIDE his sandbox only. The arbiter never executes the external party's
   code on the host — it only sequences and starts/stops prod.
2. **Window as a trusted gate.** A window is opened by an OPERATOR (or a scheduled
   cron the operator owns), NOT by the external party. Opening a window = `window-*.sh open`
   (health-gated, frees full GPU). The arbiter runs jobs only while a window is open.
3. **Drain.** While the window is open, the arbiter pops specs in FIFO order, runs
   each as `the external party` in the sandbox with a per-job `timeout`, captures exit/stdout/stderr
   to `/scratch/queue/done/<id>/`, and moves to the next. One job at a time.
4. **Close.** Window close (operator or a max-window-duration cap) → arbiter finishes
   the current job or kills it at the cap, then `window-*.sh close` restarts prod
   health-gated. Un-run specs stay queued for the next window.
5. **Budget guard.** A max-window-duration and a max-prod-downtime-per-day budget are
   enforced by the arbiter; exceeding either forces a close. the external party cannot extend it.

#

[14] CLAIM: ** A5 remains unresolved because the proposal defines neither durable crash recovery nor an independent dead-man restoration mechanism.
POINTER: queue-design.md:64-65
OPENED TEXT:
exhaust the sandbox to affect the host)?
- Where should the arbiter run and how does it survive a crash/reboot mid-drain
  without double-running a job or leaving prod DOWN?
- Booked-block scheduling (operator opens windows on a calendar) vs. a request-driven
  model where the external party's submission *notifies* the operator to open a window — which is safer?

[15] CLAIM: ** A5 remains unresolved because the proposal defines neither durable crash recovery nor an independent dead-man restoration mechanism.
POINTER: RUNBOOK.md:23
OPENED TEXT:
:8888 + dnsmasq :53 serve the island (only <island-address> allowed).
- **CT 205 the external party-bench-rtx** (12c/32G/120G local-lvm, onboot=0, unprivileged):
  on SDN vnet `cirub0` (zone `cirub`, node2-only, NO uplink — physical L2 island,
  no NAT anywhere). GPU via /dev/nvidia* bind mounts + cgroup allows c 195/510
  (same pattern as CT 201). NVIDIA user-space 610.43.02 via .run --no-kernel-modules

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }
You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: The only in-artifact mechanism for reaching into the RTX sandbox double-evaluates
POINTER: window-rtx.sh:12
OPENED TEXT:
FREE_MIB=4000                 # VRAM 'used' must drop below this to call it freed
px() { ssh "$NODE" "pct exec $CT -- $*"; }
vram_used() { ssh "$NODE" "nvidia-smi --query-gpu=memory.used --format=csv,noheader,nounits" | tr -d ' '; }
health() { px "curl -s -o /dev/null -w '%{http_code}' http://127.0.0.1:$1/health" 2>/dev/null; }

[2] CLAIM: The only in-artifact mechanism for reaching into the RTX sandbox double-evaluates
POINTER: queue-design.md:36
OPENED TEXT:
`cmd` runs as `the external party` INSIDE his sandbox only. The arbiter never executes the external party's
   code on the host — it only sequences and starts/stops prod.
2. **Window as a trusted gate.** A window is opened by an OPERATOR (or a scheduled
   cron the operator owns), NOT by the external party. Opening a window = `window-*.sh open`
   (health-gated, frees full GPU). The arbiter runs jobs only while a window is open.

[3] CLAIM: The spool lives inside the attacker's sandbox, so every arbiter read is a read of
POINTER: queue-design.md:33-34
OPENED TEXT:
1. **Submission.** the external party writes a job spec to a spool dir INSIDE his sandbox:
   `/scratch/queue/incoming/<id>.json` = `{cmd, est_seconds, artifacts_path}`.
   `cmd` runs as `the external party` INSIDE his sandbox only. The arbiter never executes the external party's
   code on the host — it only sequences and starts/stops prod.
2. **Window as a trusted gate.** A window is opened by an OPERATOR (or a scheduled

[4] CLAIM: The spool lives inside the attacker's sandbox, so every arbiter read is a read of
POINTER: RUNBOOK.md:23
OPENED TEXT:
:8888 + dnsmasq :53 serve the island (only <island-address> allowed).
- **CT 205 the external party-bench-rtx** (12c/32G/120G local-lvm, onboot=0, unprivileged):
  on SDN vnet `cirub0` (zone `cirub`, node2-only, NO uplink — physical L2 island,
  no NAT anywhere). GPU via /dev/nvidia* bind mounts + cgroup allows c 195/510
  (same pattern as CT 201). NVIDIA user-space 610.43.02 via .run --no-kernel-modules

[5] CLAIM: `close` restarts prod without any check that the GPU is free, so a job that outlives
POINTER: window-rtx.sh:24-29
OPENED TEXT:
echo -n "[open] waiting for VRAM to free"
  for i in $(seq 1 60); do
    u=$(vram_used); echo -n " ${u}MiB"
    [ "${u:-99999}" -lt "$FREE_MIB" ] && { echo; echo "[open] GO — ${u}MiB used (<$FREE_MIB). GPU is the external party's. Verify inside CT 205: nvidia-smi"; exit 0; }
    sleep 3
  done
  echo; echo "[open] NO-GO — VRAM did not free (stuck at ${u}MiB). Check for a hung vLLM proc on the RTX."; exit 1 ;;
close)
  echo "[close] restarting prod vLLM services…"
  px "systemctl start vllm.service vllm-embed.service"

[6] CLAIM: `close` restarts prod without any check that the GPU is free, so a job that outlives
POINTER: window-rtx.sh:31-32
OPENED TEXT:
close)
  echo "[close] restarting prod vLLM services…"
  px "systemctl start vllm.service vllm-embed.service"
  echo -n "[close] waiting for BOTH endpoints healthy (not just active)"
  for i in $(seq 1 100); do
    h1=$(health 8000); h2=$(health 8100)

[7] CLAIM: `close` restarts prod without any check that the GPU is free, so a job that outlives
POINTER: queue-design.md:41
OPENED TEXT:
3. **Drain.** While the window is open, the arbiter pops specs in FIFO order, runs
   each as `the external party` in the sandbox with a per-job `timeout`, captures exit/stdout/stderr
   to `/scratch/queue/done/<id>/`, and moves to the next. One job at a time.
4. **Close.** Window close (operator or a max-window-duration cap) → arbiter finishes
   the current job or kills it at the cap, then `window-*.sh close` restarts prod

[8] CLAIM: Nothing in the artifact restores prod if the arbiter or its host dies with a window
POINTER: window-rtx.sh:41
OPENED TEXT:
done
  echo; echo "[close] WARNING — a service is active but NOT healthy after wait (h8000=$h1 h8100=$h2). 'active' != healthy — investigate before declaring restored."; exit 1 ;;
status)
  echo "VRAM used: $(vram_used) MiB"
  echo "vllm.service:      $(px 'systemctl is-active vllm.service')   /health=$(health 8000)"

[9] CLAIM: Nothing in the artifact restores prod if the arbiter or its host dies with a window
POINTER: window-gb10.sh:48
OPENED TEXT:
done
  echo; echo "[close] WARNING — not all healthy after wait ($line). 'Up' != healthy — investigate."; exit 1 ;;
status)
  echo "unified mem available: $(avail_gb) G (nvidia-smi VRAM 'used' is [N/A] on Grace-Blackwell)"
  gb "docker ps --filter name=vllm --format '{{.Names}} {{.Status}}'"

[10] CLAIM: Nothing in the artifact restores prod if the arbiter or its host dies with a window
POINTER: queue-design.md:64-65
OPENED TEXT:
exhaust the sandbox to affect the host)?
- Where should the arbiter run and how does it survive a crash/reboot mid-drain
  without double-running a job or leaving prod DOWN?
- Booked-block scheduling (operator opens windows on a calendar) vs. a request-driven
  model where the external party's submission *notifies* the operator to open a window — which is safer?

[11] CLAIM: A node2 reboot leaves the external party's sandbox stopped, and the design defines no
POINTER: RUNBOOK.md:23
OPENED TEXT:
:8888 + dnsmasq :53 serve the island (only <island-address> allowed).
- **CT 205 the external party-bench-rtx** (12c/32G/120G local-lvm, onboot=0, unprivileged):
  on SDN vnet `cirub0` (zone `cirub`, node2-only, NO uplink — physical L2 island,
  no NAT anywhere). GPU via /dev/nvidia* bind mounts + cgroup allows c 195/510
  (same pattern as CT 201). NVIDIA user-space 610.43.02 via .run --no-kernel-modules

[12] CLAIM: A node2 reboot leaves the external party's sandbox stopped, and the design defines no
POINTER: queue-design.md:38-42
OPENED TEXT:
2. **Window as a trusted gate.** A window is opened by an OPERATOR (or a scheduled
   cron the operator owns), NOT by the external party. Opening a window = `window-*.sh open`
   (health-gated, frees full GPU). The arbiter runs jobs only while a window is open.
3. **Drain.** While the window is open, the arbiter pops specs in FIFO order, runs
   each as `the external party` in the sandbox with a per-job `timeout`, captures exit/stdout/stderr
   to `/scratch/queue/done/<id>/`, and moves to the next. One job at a time.
4. **Close.** Window close (operator or a max-window-duration cap) → arbiter finishes
   the current job or kills it at the cap, then `window-*.sh close` restarts prod
   health-gated. Un-run specs stay queued for the next window.

[13] CLAIM: The GB10 close path is not the inverse of open and checks nothing it runs: it
POINTER: window-gb10.sh:30
OPENED TEXT:
if all_exited && [ "${a:-0}" -ge "$FREE_GB" ]; then
      gb "docker update --memory 115g --memory-swap 115g the external party-bench" >/dev/null
      echo; echo "[open] GO — 3 containers exited, ${a}G available; bench mem cap raised to 115g of 121G unified (≈full box, ~6G host headroom). GPU is the external party's."; exit 0; fi
    sleep 3
  done

[14] CLAIM: The GB10 close path is not the inverse of open and checks nothing it runs: it
POINTER: window-gb10.sh:9
OPENED TEXT:
# open a window while a panel is running.
set -uo pipefail
GB10=gb10
CONTAINERS="vllm-qwen3-30b vllm-rerank vllm-embed"
PORTS="8200 8101 8100"

[15] CLAIM: The GB10 close path is not the inverse of open and checks nothing it runs: it
POINTER: window-rtx.sh:8
OPENED TEXT:
# AND vllm-embed.service (Qwen3-Embedding-4B, :8100). BOTH must stop for a clean window.
set -uo pipefail
NODE=<lan-address>            # node2 host
CT=201
FREE_MIB=4000                 # VRAM 'used' must drop below this to call it freed

[16] CLAIM: Rejecting Design C is correct, and for a stronger reason than the one given: CT 205
POINTER: RUNBOOK.md:101-102
OPENED TEXT:
- Bastion user has no root; egress firewall + proxy policy live where he can't touch them.
- CT 205 is on a no-uplink L2 island: LAN unreachable by construction (verified:
  a prod host cannot reach <island-address>).
- CT 205 is unprivileged; **GPU shared with CT 201 — isolation is fs/process-level,
  not GPU-level.** A driver-crashing workload still hits prod vLLM; the window
  scripts' health gates are the guard, not a hardware boundary.

[17] CLAIM: Rejecting Design C is correct, and for a stronger reason than the one given: CT 205
POINTER: RUNBOOK.md:21-22
OPENED TEXT:
cloudflared connector; sshd key-only, user `the external party` (NO sudo); nftables drops all
  private-range egress except <island-address>:22 and <lan-address>; tinyproxy
  :8888 + dnsmasq :53 serve the island (only <island-address> allowed).
- **CT 205 the external party-bench-rtx** (12c/32G/120G local-lvm, onboot=0, unprivileged):
  on SDN vnet `cirub0` (zone `cirub`, node2-only, NO uplink — physical L2 island,
  no NAT anywhere). GPU via /dev/nvidia* bind mounts + cgroup allows c 195/510

[18] CLAIM: Rejecting Design C is correct, and for a stronger reason than the one given: CT 205
POINTER: queue-design.md:53-55
OPENED TEXT:
- **Design C (reuse NATS `jobs.batch-infer`):** the estate backbone, but it puts an
  external untrusted party's control-plane on the production spine. Custody boundary
  says confidential content never transits NATS; here the risk is the reverse
  (external → prod bus). Deferred unless the panel judges a subject-scoped, separate-
  account NATS user strictly better than a local spool.

## Open questions for the panel

[19] CLAIM: "The operator opens the window, not the external party" does not satisfy criterion (1), because the
POINTER: queue-design.md:37-38
OPENED TEXT:
code on the host — it only sequences and starts/stops prod.
2. **Window as a trusted gate.** A window is opened by an OPERATOR (or a scheduled
   cron the operator owns), NOT by the external party. Opening a window = `window-*.sh open`
   (health-gated, frees full GPU). The arbiter runs jobs only while a window is open.
3. **Drain.** While the window is open, the arbiter pops specs in FIFO order, runs
   each as `the external party` in the sandbox with a per-job `timeout`, captures exit/stdout/stderr

[20] CLAIM: "The operator opens the window, not the external party" does not satisfy criterion (1), because the
POINTER: queue-design.md:35
OPENED TEXT:
`/scratch/queue/incoming/<id>.json` = `{cmd, est_seconds, artifacts_path}`.
   `cmd` runs as `the external party` INSIDE his sandbox only. The arbiter never executes the external party's
   code on the host — it only sequences and starts/stops prod.
2. **Window as a trusted gate.** A window is opened by an OPERATOR (or a scheduled
   cron the operator owns), NOT by the external party. Opening a window = `window-*.sh open`

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }
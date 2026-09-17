You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: Co-mingling the spool on the same `all_squash` export that CT 206 (and CT 205 `/queue`) get RW makes the untrusted party a same-credential writer of `incoming/`, `running/`, and `done/`, which breaks CONTRACT single-writer and at-most-once.
POINTER: unified-nfs-design.md:16-28
OPENED TEXT:
## The design
**One Synology NFS share is the shared substrate** (workspace + models + queue spool):
```
/volume1/the external party-warm/
  workspace/            # the external party's scripts/configs (RW to the external party)
  models/               # staged model files (RW to the external party, RO to jobs)
  queue/rtx/{incoming,running,done}      # per-GPU spool
  queue/gb10/{incoming,running,done}
```
Mounted via the HOST, then bind-mounted into each sandbox — so the island/internal
sandboxes get the FILES without a network ROUTE to the NAS (isolation preserved):
- **node2** mounts `the external party-warm` → bind into **CT 205** (RTX sandbox) at /workspace, /models(ro),
  /queue; and into a NEW **CT 206 `the external party-code`** (unified code-server, node2, isolated,
  bastion-reachable) RW.
- **gb10** mounts `the external party-warm` → bind into the GB10 job/bench containers (same paths).
- **node1** (arbiter host) mounts `the external party-warm` → arbiter reads/writes the spool LOCALLY
  (replaces today's over-ssh spool reads).

[2] CLAIM: Co-mingling the spool on the same `all_squash` export that CT 206 (and CT 205 `/queue`) get RW makes the untrusted party a same-credential writer of `incoming/`, `running/`, and `done/`, which breaks CONTRACT single-writer and at-most-once.
POINTER: CONTRACT.md:50-51
OPENED TEXT:
window-overrun or stale heartbeat, and on boot ensures prod-UP if no valid window.
2. **Single-writer atomic state** — one arbiter (flock); every state write is tmp+rename;
   queue claim is an atomic rename incoming→running; budget counter is monotonic per day.
3. **Cgroup kill-tree per job** — RTX: `systemd-run --scope -p RuntimeMaxSec` (systemd
   kills the whole cgroup at the deadline). GB10: ephemeral `docker run --rm` with
   `--memory/--pids-limit/--cpus` + `timeout`, arbiter `docker kill` fallback.

[3] CLAIM: Host-mount then bind-mount does not give CT 205 or the GB10 `--internal` container a new L3 route to the NAS; the unspecified CT 206 network is the residual path to the live `*` / `no_root_squash` backup exports.
POINTER: unified-nfs-design.md:24-25
OPENED TEXT:
```
Mounted via the HOST, then bind-mounted into each sandbox — so the island/internal
sandboxes get the FILES without a network ROUTE to the NAS (isolation preserved):
- **node2** mounts `the external party-warm` → bind into **CT 205** (RTX sandbox) at /workspace, /models(ro),
  /queue; and into a NEW **CT 206 `the external party-code`** (unified code-server, node2, isolated,
  bastion-reachable) RW.

[4] CLAIM: Host-mount then bind-mount does not give CT 205 or the GB10 `--internal` container a new L3 route to the NAS; the unspecified CT 206 network is the residual path to the live `*` / `no_root_squash` backup exports.
POINTER: RUNBOOK.md:23-25
OPENED TEXT:
:8888 + dnsmasq :53 serve the island (only <island-address> allowed).
- **CT 205 the external party-bench-rtx** (12c/32G/120G local-lvm, onboot=0, unprivileged):
  on SDN vnet `cirub0` (zone `cirub`, node2-only, NO uplink — physical L2 island,
  no NAT anywhere). GPU via /dev/nvidia* bind mounts + cgroup allows c 195/510
  (same pattern as CT 201). NVIDIA user-space 610.43.02 via .run --no-kernel-modules
  (matches node2 host driver EXACTLY — if node2 driver is upgraded, rerun the .run
  in 205 with the new version). User `the external party` HAS passwordless sudo (his sandbox).

[5] CLAIM: Host-mount then bind-mount does not give CT 205 or the GB10 `--internal` container a new L3 route to the NAS; the unspecified CT 206 network is the residual path to the live `*` / `no_root_squash` backup exports.
POINTER: synology-exports.txt:2
OPENED TEXT:
Export list for <lan-address>:
/volume1/proxmoxbackup/<lan-address>/24(rw,async,no_wdelay,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/proxmox<lan-address>(rw,async,no_wdelay,crossmnt,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/1-RPX-Workspace-Data                                                                                              <lan-address>
/volume1/NeoMoat-Hub                                                                                                       <lan-address>

[6] CLAIM: Host-mount then bind-mount does not give CT 205 or the GB10 `--internal` container a new L3 route to the NAS; the unspecified CT 206 network is the residual path to the live `*` / `no_root_squash` backup exports.
POINTER: unified-nfs-design.md:27-28
OPENED TEXT:
- **node2** mounts `the external party-warm` → bind into **CT 205** (RTX sandbox) at /workspace, /models(ro),
  /queue; and into a NEW **CT 206 `the external party-code`** (unified code-server, node2, isolated,
  bastion-reachable) RW.
- **gb10** mounts `the external party-warm` → bind into the GB10 job/bench containers (same paths).
- **node1** (arbiter host) mounts `the external party-warm` → arbiter reads/writes the spool LOCALLY
  (replaces today's over-ssh spool reads).

[7] CLAIM: Highest-severity hole as written is spool mutation (copy `running/<id>.json` back to `incoming/` or a new id with the same `cmd`) so the arbiter drains and runs the job twice — a tie-break failure; a close second is unconstrained RW on `/volume1` with no quota next to `proxmoxbackup`.
POINTER: CONTRACT.md:66-67
OPENED TEXT:
(health-gated, rollback on fail), writes `window.flag{open, expires_at=now+120m}`.
- The arbiter (already running) sees the open flag and drains the queue until the flag
  closes or expires.
- `queuectl --host rtx close` → `window-rtx.sh close` (retry), clears the flag.
- Hard caps: `WINDOW_MAX_MIN` per window, `DAILY_BUDGET_MIN` total prod-downtime/day.
  the external party cannot open, extend, or bypass these.

[8] CLAIM: Highest-severity hole as written is spool mutation (copy `running/<id>.json` back to `incoming/` or a new id with the same `cmd`) so the arbiter drains and runs the job twice — a tie-break failure; a close second is unconstrained RW on `/volume1` with no quota next to `proxmoxbackup`.
POINTER: synology-exports.txt:2-3
OPENED TEXT:
Export list for <lan-address>:
/volume1/proxmoxbackup/<lan-address>/24(rw,async,no_wdelay,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/proxmox<lan-address>(rw,async,no_wdelay,crossmnt,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/1-RPX-Workspace-Data                                                                                              <lan-address>
/volume1/NeoMoat-Hub                                                                                                       <lan-address>
/volume1/proxmoxbackup                                                                                                     <lan-address>

[9] CLAIM: Highest-severity hole as written is spool mutation (copy `running/<id>.json` back to `incoming/` or a new id with the same `cmd`) so the arbiter drains and runs the job twice — a tie-break failure; a close second is unconstrained RW on `/volume1` with no quota next to `proxmoxbackup`.
POINTER: unified-nfs-design.md:17-23
OPENED TEXT:
**One Synology NFS share is the shared substrate** (workspace + models + queue spool):
```
/volume1/the external party-warm/
  workspace/            # the external party's scripts/configs (RW to the external party)
  models/               # staged model files (RW to the external party, RO to jobs)
  queue/rtx/{incoming,running,done}      # per-GPU spool
  queue/gb10/{incoming,running,done}
```
Mounted via the HOST, then bind-mounted into each sandbox — so the island/internal
sandboxes get the FILES without a network ROUTE to the NAS (isolation preserved):
- **node2** mounts `the external party-warm` → bind into **CT 205** (RTX sandbox) at /workspace, /models(ro),

[10] CLAIM: Highest-severity hole as written is spool mutation (copy `running/<id>.json` back to `incoming/` or a new id with the same `cmd`) so the arbiter drains and runs the job twice — a tie-break failure; a close second is unconstrained RW on `/volume1` with no quota next to `proxmoxbackup`.
POINTER: CONTRACT.md:55-57
OPENED TEXT:
`--memory/--pids-limit/--cpus` + `timeout`, arbiter `docker kill` fallback.
4. **Sandbox quotas** — arbiter refuses to claim when the spool exceeds MAX_SPOOL_BYTES
   or MAX_SPOOL_FILES, and rejects a spec over MAX_SPEC_BYTES; per-job PID/CPU/mem caps
   via the run context. (Hard fs-quota on GB10's dir needs root — documented, deferred.)
5. **Restart retry + escalation** — window close retries prod restart; on failure it
   leaves the window.flag CLOSED so the dead-man/next-open re-asserts prod-UP, and alerts.
6. **Readiness probe** — before each job the arbiter verifies the sandbox exec responds

[11] CLAIM: A1 — copy the GB10 client opts `vers=3,soft,timeo=150,retrans=3,_netdev,nofail,nolock` for RW, not `hard`; `nofail` without a contents-level readiness probe lets a failed mount leave a local empty directory that the arbiter will treat as the spool.
POINTER: gb10-nfs-precedent.txt:1
OPENED TEXT:
<lan-address>:/volume1/1-RPX-Workspace-Data  /mnt/nas-workspace  nfs  ro,vers=3,nolock,rsize=1048576,wsize=1048576,soft,timeo=150,retrans=3,_netdev,nofail  0  0
---mount---

[12] CLAIM: A1 — copy the GB10 client opts `vers=3,soft,timeo=150,retrans=3,_netdev,nofail,nolock` for RW, not `hard`; `nofail` without a contents-level readiness probe lets a failed mount leave a local empty directory that the arbiter will treat as the spool.
POINTER: unified-nfs-design.md:44-45
OPENED TEXT:
NOT no_root_squash (unlike the existing proxmoxbackup exports).
- Mount opts mirror the working GB10 precedent: `vers=3, soft, timeo=150, retrans=3,
  _netdev, nofail, nolock`; models bind RO into sandboxes; workspace/queue RW.
- uid alignment: Synology maps anon to uid 1025; the sandbox `the external party` users are 1000/1001.
  The export will all_squash to one uid and the sandboxes' bind mounts run as that uid.

[13] CLAIM: A1 — copy the GB10 client opts `vers=3,soft,timeo=150,retrans=3,_netdev,nofail,nolock` for RW, not `hard`; `nofail` without a contents-level readiness probe lets a failed mount leave a local empty directory that the arbiter will treat as the spool.
POINTER: CONTRACT.md:60-61
OPENED TEXT:
leaves the window.flag CLOSED so the dead-man/next-open re-asserts prod-UP, and alerts.
6. **Readiness probe** — before each job the arbiter verifies the sandbox exec responds
   and (via window open) the GPU is actually free.

## Booked-block model
- `queuectl --host rtx open 120` → checks daily budget, runs `window-rtx.sh open`

[14] CLAIM: A2 — NFSv3 `rename` on one export is still a single-RPC atomic claim *if* only the arbiter can write `running/` and the arbiter mount uses `actimeo=0` (or `noac`); as proposed, `nolock` is fine (flock is local) but default attribute cache plus the external party RW on `queue/` is not.
POINTER: CONTRACT.md:21-24
OPENED TEXT:
## On-disk layout
### Management host (arbiter state, single-writer)
`~/.the external party-queue/<host>/`  (host = rtx|gb10)
- `arbiter.lock`   — flock; only one arbiter per host runs.
- `state.json`     — `{budget:{date, used_seconds}}`, updated atomically (tmp+rename).

### GPU host (readable by the LOCAL dead-man, written by the arbiter over ssh)
- node2: `/var/lib/the external party-queue/`   GB10: `/home/<login-user>/the external party-queue/`

[15] CLAIM: A2 — NFSv3 `rename` on one export is still a single-RPC atomic claim *if* only the arbiter can write `running/` and the arbiter mount uses `actimeo=0` (or `noac`); as proposed, `nolock` is fine (flock is local) but default attribute cache plus the external party RW on `queue/` is not.
POINTER: unified-nfs-design.md:44-45
OPENED TEXT:
NOT no_root_squash (unlike the existing proxmoxbackup exports).
- Mount opts mirror the working GB10 precedent: `vers=3, soft, timeo=150, retrans=3,
  _netdev, nofail, nolock`; models bind RO into sandboxes; workspace/queue RW.
- uid alignment: Synology maps anon to uid 1025; the sandbox `the external party` users are 1000/1001.
  The export will all_squash to one uid and the sandboxes' bind mounts run as that uid.

[16] CLAIM: A5 — dead-man still fail-to-prod-UP on NAS death because `window.flag` and `heartbeat` stay on GPU-host local disk; an arbiter that keeps rewriting heartbeat while spool I/O returns EIO leaves prod down until `expires_at`.
POINTER: CONTRACT.md:14-16
OPENED TEXT:
window. Reuses the health-gated `window-*.sh` (with fail-to-prod-UP rollback).
- **Dead-man** — runs ON each GPU host, INDEPENDENT of the arbiter and the management
  host. Restores prod if a window overruns its deadline or the arbiter's heartbeat
  goes stale. This is the gate-1 safety net: prod-UP survives arbiter/host death.
- **the external party** — writes job specs into his sandbox spool via `the external party-submit`; never controls
  windows or preemption.

[17] CLAIM: A5 — dead-man still fail-to-prod-UP on NAS death because `window.flag` and `heartbeat` stay on GPU-host local disk; an arbiter that keeps rewriting heartbeat while spool I/O returns EIO leaves prod down until `expires_at`.
POINTER: unified-nfs-design.md:30-31
OPENED TEXT:
- **gb10** mounts `the external party-warm` → bind into the GB10 job/bench containers (same paths).
- **node1** (arbiter host) mounts `the external party-warm` → arbiter reads/writes the spool LOCALLY
  (replaces today's over-ssh spool reads).

**Access:** retire the two per-GPU code-server endpoints; the bastion fronts ONE new
ingress `the bench hostname` → CT 206 code-server, one CF Access app (OTP), policy allows

[18] CLAIM: A dedicated `root_squash+all_squash`, 3-IP, no-`crossmnt` export is defensible; A3 is not `showmount` — prove a fourth IP cannot `mount` `the external party-warm` *or* `proxmoxbackup`, and that each sandbox has no TCP/2049 to `<lan-address>`.
POINTER: unified-nfs-design.md:41-43
OPENED TEXT:
## NFS specifics (proposed)
- New export `the external party-warm`, **root_squash + all_squash to a dedicated `the external party` uid/gid**,
  limited to the three host IPs (node1 .10, node2 .11, gb10 .225) — NOT the whole subnet,
  NOT no_root_squash (unlike the existing proxmoxbackup exports).
- Mount opts mirror the working GB10 precedent: `vers=3, soft, timeo=150, retrans=3,
  _netdev, nofail, nolock`; models bind RO into sandboxes; workspace/queue RW.
- uid alignment: Synology maps anon to uid 1025; the sandbox `the external party` users are 1000/1001.

[19] CLAIM: A dedicated `root_squash+all_squash`, 3-IP, no-`crossmnt` export is defensible; A3 is not `showmount` — prove a fourth IP cannot `mount` `the external party-warm` *or* `proxmoxbackup`, and that each sandbox has no TCP/2049 to `<lan-address>`.
POINTER: synology-exports.txt:2-3
OPENED TEXT:
Export list for <lan-address>:
/volume1/proxmoxbackup/<lan-address>/24(rw,async,no_wdelay,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/proxmox<lan-address>(rw,async,no_wdelay,crossmnt,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/1-RPX-Workspace-Data                                                                                              <lan-address>
/volume1/NeoMoat-Hub                                                                                                       <lan-address>
/volume1/proxmoxbackup                                                                                                     <lan-address>

[20] CLAIM: A dedicated `root_squash+all_squash`, 3-IP, no-`crossmnt` export is defensible; A3 is not `showmount` — prove a fourth IP cannot `mount` `the external party-warm` *or* `proxmoxbackup`, and that each sandbox has no TCP/2049 to `<lan-address>`.
POINTER: gb10-nfs-precedent.txt:1
OPENED TEXT:
<lan-address>:/volume1/1-RPX-Workspace-Data  /mnt/nas-workspace  nfs  ro,vers=3,nolock,rsize=1048576,wsize=1048576,soft,timeo=150,retrans=3,_netdev,nofail  0  0
---mount---

[21] CLAIM: A dedicated `root_squash+all_squash`, 3-IP, no-`crossmnt` export is defensible; A3 is not `showmount` — prove a fourth IP cannot `mount` `the external party-warm` *or* `proxmoxbackup`, and that each sandbox has no TCP/2049 to `<lan-address>`.
POINTER: synology-exports.txt:4
OPENED TEXT:
/volume1/proxmox<lan-address>(rw,async,no_wdelay,crossmnt,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/1-RPX-Workspace-Data                                                                                              <lan-address>
/volume1/NeoMoat-Hub                                                                                                       <lan-address>
/volume1/proxmoxbackup                                                                                                     <lan-address>

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }
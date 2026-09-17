You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: The design's safety rests entirely on the untrusted party having no IP route to
POINTER: synology-exports.txt:2-3
OPENED TEXT:
Export list for <lan-address>:
/volume1/proxmoxbackup/<lan-address>/24(rw,async,no_wdelay,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/proxmox<lan-address>(rw,async,no_wdelay,crossmnt,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/1-RPX-Workspace-Data                                                                                              <lan-address>
/volume1/NeoMoat-Hub                                                                                                       <lan-address>
/volume1/proxmoxbackup                                                                                                     <lan-address>

[2] CLAIM: The design's safety rests entirely on the untrusted party having no IP route to
POINTER: unified-nfs-design.md:27-28
OPENED TEXT:
- **node2** mounts `the external party-warm` → bind into **CT 205** (RTX sandbox) at /workspace, /models(ro),
  /queue; and into a NEW **CT 206 `the external party-code`** (unified code-server, node2, isolated,
  bastion-reachable) RW.
- **gb10** mounts `the external party-warm` → bind into the GB10 job/bench containers (same paths).
- **node1** (arbiter host) mounts `the external party-warm` → arbiter reads/writes the spool LOCALLY
  (replaces today's over-ssh spool reads).

[3] CLAIM: The design's safety rests entirely on the untrusted party having no IP route to
POINTER: RUNBOOK.md:24
OPENED TEXT:
- **CT 205 the external party-bench-rtx** (12c/32G/120G local-lvm, onboot=0, unprivileged):
  on SDN vnet `cirub0` (zone `cirub`, node2-only, NO uplink — physical L2 island,
  no NAT anywhere). GPU via /dev/nvidia* bind mounts + cgroup allows c 195/510
  (same pattern as CT 201). NVIDIA user-space 610.43.02 via .run --no-kernel-modules
  (matches node2 host driver EXACTLY — if node2 driver is upgraded, rerun the .run

[4] CLAIM: all_squash→anonuid 1025 plus unprivileged-LXC uid shifting means the party's
POINTER: unified-nfs-design.md:46-47
OPENED TEXT:
_netdev, nofail, nolock`; models bind RO into sandboxes; workspace/queue RW.
- uid alignment: Synology maps anon to uid 1025; the sandbox `the external party` users are 1000/1001.
  The export will all_squash to one uid and the sandboxes' bind mounts run as that uid.

## Open questions for the panel
- Is one shared NFS FS as workspace+models+spool the right shape, or does co-mingling

[5] CLAIM: all_squash→anonuid 1025 plus unprivileged-LXC uid shifting means the party's
POINTER: RUNBOOK.md:23
OPENED TEXT:
:8888 + dnsmasq :53 serve the island (only <island-address> allowed).
- **CT 205 the external party-bench-rtx** (12c/32G/120G local-lvm, onboot=0, unprivileged):
  on SDN vnet `cirub0` (zone `cirub`, node2-only, NO uplink — physical L2 island,
  no NAT anywhere). GPU via /dev/nvidia* bind mounts + cgroup allows c 195/510
  (same pattern as CT 201). NVIDIA user-space 610.43.02 via .run --no-kernel-modules

[6] CLAIM: Copying `soft` from the precedent onto a read-write mount is a category error: the
POINTER: gb10-nfs-precedent.txt:1
OPENED TEXT:
<lan-address>:/volume1/1-RPX-Workspace-Data  /mnt/nas-workspace  nfs  ro,vers=3,nolock,rsize=1048576,wsize=1048576,soft,timeo=150,retrans=3,_netdev,nofail  0  0
---mount---

[7] CLAIM: Copying `soft` from the precedent onto a read-write mount is a category error: the
POINTER: unified-nfs-design.md:44-45
OPENED TEXT:
NOT no_root_squash (unlike the existing proxmoxbackup exports).
- Mount opts mirror the working GB10 precedent: `vers=3, soft, timeo=150, retrans=3,
  _netdev, nofail, nolock`; models bind RO into sandboxes; workspace/queue RW.
- uid alignment: Synology maps anon to uid 1025; the sandbox `the external party` users are 1000/1001.
  The export will all_squash to one uid and the sandboxes' bind mounts run as that uid.

[8] CLAIM: Putting the spool on a share the party holds RW breaks the arbiter's single-writer
POINTER: CONTRACT.md:73-75
OPENED TEXT:
## At-most-once guarantee
Claim is `incoming→running` (atomic). On arbiter boot, any leftover `running/<id>.json`
(a job interrupted by a crash) is moved to `done/<id>/` with `status:interrupted` and is
**not** re-run. A job therefore runs zero or one times, never twice.

[9] CLAIM: Putting the spool on a share the party holds RW breaks the arbiter's single-writer
POINTER: unified-nfs-design.md:16-22
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

[10] CLAIM: No quota is specified for `the external party-warm` on a volume that also holds Proxmox backups, so
POINTER: unified-nfs-design.md:10-11
OPENED TEXT:
work goes through the queue (window-batch, prod-protected); no interactive card access.
- Large model files (tens of GB) are warm-stored on the Synology (Vault-SCP, <lan-address>,
  SSD-cached) and reused across jobs on both GPUs — not shipped per job.
- The RTX sandbox (CT 205) is a NO-UPLINK L2 island; the GB10 sandbox is a docker
  container on an `--internal` net. These isolation boundaries must NOT be weakened.

[11] CLAIM: No quota is specified for `the external party-warm` on a volume that also holds Proxmox backups, so
POINTER: synology-exports.txt:2
OPENED TEXT:
Export list for <lan-address>:
/volume1/proxmoxbackup/<lan-address>/24(rw,async,no_wdelay,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/proxmox<lan-address>(rw,async,no_wdelay,crossmnt,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/1-RPX-Workspace-Data                                                                                              <lan-address>
/volume1/NeoMoat-Hub                                                                                                       <lan-address>

[12] CLAIM: No quota is specified for `the external party-warm` on a volume that also holds Proxmox backups, so
POINTER: unified-nfs-design.md:40-47
OPENED TEXT:
## NFS specifics (proposed)
- New export `the external party-warm`, **root_squash + all_squash to a dedicated `the external party` uid/gid**,
  limited to the three host IPs (node1 .10, node2 .11, gb10 .225) — NOT the whole subnet,
  NOT no_root_squash (unlike the existing proxmoxbackup exports).
- Mount opts mirror the working GB10 precedent: `vers=3, soft, timeo=150, retrans=3,
  _netdev, nofail, nolock`; models bind RO into sandboxes; workspace/queue RW.
- uid alignment: Synology maps anon to uid 1025; the sandbox `the external party` users are 1000/1001.
  The export will all_squash to one uid and the sandboxes' bind mounts run as that uid.

## Open questions for the panel
- Is one shared NFS FS as workspace+models+spool the right shape, or does co-mingling

[13] CLAIM: A5 is satisfied — an NFS death still fails toward prod-UP — but only because
POINTER: CONTRACT.md:22-30
OPENED TEXT:
### Management host (arbiter state, single-writer)
`~/.the external party-queue/<host>/`  (host = rtx|gb10)
- `arbiter.lock`   — flock; only one arbiter per host runs.
- `state.json`     — `{budget:{date, used_seconds}}`, updated atomically (tmp+rename).

### GPU host (readable by the LOCAL dead-man, written by the arbiter over ssh)
- node2: `/var/lib/the external party-queue/`   GB10: `/home/<login-user>/the external party-queue/`
- `window.flag` — JSON `{open:bool, opened_at, expires_at, opened_by}`. Presence+expiry.
- `heartbeat`   — epoch seconds; arbiter rewrites every 15s while alive.

### Spool (in the external party's sandbox, host-visible on both GPUs)
- GB10: host `/home/<login-user>/the external party-bench/scratch/queue` = container `/scratch/queue`.
- RTX:  host `/var/lib/the external party-queue/spool` bind-mounted to CT 205 `/scratch/queue`.

[14] CLAIM: A5 is satisfied — an NFS death still fails toward prod-UP — but only because
POINTER: unified-nfs-design.md:30-31
OPENED TEXT:
- **gb10** mounts `the external party-warm` → bind into the GB10 job/bench containers (same paths).
- **node1** (arbiter host) mounts `the external party-warm` → arbiter reads/writes the spool LOCALLY
  (replaces today's over-ssh spool reads).

**Access:** retire the two per-GPU code-server endpoints; the bastion fronts ONE new
ingress `the bench hostname` → CT 206 code-server, one CF Access app (OTP), policy allows

[15] CLAIM: A5 is satisfied — an NFS death still fails toward prod-UP — but only because
POINTER: CONTRACT.md:14-16
OPENED TEXT:
window. Reuses the health-gated `window-*.sh` (with fail-to-prod-UP rollback).
- **Dead-man** — runs ON each GPU host, INDEPENDENT of the arbiter and the management
  host. Restores prod if a window overruns its deadline or the arbiter's heartbeat
  goes stale. This is the gate-1 safety net: prod-UP survives arbiter/host death.
- **the external party** — writes job specs into his sandbox spool via `the external party-submit`; never controls
  windows or preemption.

[16] CLAIM: With `nofail`, a container started before the mount bind-mounts an EMPTY host directory
POINTER: unified-nfs-design.md:44
OPENED TEXT:
NOT no_root_squash (unlike the existing proxmoxbackup exports).
- Mount opts mirror the working GB10 precedent: `vers=3, soft, timeo=150, retrans=3,
  _netdev, nofail, nolock`; models bind RO into sandboxes; workspace/queue RW.
- uid alignment: Synology maps anon to uid 1025; the sandbox `the external party` users are 1000/1001.
  The export will all_squash to one uid and the sandboxes' bind mounts run as that uid.

[17] CLAIM: With `nofail`, a container started before the mount bind-mounts an EMPTY host directory
POINTER: RUNBOOK.md:71
OPENED TEXT:
Health-gated open/close (confirm VRAM actually freed on open; confirm prod is
HEALTHY not just active on close — the estate's "active ≠ healthy" lesson):

- **RTX:** `./window-rtx.sh open | close | status`. Stops/starts BOTH prod GPU
  consumers on the card — `vllm.service` (gpt-oss-120b, :8000) AND

[18] CLAIM: With `nofail`, a container started before the mount bind-mounts an EMPTY host directory
POINTER: CONTRACT.md:60-61
OPENED TEXT:
leaves the window.flag CLOSED so the dead-man/next-open re-asserts prod-UP, and alerts.
6. **Readiness probe** — before each job the arbiter verifies the sandbox exec responds
   and (via window open) the GPU is actually free.

## Booked-block model
- `queuectl --host rtx open 120` → checks daily budget, runs `window-rtx.sh open`

[19] CLAIM: `showmount` output is not an adequate oracle for A3 — the supplied list prints option
POINTER: synology-exports.txt:2-3
OPENED TEXT:
Export list for <lan-address>:
/volume1/proxmoxbackup/<lan-address>/24(rw,async,no_wdelay,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/proxmox<lan-address>(rw,async,no_wdelay,crossmnt,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/1-RPX-Workspace-Data                                                                                              <lan-address>
/volume1/NeoMoat-Hub                                                                                                       <lan-address>
/volume1/proxmoxbackup                                                                                                     <lan-address>

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }
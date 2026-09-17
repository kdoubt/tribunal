You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: ** Redesign the storage boundary: retain shared NFS workspace/models, but keep authoritative queue state on trusted local storage behind validated submission, expose results read-only, and publish immutable model versions because a job’s RO bind does not prevent the editor changing its backing files.
POINTER: unified-nfs-design.md:19-22
OPENED TEXT:
/volume1/the external party-warm/
  workspace/            # the external party's scripts/configs (RW to the external party)
  models/               # staged model files (RW to the external party, RO to jobs)
  queue/rtx/{incoming,running,done}      # per-GPU spool
  queue/gb10/{incoming,running,done}
```
Mounted via the HOST, then bind-mounted into each sandbox — so the island/internal
sandboxes get the FILES without a network ROUTE to the NAS (isolation preserved):

[2] CLAIM: ** Redesign the storage boundary: retain shared NFS workspace/models, but keep authoritative queue state on trusted local storage behind validated submission, expose results read-only, and publish immutable model versions because a job’s RO bind does not prevent the editor changing its backing files.
POINTER: CONTRACT.md:12
OPENED TEXT:
- **Arbiter** — runs on the management host, one instance per GPU (`--host rtx|gb10`).
  Single-writer of all queue/window/budget state. Drains the spool during an open
  window. Reuses the health-gated `window-*.sh` (with fail-to-prod-UP rollback).
- **Dead-man** — runs ON each GPU host, INDEPENDENT of the arbiter and the management
  host. Restores prod if a window overruns its deadline or the arbiter's heartbeat

[3] CLAIM: ** Host mounting does not itself create a sandbox network route or make symlinks resolve against the NAS root, but retaining CT 206 on node2 requires explicit unprivileged isolation, resource limits, narrow private bind mounts and no host-management interfaces; the documented GB10 helper-compromise path remains a separate containment weakness.
POINTER: unified-nfs-design.md:24-29
OPENED TEXT:
```
Mounted via the HOST, then bind-mounted into each sandbox — so the island/internal
sandboxes get the FILES without a network ROUTE to the NAS (isolation preserved):
- **node2** mounts `the external party-warm` → bind into **CT 205** (RTX sandbox) at /workspace, /models(ro),
  /queue; and into a NEW **CT 206 `the external party-code`** (unified code-server, node2, isolated,
  bastion-reachable) RW.
- **gb10** mounts `the external party-warm` → bind into the GB10 job/bench containers (same paths).
- **node1** (arbiter host) mounts `the external party-warm` → arbiter reads/writes the spool LOCALLY
  (replaces today's over-ssh spool reads).

[4] CLAIM: ** Host mounting does not itself create a sandbox network route or make symlinks resolve against the NAS root, but retaining CT 206 on node2 requires explicit unprivileged isolation, resource limits, narrow private bind mounts and no host-management interfaces; the documented GB10 helper-compromise path remains a separate containment weakness.
POINTER: RUNBOOK.md:23-30
OPENED TEXT:
:8888 + dnsmasq :53 serve the island (only <island-address> allowed).
- **CT 205 the external party-bench-rtx** (12c/32G/120G local-lvm, onboot=0, unprivileged):
  on SDN vnet `cirub0` (zone `cirub`, node2-only, NO uplink — physical L2 island,
  no NAT anywhere). GPU via /dev/nvidia* bind mounts + cgroup allows c 195/510
  (same pattern as CT 201). NVIDIA user-space 610.43.02 via .run --no-kernel-modules
  (matches node2 host driver EXACTLY — if node2 driver is upgraded, rerun the .run
  in 205 with the new version). User `the external party` HAS passwordless sudo (his sandbox).
  Internet only via bastion tinyproxy (http/https, set in /etc/profile.d/proxy.sh,
  apt.conf.d/95proxy). Verified: HF reachable, Infisical/PBS NOT reachable.
  Excluded from the hourly PBS job (exclude=114,205) — scratch box, 120G churn.
- **GB10**: NOT built (permission classifier blocked remote execution on gb10; the owner
  runs it). Staged script: `<path> external party-bench/gb10_setup.sh` — runs as `<login-user>`

[5] CLAIM: ** A shared NAS is conditionally defensible using a dedicated filesystem export with exactly the three verified client addresses and `rw,sync,root_squash,all_squash,anonuid=<dedicated>,anongid=<dedicated>,secure,nocrossmnt,hide`, no exported ancestor or nested tenant mounts, and hard capacity limits; export options alone cannot compensate for a compromised trusted client.
POINTER: unified-nfs-design.md:41-47
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

[6] CLAIM: ** A shared NAS is conditionally defensible using a dedicated filesystem export with exactly the three verified client addresses and `rw,sync,root_squash,all_squash,anonuid=<dedicated>,anongid=<dedicated>,secure,nocrossmnt,hide`, no exported ancestor or nested tenant mounts, and hard capacity limits; export options alone cannot compensate for a compromised trusted client.
POINTER: synology-exports.txt:2-3
OPENED TEXT:
Export list for <lan-address>:
/volume1/proxmoxbackup/<lan-address>/24(rw,async,no_wdelay,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/proxmox<lan-address>(rw,async,no_wdelay,crossmnt,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/1-RPX-Workspace-Data                                                                                              <lan-address>
/volume1/NeoMoat-Hub                                                                                                       <lan-address>
/volume1/proxmoxbackup                                                                                                     <lan-address>

[7] CLAIM: ** A3 requires corroborated server configuration and end-to-end negative tests, because the supplied export listing cannot establish effective isolation or even resolve the proposal’s abbreviated client addresses.
POINTER: unified-nfs-design.md:42
OPENED TEXT:
- New export `the external party-warm`, **root_squash + all_squash to a dedicated `the external party` uid/gid**,
  limited to the three host IPs (node1 .10, node2 .11, gb10 .225) — NOT the whole subnet,
  NOT no_root_squash (unlike the existing proxmoxbackup exports).
- Mount opts mirror the working GB10 precedent: `vers=3, soft, timeo=150, retrans=3,
  _netdev, nofail, nolock`; models bind RO into sandboxes; workspace/queue RW.

[8] CLAIM: ** A3 requires corroborated server configuration and end-to-end negative tests, because the supplied export listing cannot establish effective isolation or even resolve the proposal’s abbreviated client addresses.
POINTER: synology-exports.txt:2-6
OPENED TEXT:
Export list for <lan-address>:
/volume1/proxmoxbackup/<lan-address>/24(rw,async,no_wdelay,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/proxmox<lan-address>(rw,async,no_wdelay,crossmnt,no_root_squash,insecure_locks,sec=sys,anonuid=1025,anongid=100) *
/volume1/1-RPX-Workspace-Data                                                                                              <lan-address>
/volume1/NeoMoat-Hub                                                                                                       <lan-address>
/volume1/proxmoxbackup                                                                                                     <lan-address>

[9] CLAIM: ** A1 cannot be solved by mount flags alone: use `hard,vers=3,proto=tcp,_netdev,nofail` for mutable NFS data, treat `timeo=150`/`retrans=3` as retry tuning rather than a deadline, isolate potentially blocked I/O from control services, and gate windows/jobs on verified mount identity plus a fresh write–fsync–rename–cross-client-read probe whose completion is required within a controller deadline.
POINTER: unified-nfs-design.md:44-45
OPENED TEXT:
NOT no_root_squash (unlike the existing proxmoxbackup exports).
- Mount opts mirror the working GB10 precedent: `vers=3, soft, timeo=150, retrans=3,
  _netdev, nofail, nolock`; models bind RO into sandboxes; workspace/queue RW.
- uid alignment: Synology maps anon to uid 1025; the sandbox `the external party` users are 1000/1001.
  The export will all_squash to one uid and the sandboxes' bind mounts run as that uid.

[10] CLAIM: ** A1 cannot be solved by mount flags alone: use `hard,vers=3,proto=tcp,_netdev,nofail` for mutable NFS data, treat `timeo=150`/`retrans=3` as retry tuning rather than a deadline, isolate potentially blocked I/O from control services, and gate windows/jobs on verified mount identity plus a fresh write–fsync–rename–cross-client-read probe whose completion is required within a controller deadline.
POINTER: gb10-nfs-precedent.txt:1
OPENED TEXT:
<lan-address>:/volume1/1-RPX-Workspace-Data  /mnt/nas-workspace  nfs  ro,vers=3,nolock,rsize=1048576,wsize=1048576,soft,timeo=150,retrans=3,_netdev,nofail  0  0
---mount---

[11] CLAIM: ** A1 cannot be solved by mount flags alone: use `hard,vers=3,proto=tcp,_netdev,nofail` for mutable NFS data, treat `timeo=150`/`retrans=3` as retry tuning rather than a deadline, isolate potentially blocked I/O from control services, and gate windows/jobs on verified mount identity plus a fresh write–fsync–rename–cross-client-read probe whose completion is required within a controller deadline.
POINTER: CONTRACT.md:60-61
OPENED TEXT:
leaves the window.flag CLOSED so the dead-man/next-open re-asserts prod-UP, and alerts.
6. **Readiness probe** — before each job the arbiter verifies the sandbox exec responds
   and (via window open) the GPU is actually free.

## Booked-block model
- `queuectl --host rtx open 120` → checks daily budget, runs `window-rtx.sh open`

[12] CLAIM: ** A2’s same-filesystem rename remains atomic across NFSv3 clients, but at-most-once execution needs trusted durable claim records, one lifecycle writer, checked publication errors and conservative recovery of ambiguous operations; CTO caching is not instantaneous visibility, `nolock` supplies no cross-client exclusion, and stale handles or rename errors must never trigger blind replay.
POINTER: CONTRACT.md:23-24
OPENED TEXT:
`~/.the external party-queue/<host>/`  (host = rtx|gb10)
- `arbiter.lock`   — flock; only one arbiter per host runs.
- `state.json`     — `{budget:{date, used_seconds}}`, updated atomically (tmp+rename).

### GPU host (readable by the LOCAL dead-man, written by the arbiter over ssh)
- node2: `/var/lib/the external party-queue/`   GB10: `/home/<login-user>/the external party-queue/`

[13] CLAIM: ** A2’s same-filesystem rename remains atomic across NFSv3 clients, but at-most-once execution needs trusted durable claim records, one lifecycle writer, checked publication errors and conservative recovery of ambiguous operations; CTO caching is not instantaneous visibility, `nolock` supplies no cross-client exclusion, and stale handles or rename errors must never trigger blind replay.
POINTER: unified-nfs-design.md:38
OPENED TEXT:
GPU sandbox against the SAME NFS files the external party edited; results land in `queue/<host>/done`,
visible instantly in his one workspace.

## NFS specifics (proposed)
- New export `the external party-warm`, **root_squash + all_squash to a dedicated `the external party` uid/gid**,

[14] CLAIM: ** A4 needs a coordinated migration covering NAS rules/ownership/quota, all three hosts’ persistent mounts, CT 205/206 binds and UID maps, GB10 editor/bench/ephemeral-job binds, submitter/arbiter/result paths, and ingress/provisioning/teardown definitions; otherwise expected failures include denied writes, empty local fallback directories, split queues and stale access endpoints.
POINTER: unified-nfs-design.md:26-37
OPENED TEXT:
sandboxes get the FILES without a network ROUTE to the NAS (isolation preserved):
- **node2** mounts `the external party-warm` → bind into **CT 205** (RTX sandbox) at /workspace, /models(ro),
  /queue; and into a NEW **CT 206 `the external party-code`** (unified code-server, node2, isolated,
  bastion-reachable) RW.
- **gb10** mounts `the external party-warm` → bind into the GB10 job/bench containers (same paths).
- **node1** (arbiter host) mounts `the external party-warm` → arbiter reads/writes the spool LOCALLY
  (replaces today's over-ssh spool reads).

**Access:** retire the two per-GPU code-server endpoints; the bastion fronts ONE new
ingress `the bench hostname` → CT 206 code-server, one CF Access app (OTP), policy allows
jc@the external party.ai + korey@. `the external party-submit --host rtx|gb10` writes a spec into the matching
`queue/<host>/incoming` on the share; the arbiter for that host drains it; jobs run in the
GPU sandbox against the SAME NFS files the external party edited; results land in `queue/<host>/done`,
visible instantly in his one workspace.

## NFS specifics (proposed)

[15] CLAIM: ** A4 needs a coordinated migration covering NAS rules/ownership/quota, all three hosts’ persistent mounts, CT 205/206 binds and UID maps, GB10 editor/bench/ephemeral-job binds, submitter/arbiter/result paths, and ingress/provisioning/teardown definitions; otherwise expected failures include denied writes, empty local fallback directories, split queues and stale access endpoints.
POINTER: CONTRACT.md:21-36
OPENED TEXT:
## On-disk layout
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
- `incoming/<id>.json`  — the external party's specs (he writes `*.json.tmp` then renames).
- `running/<id>.json`   — arbiter claimed it (atomic rename from incoming).
- `done/<id>/`          — `result.json` (status, exit, started/ended), `stdout`, `stderr`.

## Job spec (`incoming/<id>.json`)
```json

[16] CLAIM: ** A4 needs a coordinated migration covering NAS rules/ownership/quota, all three hosts’ persistent mounts, CT 205/206 binds and UID maps, GB10 editor/bench/ephemeral-job binds, submitter/arbiter/result paths, and ingress/provisioning/teardown definitions; otherwise expected failures include denied writes, empty local fallback directories, split queues and stale access endpoints.
POINTER: RUNBOOK.md:33-38
OPENED TEXT:
- **GB10**: NOT built (permission classifier blocked remote execution on gb10; the owner
  runs it). Staged script: `<path> external party-bench/gb10_setup.sh` — runs as `<login-user>`
  (docker group, no sudo needed): `scp <path> external party-bench/gb10_setup.sh gb10:~ && ssh gb10 bash gb10_setup.sh`.
  Design: docker-native only (no host iptables/systemd): bench container on an
  `--internal` network (no outbound), socat relay container publishes .225:2222→bench:22,
  squid proxy container (172.30.99.3:3128) allows internet but denies RFC1918.
  All state under `/home/<login-user>/the external party-bench/` on gb10.

## Access model — code-server behind CF Access (OTP), NO SSH key

[17] CLAIM: ** A4 needs a coordinated migration covering NAS rules/ownership/quota, all three hosts’ persistent mounts, CT 205/206 binds and UID maps, GB10 editor/bench/ephemeral-job binds, submitter/arbiter/result paths, and ingress/provisioning/teardown definitions; otherwise expected failures include denied writes, empty local fallback directories, split queues and stale access endpoints.
POINTER: RUNBOOK.md:23
OPENED TEXT:
:8888 + dnsmasq :53 serve the island (only <island-address> allowed).
- **CT 205 the external party-bench-rtx** (12c/32G/120G local-lvm, onboot=0, unprivileged):
  on SDN vnet `cirub0` (zone `cirub`, node2-only, NO uplink — physical L2 island,
  no NAT anywhere). GPU via /dev/nvidia* bind mounts + cgroup allows c 195/510
  (same pattern as CT 201). NVIDIA user-space 610.43.02 via .run --no-kernel-modules

[18] CLAIM: ** A5 remains unproven until NAS-loss tests show that host-local watchdogs, deadlines, heartbeat handling, job termination and production restart remain NFS-independent, boot restores production before optional storage, and teardown stops writers, removes binds/mount definitions, unmounts clients and removes only the new export.
POINTER: CONTRACT.md:14-16
OPENED TEXT:
window. Reuses the health-gated `window-*.sh` (with fail-to-prod-UP rollback).
- **Dead-man** — runs ON each GPU host, INDEPENDENT of the arbiter and the management
  host. Restores prod if a window overruns its deadline or the arbiter's heartbeat
  goes stale. This is the gate-1 safety net: prod-UP survives arbiter/host death.
- **the external party** — writes job specs into his sandbox spool via `the external party-submit`; never controls
  windows or preemption.

[19] CLAIM: ** A5 remains unproven until NAS-loss tests show that host-local watchdogs, deadlines, heartbeat handling, job termination and production restart remain NFS-independent, boot restores production before optional storage, and teardown stops writers, removes binds/mount definitions, unmounts clients and removes only the new export.
POINTER: unified-nfs-design.md:55-56
OPENED TEXT:
other Synology data or another host?
- NFS as a hard dependency: what happens to a running window / the arbiter / code-server
  if the NAS or mount drops mid-benchmark? Boot ordering (mount before containers)?
- Spool correctness on NFS: is the atomic-rename claim + at-most-once still safe over NFSv3
  across node1 (arbiter) + the sandbox writers? Lock/stale-handle risks?
- Is node2 the right home for the unified code-server, or does hosting the external party's editor next

[20] CLAIM: ** A5 remains unproven until NAS-loss tests show that host-local watchdogs, deadlines, heartbeat handling, job termination and production restart remain NFS-independent, boot restores production before optional storage, and teardown stops writers, removes binds/mount definitions, unmounts clients and removes only the new export.
POINTER: RUNBOOK.md:96-101
OPENED TEXT:
`./teardown.sh` (dry-run) → `./teardown.sh --commit`. Idempotent; removes CF
Access app + policy + DNS + tunnel, both node2 CTs, the SDN vnet+zone, all GB10
docker objects + state dir, and **removes only 205 from the backup exclude via
compare-and-swap** (NOT a hardcoded `114` — that would clobber any co-tenant
excluded mid-engagement). Then rotate anything of ours ever typed into the external party's
env (should be nothing) and drop the memory note.

## Security posture (updated 2026-08-25 after tribunal review)

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }
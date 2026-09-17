I'll read the proposal and supporting artifacts first, then take an independent position grounded in those files.**Framings considered:** (1) security-boundary — does host-mount+bind-mount leak other NAS data; (2) queue-correctness — do CONTRACT gates survive NFSv3 and co-mingling; (3) owner-criteria 1–4 with the stated tie-break; (4) minimal evolution of the GB10 RO mount. **Picked (3):** the brief already ranks data-exposure and prod-down/double-run over simplicity.

The proposal is not built (`unified-nfs-design.md:1`). I did not probe the live NAS or hosts; probes below are oracles, not observations.

---

CLAIM: Co-mingling the spool on the same `all_squash` export that CT 206 (and CT 205 `/queue`) get RW makes the untrusted party a same-credential writer of `incoming/`, `running/`, and `done/`, which breaks CONTRACT single-writer and at-most-once.
EVIDENCE: `unified-nfs-design.md:16-28` “One Synology NFS share is the shared substrate (workspace + models + queue spool)” and CT 206 bind “RW”; `:46-47` “all_squash to one uid and the sandboxes' bind mounts run as that uid”; `CONTRACT.md:50-51` “one arbiter… queue claim is an atomic rename incoming→running”; `:72-75` leftover `running/` is “not re-run”. POSIX mode bits cannot separate the external party from the arbiter under one squashed uid.
CONFIDENCE: 0.88
FALSIFIER: CT 206/205 mounts omit `queue/running` and `queue/incoming` (or those dirs are a second export mounted only on node1), and a `the external party`-uid `rename` of `running/<id>.json` → `incoming/` returns EACCES.

CLAIM: Host-mount then bind-mount does not give CT 205 or the GB10 `--internal` container a new L3 route to the NAS; the unspecified CT 206 network is the residual path to the live `*` / `no_root_squash` backup exports.
EVIDENCE: `unified-nfs-design.md:24-25` “FILES without a network ROUTE”; `RUNBOOK.md:23-25` CT 205 “NO uplink”; `:113-114` “LAN unreachable by construction”; `synology-exports.txt:2` `/volume1/proxmoxbackup/` … `no_root_squash` … `*`. CT 206 is only “isolated, bastion-reachable” (`unified-nfs-design.md:27-28`) — no vnet, no “no route to <lan-address>”.
CONFIDENCE: 0.82
FALSIFIER: `ip route get <lan-address>` inside CT 205, the GB10 bench container, *and* CT 206 all fail, and `timeout 3 bash -c 'echo >/dev/tcp/<lan-address>/2049'` fails in all three.

CLAIM: Highest-severity hole as written is spool mutation (copy `running/<id>.json` back to `incoming/` or a new id with the same `cmd`) so the arbiter drains and runs the job twice — a tie-break failure; a close second is unconstrained RW on `/volume1` with no quota next to `proxmoxbackup`.
EVIDENCE: Same as claim 1 plus `CONTRACT.md:66-67` arbiter “drains the queue”; `synology-exports.txt:2-3` other tenants on `/volume1/…`; proposal has no quota/`quota`/`MAX_*` on `workspace/` or `models/` (`unified-nfs-design.md:17-23`, `CONTRACT.md:55-57` caps apply only at claim time to the spool).
CONFIDENCE: 0.80
FALSIFIER: After a claimed `running/<id>.json` is copied to `incoming/<id>.json` (or `<id2>.json` with the same cmd) during an open window, the arbiter log shows no second start and `done/` has a single execution; `df` on `/volume1` is unchanged after writing >free-bytes in `workspace/`.

CLAIM: A1 — copy the GB10 client opts `vers=3,soft,timeo=150,retrans=3,_netdev,nofail,nolock` for RW, not `hard`; `nofail` without a contents-level readiness probe lets a failed mount leave a local empty directory that the arbiter will treat as the spool.
EVIDENCE: `gb10-nfs-precedent.txt:1` those opts (plus `ro`); `unified-nfs-design.md:44-45` copies them minus `ro`; no readiness text in the proposal; `CONTRACT.md:60-61` probe is sandbox exec + GPU free, not NFS; contested note “Active is not healthy”. Empty-mountpoint-on-`nofail` is EXTERNAL (nfs-utils: failed `nofail` mount leaves the mountpoint as a local dir).
CONFIDENCE: 0.84
FALSIFIER: With NAS down, `stat -f -c %T /mnt/the external party-warm` is still `nfs` (not `ext*`/`xfs`/`zfs`), `queuectl open` refuses, and no arbiter write appears under the local mountpoint.

CLAIM: A2 — NFSv3 `rename` on one export is still a single-RPC atomic claim *if* only the arbiter can write `running/` and the arbiter mount uses `actimeo=0` (or `noac`); as proposed, `nolock` is fine (flock is local) but default attribute cache plus the external party RW on `queue/` is not.
EVIDENCE: `CONTRACT.md:21-24` `arbiter.lock` / `state.json` live under `~/.the external party-queue/` on the management host (not NFS); `:34-35` claim is rename; `unified-nfs-design.md:44-45` `nolock`, no `actimeo`/`noac`; EXTERNAL RFC 1813 rename is atomic at the server; NFSv3 close-to-open does not disable directory attr cache.
CONFIDENCE: 0.73
FALSIFIER: Two clients, `actimeo` default: arbiter boot with a `running/<id>.json` that another client just created still always moves it to `done/... status:interrupted` and never starts `cmd` a second time, even when the external party can write `queue/`.

CLAIM: A5 — dead-man still fail-to-prod-UP on NAS death because `window.flag` and `heartbeat` stay on GPU-host local disk; an arbiter that keeps rewriting heartbeat while spool I/O returns EIO leaves prod down until `expires_at`.
EVIDENCE: `CONTRACT.md:14-16,26-29,48-49` dead-man “INDEPENDENT”, paths `/var/lib/the external party-queue/` and `/home/<login-user>/the external party-queue/` (`window.flag`, `heartbeat` every 15s); `unified-nfs-design.md:30-31` NFS replaces “over-ssh spool reads” only; `:55-56` open question, no hang/EIO policy. Proposal does not move the flag onto NFS.
CONFIDENCE: 0.86
FALSIFIER: With the NAS unplugged mid-window, `window.flag` is unreadable on the GPU host, or prod vLLM stays down after both heartbeat-stale and `expires_at` have passed.

CLAIM: A dedicated `root_squash+all_squash`, 3-IP, no-`crossmnt` export is defensible; A3 is not `showmount` — prove a fourth IP cannot `mount` `the external party-warm` *or* `proxmoxbackup`, and that each sandbox has no TCP/2049 to `<lan-address>`.
EVIDENCE: `unified-nfs-design.md:41-43` proposed options vs `synology-exports.txt:2-3` anti-pattern (`*`, `no_root_squash`, `crossmnt`); `gb10-nfs-precedent.txt:1` already IP-locks `.225` on a different share (`synology-exports.txt:4`). Oracle: from a fourth LAN IP, `mount -t nfs -o vers=3 <lan-address>:/volume1/the external party-warm /mnt` fails; same for `/volume1/proxmoxbackup`; from CT 205/206 and GB10 bench, connect to `:2049` fails. `showmount -e` alone is insufficient (mountd list ≠ enforced export).
CONFIDENCE: 0.78
FALSIFIER: Fourth-IP `mount` of `the external party-warm` succeeds, or sandbox `:2049` connect succeeds, while `showmount` still lists only the three hosts.

CLAIM: A4 skip-map (who else writes / symptom): Synology new share (other DSM admins / wrong clone of `no_root_squash,*` → backup exposure); fstab on node1/node2/gb10 (empty local dir, split-brain spool); pct bind of CT 205 and new CT 206 (jobs/editor miss files); GB10 `docker -v` (jobs miss models); arbiter + `the external party-submit` spool path move (`CONTRACT.md:31-33` old paths still used → two queues); uid 1025 vs 1000/1001 (`unified-nfs-design.md:46-47` EACCES or mixed owners); CF app `the bench hostname` (`unified-nfs-design.md:33-35`); `teardown.sh` (`RUNBOOK.md:94-101`) does not remove a NAS export — skip leaves a live RW export after engagement.
EVIDENCE: paths cited; teardown list has no Synology step.
CONFIDENCE: 0.83
FALSIFIER: After a full apply, `findmnt` on all three hosts shows nfs `the external party-warm`, arbiter and `the external party-submit` share one `queue/<host>/incoming`, and `teardown.sh --commit` deletes the Synology export (verified by fourth-IP mount fail).

---

**VERDICT INPUT**
- Q1 Architecture: **build-with-changes** (one export for models+workspace is the right warm tier; do not bind `queue/running` or the whole tree RW into CT 206/205; node2/CT 206 is acceptable only as unprivileged, no-GPU, `cirub0`, no NAS route — same blast radius as today’s CT 205 editor, not a new one).
- Q2 Security: **build-with-changes** (host-mount+bind-mount is the right isolation shape; require the A3 fourth-IP + sandbox `:2049` probes, `all_squash`+3-IP+no `crossmnt`, volume quota; do not copy `synology-exports.txt:2-3`).
- Q3 Systems: **build-with-changes** (soft/`nofail`/`_netdev` + NFS contents readiness before window/job; arbiter spool `actimeo=0`; keep flag/heartbeat local; teardown unmounts and drops the export).
- Highest-severity issue: **same-uid RW spool in the untrusted editor/sandbox mounts, enabling a second drain of the same job (tie-break: double-run).**

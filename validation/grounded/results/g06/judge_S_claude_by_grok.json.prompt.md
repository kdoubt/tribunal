You are scoring one review memo against a sealed rubric. You are not a participant; do not add your own review.
Score ONLY from the memo text. Quote the memo span that earns each point; if no span supports a point, the score is 0.

=== SEALED RUBRIC ===
# G06 sealed rubric - SEALED (arms never see this)

- **difficulty:** subtle (the network path is a red herring; the real hole is filesystem identity)
- **correct_call:** **targeted redesign / build-with-changes** (the two
  original seats differed only in label): the shape (NFS warm tier for
  models, host-mount + bind, one session) survives, but the proposal's
  single share squashed to ONE uid gives the external party the same filesystem authority
  as the arbiter over the queue's control state (`running/`, `done/`), which
  breaks at-most-once. Separate the trust domains: the external party gets a writable
  `incoming/` drop-box and read-only results; the arbiter-owned spool state
  stays off the external party's writable identity (simplest: keep it where it is today,
  not on the share). Writable paths get a `hard` mount with a live
  write-fsync-rename-read-unlink probe; models stay `ro,soft`. New distinct
  export, `no_crossmnt`, clients limited to three `/32`s, `root_squash`.
- **oracle:** orchestrator probes at panel time: from CT 205 (island) there is
  no route to the NAS and :2049 is unreachable; from a GB10 internal job
  container :2049 is unreachable; the broad `no_root_squash`/`crossmnt`
  exports exist (`synology-exports.txt:2-3`) but are not reachable from the
  sandboxes. So isolation holds and the residual exposure is identity, not
  network.
- **correct fix:** as in correct_call; plus keep dead-man flags/heartbeats
  local (CONTRACT.md:14-15, 26-29) so NAS loss stops job claims and the
  editor but the watchdog still restores prod; mount-before-container boot
  ordering; teardown unmounts and removes the export.
- **must_catch:**
  1. Identity collision: `unified-nfs-design.md:41-47` exports with
     `root_squash + all_squash to a dedicated the external party uid/gid` and puts
     `queue/<gpu>/{incoming,running,done}` on the same share (`:16-22`,
     `:30-31` the arbiter reads/writes the spool on it); the queue's
     at-most-once relies on `running/<id>.json` claimed by atomic rename and
     reconciled on restart (`CONTRACT.md:34-35,50-51`), so an external-party-controlled
     process can delete or forge `running/` and `done/` entries.
  2. The mount precedent is read-only and must not be copied for writable
     paths: `gb10-nfs-precedent.txt:1` is `ro,...,soft,...,nolock`;
     `unified-nfs-design.md:44-45` copies `soft ... nolock` for the RW share.
  3. Network isolation is preserved by host-mount + bind (the falsifier both
     seats named - sandbox reaching :2049 - fails), so the fix is
     ownership/ACLs, not more network controls.
  4. Export hardening: a new distinct shared folder; `no_crossmnt`; clients as
     three `/32`s not a subnet (`unified-nfs-design.md:42` says three host IPs
     - good - but the live exports show the subnet/`*` anti-pattern at
     `synology-exports.txt:2-3`); never touch existing exports.
  5. Verification battery, not `showmount` alone: NAS config/API inspect,
     positive mounts from the three hosts, a negative mount from an
     unauthorized host, sandbox -> :2049 probe, bind-root symlink traversal
     test, post-mount identity test.
  6. NAS loss must fail toward prod-UP: dead-man state is local
     (`CONTRACT.md:14-15,26-29`), so the design must not move heartbeats or
     window flags onto NFS.
- **landmine (confident wrong answers):** "the NFS mount gives the island a
  route to the LAN/NAS" (host-mount + bind does not; oracle-refuted); "copy
  the proven GB10 mount options" (they are read-only + soft + nolock);
  "all_squash makes it safe" (it makes the external party and the arbiter the same
  principal).
- **unverifiable:** NFSv3 rename atomicity across clients under attribute
  caching is documented behaviour, not testable from the artifact; a seat may
  label it EXTERNAL.
- **outcome_source:** `2026-08-25-external-party-nfs-review/verdict.md` (sections 1-2
  oracle record, recommendation 1-4).


=== MEMO (author unknown) ===
## Framing

Plausible framings: (a) an NFS export-hardening review; (b) a distributed-correctness review of a queue moved onto NFSv3; (c) a multi-tenancy/blast-radius review of introducing a shared prod appliance into an untrusted party's environment; (d) an availability review (NFS as a new hard dependency). I take **(c) with (b) folded in**, because the owner's tie-break explicitly ranks "any cross-tenant/data-exposure path, or any NAS failure that leaves prod down or double-runs a job" above everything else — so the load-bearing question is what new coupling the shared substrate creates, not whether the mount options are tidy.

---

```
CLAIM: The design's safety rests entirely on the untrusted party having no IP route to
<lan-address>, but it never specifies CT 206's network, and the NAS already exports
/volume1/proxmox and /volume1/proxmoxbackup to `*` with no_root_squash (and crossmnt) —
so a routable CT 206 with the party's root is full read/write of all Proxmox backups.
EVIDENCE: synology-exports.txt:2-3 `/volume1/proxmoxbackup/<lan-address>/24(rw,...,no_root_squash,...) *`
and `/volume1/proxmox<lan-address>(rw,...,crossmnt,no_root_squash,...) *`; design says only
"a NEW CT 206 `the external party-code` (unified code-server, node2, isolated, bastion-reachable)"
(unified-nfs-design.md:27-28) — no vnet named, unlike CT 205's `cirub0 ... NO uplink`
(RUNBOOK.md:24). Settling probe: `pct config 206 | grep ^net` — any `bridge=vmbr0` confirms.
CONFIDENCE: 0.75  (that this is the top hole; 0.5 that the builder would actually put 206 on vmbr0)
FALSIFIER: CT 206 is built with a single NIC on `cirub0`/10.99.0.x, no vmbr0, no NAT, and a
mount attempt to <lan-address> from inside 206 gets no route.
```

```
CLAIM: all_squash→anonuid 1025 plus unprivileged-LXC uid shifting means the party's
processes hit the share as a host uid (≈101000) that owns nothing on it, so writes fail with
EPERM unless the share is mode 0777 — the design asserts a fix it does not have.
EVIDENCE: unified-nfs-design.md:46-47 "Synology maps anon to uid 1025; the sandbox `the external party`
users are 1000/1001 ... the sandboxes' bind mounts run as that uid" — a bind mount does not
remap uids; RUNBOOK.md:23 "CT 205 ... unprivileged". Probe: `pct exec 205 -- sudo -u the external party touch
/workspace/p && ls -n /mnt/the external party-warm/workspace/p` on node2.
CONFIDENCE: 0.8
FALSIFIER: the touch succeeds with the share not world-writable (e.g. via an idmapped mount or
`lxc.idmap` aligning to 1025).
```

```
CLAIM: Copying `soft` from the precedent onto a read-write mount is a category error: the
precedent is `ro`, and `soft` on RW returns EIO mid-write after timeo*retrans, which is exactly
how a partially written result.json / spec lands on the spool.
EVIDENCE: gb10-nfs-precedent.txt:1 `ro,vers=3,nolock,...,soft,timeo=150,retrans=3,_netdev,nofail`
vs unified-nfs-design.md:44-45 "mirror the working GB10 precedent: `vers=3, soft, timeo=150,
retrans=3, _netdev, nofail, nolock`; models bind RO ... workspace/queue RW".
CONFIDENCE: 0.8
FALSIFIER: a blip-injection test (drop 2049 for 60s) on a `soft` RW mount during a rename+write
loop shows zero truncated/short files and zero EIO-swallowed writes.
```

```
CLAIM: Putting the spool on a share the party holds RW breaks the arbiter's single-writer
premise — he can create `queue/<host>/running/<id>.json` and `done/<id>/result.json` directly,
and re-dropping a claimed id into `incoming/` makes the claim-rename clobber the running record.
EVIDENCE: CONTRACT.md:73-75 "Claim is `incoming→running` (atomic). On arbiter boot, any leftover
`running/<id>.json` ... is moved to `done/<id>/`"; unified-nfs-design.md:16-22 places
`workspace/ (RW to the external party)` and `queue/rtx/{incoming,running,done}` on ONE share.
CONFIDENCE: 0.7  (partly pre-existing: CONTRACT.md:31 already puts the spool "in the external party's sandbox")
FALSIFIER: the export/mode layout gives the sandbox write access to `incoming/` only, with
`running/`/`done/` owned by the arbiter identity and not writable by the squashed uid.
```

```
CLAIM: No quota is specified for `the external party-warm` on a volume that also holds Proxmox backups, so
the party filling /volume1 with "tens of GB" model files is a cross-tenant availability hit.
EVIDENCE: unified-nfs-design.md:10-11 "Large model files (tens of GB) are warm-stored on the
Synology"; synology-exports.txt:2,6 show `/volume1/proxmoxbackup` on the same volume1; the
proposal's NFS section (unified-nfs-design.md:40-47) names squash/IP/mount opts and no quota.
CONFIDENCE: 0.85
FALSIFIER: DSM share quota (or a dedicated volume) is set on the external party-warm before export.
```

```
CLAIM: A5 is satisfied — an NFS death still fails toward prod-UP — but only because
window.flag, heartbeat and arbiter state are NOT on the share; move any of them and gate 1 dies
with the NAS.
EVIDENCE: CONTRACT.md:22-30 `arbiter.lock — flock`, `state.json` under `~/.the external party-queue/`, and
`window.flag`/`heartbeat` under node2 `/var/lib/the external party-queue/` + GB10 `/home/<login-user>/the external party-queue/`;
design moves only the spool (unified-nfs-design.md:30-31). A hung/EIO'd arbiter stops rewriting
the heartbeat → CONTRACT.md:14-16 dead-man restores prod.
CONFIDENCE: 0.8
FALSIFIER: the built layout relocates window.flag/heartbeat/arbiter.lock under /mnt/the external party-warm —
note `nolock` (unified-nfs-design.md:44) would also make `arbiter.lock` flock local-only.
```

```
CLAIM: With `nofail`, a container started before the mount bind-mounts an EMPTY host directory
and jobs write to node2 local disk instead of the NAS — the estate's "active ≠ healthy" trap —
so a content-sentinel readiness predicate is required before window open and before each job.
EVIDENCE: `nofail` in the proposed opts (unified-nfs-design.md:44); RUNBOOK.md:71 "confirm prod
is HEALTHY not just active on close — the estate's 'active ≠ healthy' lesson"; CONTRACT.md:60-61
gate 6 readiness probe is the natural hook. Probe: `umount /mnt/the external party-warm; pct start 206;
pct exec 206 -- ls /workspace` returning empty rather than failing.
CONFIDENCE: 0.85
FALSIFIER: mounts are declared `x-systemd.required-by=` the container units (or `hard` without
nofail) and container start fails closed when the share is absent.
```

```
CLAIM: `showmount` output is not an adequate oracle for A3 — the supplied list prints option
strings for two exports and none for the other three — so verification needs a negative mount
attempt from a non-allowlisted host plus an on-DSM export-config diff.
EVIDENCE: synology-exports.txt:2-3 carry `(rw,async,...)` while lines 4-6
(`/volume1/1-RPX-Workspace-Data  <lan-address>`, etc.) carry client only, no options.
Probes: from CT 204 (<lan-address>, not allowlisted) `mount -t nfs <lan-address>:/volume1/the external party-warm`
→ expect EACCES; from CT 205/206/GB10 container `nc -z <lan-address> 2049` → expect no route;
before/after diff of DSM's `/etc/exports`.
CONFIDENCE: 0.85
FALSIFIER: DSM is shown to render complete, authoritative per-export options via showmount for
every share.
```

---

## VERDICT INPUT

- **Q1 Architecture — redesign.** One share as workspace+models+spool is the wrong shape: split into `models` (RW only from the editor CT, RO-exported/RO-bound to GPU hosts) and `workspace` (RW), and leave the spool where CONTRACT.md has it (local/over-ssh), so NFS is never in the at-most-once path. node2 as code-server home is acceptable *only* if CT 206 sits on `cirub0` with no vmbr0 NIC.
- **Q2 Security — build-with-changes.** Defensible only as: dedicated share, `root_squash,all_squash,anonuid=<the external party>`, `sec=sys` with explicit per-host IPs (no `*`, no subnet), no `crossmnt`, no `insecure`, DSM quota, and a proven-absent route from every party-reachable container to <lan-address>
- **Q3 Systems/ops — build-with-changes.** `hard,_netdev` + required-by ordering for RW paths (not `soft`), `soft,ro,nofail` for models only, sentinel-file readiness in gate 6, teardown that unmounts *and* deletes the export.

**Highest-severity issue:** the pre-existing `*` + `no_root_squash` (+`crossmnt`) exports (synology-exports.txt:2-3) turn *any* NAS reachability from the party's environment into root-level access to all Proxmox backups — and the proposal never pins CT 206 to the island, which is the one control preventing it.


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}
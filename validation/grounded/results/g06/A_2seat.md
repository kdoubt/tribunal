# A_2seat = union of two isolated Round 0 memos, no ledger, no Round 1

Decision g06. Memo 1 and Memo 2 are the two panel vendors' solo outputs, verbatim, in a fixed order; no cross-exposure, no adjudication.

=== MEMO 1 ===

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

=== MEMO 2 ===

Plausible framings: storage consolidation, sandbox containment, or queue correctness under failure. I choose **queue trust and failure boundaries**, because the proposal combines RW queue access with one squashed identity (`artifact/unified-nfs-design.md:41–47`), while the contract requires single-writer state (`artifact/CONTRACT.md:12`).

**1. CLAIM:** Redesign the storage boundary: retain shared NFS workspace/models, but keep authoritative queue state on trusted local storage behind validated submission, expose results read-only, and publish immutable model versions because a job’s RO bind does not prevent the editor changing its backing files.  
**EVIDENCE:** `artifact/unified-nfs-design.md:19–22,28,45,47`; `artifact/CONTRACT.md:12,34–36,73–75`.  
**CONFIDENCE:** 0.95  
**FALSIFIER:** Effective permissions demonstrate that the external party cannot modify claimed jobs or completion records, and editor writes cannot change a running job’s model version.

**2. CLAIM:** Host mounting does not itself create a sandbox network route or make symlinks resolve against the NAS root, but retaining CT 206 on node2 requires explicit unprivileged isolation, resource limits, narrow private bind mounts and no host-management interfaces; the documented GB10 helper-compromise path remains a separate containment weakness.  
**EVIDENCE:** `artifact/unified-nfs-design.md:24–29`; `artifact/RUNBOOK.md:23–30,115–127`; EXTERNAL: [mount bind semantics](https://man7.org/linux/man-pages/man8/mount.8.html), [pathname resolution](https://man7.org/linux/man-pages/man7/path_resolution.7.html).  
**CONFIDENCE:** 0.90  
**FALSIFIER:** A sandbox can reach NAS RPC directly, traverse an exposed host mount, or access a host-management socket through the proposed configuration.

**3. CLAIM:** A shared NAS is conditionally defensible using a dedicated filesystem export with exactly the three verified client addresses and `rw,sync,root_squash,all_squash,anonuid=<dedicated>,anongid=<dedicated>,secure,nocrossmnt,hide`, no exported ancestor or nested tenant mounts, and hard capacity limits; export options alone cannot compensate for a compromised trusted client.  
**EVIDENCE:** `artifact/unified-nfs-design.md:41–47`; `artifact/synology-exports.txt:2–3` lists existing `no_root_squash`, UID 1025/GID 100 and `crossmnt`; EXTERNAL: [exports(5), client matching, squashing and subdirectory exports](https://man7.org/linux/man-pages/man5/exports.5.html).  
**CONFIDENCE:** 0.85  
**FALSIFIER:** NAS topology shows an overlapping permissive export, the anonymous identity has tenant access, or the proposed “share” is merely an inadequately confined subdirectory; require `subtree_check` for that case or reject the layout.

**4. CLAIM:** A3 requires corroborated server configuration and end-to-end negative tests, because the supplied export listing cannot establish effective isolation or even resolve the proposal’s abbreviated client addresses.  
**EVIDENCE:** `artifact/unified-nfs-design.md:42`; `artifact/synology-exports.txt:2–6`. Acceptance probes, **not executed**: compare NAS persistent rules, `exportfs -v` and active export tables; verify observed client source addresses; test permitted access from each host and denial from an unauthorized client; verify root and ordinary-user writes map to the dedicated identity; inspect sandbox routes, capabilities and mount trees, then test NAS RPC access directly and through helpers plus traversal to harmless tenant canaries.  
**CONFIDENCE:** 0.94  
**FALSIFIER:** Any unauthorized access succeeds, existing exports change, or client/server observations disagree; passing tests supports the tested configuration, not an absolute exploit-proof guarantee.

**5. CLAIM:** A1 cannot be solved by mount flags alone: use `hard,vers=3,proto=tcp,_netdev,nofail` for mutable NFS data, treat `timeo=150`/`retrans=3` as retry tuning rather than a deadline, isolate potentially blocked I/O from control services, and gate windows/jobs on verified mount identity plus a fresh write–fsync–rename–cross-client-read probe whose completion is required within a controller deadline.  
**EVIDENCE:** `artifact/unified-nfs-design.md:44–45`; the precedent is RO (`artifact/gb10-nfs-precedent.txt:1`); existing readiness covers sandbox execution/GPU availability (`artifact/CONTRACT.md:60–61`). EXTERNAL: [nfs(5)](https://man7.org/linux/man-pages/man5/nfs.5.html): soft permits integrity failures; hard retries indefinitely; [systemd.mount(5)](https://man7.org/linux/man-pages/man5/systemd.mount.5.html): boot semantics.  
**CONFIDENCE:** 0.94  
**FALSIFIER:** During NAS loss the controller blocks awaiting its probe, admits work using cached/local fallback paths, or production recovery waits for NFS; editor file operations may still block.

**6. CLAIM:** A2’s same-filesystem rename remains atomic across NFSv3 clients, but at-most-once execution needs trusted durable claim records, one lifecycle writer, checked publication errors and conservative recovery of ambiguous operations; CTO caching is not instantaneous visibility, `nolock` supplies no cross-client exclusion, and stale handles or rename errors must never trigger blind replay.  
**EVIDENCE:** `artifact/CONTRACT.md:23–24,50–51,73–75`; `artifact/unified-nfs-design.md:38,45`. EXTERNAL: [RFC 1813 §3.3.14](https://www.rfc-editor.org/info/rfc1813/), [rename(2), BUGS](https://man7.org/linux/man-pages/man2/rename.2.html), [nfs(5), coherence and locking](https://man7.org/linux/man-pages/man5/nfs.5.html).  
**CONFIDENCE:** 0.95  
**FALSIFIER:** Crash/reboot tests around publication, claim and dispatch produce two launches of one accepted ID, or recovery treats an uncertain claim as unclaimed; no stable position on implementation correctness without the queue code.

**7. CLAIM:** A4 needs a coordinated migration covering NAS rules/ownership/quota, all three hosts’ persistent mounts, CT 205/206 binds and UID maps, GB10 editor/bench/ephemeral-job binds, submitter/arbiter/result paths, and ingress/provisioning/teardown definitions; otherwise expected failures include denied writes, empty local fallback directories, split queues and stale access endpoints.  
**EVIDENCE:** `artifact/unified-nfs-design.md:26–37,46–47`; `artifact/CONTRACT.md:21–36,52–54`; `artifact/RUNBOOK.md:33–38,48–61,94–101`. Writer inventory: operator/provisioners configure infrastructure; the external party writes submissions/workspace; arbiter writes lifecycle state; jobs write outputs—these actors need separate authority. CT 205 is unprivileged (`RUNBOOK.md:23`), so numeric UID equality alone is insufficient.  
**CONFIDENCE:** 0.91  
**FALSIFIER:** A traced submission-to-result survives reprovisioning/reboot with every actor on the intended paths and correct effective identities despite an omitted migration item.

**8. CLAIM:** A5 remains unproven until NAS-loss tests show that host-local watchdogs, deadlines, heartbeat handling, job termination and production restart remain NFS-independent, boot restores production before optional storage, and teardown stops writers, removes binds/mount definitions, unmounts clients and removes only the new export.  
**EVIDENCE:** `artifact/CONTRACT.md:14–16,26–29,48–59`; `artifact/unified-nfs-design.md:55–56`; current teardown scope: `artifact/RUNBOOK.md:96–101`. **SPECULATIVE:** scripts were not supplied; their NFS dependencies and blocked-process behavior cannot be verified.  
**CONFIDENCE:** 0.93  
**FALSIFIER:** NAS reboot/drop during a window prevents healthy production restoration by the deadline, corrupts accepted queue state, replays work, or teardown changes an existing export.

**VERDICT INPUT**

- **Q1: redesign** — separate authoritative queue control from external-party RW storage (claims 1, 6).
- **Q2: build-with-changes** — conditional containment; no demonstrated direct cross-export escape (claims 2–4).
- **Q3: redesign** — remove NFS from production recovery and authoritative queue state (claims 5–8).
- **Highest-severity established issue:** external-party RW access to queue lifecycle directories under the same squashed identity as the arbiter undermines trusted claim/completion state (`artifact/unified-nfs-design.md:45–47`; `artifact/CONTRACT.md:35,73–75`).

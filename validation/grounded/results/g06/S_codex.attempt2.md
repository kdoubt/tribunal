Plausible framings are storage convenience, tenant isolation, and preservation of queue/recovery guarantees. I choose the third, with isolation as a prerequisite: the existing contract promises single-writer state and independent production recovery (`artifact/CONTRACT.md:12-16`). Recommendations below are proposed changes; probes are acceptance tests, not executed observations.

**1. CLAIM:** Separate the authoritative queue from tenant-writable NFS storage, while retaining the unified editor on node2 in a restricted CT 206.
**EVIDENCE:** `artifact/unified-nfs-design.md:26-28,41-47` grants RW queue access and maps clients to one identity; `artifact/CONTRACT.md:34-36,73-75` makes queue entries authoritative execution records.
**CONFIDENCE:** 0.96
**FALSIFIER:** Effective permissions demonstrate that tenant processes cannot alter, delete, or recreate authoritative claimed/completed records.

Recommended shape: local arbiter-owned durable queue/ledger; untrusted submission inbox and separately published results; RW workspace; separate published model tier, immutable during jobs and RO to consumers. Separate directories with identical RW credentials do not establish that boundary. CT 206 should be unprivileged, resource-limited, without GPU devices, host-management sockets, or parent-directory binds. Node2 already shares the production GPU fault domain (`artifact/RUNBOOK.md:115-117`).

**2. CLAIM:** Host mounting need not create a sandbox network route, but the artifacts do not establish complete cross-tenant isolation.
**EVIDENCE:** `artifact/unified-nfs-design.md:24-29`; `artifact/RUNBOOK.md:121-127` explicitly records LAN-routable GB10 helpers; `artifact/synology-exports.txt:2-3` records broad backup exports with `no_root_squash`.
**CONFIDENCE:** 0.93
**FALSIFIER:** Namespace, routing, firewall, and helper-compromise tests demonstrate that every relevant sandbox/helper remains unable to reach unintended NAS exports or hosts.

A compromised helper is a documented conditional LAN path; successful backup access through it remains **SPECULATIVE** without effective export/credential tests. Neither another export’s `no_root_squash` nor an ordinary sandbox symlink alone proves an escape. Host-side consumption of attacker-controlled paths needs separate scrutiny; implementation is absent from these artifacts.

**3. CLAIM:** A shared-NAS export is conditionally defensible only with explicit identity, filesystem, and client boundaries beyond the proposed squashing.
**EVIDENCE:** `artifact/unified-nfs-design.md:41-47`; existing anonymous mapping is `1025:100` at `artifact/synology-exports.txt:2-3`. **EXTERNAL:** [exports(5), “User ID Mapping,” “crossmnt,” and “Subdirectory Exports”](https://man7.org/linux/man-pages/man5/exports.5.html).
**CONFIDENCE:** 0.87
**FALSIFIER:** Effective export configuration or access tests reveal broader clients, shared privileged identity, or access outside the dedicated filesystem.

Recommended export baseline: exact verified host source IPs; `rw,sync,root_squash,all_squash,anonuid=<dedicated>,anongid=<dedicated>,secure,hide,nocrossmnt`. Published models should be `ro` for consumers. Verify that the anonymous account/group has no other-tenant permissions; do not assume 1025 is dedicated. Prefer a dedicated filesystem root with no nested production mounts. If only a subdirectory is available, require `subtree_check` and test its stale-handle consequences. Check overlapping parent exports; do not modify existing exports.

**4. CLAIM:** A1 is unsatisfied because the copied RO precedent does not establish safe writable-storage behavior or live readiness.
**EVIDENCE:** `artifact/gb10-nfs-precedent.txt:1`; `artifact/unified-nfs-design.md:44-45`; readiness currently checks sandbox execution and GPU availability (`artifact/CONTRACT.md:60-61`). **EXTERNAL:** [nfs(5), mount options](https://man7.org/linux/man-pages/man5/nfs.5.html); [systemd.mount(5), `nofail`](https://man7.org/linux/man-pages/man5/systemd.mount.5.html).
**CONFIDENCE:** 0.97
**FALSIFIER:** Outage tests demonstrate bounded control-path response, checked write failures, and rejection of unavailable mounts before opening windows.

Use `vers=3,proto=tcp,hard,_netdev,nofail` for integrity-sensitive data. `soft` can return I/O errors and risk corruption; `hard` can wait indefinitely. `timeo=150` means 15 seconds initially; `retrans=3` is not an overall hard-mount deadline. `nofail` permits boot without storage; `_netdev` supplies network ordering, not health. Require correct mount identity plus fresh read and write–fsync–rename–readback probes before windows/jobs. A separate supervisor must bound admission waiting without assuming the blocked I/O worker terminates.

**5. CLAIM:** A2’s rename primitive survives NFSv3, but the claimed at-most-once guarantee needs additional durability and recovery rules.
**EVIDENCE:** `artifact/CONTRACT.md:50-51,73-75`. **EXTERNAL:** [RFC 1813 §3.3.14](https://www.rfc-editor.org/rfc/rfc1813.html#section-3.3.14); [rename(2), BUGS](https://man7.org/linux/man-pages/man2/rename.2.html); [nfs(5), coherence and locking](https://man7.org/linux/man-pages/man5/nfs.5.html).
**CONFIDENCE:** 0.96
**FALSIFIER:** Crash/reply-loss tests show a durable, tenant-inaccessible execution ledger prevents every duplicate dispatch.

Same-filesystem rename is atomic across clients; failed RPC completion can nevertheless mean rename occurred. Close-to-open consistency and attribute caching do not provide transactions or instant visibility. `nolock` provides no cross-client exclusion, although it does not invalidate rename or the existing local arbiter lock. Require immutable accepted specs, durable IDs/claims before dispatch, checked persistence errors, and quarantine on ambiguous completion or `ESTALE`; never blindly requeue. Keep lock/budget state local.

**6. CLAIM:** A3 requires effective server configuration and adversarial access tests, not export-list output alone.
**EVIDENCE:** `artifact/synology-exports.txt:1-6` contains no effective configuration for the proposed export; the design is explicitly unbuilt (`artifact/unified-nfs-design.md:3`).
**CONFIDENCE:** 0.98
**FALSIFIER:** A recorded acceptance matrix already supplies the following evidence.

Inspect NAS effective exports (`exportfs -v` or Synology equivalent), backing mounts, ownership, ACLs, and parent exports; inspect each host’s `findmnt` and sandbox mount namespace. Verify allowed hosts succeed and a fourth client is denied. Check server-observed ownership of canary writes from sandbox root and ordinary users. From CT 205, CT 206, and GB10, test direct and proxy-mediated NAS RPC access, unintended-export reads, parent traversal, symlinks, and RO-remount attempts. Use dedicated canaries, not production-data reads.

**7. CLAIM:** A4 requires coordinated migration across all producers, consumers, and configuration owners.
**EVIDENCE:** `artifact/unified-nfs-design.md:26-37,46-47`; `artifact/CONTRACT.md:21-36`; `artifact/RUNBOOK.md:48-57`.
**CONFIDENCE:** 0.96
**FALSIFIER:** A configuration inventory demonstrates every actor already uses the intended paths and identities.

Required ownership map; consequences are **ASSUMPTION** predictions:

- NAS administrator: export, ACLs, quota, identity; omission causes denial or excessive access.
- Three host administrators: fstab/mount units; omission causes missing or divergent storage.
- Node2 administrator: CT 205/206 binds and UID maps; omission causes permission failures or excessive visibility.
- GB10 container definitions: editor/bench/job binds and users; omission leaves jobs on old storage.
- Queue maintainer: submitter, arbiter, job/result writers, migration/deduplication; omission strands or duplicates submissions.
- Access administrator: bastion ingress/firewall and CF applications; omission preserves obsolete endpoints.

**8. CLAIM:** A5 remains unproven until NAS-failure recovery demonstrably restores healthy production without waiting for NFS.
**EVIDENCE:** Local flags/heartbeat and watchdog guarantees: `artifact/CONTRACT.md:26-29,48-59`; unanswered outage question: `artifact/unified-nfs-design.md:55-56`; existing teardown scope: `artifact/RUNBOOK.md:96-101`.
**CONFIDENCE:** 0.95
**FALSIFIER:** NAS blackout/reboot tests during GPU I/O restore healthy production by deadline and preserve execution records.

Keep watchdogs, deadlines, heartbeat, recovery executables, logs, and authoritative state local. Test stale heartbeat and continuing heartbeat separately; blocked job teardown must not prevent recovery. Boot production/watchdogs independently; gate tenant containers on mounted-and-serving storage. Teardown must close windows, restore production, stop consumers, remove binds, unmount all three hosts, remove mount configuration, then remove only the new exports.

**VERDICT INPUT**

Q1: **redesign** — separate authoritative queue and mutable storage (claim 1).  
Q2: **build-with-changes** — conditional acceptance after containment/export verification (claims 2–3, 6).  
Q3: **redesign** — remove NFS from production recovery and authoritative queue dependencies (claims 4–5, 8).  
Highest-severity established issue: **tenant write authority over execution records**, undermining queue integrity and at-most-once protection (claim 1).

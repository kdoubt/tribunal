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

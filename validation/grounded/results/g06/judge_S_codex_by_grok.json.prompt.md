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


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}
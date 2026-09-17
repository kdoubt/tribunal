Plausible framings are network isolation, queue consistency, and end-to-end failure containment. I choose the third: the decision requires isolation **and** production recovery. Recommendations below are review judgments; probes are proposed, not executed.

**1. CLAIM:** The shared writable spool violates the intended authority boundary: separate trusted queue state onto arbiter-local storage, expose submission intake and result views, and retain NFS for workspace/models, with versioned models if jobs require stable contents.  
**EVIDENCE:** `artifact/unified-nfs-design.md:19–22,28,41–47` grants RW queue access under one squashed identity; `artifact/CONTRACT.md:12,35,73–75` makes arbiter-owned claims and surviving `running` records authoritative; proposal line 20 permits editor writes despite job-side RO models.  
**CONFIDENCE:** 0.95  
**FALSIFIER:** Permission tests show the external party cannot alter/delete claimed records or authoritative results, and cannot replay an accepted ID; an independently protected ledger enforces this despite writable intake.

**2. CLAIM:** Host-mounted bind mounts can preserve network isolation, but they do not establish complete containment; keep CT 206 unprivileged and resource-limited on node2, without GPUs, host-management sockets or unrelated mounts, and close the documented GB10 helper route before accepting the stronger isolation claim.  
**EVIDENCE:** `artifact/unified-nfs-design.md:24–29`; `artifact/RUNBOOK.md:23–30,115–127`, especially “dual-homed” and “host-NAT-routable to the LAN.”  
**CONFIDENCE:** 0.88  
**FALSIFIER:** Namespace inspection and traffic tests demonstrate CT 206 has stronger isolation already and compromised GB10 helpers cannot reach private networks. No stable position on an exploitable NAS escape: the artifacts establish a conditional route, not a demonstrated compromise.

**3. CLAIM:** Shared-NAS use is defensible only conditionally: give the new export an isolated filesystem root, exact three verified source IPs, `rw,sync,root_squash,all_squash,anonuid=<dedicated>,anongid=<dedicated>,secure,hide,nocrossmnt,no_subtree_check`, no overlapping parent export, enforced capacity limits, and narrowly scoped binds; do not assume UID 1025 is dedicated or alter existing exports.  
**EVIDENCE:** `artifact/unified-nfs-design.md:41–47`; `artifact/synology-exports.txt:2–3` already names `anonuid=1025,anongid=100` and `no_root_squash`; **EXTERNAL:** [exports(5), User ID Mapping, Subdirectory Exports, crossmnt](https://man7.org/linux/man-pages/man5/exports.5.html). Filesystem-root isolation is an acceptance condition; a subdirectory instead requires separate subtree/filehandle analysis.  
**CONFIDENCE:** 0.85  
**FALSIFIER:** Effective server configuration reveals overlapping exports, shared identity privileges, or reachable sibling filesystems; alternatively, tests establish equivalent confinement through another supported configuration.

**4. CLAIM:** A3 requires server-side configuration inspection plus adversarial access tests, because export listings alone cannot establish either identity squashing or sandbox confinement.  
**EVIDENCE:** Decisive acceptance probe: compare NAS persistent configuration with effective `exportfs -v`/server export tables; create disposable files from each allowed host as root and ordinary users and inspect server ownership; attempt mounting the new export from a disallowed client; from every sandbox test direct and proxy-assisted NAS RPC access and attempts to access canaries in unintended exports; inspect mount namespaces and test absolute/relative symlinks and `..`, including privileged intake processing. Relevant inventory: `artifact/synology-exports.txt:2–6`; helper path: `artifact/RUNBOOK.md:121–127`.  
**CONFIDENCE:** 0.93  
**FALSIFIER:** Any denied-client mount, unsquashed ownership, unintended canary read, or host-side symlink traversal succeeds. A normal symlink alone is not proof of server-side export escape.

**5. CLAIM:** A1 should use `vers=3,proto=tcp,hard,_netdev` for writable data, with explicit benchmark-service mount dependencies and isolated I/O workers, because `soft` trades integrity for responsiveness and no mount option guarantees both bounded I/O and integrity during an indefinite outage.  
**EVIDENCE:** `artifact/unified-nfs-design.md:44–45` copies the RO precedent at `artifact/gb10-nfs-precedent.txt:1`; **EXTERNAL:** [nfs(5), soft/hard, timeo, retrans](https://man7.org/linux/man-pages/man5/nfs.5.html) and [systemd.mount(5), nofail](https://man7.org/linux/man-pages/man5/systemd.mount.5.html). `timeo=150` means 15 seconds per initial timeout; `retrans=3` is not an application deadline.  
**CONFIDENCE:** 0.95  
**FALSIFIER:** Outage testing shows bounded, checked failure without corruption under the proposed options. Required readiness oracle: before opening and each job, verify mount identity plus fresh write/fsync/rename/readback in a disposable directory, cross-client where needed; a supervisor must reject late probes without waiting indefinitely. `nofail` may preserve host boot, not authorize benchmark startup.

**6. CLAIM:** A2 preserves same-filesystem server-side rename atomicity, but that alone does not prove at-most-once execution across hostile writers, caching, ambiguous RPC outcomes or recovery; retain local singleton locking and durable accepted-ID/dispatch records, and quarantine uncertain claims instead of retrying execution.  
**EVIDENCE:** `artifact/CONTRACT.md:23–24,50–51,73–75`; proposed `nolock` at `artifact/unified-nfs-design.md:45`; **EXTERNAL:** [RFC 1813 §3.3.14 and §4.5](https://www.rfc-editor.org/rfc/rfc1813.html), [nfs(5), close-to-open and nolock](https://man7.org/linux/man-pages/man5/nfs.5.html): directory caching can delay discovery; `nolock` supplies no cross-client exclusion.  
**CONFIDENCE:** 0.93  
**FALSIFIER:** Two-client tests involving same-ID replacement, lost rename replies, stale handles and NAS/arbiter restarts consistently preserve the claimed guarantee through an independently verified recovery protocol.

**7. CLAIM:** A4 needs coordinated migration of every writer and reader, including NAS configuration/ownership, all three hosts’ fstab and mount units, CT 205/206 binds, GB10 persistent and ephemeral container definitions, submitter/arbiter/result paths, and actual LXC/Docker UID mappings.  
**EVIDENCE:** `artifact/unified-nfs-design.md:26–37,46–47`; old paths and writers at `artifact/CONTRACT.md:21–36,53–54`; unprivileged LXC at `artifact/RUNBOOK.md:23`. Acceptance matrix: omitted NAS mapping → denied/shared-identity writes; omitted host dependency → unavailable or underlying-local paths; omitted sandbox bind → stale workspace/models; omitted submitter/arbiter/result path → divergent queues; omitted UID mapping → access failures.  
**CONFIDENCE:** 0.94  
**FALSIFIER:** Inventory and end-to-end submissions from CT 206 to both GPU contexts show another actor/store participates, or a listed omission demonstrably cannot affect this deployment.

**8. CLAIM:** A5 remains unproven until NAS-loss tests demonstrate NFS-independent production recovery and boot, with local watchdog state, progress-sensitive heartbeats, recovery that does not wait for blocked NFS tasks, and teardown extended to remove new binds, host mounts/fstab entries and only the new export.  
**EVIDENCE:** `artifact/CONTRACT.md:26–29,48–59` specifies local watchdog state and recovery; `artifact/unified-nfs-design.md:55–58` leaves outage behavior open; `artifact/RUNBOOK.md:94–101` lists existing teardown. **SPECULATIVE:** implementation-level recovery under blocked NFS; recovery scripts are not supplied.  
**CONFIDENCE:** 0.92  
**FALSIFIER:** Mid-window NAS reboot/drop tests show timely healthy production, no duplicate dispatch or corrupted authoritative state, successful boot without NAS, and clean teardown without modifying other exports.

**VERDICT INPUT**

- **Q1:** redesign — separate trusted queue authority from writable warm storage; retain node2 placement with claim 2’s containment.
- **Q2:** build-with-changes — conditional on claims 2–4’s confinement and verification.
- **Q3:** redesign — implement claims 5–8’s independent control and recovery.
- **Highest-severity established issue:** external-party write authority over authoritative queue records undermines integrity and replay protection (claim 1); cross-tenant NAS compromise remains conditional, not demonstrated.

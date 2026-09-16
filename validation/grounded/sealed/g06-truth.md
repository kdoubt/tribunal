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

## 1. ATTACK

**Peer-3 (0.90) is the one that's wrong where it matters.** "Host mounting does not itself create a sandbox network route" is true and useless as reassurance, and filing the GB10 helper dual-homing as "a separate containment weakness" is the error. It is not separate. `RUNBOOK.md:121-127`: `the external party-proxy` (squid) and `the external party-ssh-relay` (socat) are "dual-homed on the default docker bridge (host-NAT-routable to the LAN)", containment rests on "the squid ACL + socat range, **not on routing**", and the closing rule (`iptables -I DOCKER-USER -s 172.17.0.0/16 -d <lan-address>/16 -j DROP`) is **deferred, unrun**. <lan-address> is inside <lan-address>/16. So a helper compromise lands on the NAS, where `synology-exports.txt:2-3` exports `/volume1/proxmox*` to `*` with `no_root_squash` (one `crossmnt`) — root RW on all Proxmox backups. Peer-3's own falsifier ("a sandbox can reach NAS RPC directly") is already half-satisfied by its own cited lines. 0.90 is not supportable; and it never addresses CT 206's unnamed network (`unified-nfs-design.md:27-28`), the other unpinned route.

**Peer-2 (0.94)** is internally inconsistent: it keeps `nofail` on *mutable* data while its falsifier names "admits work using cached/local fallback paths" as disproof. `nofail` is precisely how a container starts over an empty host dir and jobs write node2 local disk (`unified-nfs-design.md:44`; "active ≠ healthy", `RUNBOOK.md:71`). `nofail` is defensible only paired with `x-systemd.required-by=` the container units plus a content sentinel. Also over-engineered: "isolate blocked I/O from control services" is solved by not mounting the share on the arbiter host at all. 0.94 on an unrun configuration is inflated.

**Peer-1 (0.91)**: correct inventory, but a compound conjunction at 0.91 and a near-unfalsifiable falsifier ("despite an omitted migration item" — which item?). It hedges "numeric UID equality alone is insufficient" without naming the mechanism: unprivileged LXC shifts uid 1000 → ~101000 on the host, so the squashed anon uid 1025 (`unified-nfs-design.md:46`) owns nothing the party's process can write — EPERM unless 0777. It also omits the NAS quota owner.

**Peer-4 (0.95)**: right, but under-specified on enforcement — a job's RO *bind* is a sandbox-side property; the editor still writes the backing files over its RW export. Immutability must be a second RO export to the GPU hosts (or dirs unwritable by the squashed uid), not a bind flag. And 0.95 on a recommendation bundle is a category error: its falsifier tests the *fix*, not the claim.

**Peer-5 (0.93)**: the teardown half is confirmed — `RUNBOOK.md:96-101` enumerates CF objects, both CTs, vnet/zone, docker objects, backup-exclude CAS, and **no NFS export or mount removal**. The A5 half correctly beats my Own-6 on one axis (below).

## 2. CONCEDE

- Peer-5: A5 is *not* fully settled by layout. `hard` mounts put blocked writers in uninterruptible sleep — job kill, window close, and teardown can stall even with heartbeat local. Unproven without a NAS-drop test. Teardown must delete the export.
- Peer-4: results RO, models immutable/versioned, authoritative queue state local.
- Peer-1: A4 needs actor-separated authority, not just path moves.
- Peer-2: `timeo`/`retrans` are retry tuning, not a deadline; the controller needs its own deadline.

## 3. REVISE

**Own-6**: new text — "The layout preserves fail-toward-prod-UP for `window.flag`/`heartbeat`/`arbiter.lock` (`CONTRACT.md:22-30`), but this is a design-intent property only: with `hard` RW mounts, writers in uninterruptible sleep may block job termination, window close and teardown, and no scripts were supplied. Unproven until a NAS-drop test." **0.8 → 0.6.**

**Own-1**: widen — CT 206's unnamed network is *one* of two unpinned routes; the deferred `DOCKER-USER` rule (`RUNBOOK.md:125-127`) is the other, and it is live today. Confidence 0.75 → **0.85**.

No other revisions.

## 4. VERDICT INPUT

- **Q1 Architecture — redesign.** Split models (RO-exported to GPU hosts) / workspace (RW); spool stays local per `CONTRACT.md`; node2 acceptable only with CT 206 pinned to `cirub0`, no vmbr0.
- **Q2 Security — build-with-changes.** Dedicated export, three host IPs, `root_squash,all_squash,anonuid`, no `*`/subnet/`crossmnt`/`insecure`, DSM quota, **and the deferred GB10 `DOCKER-USER` DROP run first** — it is now a precondition, not a residual.
- **Q3 Systems/ops — build-with-changes.** `hard,_netdev` + `x-systemd.required-by` (not bare `nofail`) on RW; sentinel readiness in gate 6; controller-side deadline; teardown unmounts and deletes the export.

**Highest-severity:** any NAS reachability from the party's environment — via unpinned CT 206 *or* the unrun GB10 `DOCKER-USER` rule — becomes root RW on all Proxmox backups through the pre-existing `*` + `no_root_squash` exports.

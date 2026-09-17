# Verdict - g06: a unified NFS-backed single-session workspace for an untrusted external party

<!-- Mechanical fill of core/templates/verdict.md from ledger.md rows only. No new arguments. -->

## 1. Independent agreement (agreed-r0)

The panel concludes, from claims both seats made before cross-exposure:
- A3/B5: copying `soft` from the RO precedent onto RW paths is wrong; RW paths need `hard` (+`_netdev`) and a readiness predicate that the mount is live and serving before a window opens and before each job (A7/B5, B7: an absent mount otherwise yields an empty local directory that jobs write into).
- A4/B1/B6: the queue spool must not sit on a share the party holds RW under the arbiter's squashed identity; authoritative claim/lifecycle state stays single-writer on trusted storage; at-most-once over NFSv3 requires trusted claim records and `nolock` gives no cross-client exclusion.
- A5/B3: a dedicated export limited to exactly the three host IPs, `root_squash` + `all_squash` to a dedicated uid/gid, no `*`/subnet, no `crossmnt`, plus a hard quota, is the only defensible export shape.
- A8/B4: `showmount` alone cannot verify the export; a negative mount from a non-allowlisted host plus an on-NAS export-configuration comparison is required.
- Q1 redesign and Q2 build-with-changes, from both seats.

## 2. Resolved after Round 0

**Oracle-settled (`verified`):**
- Pointer resolution: 19 of 19 (A) and 34 of 34 (B) file:line pointers open in the artifact.
- synology-exports.txt:2-3 → the existing `/volume1/proxmox*` exports are to `*` with `no_root_squash` (one `crossmnt`); lines 4-6 carry no options (A1 premise, A8, B3, B4).
- unified-nfs-design.md:27-28 → no network named for CT 206; RUNBOOK.md:24 → CT 205 pinned to the no-uplink vnet (A1 premise).
- unified-nfs-design.md:44-45 vs gb10-nfs-precedent.txt:1 → the RO precedent's `soft,nofail,nolock` copied onto RW paths (A3, A7, B5).
- CONTRACT.md:22-30 + unified-nfs-design.md:30-31 → arbiter.lock/state/window.flag/heartbeat stay local; only the spool moves (A6 premise, B8 pointers).
- RUNBOOK.md:96-101 → teardown lists no NFS export or mount removal (B8 teardown half, confirmed by A in R1).

**Cross-examination-settled (`conceded` / `revised`):**
- A6 → revised by A toward B8: A5 is unproven until a NAS-drop test (`hard` writers can block kill/close/teardown); conf 0.8→0.6.
- A7/B5 → converged: `nofail` only with explicit unit dependencies and a readiness gate (B revised Own-5).
- B1 → conceded by A: results RO, models immutable via a second RO export, queue authority local.
- B7 → conceded by A: actor-separated authority across the A4 inventory.
- A1 → widened by A (second unpinned route: the deferred GB10 `DOCKER-USER` DROP, RUNBOOK.md:121-127); B conceded CT 206 needs specified, verified isolation because the existing exports make unintended NAS access catastrophic.

## 3. Surviving dissent

- **Q3 mode label.** A: build-with-changes (`hard,_netdev` + required-by, sentinel readiness, controller deadline, teardown deletes the export). B: redesign (independent recovery, mount gates, NAS-loss acceptance tests). Substance converged; the label did not. Cheapest discriminating test: none mechanical; it is a naming call on the same change list.
- **Highest-severity issue.** A: NAS reachability from the party's environment (unpinned CT 206 or the unrun `DOCKER-USER` rule) becomes root RW on all Proxmox backups through the pre-existing `*` + `no_root_squash` exports. B: that path is the highest *potential* severity but conditional and unproven; the strongest *established* defect is party-writable authoritative queue state. Cheapest test: `pct config 206 | grep ^net` once built, plus an RPC-reachability probe from CT 206 and from a GB10 helper to the NAS.
- **A2 (uid EPERM).** A: writes fail unless the share is 0777. B: not established; `all_squash` maps shifted uids server-side. Cheapest test: `sudo -u the external party touch` inside CT 205 and `ls -n` on the host share.

## Recommendation

**Mode:** `dont` (as proposed; the agreed bucket is redesign on Q1). Build only as the redesigned shape in bucket 1 (queue authority off the share; models RO-exported; dedicated squashed export with quota and three host IPs; `hard` RW with unit dependencies and a serving-sentinel gate), with the Q3 label, the severity ranking, and the uid question left as bucket-3 human calls; the uid and CT 206 probes above are the cheapest checks.

## Record

- Open (unexamined, not endorsed): none.
- Verified vs merely agreed: verified = the six oracle facts above; agreed-only = the export-option list (B3), NFSv3 semantics (B6), migration inventory (B7).
- Rounds run / stop rule hit: R0+R1; stopped: no verdict input crossed a line in R1, no new claim IDs.
- Transformations applied to relayed text: ORCH-SUMMARY ledger note (agreed items + oracle facts); rival claims relayed verbatim under Peer tags, shuffled; no tallies; see transformations.log.

VERDICT INPUT
- Q1 Architecture: redesign (agreed-r0).
- Q2 Security: build-with-changes (agreed-r0): dedicated export, three host IPs, root_squash+all_squash to a dedicated uid/gid, no `*`/subnet/crossmnt/insecure, quota, verified absence of any NAS route from CT 206 and from the GB10 helpers.
- Q3 Systems/operational: build-with-changes (A) / redesign (B) - surviving dissent on the label; converged change list: `hard,_netdev` + unit dependencies, serving-sentinel readiness before window open and each job, controller-side deadline, NAS-loss acceptance test before go-live, teardown that unmounts and deletes the export.
- Highest-severity issue: NAS reachability from the party's environment × the pre-existing `*` + `no_root_squash` exports (A, widened; B: highest potential) / party-writable authoritative queue state (B: strongest established) - surviving dissent on the ranking.

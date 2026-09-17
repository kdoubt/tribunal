You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: Upgrade 2.14.2→2.14.5 is justified for this R3 file-store cluster by JetStream/Raft/filestore fixes, not by the owner’s “auth/permission CVEs”; those CVE-shaped items need TLS, JWT, MQTT, or leafnodes, which this config does not enable.
POINTER: release-notes-v2.14.5.md:29
OPENED TEXT:
- Fix a bug that could result in potential data loss when handling idempotent stream creates when an offline node catches up from a metalayer snapshot, caused by an incorrect update to the create time in the stream assignment (#8449)

### Complete Changes

[2] CLAIM: Upgrade 2.14.2→2.14.5 is justified for this R3 file-store cluster by JetStream/Raft/filestore fixes, not by the owner’s “auth/permission CVEs”; those CVE-shaped items need TLS, JWT, MQTT, or leafnodes, which this config does not enable.
POINTER: v2.14.4.md:56
OPENED TEXT:
- Filestore blocks with unsynced or truncated key files are now removed and counted as lost data instead of failing to recover altogether (#8365)
- Filestore encryption key files are now synced to disk more aggressively (#8366)
- Raft now handles the append entry iterator returning no more entries correctly (#8372)
- Fixed string ownership when handling the expected last sequence per subject in a batch (#8377)
- Fixed a race condition between concurrent message removals via limits that could unexpectedly disable writes into a filestore (#8378)

[3] CLAIM: Upgrade 2.14.2→2.14.5 is justified for this R3 file-store cluster by JetStream/Raft/filestore fixes, not by the owner’s “auth/permission CVEs”; those CVE-shaped items need TLS, JWT, MQTT, or leafnodes, which this config does not enable.
POINTER: v2.14.4.md:68
OPENED TEXT:
- Raft proposals now require the term to be passed down from JetStream, preventing situations where stale proposals from a previous term could make changes in a new term after a fast election (#8370)
- Replicated streams that were recreated while a node was down are no longer treated as an update by a returning node processing a snapshot, avoiding stale Raft groups from continuing to run and unexpected behaviour with consumers (#8413)
- Stream snapshot endpoints now more strictly check the reply subject for validity

MQTT

[4] CLAIM: Upgrade 2.14.2→2.14.5 is justified for this R3 file-store cluster by JetStream/Raft/filestore fixes, not by the owner’s “auth/permission CVEs”; those CVE-shaped items need TLS, JWT, MQTT, or leafnodes, which this config does not enable.
POINTER: v2.14.3.md:76
OPENED TEXT:
- Consumer ack subscriptions now match correctly when consumer names contain `%` (#8301)
- Observer state is now cleared correctly during `js_cluster_migrate` when a leaf remote is removed (#8304)
- Atomic batch end-of-batch max-size checks and R1 message rewrites have been fixed (#8305)
- Schedule drift, failed fast batch commits with `gapOk` and stale `/varz` leaf remote state have been fixed (#8308)
- Peer state decoding now bounds peer ID reads to the buffer length (#8310)

[5] CLAIM: Upgrade 2.14.2→2.14.5 is justified for this R3 file-store cluster by JetStream/Raft/filestore fixes, not by the owner’s “auth/permission CVEs”; those CVE-shaped items need TLS, JWT, MQTT, or leafnodes, which this config does not enable.
POINTER: v2.14.3.md:85
OPENED TEXT:
- `MultiLastSeqs` no longer reorders stream config subjects through `filterIsAll` handling (#8315)
- Meta recovery snapshots no longer leave phantom streams or consumers behind (#8324)
- Skipped messages last time no longer violates ordering that could lead to issues with starting by time (#8237)
- Raft now reverts uncommitted membership changes correctly when truncating or snapshotting (#8332)

[6] CLAIM: Upgrade 2.14.2→2.14.5 is justified for this R3 file-store cluster by JetStream/Raft/filestore fixes, not by the owner’s “auth/permission CVEs”; those CVE-shaped items need TLS, JWT, MQTT, or leafnodes, which this config does not enable.
POINTER: v2.14.3.md:80
OPENED TEXT:
- Peer state decoding now bounds peer ID reads to the buffer length (#8310)
- Counter stream staging no longer corrupts the committed running total (#8311)
- Filestore compaction no longer corrupts compressed or encrypted blocks (#8312)
- Memory store `NumPending` no longer overcounts for `DeliverLastPerSubject` consumers (#8313)
- Consumer inactive-delete grace period handling and pull request `MaxBytes` budgeting have been fixed (#8314)

[7] CLAIM: Upgrade 2.14.2→2.14.5 is justified for this R3 file-store cluster by JetStream/Raft/filestore fixes, not by the owner’s “auth/permission CVEs”; those CVE-shaped items need TLS, JWT, MQTT, or leafnodes, which this config does not enable.
POINTER: live-state.md:15-21
OPENED TEXT:
- STREAM JOBS: subjects=['jobs.>'] retention=workqueue max_age=7d max_bytes=5368709120 discard=old R3 storage=file | msgs=0 bytes=0 leader=nats3
    - consumer clickup-ingest: filter=jobs.clickup.events ack_wait=120s max_deliver=5 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=1
    - consumer embed-workers: filter=jobs.embed.batch ack_wait=600s max_deliver=5 max_ack_pending=16 pending=0 redelivered=0 waiting_pulls=0
    - consumer hale-eval-workers: filter=jobs.hale.eval ack_wait=300s max_deliver=4 max_ack_pending=8 pending=0 redelivered=0 waiting_pulls=0
    - consumer rtx-serve: filter=jobs.serve.> ack_wait=30s max_deliver=-1 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=0
- STREAM RESULTS: subjects=['results.>'] retention=limits max_age=3d max_bytes=2147483648 discard=old R3 storage=file | msgs=380 bytes=63214 leader=nats1
    - consumer hale-eval-collector: filter=results.hale.eval ack_wait=60s max_deliver=-1 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=0
- STREAM EVENTS: subjects=['events.support.>', 'events.slack.>'] retention=limits max_age=30d max_bytes=-1 discard=old R3 storage=file | msgs=4 bytes=883 leader=nats1
    - consumer supp

[8] CLAIM: Upgrade 2.14.2→2.14.5 is justified for this R3 file-store cluster by JetStream/Raft/filestore fixes, not by the owner’s “auth/permission CVEs”; those CVE-shaped items need TLS, JWT, MQTT, or leafnodes, which this config does not enable.
POINTER: v2.14.4.md:39
OPENED TEXT:
- Several JetStream and MQTT endpoints now correctly guard against null values in JSON
- Fixed an authentication bypass with TLS `verify_and_map` authenticating users with blank passwords

Monitoring

[9] CLAIM: Upgrade 2.14.2→2.14.5 is justified for this R3 file-store cluster by JetStream/Raft/filestore fixes, not by the owner’s “auth/permission CVEs”; those CVE-shaped items need TLS, JWT, MQTT, or leafnodes, which this config does not enable.
POINTER: v2.14.3.md:53-59
OPENED TEXT:
MQTT

- Partial `CONNECT` packets can no longer exhaust pre-authentication memory
- `PUBLISH` remaining-length underflow no longer causes a server panic
- Subscriptions to internal `$MQTT.deliver.pubrel` subjects are now rejected
- Subscribe deny rules are now enforced on retained message and QoS replay paths
- WebSocket `/mqtt` upgrades no longer panic when MQTT is disabled

Monitoring

[10] CLAIM: Upgrade 2.14.2→2.14.5 is justified for this R3 file-store cluster by JetStream/Raft/filestore fixes, not by the owner’s “auth/permission CVEs”; those CVE-shaped items need TLS, JWT, MQTT, or leafnodes, which this config does not enable.
POINTER: v2.14.3.md:41-42
OPENED TEXT:
- Inherited JWT default permissions are now refreshed when account claims are updated (#8276)
- External auth configuration is now cleared correctly when account claims are updated (#8275)
- PROXY protocol detection, TLS sniffing with `allow_non_tls` and PROXY v1 address-family parsing have been fixed (#8302)
- A race in gateway `CONNECT` handling has been fixed (#8306)
- Trusted proxy tracking no longer leaks closed clients during concurrent updates (#8307)
- Service import replies can now be delivered across cluster routes (#8317)

[11] CLAIM: Upgrade 2.14.2→2.14.5 is justified for this R3 file-store cluster by JetStream/Raft/filestore fixes, not by the owner’s “auth/permission CVEs”; those CVE-shaped items need TLS, JWT, MQTT, or leafnodes, which this config does not enable.
POINTER: v2.14.3.md:49
OPENED TEXT:
- `NoAuthUser` now checks connection restrictions
- Leaf connections no longer bypass `Nats-Trace-Dest` publish permission checks
- `CONNZ` and `SUBSZ` pagination now guard against `Offset` and `Limit` integer overflow panics
- Fixed a nil pointer panic when starting up when the resolver parent directory is missing (#8329)

[12] CLAIM: Upgrade 2.14.2→2.14.5 is justified for this R3 file-store cluster by JetStream/Raft/filestore fixes, not by the owner’s “auth/permission CVEs”; those CVE-shaped items need TLS, JWT, MQTT, or leafnodes, which this config does not enable.
POINTER: v2.14.3.md:91
OPENED TEXT:
While CVE advisory notices are credited individually, a number of fixes in this release were the result of non-CVE reports from the following contributors:

- Koda Reef
- @Emin-ACIKGOZ

[13] CLAIM: Roll followers first, meta leader last: nats2 → nats1 → nats3; freeze stream/consumer creates; avoid PVE `:00` `vzdump`; copy the 2.14.2 binary aside and tar `/var/lib/nats/js` on the node before replacing it.
POINTER: live-state.md:7-8
OPENED TEXT:
| nats1 | CT 121 | node1 (zfs subvol) | /usr/local/bin/nats-server (17968728 B, dated Jun 22) | v2.14.2 | 10349df4… | 2026-06-22 03:31 UTC | 0 | Debian 12 |
| nats2 | CT 122 | node2 (lvm) | same binary/hash | v2.14.2 | 10349df4… | 2026-07-26 21:07 UTC (CT rebooted then) | 0 | Debian 12 |
| nats3 | CT 123 | node3 (lvm), seed | same binary/hash | v2.14.2 | 10349df4… | 2026-07-28 13:28 UTC (CT rebooted then) | 0 | Debian 12 |

All 3 LXCs: unprivileged, nesting=1, 2c/2G/30G, onboot=1, startup order=1. JetStream store `/var/lib/nats/js` uses ~0.5–1.1 MB of a 15 GB cap. The binary was installed by `install-nats.sh` (see evidence), which fetches GitHub **latest** with no version pin; the binary has not changed since Jun 22. There is no apt package, no second copy of the binary, and no documented rollback path on the nodes.

[14] CLAIM: Roll followers first, meta leader last: nats2 → nats1 → nats3; freeze stream/consumer creates; avoid PVE `:00` `vzdump`; copy the 2.14.2 binary aside and tar `/var/lib/nats/js` on the node before replacing it.
POINTER: live-state.md:44
OPENED TEXT:
## Backup / maintenance windows
- PVE hourly `vzdump` (all guests, snapshot mode, to iscsi-fast) runs at the top of every hour. Estate rule: check cluster tasks and avoid the :00 windows before any disruptive op (a reboot during vzdump once locked a live-trading CT for 20 h).
- `nats-store-monitor.timer` on node1 polls `/jsz` on all three nodes every 15 min and alerts at >80% of 15 GB; the last runs show 0%. It only checks store bytes — not version, quorum, leader, or consumer lag.
- Cross-node `estate-deadman` watchdog exists (pings node1/2/3), unrelated to NATS health.

[15] CLAIM: Roll followers first, meta leader last: nats2 → nats1 → nats3; freeze stream/consumer creates; avoid PVE `:00` `vzdump`; copy the 2.14.2 binary aside and tar `/var/lib/nats/js` on the node before replacing it.
POINTER: live-state.md:10
OPENED TEXT:
All 3 LXCs: unprivileged, nesting=1, 2c/2G/30G, onboot=1, startup order=1. JetStream store `/var/lib/nats/js` uses ~0.5–1.1 MB of a 15 GB cap. The binary was installed by `install-nats.sh` (see evidence), which fetches GitHub **latest** with no version pin; the binary has not changed since Jun 22. There is no apt package, no second copy of the binary, and no documented rollback path on the nodes.

## JetStream (jsz)
api level 4 | api calls 1607 errors 46 | ha_assets 10 | msgs 384 bytes 64097 | meta leader nats3 | last meta snapshot 2026-08-15T20:43:08

[16] CLAIM: Roll followers first, meta leader last: nats2 → nats1 → nats3; freeze stream/consumer creates; avoid PVE `:00` `vzdump`; copy the 2.14.2 binary aside and tar `/var/lib/nats/js` on the node before replacing it.
POINTER: live-state.md:39
OPENED TEXT:
- 2026-08-15 20:31:54 on nats1 and nats3: `[ERR] <lan-address> … authentication error - User "route"` — something on the node1 PVE host tried the cluster-route user on the CLIENT port 4222 (the route password lives in plaintext in each node's config file, in the `routes` URLs).
- 2026-08-15 20:35–20:42: stream `ESTATE > FIX2SCRATCH` with consumers `s2` and `scratch-serve` got leaders — a scratch stream created ad hoc on the production spine during some fix session; it is not in the current stream list (deleted since).
- 2026-08-15 06:03: nats3 became meta leader ("Self is new JetStream cluster metadata leader") after a node event; the last meta snapshot is from 2026-08-15 20:43.
- No restarts of nats.service on any node since each CT's last boot (NRestarts=0).

[17] CLAIM: `systemctl is-active` is not “back and current”; gate the next node from a **surviving** `:8222` with a 2s poll, 3 consecutive passes, 120s hard stop.
POINTER: live-state.md:13-21
OPENED TEXT:
## JetStream (jsz)
api level 4 | api calls 1607 errors 46 | ha_assets 10 | msgs 384 bytes 64097 | meta leader nats3 | last meta snapshot 2026-08-15T20:43:08
- STREAM JOBS: subjects=['jobs.>'] retention=workqueue max_age=7d max_bytes=5368709120 discard=old R3 storage=file | msgs=0 bytes=0 leader=nats3
    - consumer clickup-ingest: filter=jobs.clickup.events ack_wait=120s max_deliver=5 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=1
    - consumer embed-workers: filter=jobs.embed.batch ack_wait=600s max_deliver=5 max_ack_pending=16 pending=0 redelivered=0 waiting_pulls=0
    - consumer hale-eval-workers: filter=jobs.hale.eval ack_wait=300s max_deliver=4 max_ack_pending=8 pending=0 redelivered=0 waiting_pulls=0
    - consumer rtx-serve: filter=jobs.serve.> ack_wait=30s max_deliver=-1 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=0
- STREAM RESULTS: subjects=['results.>'] retention=limits max_age=3d max_bytes=2147483648 discard=old R3 storage=file | msgs=380 bytes=63214 leader=nats1
    - consumer hale-eval-collector: filter=results.hale.eval ack_wait=60s max_deliver=-1 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=0
- STREAM EVENTS: subjects=['e

[18] CLAIM: `systemctl is-active` is not “back and current”; gate the next node from a **surviving** `:8222` with a 2s poll, 3 consecutive passes, 120s hard stop.
POINTER: live-state.md:26-28
OPENED TEXT:
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2
- via nats1: client_ip=<lan-address> authorized_user=control name=hale-support lang=nats.js lib=2.29.3 subs=1
- via nats2: client_ip=<lan-address> authorized_user=control name=hale-support-events lang=nats.js lib=2.29.3 subs=2

(client .13 = CT 201 "halebrain" on the RTX inference box, python nats.py 2.15.0; client .16 = CT 116 "hale-support" desk, nats.js 2.29.3. BOTH authenticate as the full-privilege `control` user, although least-privilege users `rtx`/`arm` exist in the config and a `support` user was DESIGNED in the HAL repo doc `docs/NATS-SUPPORT-USER.md` but never added to `nats-server.conf`.)

[19] CLAIM: `systemctl is-active` is not “back and current”; gate the next node from a **surviving** `:8222` with a 2s poll, 3 consecutive passes, 120s hard stop.
POINTER: live-state.md:10
OPENED TEXT:
All 3 LXCs: unprivileged, nesting=1, 2c/2G/30G, onboot=1, startup order=1. JetStream store `/var/lib/nats/js` uses ~0.5–1.1 MB of a 15 GB cap. The binary was installed by `install-nats.sh` (see evidence), which fetches GitHub **latest** with no version pin; the binary has not changed since Jun 22. There is no apt package, no second copy of the binary, and no documented rollback path on the nodes.

## JetStream (jsz)
api level 4 | api calls 1607 errors 46 | ha_assets 10 | msgs 384 bytes 64097 | meta leader nats3 | last meta snapshot 2026-08-15T20:43:08

[20] CLAIM: After each restart, Raft catch-up, possible stream/meta leader moves, and ack-wait redelivery are expected; with 4 connections this is not a reconnect storm; duplicate desk events are possible because `support-intake` has `max_deliver=-1`, while loss of the NATS path is covered by the 10-minute poll.
POINTER: live-state.md:22
OPENED TEXT:
- STREAM EVENTS: subjects=['events.support.>', 'events.slack.>'] retention=limits max_age=30d max_bytes=-1 discard=old R3 storage=file | msgs=4 bytes=883 leader=nats1
    - consumer support-intake: filter=events.support.intake ack_wait=30s max_deliver=-1 max_ack_pending=1 pending=0 redelivered=0 waiting_pulls=0

## Live client connections (connz?auth=true on all 3 nodes) — NOTE the authorized_user column
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2

[21] CLAIM: After each restart, Raft catch-up, possible stream/meta leader moves, and ack-wait redelivery are expected; with 4 connections this is not a reconnect storm; duplicate desk events are possible because `support-intake` has `max_deliver=-1`, while loss of the NATS path is covered by the 10-minute poll.
POINTER: live-state.md:25-28
OPENED TEXT:
## Live client connections (connz?auth=true on all 3 nodes) — NOTE the authorized_user column
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2
- via nats1: client_ip=<lan-address> authorized_user=control name=hale-support lang=nats.js lib=2.29.3 subs=1
- via nats2: client_ip=<lan-address> authorized_user=control name=hale-support-events lang=nats.js lib=2.29.3 subs=2

(client .13 = CT 201 "halebrain" on the RTX inference box, python nats.py 2.15.0; client .16 = CT 116 "hale-support" desk, nats.js 2.29.3. BOTH authenticate as the full-privilege `control` user, although least-privilege users `rtx`/`arm` exist in the config and a `support` user was DESIGNED in the HAL repo doc `docs/NATS-SUPPORT-USER.md` but never added to `nats-server.conf`.)

[22] CLAIM: After each restart, Raft catch-up, possible stream/meta leader moves, and ack-wait redelivery are expected; with 4 connections this is not a reconnect storm; duplicate desk events are possible because `support-intake` has `max_deliver=-1`, while loss of the NATS path is covered by the 10-minute poll.
POINTER: live-state.md:49
OPENED TEXT:
## Who depends on the spine right now
- CT 116 hale-support (durable `support-intake` on EVENTS; `clickup-ingest`-adjacent) — the LIVE support desk (email → tickets). NATS is a freshness path; CT116 falls back to 10-min polling if NATS is down.
- CT 201 halebrain — `embed-workers`, `hale-eval-workers`/`collector`, `clickup-ingest` (HAL ingestion). Batch/async; restart-survivable by design.
- Nothing else. GPU workers (`rtx`/`arm` users) are not yet wired in.

[23] CLAIM: Pin the installer to `v2.14.5`; do not run `install-nats.sh` as written; filestore downgrade 2.14.5→2.14.2 without restoring `js` is SPECULATIVE — rollback is saved 2.14.2 binary plus the pre-node `js` tarball, then catch-up from the two still-old peers.
POINTER: install-nats.sh:6-10
OPENED TEXT:
apt-get install -y -qq curl ca-certificates >/dev/null 2>&1
NV=$(curl -fsSL https://api.github.com/repos/nats-io/nats-server/releases/latest | grep '"tag_name"' | head -1 | sed -E 's/.*"(v[^"]+)".*/\1/')
cd /tmp
curl -fsSL "https://github.com/nats-io/nats-server/releases/download/${NV}/nats-server-${NV}-linux-amd64.tar.gz" -o n.tgz
tar xzf n.tgz
install -m755 nats-server-*-linux-amd64/nats-server /usr/local/bin/nats-server
id nats >/dev/null 2>&1 || useradd --system --home-dir /var/lib/nats --shell /usr/sbin/nologin nats
mkdir -p /var/lib/nats/js /etc/nats && chown -R nats:nats /var/lib/nats
echo "nats-server $(/usr/local/bin/nats-server --version)"

[24] CLAIM: Pin the installer to `v2.14.5`; do not run `install-nats.sh` as written; filestore downgrade 2.14.5→2.14.2 without restoring `js` is SPECULATIVE — rollback is saved 2.14.2 binary plus the pre-node `js` tarball, then catch-up from the two still-old peers.
POINTER: live-state.md:10
OPENED TEXT:
All 3 LXCs: unprivileged, nesting=1, 2c/2G/30G, onboot=1, startup order=1. JetStream store `/var/lib/nats/js` uses ~0.5–1.1 MB of a 15 GB cap. The binary was installed by `install-nats.sh` (see evidence), which fetches GitHub **latest** with no version pin; the binary has not changed since Jun 22. There is no apt package, no second copy of the binary, and no documented rollback path on the nodes.

## JetStream (jsz)
api level 4 | api calls 1607 errors 46 | ha_assets 10 | msgs 384 bytes 64097 | meta leader nats3 | last meta snapshot 2026-08-15T20:43:08

[25] CLAIM: Pin the installer to `v2.14.5`; do not run `install-nats.sh` as written; filestore downgrade 2.14.5→2.14.2 without restoring `js` is SPECULATIVE — rollback is saved 2.14.2 binary plus the pre-node `js` tarball, then catch-up from the two still-old peers.
POINTER: v2.14.5.md:5
OPENED TEXT:
Refer to the [2.14 Upgrade Guide](https://docs.nats.io/release-notes/whats_new/whats_new_214) for backwards compatibility notes with 2.12.x. Please note that the 2.13.x version was skipped.

### Go Version

- 1.26.5

[26] CLAIM: Pin the installer to `v2.14.5`; do not run `install-nats.sh` as written; filestore downgrade 2.14.5→2.14.2 without restoring `js` is SPECULATIVE — rollback is saved 2.14.2 binary plus the pre-node `js` tarball, then catch-up from the two still-old peers.
POINTER: v2.14.3.md:80
OPENED TEXT:
- Peer state decoding now bounds peer ID reads to the buffer length (#8310)
- Counter stream staging no longer corrupts the committed running total (#8311)
- Filestore compaction no longer corrupts compressed or encrypted blocks (#8312)
- Memory store `NumPending` no longer overcounts for `DeliverLastPerSubject` consumers (#8313)
- Consumer inactive-delete grace period handling and pull request `MaxBytes` budgeting have been fixed (#8314)

[27] CLAIM: Pin the installer to `v2.14.5`; do not run `install-nats.sh` as written; filestore downgrade 2.14.5→2.14.2 without restoring `js` is SPECULATIVE — rollback is saved 2.14.2 binary plus the pre-node `js` tarball, then catch-up from the two still-old peers.
POINTER: v2.14.4.md:56-59
OPENED TEXT:
- Filestore blocks with unsynced or truncated key files are now removed and counted as lost data instead of failing to recover altogether (#8365)
- Filestore encryption key files are now synced to disk more aggressively (#8366)
- Raft now handles the append entry iterator returning no more entries correctly (#8372)
- Fixed string ownership when handling the expected last sequence per subject in a batch (#8377)
- Fixed a race condition between concurrent message removals via limits that could unexpectedly disable writes into a filestore (#8378)
- Fixed a bug in the filestore which could prevent some filestore block cache references from being weakened correctly, which could result in unexpected memory usage and GC pressure (#8380)
- Attempting to update the consumer storage type now correctly returns an error (#8382)
- Stream publish checks will now correctly reject messages that exceed the maximum store size before proposal (#8389)

[28] CLAIM: A mixed 2.14.2/2.14.5 cluster after 1 of 3 is acceptable for minutes (the intended roll), not for hours/days; if the new binary fails to start, roll back **that node only** and do not touch the other two.
POINTER: v2.14.5.md:29
OPENED TEXT:
- Fix a bug that could result in potential data loss when handling idempotent stream creates when an offline node catches up from a metalayer snapshot, caused by an incorrect update to the create time in the stream assignment (#8449)

### Complete Changes

[29] CLAIM: A mixed 2.14.2/2.14.5 cluster after 1 of 3 is acceptable for minutes (the intended roll), not for hours/days; if the new binary fails to start, roll back **that node only** and do not touch the other two.
POINTER: live-state.md:13
OPENED TEXT:
## JetStream (jsz)
api level 4 | api calls 1607 errors 46 | ha_assets 10 | msgs 384 bytes 64097 | meta leader nats3 | last meta snapshot 2026-08-15T20:43:08
- STREAM JOBS: subjects=['jobs.>'] retention=workqueue max_age=7d max_bytes=5368709120 discard=old R3 storage=file | msgs=0 bytes=0 leader=nats3
    - consumer clickup-ingest: filter=jobs.clickup.events ack_wait=120s max_deliver=5 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=1
    - consumer embed-workers: filter=jobs.embed.batch ack_wait=600s max_deliver=5 max_ack_pending=16 pending=0 redelivered=0 waiting_pulls=0

[30] CLAIM: Highest real risk to the LIVE desk in the captured state is both CT 116 and CT 201 authenticating as `control` (`publish/subscribe allow [">"]`); add `support` and move clients only **after** the binary roll — do not bundle.
POINTER: live-state.md:26-30
OPENED TEXT:
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2
- via nats1: client_ip=<lan-address> authorized_user=control name=hale-support lang=nats.js lib=2.29.3 subs=1
- via nats2: client_ip=<lan-address> authorized_user=control name=hale-support-events lang=nats.js lib=2.29.3 subs=2

(client .13 = CT 201 "halebrain" on the RTX inference box, python nats.py 2.15.0; client .16 = CT 116 "hale-support" desk, nats.js 2.29.3. BOTH authenticate as the full-privilege `control` user, although least-privilege users `rtx`/`arm` exist in the config and a `support` user was DESIGNED in the HAL repo doc `docs/NATS-SUPPORT-USER.md` but never added to `nats-server.conf`.)

Observed config-vs-doctrine deviations (for seats to weigh, not conclusions):
- The build doctrine said "explicit stream limits or silent disk-full". Stream EVENTS has `max_bytes=-1` (unlimited) with max_age 30d; the 15 GB `max_file_store` is the only backstop.

[31] CLAIM: Highest real risk to the LIVE desk in the captured state is both CT 116 and CT 201 authenticating as `control` (`publish/subscribe allow [">"]`); add `support` and move clients only **after** the binary roll — do not bundle.
POINTER: live-state.md:30
OPENED TEXT:
(client .13 = CT 201 "halebrain" on the RTX inference box, python nats.py 2.15.0; client .16 = CT 116 "hale-support" desk, nats.js 2.29.3. BOTH authenticate as the full-privilege `control` user, although least-privilege users `rtx`/`arm` exist in the config and a `support` user was DESIGNED in the HAL repo doc `docs/NATS-SUPPORT-USER.md` but never added to `nats-server.conf`.)

Observed config-vs-doctrine deviations (for seats to weigh, not conclusions):
- The build doctrine said "explicit stream limits or silent disk-full". Stream EVENTS has `max_bytes=-1` (unlimited) with max_age 30d; the 15 GB `max_file_store` is the only backstop.

[32] CLAIM: Highest real risk to the LIVE desk in the captured state is both CT 116 and CT 201 authenticating as `control` (`publish/subscribe allow [">"]`); add `support` and move clients only **after** the binary roll — do not bundle.
POINTER: live-state.md:22
OPENED TEXT:
- STREAM EVENTS: subjects=['events.support.>', 'events.slack.>'] retention=limits max_age=30d max_bytes=-1 discard=old R3 storage=file | msgs=4 bytes=883 leader=nats1
    - consumer support-intake: filter=events.support.intake ack_wait=30s max_deliver=-1 max_ack_pending=1 pending=0 redelivered=0 waiting_pulls=0

## Live client connections (connz?auth=true on all 3 nodes) — NOTE the authorized_user column
- via nats1: client_ip=<lan-address> authorized_user=control name=None lang=python3 lib=2.15.0 subs=2

[33] CLAIM: Highest real risk to the LIVE desk in the captured state is both CT 116 and CT 201 authenticating as `control` (`publish/subscribe allow [">"]`); add `support` and move clients only **after** the binary roll — do not bundle.
POINTER: live-state.md:21
OPENED TEXT:
- consumer hale-eval-collector: filter=results.hale.eval ack_wait=60s max_deliver=-1 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=0
- STREAM EVENTS: subjects=['events.support.>', 'events.slack.>'] retention=limits max_age=30d max_bytes=-1 discard=old R3 storage=file | msgs=4 bytes=883 leader=nats1
    - consumer support-intake: filter=events.support.intake ack_wait=30s max_deliver=-1 max_ack_pending=1 pending=0 redelivered=0 waiting_pulls=0

## Live client connections (connz?auth=true on all 3 nodes) — NOTE the authorized_user column

[34] CLAIM: Highest real risk to the LIVE desk in the captured state is both CT 116 and CT 201 authenticating as `control` (`publish/subscribe allow [">"]`); add `support` and move clients only **after** the binary roll — do not bundle.
POINTER: nats-store-monitor.sh:2-9
OPENED TEXT:
#!/usr/bin/env bash
# Alert if any estate-spine node's JetStream file store exceeds 80% of its 15GB limit.
LIMIT=$((15*1024*1024*1024)); THRESH=80; LOG=/var/log/nats-monitor.log
for n in <lan-address> <lan-address> <lan-address>; do
  used=$(curl -s --max-time 6 "http://$n:8222/jsz" | python3 -c 'import sys,json;print(json.load(sys.stdin).get("store",0))' 2>/dev/null)
  [ -z "$used" ] && { echo "$(date -u +%FT%TZ) WARN $n unreachable" | tee -a "$LOG"; continue; }
  pct=$(( used*100/LIMIT ))
  msg="$(date -u +%FT%TZ) $n store ${pct}% ($((used/1024/1024))MB/15GB)"
  if [ "$pct" -ge "$THRESH" ]; then echo "$msg ALERT>=${THRESH}%" | tee -a "$LOG"; else echo "$msg" >> "$LOG"; fi
done

[35] CLAIM: Highest real risk to the LIVE desk in the captured state is both CT 116 and CT 201 authenticating as `control` (`publish/subscribe allow [">"]`); add `support` and move clients only **after** the binary roll — do not bundle.
POINTER: live-state.md:38
OPENED TEXT:
## Recent journal signals (from `journalctl -u nats` on the nodes)
- 2026-08-15 20:31:54 on nats1 and nats3: `[ERR] <lan-address> … authentication error - User "route"` — something on the node1 PVE host tried the cluster-route user on the CLIENT port 4222 (the route password lives in plaintext in each node's config file, in the `routes` URLs).
- 2026-08-15 20:35–20:42: stream `ESTATE > FIX2SCRATCH` with consumers `s2` and `scratch-serve` got leaders — a scratch stream created ad hoc on the production spine during some fix session; it is not in the current stream list (deleted since).
- 2026-08-15 06:03: nats3 became meta leader ("Self is new JetStream cluster metadata leader") after a node event; the last meta snapshot is from 2026-08-15 20:43.
- No restarts of nats.service on any node since each CT's last boot (NRestarts=0).

[36] CLAIM: Highest real risk to the LIVE desk in the captured state is both CT 116 and CT 201 authenticating as `control` (`publish/subscribe allow [">"]`); add `support` and move clients only **after** the binary roll — do not bundle.
POINTER: live-state.md:35
OPENED TEXT:
- Consumers `rtx-serve`, `hale-eval-collector`, `support-intake` have `max_deliver=-1` (infinite redelivery).
- `rtx-serve` (filter jobs.serve.>) has never had a live subscriber in this snapshot; it is a demo consumer from build day.

## Recent journal signals (from `journalctl -u nats` on the nodes)
- 2026-08-15 20:31:54 on nats1 and nats3: `[ERR] <lan-address> … authentication error - User "route"` — something on the node1 PVE host tried the cluster-route user on the CLIENT port 4222 (the route password lives in plaintext in each node's config file, in the `routes` URLs).

[37] CLAIM: Newsletter items the owner ignored (OpenTelemetry collector, agent-SDK session store, “coordinating AI agents”, file-vs-memory KV mirror) are correctly ignored; seed designation, credentials, and client reconnect config must not change in this change-set.
POINTER: live-state.md:15-22
OPENED TEXT:
- STREAM JOBS: subjects=['jobs.>'] retention=workqueue max_age=7d max_bytes=5368709120 discard=old R3 storage=file | msgs=0 bytes=0 leader=nats3
    - consumer clickup-ingest: filter=jobs.clickup.events ack_wait=120s max_deliver=5 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=1
    - consumer embed-workers: filter=jobs.embed.batch ack_wait=600s max_deliver=5 max_ack_pending=16 pending=0 redelivered=0 waiting_pulls=0
    - consumer hale-eval-workers: filter=jobs.hale.eval ack_wait=300s max_deliver=4 max_ack_pending=8 pending=0 redelivered=0 waiting_pulls=0
    - consumer rtx-serve: filter=jobs.serve.> ack_wait=30s max_deliver=-1 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=0
- STREAM RESULTS: subjects=['results.>'] retention=limits max_age=3d max_bytes=2147483648 discard=old R3 storage=file | msgs=380 bytes=63214 leader=nats1
    - consumer hale-eval-collector: filter=results.hale.eval ack_wait=60s max_deliver=-1 max_ack_pending=1000 pending=0 redelivered=0 waiting_pulls=0
- STREAM EVENTS: subjects=['events.support.>', 'events.slack.>'] retention=limits max_age=30d max_bytes=-1 discard=old R3 storage=file | msgs=4 bytes=883 leader=nats1
    - consumer supp

[38] CLAIM: Newsletter items the owner ignored (OpenTelemetry collector, agent-SDK session store, “coordinating AI agents”, file-vs-memory KV mirror) are correctly ignored; seed designation, credentials, and client reconnect config must not change in this change-set.
POINTER: install-nats.sh:6
OPENED TEXT:
apt-get install -y -qq curl ca-certificates >/dev/null 2>&1
NV=$(curl -fsSL https://api.github.com/repos/nats-io/nats-server/releases/latest | grep '"tag_name"' | head -1 | sed -E 's/.*"(v[^"]+)".*/\1/')
cd /tmp
curl -fsSL "https://github.com/nats-io/nats-server/releases/download/${NV}/nats-server-${NV}-linux-amd64.tar.gz" -o n.tgz
tar xzf n.tgz

[39] CLAIM: Newsletter items the owner ignored (OpenTelemetry collector, agent-SDK session store, “coordinating AI agents”, file-vs-memory KV mirror) are correctly ignored; seed designation, credentials, and client reconnect config must not change in this change-set.
POINTER: live-state.md:6
OPENED TEXT:
|---|---|---|---|---|---|---|---|---|
| nats1 | CT 121 | node1 (zfs subvol) | /usr/local/bin/nats-server (17968728 B, dated Jun 22) | v2.14.2 | 10349df4… | 2026-06-22 03:31 UTC | 0 | Debian 12 |
| nats2 | CT 122 | node2 (lvm) | same binary/hash | v2.14.2 | 10349df4… | 2026-07-26 21:07 UTC (CT rebooted then) | 0 | Debian 12 |
| nats3 | CT 123 | node3 (lvm), seed | same binary/hash | v2.14.2 | 10349df4… | 2026-07-28 13:28 UTC (CT rebooted then) | 0 | Debian 12 |

[40] CLAIM: Newsletter items the owner ignored (OpenTelemetry collector, agent-SDK session store, “coordinating AI agents”, file-vs-memory KV mirror) are correctly ignored; seed designation, credentials, and client reconnect config must not change in this change-set.
POINTER: live-state.md:8
OPENED TEXT:
| nats2 | CT 122 | node2 (lvm) | same binary/hash | v2.14.2 | 10349df4… | 2026-07-26 21:07 UTC (CT rebooted then) | 0 | Debian 12 |
| nats3 | CT 123 | node3 (lvm), seed | same binary/hash | v2.14.2 | 10349df4… | 2026-07-28 13:28 UTC (CT rebooted then) | 0 | Debian 12 |

All 3 LXCs: unprivileged, nesting=1, 2c/2G/30G, onboot=1, startup order=1. JetStream store `/var/lib/nats/js` uses ~0.5–1.1 MB of a 15 GB cap. The binary was installed by `install-nats.sh` (see evidence), which fetches GitHub **latest** with no version pin; the binary has not changed since Jun 22. There is no apt package, no second copy of the binary, and no documented rollback path on the nodes.

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }
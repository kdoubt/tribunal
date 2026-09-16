# Decision G08 - a 3-node NATS JetStream spine: act on the vendor newsletter and upgrade 2.14.2 -> 2.14.5?

## Artifact(s)

All under `./artifact/` (sanitized, no secrets; read them directly; cite `file:line`):

- `release-notes-v2.14.3.md`, `release-notes-v2.14.4.md`,
  `release-notes-v2.14.5.md` - the official release bodies (EXTERNAL, primary source).
- `live-state.md` - captured cluster state: node inventory, binary provenance,
  streams, consumers, live client connections with authorized user, journal
  signals, windows.
- `nats-server.conf.sanitized` - node 1's config (identical structure on all nodes).
- `nats.service` - the systemd unit.
- `install-nats.sh` - how the binary was installed.
- `nats-store-monitor.sh` - the only NATS-specific monitoring that exists.

Context (USER-FACT): a 3-node NATS JetStream cluster "estate-spine" (R3,
native nats-server v2.14.2 under systemd, in three unprivileged Debian LXCs,
one per Proxmox node) is the estate's async job/event bus. Consumers today: a
LIVE customer support desk (CT 116, durable consumer on stream EVENTS, with a
10-minute polling fallback) and a knowledge-base ingestion/eval pipeline
(CT 201, work-queue consumers on JOBS/RESULTS). Volume is tiny (384 msgs,
64 KB). Build doctrines: cluster from day one, explicit stream limits,
accounts plus least-privilege users per workload, boot order so the spine
precedes workers, monitor store usage.

The August 2026 vendor newsletter arrived. The owner's pre-review read:
the only action item is that nats-server is three patch releases behind
(2.14.2 -> 2.14.5), and the release notes cover auth/permission CVEs, Raft
election fixes, and a "data loss when an offline node catches up from a
metalayer snapshot" fix, all of which map onto the R3 + least-privilege
design. Other newsletter items (an OpenTelemetry collector, an agent-SDK
session store, "coordinating AI agents on NATS", a file-store-vs-memory-store
KV-mirror discussion) were judged not actionable. This review exists to
CHALLENGE that read and to design the upgrade.

## Question under review

- **Q1 Should we upgrade to 2.14.5, and how?** Is the upgrade justified by the
  release notes for THIS deployment (R3, bcrypt accounts, no MQTT/leafnodes/
  JWT/TLS)? Name the specific fixed issues that actually apply and any the
  owner over-weighted. Then specify the rolling procedure: node order
  (followers first? meta leader last?), pre/post checks per node, what
  "healthy" means before the next node (not just `active`), how to pin the
  version, and the rollback path (is a 2.14.5 -> 2.14.2 downgrade of the
  file store safe? If you cannot establish this from the notes, say
  SPECULATIVE and name the discriminating test).
- **Q2 What ELSE in the newsletter or the captured state should we act on?**
  Include the observed deviations in `live-state.md`. Rank by real risk to a
  LIVE support desk. Say which newsletter items are correctly ignored.
- **Q3 Operational readiness of the upgrade (A1-A5, mandatory):**
  - A1: for each restarted node, what predicate, poll interval, timeout, and
    retry decide "back and current in the Raft group" before the next node?
  - A2: what is eventually consistent after a restart (Raft catch-up,
    leader elections, redelivery of in-flight acks)? Which clients see
    redeliveries or reconnect storms, and is that harmless?
  - A3: how to verify genuine cluster health (quorum, all peers `current`, no
    `pending` Raft entries, streams/consumers have leaders, clients
    reconnected) from a vantage point that is not the restarted node?
  - A4: every store/actor that must change: the binary on 3 nodes, the
    installer script (pin), any docs recording the version, the store
    monitor, the seed node designation, credentials, client reconnect config.
  - A5: if the operator is interrupted after node 1 of 3, is a mixed
    2.14.2/2.14.5 cluster safe to leave for hours or days? If a node fails to
    start on the new binary, the exact rollback step.

## Precedent (contested evidence)

- "Active is not healthy": `systemctl is-active` proves nothing about serving.
- A reboot during the hourly backup locked a live-trading CT for 20 h; check
  cluster tasks and avoid :00 windows before any disruptive op.
- Co-resident writers with no lock caused a lost update; applies to any shared
  flag an upgrade script leaves behind.
- An internal doc already prescribes the safe rollout for CONFIG changes:
  validate with `nats-server -t`, reload one node at a time, verify health
  between each. No equivalent recipe exists for BINARY upgrades.
- The June build verified HA on 2.14.2: stop the meta leader -> another node
  elected, streams survived on 2/3, publish+pull+ack worked with a node down,
  the node rejoined. That was before the 2.14.5 offline-node catch-up fix.
- Searched for a prior nats-server upgrade runbook in the estate: none.

## Decision criteria (owner-supplied)

(1) the LIVE support desk never loses or duplicates a ticket event and the
spine never loses quorum unattended; (2) the security fixes land; (3) the
procedure is boring, repeatable, and leaves a pinned, documented state; (4)
the simplest thing that achieves 1-3. Tie-break: any step that can leave the
cluster without quorum or a client silently disconnected outranks all
convenience; a security fix outranks "wait and see".

## Constraints

- Binary-only upgrade path (tarball from GitHub releases, `install -m755`);
  no docker, no apt repo. Operator runs as root on the hosts via `pct exec`.
  The store is ~1 MB, so backup/restore of `/var/lib/nats/js` is trivial.
- The monitor port :8222 is reachable from the hosts (read-only JSON).
- Auth changes (adding a `support` user, moving clients off `control`) need a
  config reload on all three nodes and credential delivery to CT 116/201: a
  separate change; you may recommend sequencing but do not bundle it unless
  you argue why bundling is safer.

## Output contract

Maximum 8 claims, each as:

```
CLAIM: <one sentence>
EVIDENCE: <artifact file:line, verbatim span, or the exact probe that would settle it; else ASSUMPTION / SPECULATIVE (name it) / EXTERNAL (source)>
CONFIDENCE: <0-1 probability, calibrated>
FALSIFIER: <what concrete observation would prove this claim wrong>
```

Plus **VERDICT INPUT**: one line per question (proceed-as-read /
proceed-with-changes / do-not-upgrade-yet) and the single highest-severity
issue overall. Maximum 1200 words.

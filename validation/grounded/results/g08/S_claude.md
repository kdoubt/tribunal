## Framing

Alternatives considered: (a) *patch-hygiene* — "three behind, apply it"; (b) *fix-applicability audit* — does anything in 2.14.3–5 touch **this** deployment; (c) *the upgrade is the small risk; the captured state is the big one*. I pick (b)+(c) jointly: the applicability audit changes *why* you upgrade (and reorders the procedure), and `live-state.md` contains a larger, cheaper-to-fix exposure than any CVE in the notes.

---

```
CLAIM: The upgrade is justified, but almost none of the auth/permission CVEs the owner cited apply; the real justification is Raft/metalayer correctness.
EVIDENCE: Non-applicable: "Fixed an authentication bypass with TLS `verify_and_map` authenticating users with blank passwords" (v2.14.4:39 — no TLS in nats-server.conf.sanitized:1-41); all MQTT items (v2.14.3:55-59, v2.14.4:71-76); "`NoAuthUser` now checks connection restrictions" / "Leaf connections..." (v2.14.3:48-49); JWT/account-claims items (v2.14.3:40-41, v2.14.4:36 — static accounts, conf:24-41). Applicable: "Raft nodes no longer participate in voting or candidacy after write errors (#8290)" (v2.14.3:72), "Raft `ApplyCommit` now handles the post-snapshot index correctly (#8321)" (v2.14.3:74), "Raft elections now correctly ignore votes from removed peers (#8353)" (v2.14.4:54), "#8370 ... stale proposals from a previous term" (v2.14.4:67).
CONFIDENCE: 0.85
FALSIFIER: A `tls{}`, `mqtt{}`, `leafnodes{}` or `no_auth_user` block on nats2/nats3 (artifact asserts "identical structure on all nodes" — unverified for 2 of 3).
```
```
CLAIM: The single strongest reason to go to 2.14.5 is that the rolling upgrade itself is the exact scenario the #8449 data-loss fix covers, and one #8413 hazard has already occurred on this spine.
EVIDENCE: v2.14.5:29 "potential data loss when handling idempotent stream creates when an offline node catches up from a metalayer snapshot"; live-state.md:13 "last meta snapshot 2026-08-15T20:43:08"; every restarted node is an offline node catching up. v2.14.4:68 "#8413 Replicated streams that were recreated while a node was down are no longer treated as an update by a returning node processing a snapshot" ↔ live-state.md:39 ad-hoc `FIX2SCRATCH` stream created and deleted on the production spine.
CONFIDENCE: 0.8
FALSIFIER: Clients never issue idempotent `add_stream` on connect (would need CT116/CT201 source, not in artifact) — then #8449 is latent only.
```
```
CLAIM: Order must be nats2 → nats1 → nats3, not "seed first"; the "seed" label is documentation-only and carries no config meaning.
EVIDENCE: live-state.md:13 "meta leader nats3"; :14/:19/:21 leaders JOBS=nats3, RESULTS=nats1, EVENTS=nats1; client counts from connz — nats1 has 3 (:25-27), nats2 has 1 (:28), nats3 has 0. conf:17-21 lists all three routes including self — no distinguished seed despite live-state.md:8 "node3 (lvm), seed".
CONFIDENCE: 0.8
FALSIFIER: nats2/nats3 configs carry an asymmetric `routes` list (only nats1's config is in the artifact).
```
```
CLAIM: A1/A3 — `systemctl is-active` and even the restarted node's own `/healthz` are insufficient; the gate must be read from a peer that was not restarted.
EVIDENCE: nats-store-monitor.sh:5 polls only `/jsz` `.store` — the only NATS monitoring that exists (script header line 2). Gate predicate, run from the two *other* nodes: `curl -s http://<peer>:8222/jsz?acc=ESTATE&streams=true&consumers=true` → restarted node appears in every stream's `cluster.replicas` with `current:true`, `offline:false`, `lag:0`; plus `/varz` `version` == `2.14.5` and `/healthz` HTTP 200 on the restarted node; plus `connz?auth=true` shows the displaced clients re-established. Poll 5 s, timeout 180 s, no auto-retry — on timeout stop and do not touch node N+1.
CONFIDENCE: 0.85
FALSIFIER: `/jsz` on this build omits per-replica `current`/`lag` (settle by running the probe on 2.14.2 today, pre-upgrade).
```
```
CLAIM: A2 — duplicate ticket events are possible on restart, so decision criterion (1) "never duplicates" is unachievable by the spine and must be met by CT 116 idempotency; additionally `support-intake` can be head-of-line blocked forever by one poison message.
EVIDENCE: live-state.md:22 `support-intake: ... ack_wait=30s max_deliver=-1 max_ack_pending=1`. An in-flight unacked message at leader restart redelivers after 30 s; `max_deliver=-1` never gives up; `max_ack_pending=1` means nothing else is delivered until it acks. Mitigation gate: confirm `pending=0` and `ack_pending=0` on `support-intake` immediately before restarting nats1 (current snapshot: `pending=0 redelivered=0`).
CONFIDENCE: 0.8
FALSIFIER: CT 116 already dedupes on `Nats-Msg-Id`/ticket ID (CT116 code not in artifact).
```
```
CLAIM: A downgrade of the on-disk file store from 2.14.5 back to 2.14.2 is not establishable from the notes — SPECULATIVE — so the rollback path must not depend on it.
EVIDENCE: No format/API-level change is stated in any of the three notes; the only compat pointer is v2.14.5:4 "backwards compatibility notes with 2.12.x". But filestore/encode paths changed: v2.14.3:81, v2.14.4:51 "Decoded AVL sequence set node counts ... on 32-bit systems", v2.14.4:27 "Stream snapshots now attempt to determine the correct encode buffer size up front". Discriminating test: on a throwaway 4th LXC, single-node JetStream, create stream+durable+msgs on 2.14.5, stop, `install -m755` the 2.14.2 binary, start, compare `/jsz` msgs and consumer ack floor. A5 rollback (format-independent): `systemctl stop nats` → restore the saved 2.14.2 binary → `rm -rf /var/lib/nats/js` → restore the `cp -a` copy taken before *that node's* restart → start → re-run the A1 gate from a peer.
CONFIDENCE: 0.75 (that the notes cannot settle it)
FALSIFIER: An explicit "no downgrade" / format-version line in the linked 2.14 Upgrade Guide (EXTERNAL, not in artifact).
```
```
CLAIM: A4 — the highest-value non-binary change is pinning `install-nats.sh`, which today re-installs whatever GitHub calls "latest" and would silently produce a 4th version on any re-run.
EVIDENCE: install-nats.sh:6 `NV=$(curl ... releases/latest ... )`; live-state.md:10 "fetches GitHub **latest** with no version pin ... no documented rollback path on the nodes". Fix: `NV=v2.14.5` literal + `sha256sum -c` against the recorded hash (the estate already records hashes — live-state.md:6 `10349df4…`). Also touch: 3 binaries, `docs/NATS-SUPPORT-USER.md` and live-state (version), nats-store-monitor.sh (add `/varz` version + `/healthz` + meta-leader checks). Unchanged: conf, credentials, seed designation.
CONFIDENCE: 0.9
FALSIFIER: A pinned wrapper or Ansible/pve script elsewhere in the estate supersedes install-nats.sh (search returned "none" for an upgrade runbook, per brief).
```
```
CLAIM: Q2's top risk is not in the newsletter at all: both live workloads authenticate as `control`, which holds publish+subscribe `>` — including full `$JS.API` destructive access to EVENTS/JOBS/RESULTS.
EVIDENCE: live-state.md:25-28 `authorized_user=control` on all four connections, including `name=hale-support`; conf:29-30 `{ user: control ... publish: { allow: [">"] }, subscribe: { allow: [">"] } }`; live-state.md:30 the designed `support` user "never added to `nats-server.conf`". Secondary: the plaintext route password (conf:16-21) was probed against the client port — live-state.md:38 `authentication error - User "route"` from <lan-address>
CONFIDENCE: 0.85
FALSIFIER: An external firewall restricts 4222 to .13/.16 only, bounding blast radius (not shown in artifact).
```

**A5 mixed-version dwell:** leaving 2.14.5/2.14.2/2.14.2 for hours is acceptable — quorum is 3/3, api level is unchanged (live-state.md:13 "api level 4") and the notes state no intra-2.14 incompatibility — but do it only *after* pinning the installer, and don't leave it across a `vzdump` maintenance day. **Correctly ignored newsletter items:** OTel collector, agent-SDK session store, "AI agents on NATS", KV file-vs-memory mirror (no `KV_*` streams exist — live-state.md:14-22). **Do not bundle** the `support`-user auth change with the binary upgrade: it needs a reload on all 3 nodes plus credential delivery, and v2.14.4:37 (queue-sub permission literal handling) means the new least-privilege rules are *better enforced* on 2.14.5 — so sequence auth **after** the upgrade, not with it.

---

**VERDICT INPUT**
- **Q1:** proceed-with-changes — upgrade to 2.14.5, but on Raft/#8449 grounds, not the auth CVEs; order nats2 → nats1 → nats3; per-node `cp -a /var/lib/nats/js` before restart; rollback = restore binary + that node's store copy.
- **Q2:** proceed-with-changes — the `control`-for-everything finding and `support-intake`'s `max_ack_pending=1`/`max_deliver=-1` outrank the patch level; newsletter dismissals are correct.
- **Q3:** proceed-with-changes — A1/A3 gate must be read from a non-restarted peer; A2 duplicates are possible and need CT 116 idempotency; A4 installer pin is mandatory; A5 rollback is store-restore, not downgrade.

**Highest-severity issue overall:** the LIVE support desk (CT 116) holds `>` publish/subscribe as `control` (live-state.md:27-28, conf:29-30) — a single compromised desk can destroy every stream on the spine, which is strictly worse than any issue 2.14.5 fixes.

You are scoring one review memo against a sealed rubric. You are not a participant; do not add your own review.
Score ONLY from the memo text. Quote the memo span that earns each point; if no span supports a point, the score is 0.

=== SEALED RUBRIC ===
# G05 sealed rubric - SEALED (arms never see this)

- **difficulty:** subtle (two oracle-verified holes; one plausible wrong claim was overturned)
- **correct_call:** **harden-first, then onboard.** The RTX island holds; the
  bastion is a real chokepoint for the island; but (a) the Cloudflare identity
  gate is bypassable from the LAN because both SSH entry points accept any
  LAN source, and (b) the shared, unpartitioned GPU is the top production
  fault domain. Plus two verified operational gaps (no healthchecks, a fixed
  `sleep 3`; teardown restores a literal backup-exclusion value).
- **oracle:** orchestrator probes at panel time: `/dev/tcp` from a production
  LAN host reached the bastion's LAN address :22 AND the GB10 host :2222
  (both OPEN); the island SSH address unreachable from the LAN (island holds);
  `docker inspect external-bench .Config.Healthcheck` = NONE; live backup job
  exclude = `114,205`. Artifact lines in must_catch.
- **correct fix:** source-restrict both SSH entry points (bastion :22 only from
  the cloudflared loopback path / a management CIDR; GB10 relay bound
  host-local or firewalled to the bastion); treat the shared GPU as a
  production fault domain with readiness AND liveness checks on window
  open/close; container healthchecks + a readiness predicate instead of
  `sleep 3`; teardown removes 205 from the live exclusion value
  (compare-and-swap) rather than writing `114`.
- **must_catch:**
  1. Access bypass from the LAN: `bastion-nftables.conf:8` accepts
     `tcp dport 22` from any source (comment "cloudflared localhost + LAN
     mgmt"); `ct204.conf` puts eth0 on the production bridge; the GB10 relay
     publishes `2222` on the host's LAN address (`gb10_setup.sh:48-50`;
     `gb10-docker-inspect.txt` line 3 `ports={"2222/tcp":[{"HostIp":...}]}`).
     The RUNBOOK's claim "Identity gate at CF edge BEFORE any packet reaches
     sshd" (`RUNBOOK.md:92`) is false for LAN-origin traffic; only SSH key
     auth remains there.
  2. Shared unpartitioned GPU = production fault domain: `ct205.conf:14-20`
     binds `/dev/nvidia*` with `cgroup2 devices.allow c 195/510`, the same
     nodes CT 201 uses; no MIG; only manual window discipline guards it.
  3. The RTX island is real: `ct205.conf:8` (`bridge=cirub0`, no uplink),
     `sdn.cfg:1-5` (simple zone, one node), `bastion-nftables.conf:19-24`
     (egress only to the island SSH address, the GB10 relay, DNS; all other RFC1918
     dropped).
  4. Asymmetry: the GB10 boundary is composed (internal net + squid ACL
     `gb10-squid-dockerfile.txt:2-10` + socat relay) and its two helper
     containers are dual-homed on the default bridge
     (`gb10-docker-inspect.txt` lines 2-3: `bridge` 172.17.x AND `the external party`),
     so a helper compromise inherits LAN routing.
  5. No readiness gates: `bastion-cloudflared.service` has `Restart=on-failure`
     but nothing verifies tunnel health; `gb10_setup.sh:60-61` is a fixed
     `sleep 3` then a fatal `docker exec`; no container `HEALTHCHECK` appears
     in `gb10-docker-inspect.txt` or `gb10_setup.sh`.
  6. Teardown clobber: `RUNBOOK.md:86` restores the backup exclusion to the
     literal `114`, while the live value is `114,205` (`backup-job.json`);
     any co-tenant excluded mid-engagement loses that exclusion at teardown.
  7. Cache-naive verification: the RUNBOOK's DNS/Access/TLS checks self-query
     (A2/A3); no propagation awareness.
- **landmine (confident wrong answers):** "`gb10_setup.sh` is not idempotent,
  it fails if the network exists" - FALSE: `gb10_setup.sh:38-39` guards the
  create with `docker network inspect ... ||` and `:41` uses `rm -f`
  (this claim was made and overturned by the oracle in the original panel);
  "CF Access makes the SSH endpoints unreachable without identity" (only from
  the WAN); "the bastion is theatre" (its nftables egress drop is the island's
  real chokepoint).
- **unverifiable:**
  - the RUNBOOK in this bundle does not state the dual-homing residual or a
    sshd HEALTHCHECK intent (an earlier draft cited lines past its end); those
    two facts are scored only through the inspect/setup files above;
  - Access policy load-bearing status at review time (policy is owner-only by
    design pre-onboarding).
- **outcome_source:** `2026-08-25-external-bench-access-review/ledger.md` (A3/B2,
  A5, A7, B1, B4 verified; B5 overturned) and `verdict.md` (recommendation
  1-3; surviving dissent = severity ranking only).


=== MEMO (author unknown) ===
# Verdict - g05 gated remote GPU benchmark access for an external party

Built from ledger rows only (ledger.md). Seats A and B; authorship in the ledger.

## 1. Independent agreement (agreed-r0)

The panel concludes, from claims both seats made before cross-exposure:
- The identity gate is load-bearing only on the WAN path: bastion :22 accepts from any source and the GB10 sshd is published on the LAN address, so any LAN peer bypasses Access and faces only `authorized_keys` (A2, B2).
- The two hosts use different isolation models: the RTX side is a no-uplink L2 island behind a bastion whose forward chain drops (a real chokepoint); the GB10 side is docker-policy isolation with a LAN-published listener, the weaker side, where the bastion gates nothing (A4, B1).
- Teardown writes shared mutable stores (backup-job exclude, cluster SDN, and on B7 also docker's published-port rules) by blind read-modify-write; CT 201's nvidia device nodes are simultaneously accessible to CT 205 (A6, B7).
- Readiness: only cloudflared has a restart/readiness story; GB10 bring-up is a fixed `sleep 3` with services attached to the internal network after they start; CT 205 `onboot: 0` vs bastion `onboot: 1` (A7, B5).
- The GPU is shared with production at the device level with no enforcement of exclusivity, and both seats rank that as the top-severity class (A1, B3).

## 2. Resolved after Round 0

**Oracle-settled (`verified`):**
- Every file:line pointer from both seats opens and matches (oracles.md); one off-by-one (`ct205.conf:10` -> :9 for `onboot: 0`).
- A5 narrowed: `gb10_setup.sh:68, :69, :71` cannot fail the build (`|| true` / `|| echo good`), but :70 has no `|| true` and can exit non-zero under `set -e`.

**Cross-examination-settled (`conceded` / revised):**
- A conceded B3's distinct gap: the documented window close never detaches the GPU on either host and `docker update --memory 20g` governs host RAM, not VRAM; A1 revised to two enforcement gaps (0.92).
- A conceded B4: 172.30.99.1 and host-published ports are an untested reachability surface from the external party-bench; A added the dual-homed relay/proxy (172.17.0.x) and 169.254/100.64/IPv6 ACL gaps; B revised to 0.80.
- A conceded B8's core: teardown omits restarting CT 201's vLLM and GB10's three vLLMs; PBS snapshots of CT 204 retain the guest pubkey. A added: a setup re-run's `docker rm -f` destroys an in-flight benchmark.
- A conceded B6's conclusion (no cache-proof DNS/Access/TLS validation recorded) while rejecting two of its supports.
- B conceded A3 as a ruleset fact (IPv4-only denylists) and narrowed it: CT 205's block is family-agnostic; the gap matters on the GB10/squid side; 169.254.0.0/16 is an IPv4 hole in both ACLs.
- B conceded A8's fact and A revised materiality to 0.3 (token dead after tunnel delete; residue = pubkey and host keys).
- B conceded A7's facts and reframed `onboot: 0` as GPU-safety; A accepted the :9 pointer.

## 3. Surviving dissent

- **Framing of the highest-severity finding.** A: hostile root-level ioctl on shared `/dev/nvidia*` on both hosts plus no window-close detach (0.92; kernel escape only 0.25). B: GPU exclusivity is a runbook convention, not enforcement, with the external party-bench `--gpus all`/`unless-stopped` outside windows (0.90; explicitly not a kernel escape). Same class, different mechanism named first. Cheapest discriminating test: none needed for the ranking (both rank it first); to settle the escape sub-claim, a driver-fault test from CT 205 while CT 201's vLLM is up would show whether the outage is VRAM/driver-level only.
- **Q3 label.** A: don't-ship-as-written (window close restores neither exclusivity nor prod vLLMs; blind RMW; no assertion can fail). B: harden-first with the same change list. Substance converged; the label is an owner's call.
- **Confidence on B6 and B8.** A puts B6 at ~0.6 (two supports rejected: session_duration is revocation latency; no competing DOCKER-USER writer) and B8 at ~0.8 (images and DOCKER-USER are noise; `from=` decorative in CT 205); B holds 0.86 and 0.91. Test: none mechanical; calibration dispute.
- **A3 exploitability.** IPv6 presence on either segment is unverifiable in the artifact (0.6, unchanged); B notes GB10 inspect shows empty IPv6 fields.

## Recommendation

**Mode:** `dont` (as written) / `harden-first` - both seats agree the shape is right and the RTX island is real, and both agree the build must not run a window as configured: the GPU is shared at the device level with no exclusivity enforcement and no close-time detach (A1, B3), the GB10 sshd is on the LAN and the bastion is WAN-only gating (A2, B2, B1), the GB10 verify block cannot fail on its isolation assertions (A5), window close does not restore production vLLMs and teardown RMWs shared stores (A6, B7, B8). Owner picks the label (A: don't-as-written; B: harden-first); the required changes are the same set.

## Record

- Open (unexamined, not endorsed): none.
- Verified vs merely agreed: verified = every cited config fact (oracles.md), A5 narrowing; agreed-only = consequence claims (crashloop, kernel escape 0.25, materiality of PBS residue, reachability of 172.30.99.1).
- Rounds run / stop rule hit: R0+R1; stopped: no overturned claim, no load-bearing flip, no new claim IDs.
- Transformations applied to relayed text: neutral Peer labels, order shuffle, ORCH-SUMMARY ledger note (settled items + oracle results); nothing else.

VERDICT INPUT: Q1 Architecture = harden-first (shape right; RTX island real; GB10 docker-policy side must reach parity or be accepted in writing as weaker) ; Q2 Security = harden-first (bind :2222 off the LAN, source-restrict bastion :22, `from=` keys, enforce GPU exclusivity with a close-time detach) ; Q3 Systems/operational = don't-ship-as-written (A) / harden-first (B) with one converged change list (fail-closed verify, single-writer RMW for exclude and SDN, exclude 204 or purge, start/stop 205 with the window, restore vLLMs at close and teardown) ; Highest-severity finding = production GPUs shared at the device level with a sudo-holding guest and no exclusivity enforcement or close-time detach - co-tenant vLLM outage by construction, unaffected by any network control.


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}
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

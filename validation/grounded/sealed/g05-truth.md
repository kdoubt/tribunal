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

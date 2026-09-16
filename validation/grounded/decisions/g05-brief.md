# Decision G05 - gated remote GPU benchmark access for an external party: architecture, security, operations review

## Artifact(s)

All under `./artifact/` (read them directly; cite `file:line`):

- `RUNBOOK.md` - the intended architecture, IDs, window/teardown procedure.
- `cloudflare-objects.json` - live Cloudflare tunnel ingress + Access app + policy.
- `ct204.conf`, `ct205.conf` - Proxmox LXC configs (bastion; RTX bench CT).
- `bastion-nftables.conf` - live nftables on the bastion (CT 204).
- `bastion-services.conf` - bastion sshd drop-in + tinyproxy + dnsmasq.
- `bastion-cloudflared.service` - cloudflared unit (token redacted).
- `ct205-services.conf` - RTX bench CT sshd drop-in + sudoers + proxy env.
- `gb10_setup.sh` - the script that built the GB10 side (docker-native).
- `gb10-squid-dockerfile.txt` - live squid.conf + bench Dockerfile on the GB10 host.
- `gb10-docker-inspect.txt` - live docker network/port/mount/mem facts on the GB10 host.
- `sdn.cfg`, `backup-job.json` - SDN island definition; backup-job exclusions.

Context (USER-FACT; verify against the artifacts, do not trust this prose):
A third party ("the external party", a hardware LLM-inference benchmarking lab that publishes
dashboards publicly) is being granted temporary, gated, remote SSH access to
benchmark two GPUs in a small production estate: an RTX Pro 6000 on a Proxmox
node ("the RTX host"), normally driving a production vLLM in CT 201 that
backs internal services; and an NVIDIA GB10 (Grace-Blackwell ARM, standalone,
"the GB10 host") that also runs three production vLLM containers. The whole
LAN is a flat production network holding a secrets store, a backup server, a
message spine, other tenants' dev boxes, and live trading workloads. Goal:
the external party can use the two GPUs but can reach nothing else on the LAN, there is no
inbound WAN exposure, and teardown is clean. Access identity is gated at the
Cloudflare edge.

Topology as built (USER-FACT):
```
the external party -> the bench hostname (CF Access, email allowlist, self-hosted app)
     -> cloudflared tunnel (outbound-only from CT 204)
     -> CT 204 bastion (eth0 on the production bridge; eth1 on the bastion island address on an
        isolated SDN vnet cirub0)
        |-> CT 205 bench-rtx <bench island address>:22 (on cirub0, NO uplink; RTX via
        |     /dev/nvidia* bind-mounts SHARED with prod CT 201; user has sudo;
        |     internet only via bastion tinyproxy <bastion island address>:8888)
        |-> GB10 bench container <GB10 host LAN address>:2222 (docker: bench on
              an --internal network, socat relay publishes :2222, squid egress
              proxy denies RFC1918)
```

## Question under review

Review as three lenses (every seat addresses all three):
- **Q1 Architecture.** Is this the right shape? Do the two GPU hosts use
  materially different isolation models (SDN L2 island vs docker internal net
  + proxy) for the same goal, and does that asymmetry create risk or
  fragility? Is the bastion a real chokepoint or theatre?
- **Q2 Security.** Can the external party (or an attacker who compromises the external party's session or
  the bench container) reach anything else on the LAN, escalate off the GPU
  host, read another tenant's data, or persist past teardown? Is the identity
  gate actually load-bearing? Name the highest-severity real hole.
- **Q3 Systems/operational.** Will this break production or fail to tear down
  cleanly? Answer A1-A5 below; they are mandatory review surface.

## Operational readiness (mandatory, A1-A5)

- **A1** Readiness/timeout/retry for each created dependency (cloudflared
  connector, tunnel health, squid, socat relay, tinyproxy, dnsmasq, the two
  sshd endpoints): readiness predicate and restart policy, or one-shot/racy?
- **A2** Eventual consistency: any state (DNS CNAME, Access policy propagation,
  docker iptables ordering) that could give a false pass/fail if checked
  immediately?
- **A3** Was DNS/Access/TLS validated in a way that could be fooled by caching?
- **A4** Enumerate every store that must change, who else writes it, and the
  symptom if skipped: the shared backup job exclusions, the SDN config, host
  docker iptables (DOCKER-USER), CT 201's GPU device nodes.
- **A5** Idempotent resume and teardown completeness: does a re-run
  double-create or wedge? Does the documented teardown remove every created
  object (CF app+policy+DNS+tunnel, both CTs, SDN vnet+zone, all three GB10
  containers + network + image + the bench home dir, backup-exclude restore)?

## Precedent (contested evidence)

- The production CT 201 uses the same `/dev/nvidia*` bind-mount +
  `cgroup2 devices.allow c 195/510` pattern that `ct205.conf` reuses: the RTX
  GPU is physically shared between prod and the guest; no MIG partition.
- Prior estate lesson: a co-resident vLLM crashloop happened when GPU memory
  fractions summed above 1.0; "active is not healthy".
- Prior estate lesson: a read-modify-write of a shared map by two
  cross-process writers with no lock lost an update; watch shared mutable
  host state (DOCKER-USER chain, SDN config, backup job).
- A squid gotcha was already hit and fixed during build (log to stdout as
  user `proxy` is fatal; now a daemon log file).
- The GB10 login user is in the docker group with no passwordless sudo; the
  RTX bench user in CT 205 has passwordless sudo (unprivileged LXC).

## Decision criteria (owner-supplied)

(1) no lateral reach from the guest to any other LAN host or tenant; (2) no
production outage or co-tenant breakage from the build or a benchmark window;
(3) clean, complete, idempotent teardown; (4) the simplest architecture that
achieves 1-3. Tie-break: rank findings by blast radius x likelihood; a
confirmed lateral-reach or co-tenant-outage path outranks any hardening nit.

## Constraints

- Cannot re-VLAN the estate for this engagement.
- RTX bench requires stopping prod vLLM in CT 201 during a window (accepted);
  the GB10 bench requires stopping its three vLLM containers.
- Time-boxed; end date not set. the external party's SSH pubkey and email are not yet
  installed (authorized_keys empty, policy owner-only), so review the
  mechanism and flag what must be true when they are added.

## Output contract

Maximum 8 claims, each as:

```
CLAIM: <one sentence>
EVIDENCE: <artifact file:line, verbatim span, or the exact probe that would settle it; else ASSUMPTION / SPECULATIVE (name it) / EXTERNAL (source)>
CONFIDENCE: <0-1 probability, calibrated>
FALSIFIER: <what concrete observation would prove this claim wrong>
```

Plus **VERDICT INPUT**: one line per question (ship / harden-first / don't)
and the single highest-severity finding overall. Maximum 1200 words.

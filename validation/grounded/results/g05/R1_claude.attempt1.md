## 1. ATTACK

**[Peer-1] (GPU exclusivity is convention, not enforcement) — right, but under-argued and mis-scoped.** The strongest evidence is missing from it: the documented close path `docker update --memory 20g the external party-bench` (`RUNBOOK.md:75`) governs *host RAM*, not VRAM — it does nothing about the GPU, and `gb10-docker-inspect.txt:1` confirms `the external party-bench` sits at `mem=21474836480 restart=unless-stopped` with `--gpus all` (`gb10_setup.sh:53-54`) *while closed*, i.e. co-resident with the three vLLMs on a reboot. That is the estate's own "GPU fractions sum >1.0 crashloop" precedent instantiated. Scope objection: framing the hole as *window discipline* implies a procedural fix. It isn't sufficient — `ct205.conf:14-19` plus `NOPASSWD:ALL` (`ct205-services.conf:5`, `gb10_setup.sh:15`) gives a hostile guest **root-level ioctl on device nodes shared with prod**, which no window schedule closes. Also, `RUNBOOK.md:13,32` saying `[NOT YET BUILT]` is contradicted by live inspect output — that's doc drift, evidence of stale operational truth, not of exclusivity. 0.88 is fine for the claim as stated.

**[Peer-2] (A2/A3 false-pass, no cache-proof validation) — right conclusion, two wrong supports; 0.74 is too high.** `cloudflare-objects.json:20 session_duration: "24h"` is a *revocation-latency* fact, not a caching/false-pass fact; citing it for A2/A3 is category error. "Docker iptables ordering vs other engine rules" is not instantiated: `gb10_setup.sh:5` explicitly makes *no* host iptables/systemd changes and `DOCKER-USER` is never written, so there is no competing writer to order against — the real A2 exposure is that internal-net isolation is pure docker-generated rules that a host firewall reload can transiently flush, which the claim doesn't say. The A3 half is self-labelled ASSUMPTION (absence of a recorded method). Net: correct that nothing fails closed (`sleep 3` at `:60`, `|| true` at `:68,71`), but the confidence should be ~0.6.

**[Peer-3] (GB10 LAN path; `172.30.99.1` never probed) — the sharpest rival claim; I'd raise it, not attack it.** One correction and one addition. Correction: `the external party-bench`'s `Gateway:""` (`gb10-docker-inspect.txt:1`) means no *route* off-subnet, but `172.30.99.1` is a host-local bridge address — traffic to it hits the host INPUT path, not the forward chain docker drops on, so it is very likely reachable, and `gb10_setup.sh:69-71` probes only `<lan-address>`. Addition they miss: the higher-value bridgehead is the **dual-homed relay and proxy** (`172.17.0.5`/`172.17.0.6`, `Gateway:"172.17.0.1"`), both NAT'd to the LAN; squid's `acl lan` (`gb10_setup.sh:26`) is dst-IP based so DNS tricks fail, but `169.254.0.0/16`, `100.64.0.0/10` and IPv6 are uncovered. 0.78 is well calibrated.

**[Peer-4] (A5 teardown incompleteness) — core is right, 0.9 on a five-part conjunction is not calibrated.** Two items are noise under the owner's tie-break (blast-radius ranking): leftover pulled base images and an unwritten `DOCKER-USER` chain (never modified, so nothing to restore). The PBS-snapshot item is material but its severity is capped — teardown step 1 (`RUNBOOK.md:80`) deletes the tunnel, so the captured `--token` is dead; the durable residue is the guest's installed pubkey and CT host keys. The `from=` point is only load-bearing in CT 204 and the GB10 container; in CT 205 the guest has `NOPASSWD:ALL` and can edit his own `authorized_keys`, so `from=` there is decorative. Also missed: `gb10_setup.sh:41` `docker rm -f` makes a re-run silently destroy an in-flight guest benchmark and reset `--memory` to 20g mid-window. Fair confidence ~0.8.

## 2. CONCEDE

- **Peer-3:** `172.30.99.1` and host-published ports are an untested reachability surface; my Own-4's "GB10 is weaker" needs this concrete unprobed path attached.
- **Peer-1:** the window *close* procedure never detaches the GPU on either host; `the external party-bench` runs GPU-attached with `restart=unless-stopped` outside windows. This is a distinct, fixable failure from my Own-1's root-ioctl framing and both belong in the final answer.
- **Peer-4:** teardown omits restarting CT 201's vLLM and GB10's three vLLMs — a production-restoration gap I did not raise; and PBS snapshots of CT 204 retain the guest's pubkey.
- **Peer-2:** nothing in the artifact records a cache-proof DNS/Access/TLS validation.

## 3. REVISE

- **Own-1** — amend to two enforcement gaps, not one: (a) root ioctl on shared `/dev/nvidia*`; (b) no window-close GPU detach, and `docker update --memory 20g` is host RAM, not VRAM, so the documented close is a non-remedy for the crashloop precedent. Confidence 0.9 → **0.92**.
- **Own-4** — add: the bastion chokepoint verdict stands, but GB10's weakness is now concretely two untested paths (`172.30.99.1`; dual-homed relay/proxy at `172.17.0.x`). Confidence 0.8 → **0.82**.
- **Own-8** — narrow: residue is the guest pubkey and host keys, not a live token (tunnel deleted at `RUNBOOK.md:80`). Materiality 0.35 → **0.3**; fact stays 0.85.
- Own-2, 3, 5, 6, 7 — no revision; Own-7's `onboot` pointer is `ct205.conf:9`, not `:10`.

## 4. VERDICT INPUT

- **Q1 Architecture:** harden-first — shape is right, the RTX island is real, but GB10's docker-policy model must be brought to parity (or accepted as explicitly weaker in writing).
- **Q2 Security:** harden-first — identity gate is load-bearing on WAN only; both bench sshd endpoints are LAN-reachable and face `authorized_keys` alone.
- **Q3 Systems/operational:** don't-ship-as-written — window close restores neither GPU exclusivity nor prod vLLMs; teardown is blind read-modify-write on two shared stores; no assertion in `gb10_setup.sh` can fail the build.
- **Highest-severity finding overall:** guest gets root-level ioctl access to GPU char devices shared with production on both hosts, with no window-close detach and no VRAM-level enforcement — co-tenant outage by construction, bypassing every network control.

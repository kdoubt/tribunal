I'll read the artifacts and check each disputed claim against the frozen brief before writing the cross-examination.I'll verify each disputed claim against the cited artifact spans.Checking a few remaining line pointers so the attacks land on the actual files.1. **ATTACK**

**Peer-1.** Ruleset fact is right (`bastion-nftables.conf:16,24` `ip daddr` in `table inet` output/`policy accept`; `gb10-squid-dockerfile.txt:3,6,9` IPv4 `lan` then `allow benchnet`). The leap to both isolation guarantees failing is not. CT 205’s LAN block is no-uplink `cirub0` plus `forward` `policy drop` (`bastion-nftables.conf:14`) in an `inet` table — family-agnostic — not that output drop. Bastion output IPv6 is a pivot from CT 204, not a guest→LAN path from 205. GB10 docker inspect shows empty `GlobalIPv6Address`/`IPv6Gateway` on all three containers. IPv6 presence is oracle-uncheckable; 0.6 is the right number and the claim sentence should have carried it. 169.254 is a real IPv4 hole in both ACLs; it is not “IPv6.”

**Peer-2.** Shared-GPU co-tenant outage is the top class (settled). The claim sentence then smuggles an unevidenced host-kernel escape past “every network control.” Their own 0.25 on escape is the honest number; 0.9 is only defensible for ranking GPU ioctl/VRAM, not for escape. CT 205 is `unprivileged: 1` (`ct205.conf:13`): NOPASSWD is userns root + bind-mounted `c 195/510`, not host root. GB10 `NOPASSWD` is container-local (`gb10_setup.sh:15`); no `--cap-drop` is a hardening nit, not a demonstrated breakout. Falsifier (“vLLM stopped for the whole engagement”) does not kill driver-crash or out-of-window 205/`unless-stopped --gpus all`.

**Peer-3.** `onboot: 0` is `ct205.conf:9`, not `:10` (oracle off-by-one). `onboot: 0` is GPU-safety (don’t auto-grab the shared RTX), not only A1 fragility. Reboot = live bastion + dead 205 is availability, not a new reachability hole; GB10 is a different host with `unless-stopped`. CLAIM states cloudflared “will” 15s-notify loop; confidence on that half is 0.3. Don’t sell 0.3 as will.

**Peer-4.** Exclude fact is certain (`backup-job.json:1` `"exclude":"114,205"`). The security claim is not. Tunnel delete (`RUNBOOK.md:80`) revokes the token; a pubkey is not a secret; 0.35 materiality is their number and is missing from the CLAIM. Hourly copies to `iscsi-fast` are operator-backup persistence, not guest→LAN. This must not outrank GPU or `.225:2222`.

**Peer-5.** Isolation tests cannot fail-closed: `:69` ends in `|| echo "…(good)"`, `:71` is `|| true`. “Verification block cannot fail the build” is too broad — `:70` (huggingface via squid) has no `|| true` and *can* non-zero under `set -e`. 0.95 should be ~0.90.

2. **CONCEDE**

- Egress denylists are IPv4-private only; GB10 squid is the side where that matters.
- Top hole: guest root/sudo + shared `/dev/nvidia*` ioctl, no MIG, no window enforcement — co-tenant VRAM/driver crash by construction (`RUNBOOK.md:95-96`, `ct205.conf:14-20`, `gb10_setup.sh:54`).
- Bastion `onboot: 1` / 205 `onboot: 0` reboot split; GB10 `sleep 3`; only cloudflared has `Restart=`.
- CT 204 is live on the hourly job; token in `bastion-cloudflared.service:9`; teardown has no PBS purge.
- GB10 isolation asserts are fail-open.

3. **REVISE**

- **Own-3** (0.88→0.90): keep as highest-severity: GPU exclusivity is convention; guest can ioctl shared nvidia for CT 205’s lifetime and GB10 `the external party-bench` is `--gpus all`/`unless-stopped`/20g. Explicitly *not* a demonstrated kernel escape (0.25).
- **Own-4** (0.78→0.80): add that squid/nft “RFC1918 deny” is IPv4-only; CT 205 still does not depend on that output chain; GB10 inspect currently shows no container IPv6.
- **Own-5** (0.86→0.88): add 204 `onboot: 1` vs 205 `onboot: 0` (`ct205.conf:9`).
- **Own-6** (0.74→0.86): GB10 verify cannot fail-closed on LAN/squid (`gb10_setup.sh:69,71`).
- **Own-7** (0.84→0.86): A4 skip: 204 omitted from exclude → token snapshotted hourly.
- **Own-8** (0.90→0.91): teardown also leaves PBS history of 204 for `keep-monthly:3`.
- **Own-1, Own-2:** no revision. GB10 `:2222` on `.225` and bastion `:22` with no `ip saddr` still make Access WAN-only and the bastion theatre on the GB10 path.

4. **VERDICT INPUT**

Q1 Architecture: **harden-first** (keep L2 island; GB10 docker+LAN publish is the weak/asymmetric side).  
Q2 Security: **harden-first** (bind `:2222` to bastion, `from=` keys, enforce GPU exclusive).  
Q3 Ops: **harden-first** (fail-closed verify, lock RMW, exclude 204, start/stop 205 with the window).  
Highest-severity: **shared GPU with no exclusivity enforcement — guest ioctl/VRAM can crash co-tenant vLLM.**

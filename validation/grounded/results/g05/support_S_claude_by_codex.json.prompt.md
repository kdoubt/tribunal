You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: The highest-severity hole is that a hostile guest gets **root with direct ioctl access to GPU char devices shared with production**, on both hosts — no MIG, no GPU-level isolation — giving a co-tenant outage path by construction and a host-kernel escape surface that bypasses every network control.
POINTER: ct205.conf:14-19
OPENED TEXT:
unprivileged: 1
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 510:* rwm
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file

[2] CLAIM: The highest-severity hole is that a hostile guest gets **root with direct ioctl access to GPU char devices shared with production**, on both hosts — no MIG, no GPU-level isolation — giving a co-tenant outage path by construction and a host-kernel escape surface that bypasses every network control.
POINTER: ct205-services.conf:5
OPENED TEXT:
AllowUsers the external party
the external party ALL=(ALL) NOPASSWD:ALL
Defaults:the external party env_keep += "http_proxy https_proxy HTTP_PROXY HTTPS_PROXY no_proxy"
export http_proxy=http://<island-address>:8888
export https_proxy=http://<island-address>:8888

[3] CLAIM: The highest-severity hole is that a hostile guest gets **root with direct ioctl access to GPU char devices shared with production**, on both hosts — no MIG, no GPU-level isolation — giving a co-tenant outage path by construction and a host-kernel escape surface that bypasses every network control.
POINTER: gb10_setup.sh:15
OPENED TEXT:
useradd -m -s /bin/bash the external party && \
    echo 'the external party ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/the external party && chmod 440 /etc/sudoers.d/the external party && \
    printf 'PermitRootLogin no\nPasswordAuthentication no\nKbdInteractiveAuthentication no\nAllowUsers the external party\n' > /etc/ssh/sshd_config.d/90-bench.conf && \
    printf 'http_proxy=http://172.30.99.3:3128\nhttps_proxy=http://172.30.99.3:3128\nHTTP_PROXY=http://172.30.99.3:3128\nHTTPS_PROXY=http://172.30.99.3:3128\nno_proxy=localhost,127.0.0.1\n' >> /etc/environment
EXPOSE 22

[4] CLAIM: The highest-severity hole is that a hostile guest gets **root with direct ioctl access to GPU char devices shared with production**, on both hosts — no MIG, no GPU-level isolation — giving a co-tenant outage path by construction and a host-kernel escape surface that bypasses every network control.
POINTER: gb10_setup.sh:54
OPENED TEXT:
docker run -d --name the external party-bench --restart unless-stopped \
  --gpus all --network cirubench --ip 172.30.99.10 \
  --memory 20g \
  -v /home/<login-user>/the external party-bench/home:/home/the external party \
  -v /home/<login-user>/the external party-bench/scratch:/scratch \

[5] CLAIM: The highest-severity hole is that a hostile guest gets **root with direct ioctl access to GPU char devices shared with production**, on both hosts — no MIG, no GPU-level isolation — giving a co-tenant outage path by construction and a host-kernel escape surface that bypasses every network control.
POINTER: RUNBOOK.md:95-96
OPENED TEXT:
- CT 205 is on a no-uplink L2 island: LAN unreachable by construction, not by rule.
- CT 205 is unprivileged; GPU shared with CT 201 — isolation is fs/process-level,
  not GPU-level (a driver-crashing workload would still hit vLLM).
- GB10 bench container (once built): internal network, egress only via squid ACL.

[6] CLAIM: The Cloudflare identity gate is load-bearing **only for the WAN path**; both bench SSH endpoints are directly reachable from the flat production LAN, so any co-tenant dev box or compromised prod container bypasses Access entirely and faces only `authorized_keys`.
POINTER: bastion-nftables.conf:8
OPENED TEXT:
ct state established,related accept
    tcp dport 22 accept comment "sshd (cloudflared localhost + LAN mgmt)"
    ip saddr <island-address> tcp dport 8888 accept comment "bench -> tinyproxy"
    ip saddr <island-address> udp dport 53 accept comment "bench -> dns"
    ip saddr <island-address> tcp dport 53 accept

[7] CLAIM: The Cloudflare identity gate is load-bearing **only for the WAN path**; both bench SSH endpoints are directly reachable from the flat production LAN, so any co-tenant dev box or compromised prod container bypasses Access entirely and faces only `authorized_keys`.
POINTER: gb10-docker-inspect.txt:3
OPENED TEXT:
/the external party-proxy nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e8416a31b996db6b4024","EndpointID":"efdb9650be3b86a3ead899a73511cac7bc4968ca5c7dbfb5eb364155a0c09d51","Gateway":"172.17.0.1","IPAddress":"172.17.0.6","MacAddress":"d6:af:c8:9a:6a:75","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":null},"cirubench":{"IPAMConfig":{"IPv4Address":"172.30.99.3"},"Links":null,"Aliases":[],"DriverOpts":{},"GwPriority":0,"NetworkID":"8c1484b020c6d40bcfb6500b9f14dafc3eb3db58bc8353cf576db62d526ed4bb","EndpointID":"cf116760b8078307321df092f27dacc895f7d12303663a5da0955e998ea946be","Gateway":"","IPAddress":"172.30.99.3","MacAddress":"b6:9d:a2:cd:e6:f7","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":["the external party-proxy","ceafb1148a9c"]}} ports={} mounts=["/home/<login-user>/the external party-bench/proxy/squid.conf:/etc/squid/squid.conf:ro"] mem=0 restart=unless-stopped
/the external party-ssh-relay nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e841

[8] CLAIM: The Cloudflare identity gate is load-bearing **only for the WAN path**; both bench SSH endpoints are directly reachable from the flat production LAN, so any co-tenant dev box or compromised prod container bypasses Access entirely and faces only `authorized_keys`.
POINTER: gb10_setup.sh:5
OPENED TEXT:
# Outbound: squid proxy container with RFC1918 destinations denied.
# Docker-native isolation only — no host iptables/systemd changes.
set -euo pipefail

mkdir -p /home/<login-user>/the external party-bench/{home,scratch,build,proxy}

[9] CLAIM: The Cloudflare identity gate is load-bearing **only for the WAN path**; both bench SSH endpoints are directly reachable from the flat production LAN, so any co-tenant dev box or compromised prod container bypasses Access entirely and faces only `authorized_keys`.
POINTER: RUNBOOK.md:92
OPENED TEXT:
- No inbound WAN exposure anywhere (tunnel is outbound-only from CT 204).
- Identity gate at CF edge (email allowlist) BEFORE any packet reaches sshd.
- Bastion user has no root; egress firewall + proxy policy live where he can't touch them.
- CT 205 is on a no-uplink L2 island: LAN unreachable by construction, not by rule.
- CT 205 is unprivileged; GPU shared with CT 201 — isolation is fs/process-level,

[10] CLAIM: Both egress denylists cover **IPv4 private ranges only** — no IPv6, no link-local — so if either segment has IPv6 (SLAAC/RA), the "no private egress" and "denies RFC1918" guarantees do not hold on that family.
POINTER: bastion-nftables.conf:24
OPENED TEXT:
ip daddr 1.1.1.1 tcp dport 53 accept
    ip daddr { <lan-address>/16, 10.0.0.0/8, 172.16.0.0/12 } drop comment "no other private egress"
  }
}

[11] CLAIM: Both egress denylists cover **IPv4 private ranges only** — no IPv6, no link-local — so if either segment has IPv6 (SLAAC/RA), the "no private egress" and "denies RFC1918" guarantees do not hold on that family.
POINTER: gb10-squid-dockerfile.txt:3
OPENED TEXT:
acl benchnet src 172.30.99.0/24
acl lan dst <lan-address>/16 10.0.0.0/8 172.16.0.0/12
acl SSL_ports port 443
acl Safe_ports port 80 443
http_access deny lan

[12] CLAIM: The RTX side's SDN island is genuinely strong and the bastion is a **real chokepoint, not theatre** — the asymmetry is real, and the GB10 docker model is the weaker of the two.
POINTER: ct205.conf:8
OPENED TEXT:
nameserver: <island-address>
net0: name=eth0,bridge=cirub0,hwaddr=BC:24:11:57:D0:9E,ip=<island-address>/24,type=veth
onboot: 0
ostype: debian
rootfs: local-lvm:vm-205-disk-0,size=120G

[13] CLAIM: The RTX side's SDN island is genuinely strong and the bastion is a **real chokepoint, not theatre** — the asymmetry is real, and the GB10 docker model is the weaker of the two.
POINTER: bastion-nftables.conf:14
OPENED TEXT:
}
  chain forward { type filter hook forward priority 0; policy drop; }
  chain output {
    type filter hook output priority 0; policy accept;
    oif "lo" accept

[14] CLAIM: The RTX side's SDN island is genuinely strong and the bastion is a **real chokepoint, not theatre** — the asymmetry is real, and the GB10 docker model is the weaker of the two.
POINTER: bastion-services.conf:8-17
OPENED TEXT:
AllowTcpForwarding yes
User tinyproxy
Group tinyproxy
Port 8888
Listen <island-address>
Timeout 600
LogLevel Notice
MaxClients 20
Allow <island-address>
ConnectPort 443
ConnectPort 80
listen-address=<island-address>
bind-interfaces
no-resolv

[15] CLAIM: The RTX side's SDN island is genuinely strong and the bastion is a **real chokepoint, not theatre** — the asymmetry is real, and the GB10 docker model is the weaker of the two.
POINTER: bastion-nftables.conf:24
OPENED TEXT:
ip daddr 1.1.1.1 tcp dport 53 accept
    ip daddr { <lan-address>/16, 10.0.0.0/8, 172.16.0.0/12 } drop comment "no other private egress"
  }
}

[16] CLAIM: The RTX side's SDN island is genuinely strong and the bastion is a **real chokepoint, not theatre** — the asymmetry is real, and the GB10 docker model is the weaker of the two.
POINTER: gb10-docker-inspect.txt:2
OPENED TEXT:
/the external party-bench nets={"cirubench":{"IPAMConfig":{"IPv4Address":"172.30.99.10"},"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"8c1484b020c6d40bcfb6500b9f14dafc3eb3db58bc8353cf576db62d526ed4bb","EndpointID":"6db1f19a9564a0b47e9f73f6adeee0a5e853e302cee2c90787a8a79bbd307fe9","Gateway":"","IPAddress":"172.30.99.10","MacAddress":"b6:aa:82:d6:d6:55","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":["the external party-bench","2e58d4795efe"]}} ports={} mounts=["/home/<login-user>/the external party-bench/home:/home/the external party","/home/<login-user>/the external party-bench/scratch:/scratch"] mem=21474836480 restart=unless-stopped
/the external party-proxy nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e8416a31b996db6b4024","EndpointID":"efdb9650be3b86a3ead899a73511cac7bc4968ca5c7dbfb5eb364155a0c09d51","Gateway":"172.17.0.1","IPAddress":"172.17.0.6","MacAddress":"d6:af:c8:9a:6a:75","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":null},"cirubench":{"IPAMConfig":{"IPv4Address":"172.30.99.3"},"Links":null,"Aliases":[],"DriverOpts":{},"GwPriority":0,"Network

[17] CLAIM: `gb10_setup.sh`'s verification block **cannot fail the build** — every isolation assertion is echo-only or `|| true`, so a squid misconfiguration that permits LAN egress still exits 0.
POINTER: gb10_setup.sh:66-71
OPENED TEXT:
echo "=== verify ==="
docker ps --filter name=the external party --format '{{.Names}}\t{{.Status}}\t{{.Ports}}'
docker exec the external party-bench nvidia-smi --query-gpu=name,driver_version --format=csv,noheader || true
docker exec the external party-bench bash -c 'timeout 3 bash -c "echo > /dev/tcp/<lan-address>/8080" 2>/dev/null && echo "LAN REACHABLE (BAD)" || echo "LAN direct: blocked (good)"'
docker exec the external party-bench bash -c 'curl -s -x http://172.30.99.3:3128 -o /dev/null -w "internet-via-squid: %{http_code}\n" --max-time 20 https://huggingface.co/api/models?limit=1'
docker exec the external party-bench bash -c 'curl -s -x http://172.30.99.3:3128 -o /dev/null -w "squid-to-LAN: %{http_code}\n" --max-time 10 http://<lan-address> || true'

[18] CLAIM: Teardown touches two shared mutable stores with blind read-modify-write, matching the estate's own lost-update precedent: `pvesh set /cluster/sdn` applies **all** pending cluster-wide SDN state, and the backup-exclude restore hardcodes `114`.
POINTER: RUNBOOK.md:82
OPENED TEXT:
2. node2: `pct stop 204 205 && pct destroy 204 205`; remove SDN vnet `cirub0` +
   zone `cirub` (`pvesh delete /cluster/sdn/vnets/cirub0`, `.../zones/cirub`,
   then `pvesh set /cluster/sdn`).
3. GB10: `docker rm -f the external party-bench the external party-proxy the external party-ssh-relay`,
   `docker network rm cirubench`, `rm -rf /home/<login-user>/the external party-bench`, `docker rmi the external party-bench:1`.

[19] CLAIM: Teardown touches two shared mutable stores with blind read-modify-write, matching the estate's own lost-update precedent: `pvesh set /cluster/sdn` applies **all** pending cluster-wide SDN state, and the backup-exclude restore hardcodes `114`.
POINTER: RUNBOOK.md:86
OPENED TEXT:
`docker network rm cirubench`, `rm -rf /home/<login-user>/the external party-bench`, `docker rmi the external party-bench:1`.
4. Backup job: restore exclude to `114` only.
5. If anything of ours was ever typed into his environment (it shouldn't be): rotate.

## Security posture summary

[20] CLAIM: Teardown touches two shared mutable stores with blind read-modify-write, matching the estate's own lost-update precedent: `pvesh set /cluster/sdn` applies **all** pending cluster-wide SDN state, and the backup-exclude restore hardcodes `114`.
POINTER: backup-job.json:1
OPENED TEXT:
{"all":1,"comment":"Hourly backup with deduplication - 24hr granularity","compress":"zstd","enabled":1,"exclude":"114,205","id":"cd5e47ba-81f1-4878-ac1d-aa685e22f6b1","mailnotification":"failure","mailto":"<email>","mode":"snapshot","prune-backups":{"keep-daily":"7","keep-hourly":"24","keep-monthly":"3","keep-weekly":"4"},"schedule":"hourly","storage":"iscsi-fast","type":"vzdump"}

[21] CLAIM: A1 readiness is weak on the LXC side and racy on GB10: CT 205 is `onboot: 0` while the bastion is `onboot: 1`, so a node reboot leaves the guest with a live entry point and a dead target; the GB10 build uses a fixed `sleep 3` as its only readiness predicate.
POINTER: ct205.conf:10
OPENED TEXT:
onboot: 0
ostype: debian
rootfs: local-lvm:vm-205-disk-0,size=120G
swap: 8192
unprivileged: 1

[22] CLAIM: A1 readiness is weak on the LXC side and racy on GB10: CT 205 is `onboot: 0` while the bastion is `onboot: 1`, so a node reboot leaves the guest with a live entry point and a dead target; the GB10 build uses a fixed `sleep 3` as its only readiness predicate.
POINTER: ct204.conf:10
OPENED TEXT:
net1: name=eth1,bridge=cirub0,hwaddr=BC:24:11:EC:45:25,ip=<island-address>/24,type=veth
onboot: 1
ostype: debian
rootfs: local-lvm:vm-204-disk-0,size=8G
swap: 512

[23] CLAIM: A1 readiness is weak on the LXC side and racy on GB10: CT 205 is `onboot: 0` while the bastion is `onboot: 1`, so a node reboot leaves the guest with a live entry point and a dead target; the GB10 build uses a fixed `sleep 3` as its only readiness predicate.
POINTER: gb10_setup.sh:60
OPENED TEXT:
sleep 3
docker exec the external party-bench bash -c '
  cp -rT /etc/skel /home/the external party 2>/dev/null || true
  mkdir -p /home/the external party/.ssh && touch /home/the external party/.ssh/authorized_keys

[24] CLAIM: CT 204 is **not** excluded from the hourly vzdump job, so the cloudflared tunnel token and the guest's installed pubkey are copied hourly to shared `iscsi-fast` and survive teardown for up to three months.
POINTER: backup-job.json:1
OPENED TEXT:
{"all":1,"comment":"Hourly backup with deduplication - 24hr granularity","compress":"zstd","enabled":1,"exclude":"114,205","id":"cd5e47ba-81f1-4878-ac1d-aa685e22f6b1","mailnotification":"failure","mailto":"<email>","mode":"snapshot","prune-backups":{"keep-daily":"7","keep-hourly":"24","keep-monthly":"3","keep-weekly":"4"},"schedule":"hourly","storage":"iscsi-fast","type":"vzdump"}

[25] CLAIM: CT 204 is **not** excluded from the hourly vzdump job, so the cloudflared tunnel token and the guest's installed pubkey are copied hourly to shared `iscsi-fast` and survive teardown for up to three months.
POINTER: RUNBOOK.md:78-87
OPENED TEXT:
## Teardown (end of engagement)

1. CF: delete Access app 493b20ef… (+policy), DNS CNAME `ssh-bench`, tunnel 4b1cfef2….
2. node2: `pct stop 204 205 && pct destroy 204 205`; remove SDN vnet `cirub0` +
   zone `cirub` (`pvesh delete /cluster/sdn/vnets/cirub0`, `.../zones/cirub`,
   then `pvesh set /cluster/sdn`).
3. GB10: `docker rm -f the external party-bench the external party-proxy the external party-ssh-relay`,
   `docker network rm cirubench`, `rm -rf /home/<login-user>/the external party-bench`, `docker rmi the external party-bench:1`.
4. Backup job: restore exclude to `114` only.
5. If anything of ours was ever typed into his environment (it shouldn't be): rotate.

## Security posture summary

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }
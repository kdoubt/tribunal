You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: ** The bastion provides meaningful IPv4 filtering, but the asymmetric design makes it an incomplete chokepoint.
POINTER: ct205.conf:8
OPENED TEXT:
nameserver: <island-address>
net0: name=eth0,bridge=cirub0,hwaddr=BC:24:11:57:D0:9E,ip=<island-address>/24,type=veth
onboot: 0
ostype: debian
rootfs: local-lvm:vm-205-disk-0,size=120G

[2] CLAIM: ** The bastion provides meaningful IPv4 filtering, but the asymmetric design makes it an incomplete chokepoint.
POINTER: sdn.cfg:1-5
OPENED TEXT:
/etc/pve/sdn/vnets.cfg:vnet: cirub0
/etc/pve/sdn/vnets.cfg:	zone cirub
/etc/pve/sdn/vnets.cfg-
/etc/pve/sdn/zones.cfg:simple: cirub
/etc/pve/sdn/zones.cfg-	nodes the RTX host
/etc/pve/sdn/zones.cfg-

[3] CLAIM: ** The bastion provides meaningful IPv4 filtering, but the asymmetric design makes it an incomplete chokepoint.
POINTER: bastion-nftables.conf:14
OPENED TEXT:
}
  chain forward { type filter hook forward priority 0; policy drop; }
  chain output {
    type filter hook output priority 0; policy accept;
    oif "lo" accept

[4] CLAIM: ** The bastion provides meaningful IPv4 filtering, but the asymmetric design makes it an incomplete chokepoint.
POINTER: gb10-docker-inspect.txt:2-3
OPENED TEXT:
/the external party-bench nets={"cirubench":{"IPAMConfig":{"IPv4Address":"172.30.99.10"},"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"8c1484b020c6d40bcfb6500b9f14dafc3eb3db58bc8353cf576db62d526ed4bb","EndpointID":"6db1f19a9564a0b47e9f73f6adeee0a5e853e302cee2c90787a8a79bbd307fe9","Gateway":"","IPAddress":"172.30.99.10","MacAddress":"b6:aa:82:d6:d6:55","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":["the external party-bench","2e58d4795efe"]}} ports={} mounts=["/home/<login-user>/the external party-bench/home:/home/the external party","/home/<login-user>/the external party-bench/scratch:/scratch"] mem=21474836480 restart=unless-stopped
/the external party-proxy nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e8416a31b996db6b4024","EndpointID":"efdb9650be3b86a3ead899a73511cac7bc4968ca5c7dbfb5eb364155a0c09d51","Gateway":"172.17.0.1","IPAddress":"172.17.0.6","MacAddress":"d6:af:c8:9a:6a:75","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":null},"cirubench":{"IPAMConfig":{"IPv4Address":"172.30.99.3"},"Links":null,"Aliases":[],"DriverOpts":{},"GwPriority":0,"Network

[5] CLAIM: ** The bastion provides meaningful IPv4 filtering, but the asymmetric design makes it an incomplete chokepoint.
POINTER: bastion-nftables.conf:16
OPENED TEXT:
chain output {
    type filter hook output priority 0; policy accept;
    oif "lo" accept
    ct state established,related accept
    ip daddr <island-address> tcp dport 22 accept comment "onward ssh to bench CT"

[6] CLAIM: ** Cloudflare gates the intended public route but is not a mandatory identity gate for every SSH entry path.
POINTER: cloudflare-objects.json:5
OPENED TEXT:
{
    "service": "ssh://localhost:22",
    "hostname": "the bench hostname"
   },
   {

[7] CLAIM: ** Cloudflare gates the intended public route but is not a mandatory identity gate for every SSH entry path.
POINTER: bastion-nftables.conf:8
OPENED TEXT:
ct state established,related accept
    tcp dport 22 accept comment "sshd (cloudflared localhost + LAN mgmt)"
    ip saddr <island-address> tcp dport 8888 accept comment "bench -> tinyproxy"
    ip saddr <island-address> udp dport 53 accept comment "bench -> dns"
    ip saddr <island-address> tcp dport 53 accept

[8] CLAIM: ** Cloudflare gates the intended public route but is not a mandatory identity gate for every SSH entry path.
POINTER: ct204.conf:8
OPENED TEXT:
nameserver: 1.1.1.1
net0: name=eth0,bridge=vmbr0,gw=<lan-address>,hwaddr=BC:24:11:93:CA:E7,ip=<lan-address>/24,type=veth
net1: name=eth1,bridge=cirub0,hwaddr=BC:24:11:EC:45:25,ip=<island-address>/24,type=veth
onboot: 1
ostype: debian

[9] CLAIM: ** Cloudflare gates the intended public route but is not a mandatory identity gate for every SSH entry path.
POINTER: gb10-docker-inspect.txt:3
OPENED TEXT:
/the external party-proxy nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e8416a31b996db6b4024","EndpointID":"efdb9650be3b86a3ead899a73511cac7bc4968ca5c7dbfb5eb364155a0c09d51","Gateway":"172.17.0.1","IPAddress":"172.17.0.6","MacAddress":"d6:af:c8:9a:6a:75","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":null},"cirubench":{"IPAMConfig":{"IPv4Address":"172.30.99.3"},"Links":null,"Aliases":[],"DriverOpts":{},"GwPriority":0,"NetworkID":"8c1484b020c6d40bcfb6500b9f14dafc3eb3db58bc8353cf576db62d526ed4bb","EndpointID":"cf116760b8078307321df092f27dacc895f7d12303663a5da0955e998ea946be","Gateway":"","IPAddress":"172.30.99.3","MacAddress":"b6:9d:a2:cd:e6:f7","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":["the external party-proxy","ceafb1148a9c"]}} ports={} mounts=["/home/<login-user>/the external party-bench/proxy/squid.conf:/etc/squid/squid.conf:ro"] mem=0 restart=unless-stopped
/the external party-ssh-relay nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e841

[10] CLAIM: ** Cloudflare gates the intended public route but is not a mandatory identity gate for every SSH entry path.
POINTER: gb10_setup.sh:48-50
OPENED TEXT:
docker run -d --name the external party-ssh-relay --restart unless-stopped \
  -p <lan-address>:2222 \
  alpine/socat tcp-listen:2222,fork,reuseaddr tcp:172.30.99.10:22
docker network connect cirubench the external party-ssh-relay

docker run -d --name the external party-bench --restart unless-stopped \

[11] CLAIM: ** Cloudflare gates the intended public route but is not a mandatory identity gate for every SSH entry path.
POINTER: RUNBOOK.md:42-46
OPENED TEXT:
1. Add his email to the Access policy (script `cf_add_email.py` pattern, or dash:
   Zero Trust → Access → Applications → "the external party bench SSH" → policy → add email).
2. Append his pubkey to `/home/the external party/.ssh/authorized_keys` in CT 204 AND CT 205
   (`pct exec` from node2), and in the GB10 container home once built
   (`/home/<login-user>/the external party-bench/home/.ssh/authorized_keys` on gb10).
3. Send him the client config (needs `cloudflared` installed locally):

```

[12] CLAIM: ** The highest-severity demonstrated design hole is retained guest GPU access when production resumes, allowing benchmark workloads to compete with production outside the accepted window.
POINTER: ct205.conf:14-20
OPENED TEXT:
unprivileged: 1
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 510:* rwm
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file

[13] CLAIM: ** The highest-severity demonstrated design hole is retained guest GPU access when production resumes, allowing benchmark workloads to compete with production outside the accepted window.
POINTER: gb10_setup.sh:53-55
OPENED TEXT:
docker run -d --name the external party-bench --restart unless-stopped \
  --gpus all --network cirubench --ip 172.30.99.10 \
  --memory 20g \
  -v /home/<login-user>/the external party-bench/home:/home/the external party \
  -v /home/<login-user>/the external party-bench/scratch:/scratch \
  the external party-bench:1

[14] CLAIM: ** The highest-severity demonstrated design hole is retained guest GPU access when production resumes, allowing benchmark workloads to compete with production outside the accepted window.
POINTER: RUNBOOK.md:69-75
OPENED TEXT:
fails over down limp-router rungs; expect degraded latency, not outage).
  Verify VRAM free: `pct exec 205 -- nvidia-smi`. **Close:** start vllm back,
  verify LiteLLM route healthy. Check cluster tasks before touching anything
  (avoid :00 vzdump windows).
- **GB10 window open:** `docker stop vllm-qwen3-30b vllm-rerank vllm-embed` and
  `docker update --memory 100g the external party-bench`. Note: GB10 Qwen3 seat is the tribunal
  backstop — don't open a window while a panel run is in flight. **Close:**
  `docker update --memory 20g the external party-bench`, start the three vLLM containers.
- Coordinate against hale-brain teacher/distillation runs on GB10.

## Teardown (end of engagement)

[15] CLAIM: ** The highest-severity demonstrated design hole is retained guest GPU access when production resumes, allowing benchmark workloads to compete with production outside the accepted window.
POINTER: RUNBOOK.md:95-96
OPENED TEXT:
- CT 205 is on a no-uplink L2 island: LAN unreachable by construction, not by rule.
- CT 205 is unprivileged; GPU shared with CT 201 — isolation is fs/process-level,
  not GPU-level (a driver-crashing workload would still hit vLLM).
- GB10 bench container (once built): internal network, egress only via squid ACL.

[16] CLAIM: ** GB10’s internal network does not establish isolation from services on the GB10 host itself.
POINTER: gb10-docker-inspect.txt:4
OPENED TEXT:
/the external party-ssh-relay nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e8416a31b996db6b4024","EndpointID":"54ce145d19c4a52f1fb450a78a02491a71d0dc8bb05024c2379a2315643c8d35","Gateway":"172.17.0.1","IPAddress":"172.17.0.5","MacAddress":"ce:ed:c7:2e:44:e7","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":null},"cirubench":{"IPAMConfig":{},"Links":null,"Aliases":[],"DriverOpts":{},"GwPriority":0,"NetworkID":"8c1484b020c6d40bcfb6500b9f14dafc3eb3db58bc8353cf576db62d526ed4bb","EndpointID":"173cbf3e4338c98993929cd0f755a092cc1d1e3135a19c225edadf7a6683f64d","Gateway":"","IPAddress":"172.30.99.2","MacAddress":"3e:d9:47:bf:c2:96","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":["the external party-ssh-relay","d81a094e45f0"]}} ports={"2222/tcp":[{"HostIp":"<lan-address>","HostPort":"2222"}]} mounts=null mem=0 restart=unless-stopped
cirubench internal=true subnet=[{"Subnet":"172.30.99.0/24","Gateway":"172.30.99.1"}]

[17] CLAIM: ** GB10’s internal network does not establish isolation from services on the GB10 host itself.
POINTER: gb10_setup.sh:38-39
OPENED TEXT:
docker network inspect cirubench >/dev/null 2>&1 || \
  docker network create --internal --subnet 172.30.99.0/24 cirubench

docker rm -f the external party-bench the external party-proxy the external party-ssh-relay 2>/dev/null || true

[18] CLAIM: ** GB10’s internal network does not establish isolation from services on the GB10 host itself.
POINTER: gb10_setup.sh:69
OPENED TEXT:
docker exec the external party-bench nvidia-smi --query-gpu=name,driver_version --format=csv,noheader || true
docker exec the external party-bench bash -c 'timeout 3 bash -c "echo > /dev/tcp/<lan-address>/8080" 2>/dev/null && echo "LAN REACHABLE (BAD)" || echo "LAN direct: blocked (good)"'
docker exec the external party-bench bash -c 'curl -s -x http://172.30.99.3:3128 -o /dev/null -w "internet-via-squid: %{http_code}\n" --max-time 20 https://huggingface.co/api/models?limit=1'
docker exec the external party-bench bash -c 'curl -s -x http://172.30.99.3:3128 -o /dev/null -w "squid-to-LAN: %{http_code}\n" --max-time 10 http://<lan-address> || true'

[19] CLAIM: ** A1 readiness is incomplete: restart settings exist for some dependencies, but end-to-end readiness and bounded recovery are not established.
POINTER: gb10_setup.sh:43-58
OPENED TEXT:
docker run -d --name the external party-proxy --restart unless-stopped \
  -v /home/<login-user>/the external party-bench/proxy/squid.conf:/etc/squid/squid.conf:ro \
  ubuntu/squid
docker network connect --ip 172.30.99.3 cirubench the external party-proxy

docker run -d --name the external party-ssh-relay --restart unless-stopped \
  -p <lan-address>:2222 \
  alpine/socat tcp-listen:2222,fork,reuseaddr tcp:172.30.99.10:22
docker network connect cirubench the external party-ssh-relay

docker run -d --name the external party-bench --restart unless-stopped \
  --gpus all --network cirubench --ip 172.30.99.10 \
  --memory 20g \
  -v /home/<login-user>/the external party-bench/home:/home/the external party \
  -v /home/<login-user>/the external party-bench/scratch:/scratch \
  the external party-bench:1

sleep 3
docker exec the external party-bench bash -c '

[20] CLAIM: ** A1 readiness is incomplete: restart settings exist for some dependencies, but end-to-end readiness and bounded recovery are not established.
POINTER: bastion-services.conf:12
OPENED TEXT:
Listen <island-address>
Timeout 600
LogLevel Notice
MaxClients 20
Allow <island-address>

[21] CLAIM: ** A1 readiness is incomplete: restart settings exist for some dependencies, but end-to-end readiness and bounded recovery are not established.
POINTER: bastion-services.conf:1-21
OPENED TEXT:
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
AllowUsers the external party
X11Forwarding no
AllowAgentForwarding yes
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
server=1.1.1.1

[22] CLAIM: ** A1 readiness is incomplete: restart settings exist for some dependencies, but end-to-end readiness and bounded recovery are not established.
POINTER: ct205-services.conf:1-10
OPENED TEXT:
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
AllowUsers the external party
the external party ALL=(ALL) NOPASSWD:ALL
Defaults:the external party env_keep += "http_proxy https_proxy HTTP_PROXY HTTPS_PROXY no_proxy"
export http_proxy=http://<island-address>:8888
export https_proxy=http://<island-address>:8888
export HTTP_PROXY=$http_proxy HTTPS_PROXY=$https_proxy
export no_proxy=localhost,127.0.0.1,<island-address>/24

[23] CLAIM: ** A2–A3 validation does not establish convergence or cache-independent DNS/Access/TLS correctness.
POINTER: RUNBOOK.md:16-18
OPENED TEXT:
- **CF side** (account 3e3fc5cb…, zone <estate-domain>): tunnel `the external party-bench-bastion`,
  CNAME `ssh-bench` → tunnel, Access app `the external party bench SSH` (493b20ef…), policy
  `the external party-bench-allowed-emails` (43af5936…) — currently <email> only.
- **CT 204 the external party-bastion** (1c/1G/8G, vmbr0 .204 + cirub0 <island-address>, onboot=1):
  cloudflared connector; sshd key-only, user `the external party` (NO sudo); nftables drops all
  private-range egress except <island-address>:22 and <lan-address>; tinyproxy

[24] CLAIM: ** A2–A3 validation does not establish convergence or cache-independent DNS/Access/TLS correctness.
POINTER: gb10_setup.sh:43-60
OPENED TEXT:
docker run -d --name the external party-proxy --restart unless-stopped \
  -v /home/<login-user>/the external party-bench/proxy/squid.conf:/etc/squid/squid.conf:ro \
  ubuntu/squid
docker network connect --ip 172.30.99.3 cirubench the external party-proxy

docker run -d --name the external party-ssh-relay --restart unless-stopped \
  -p <lan-address>:2222 \
  alpine/socat tcp-listen:2222,fork,reuseaddr tcp:172.30.99.10:22
docker network connect cirubench the external party-ssh-relay

docker run -d --name the external party-bench --restart unless-stopped \
  --gpus all --network cirubench --ip 172.30.99.10 \
  --memory 20g \
  -v /home/<login-user>/the external party-bench/home:/home/the external party \
  -v /home/<login-user>/the external party-bench/scratch:/scratch \
  the external party-bench:1

sleep 3
docker exec the external party-bench bash -c '
  cp -rT /etc/skel /home/the external party 2>/dev/null || true
  mkdir -p /home/the external party/.ssh && touch /home/the external party/.ssh/authorized_keys

[25] CLAIM: ** A4 shared-state changes lack a documented concurrency-safe reconciliation procedure.
POINTER: backup-job.json:1
OPENED TEXT:
{"all":1,"comment":"Hourly backup with deduplication - 24hr granularity","compress":"zstd","enabled":1,"exclude":"114,205","id":"cd5e47ba-81f1-4878-ac1d-aa685e22f6b1","mailnotification":"failure","mailto":"<email>","mode":"snapshot","prune-backups":{"keep-daily":"7","keep-hourly":"24","keep-monthly":"3","keep-weekly":"4"},"schedule":"hourly","storage":"iscsi-fast","type":"vzdump"}

[26] CLAIM: ** A4 shared-state changes lack a documented concurrency-safe reconciliation procedure.
POINTER: RUNBOOK.md:86
OPENED TEXT:
`docker network rm cirubench`, `rm -rf /home/<login-user>/the external party-bench`, `docker rmi the external party-bench:1`.
4. Backup job: restore exclude to `114` only.
5. If anything of ours was ever typed into his environment (it shouldn't be): rotate.

## Security posture summary

[27] CLAIM: ** A4 shared-state changes lack a documented concurrency-safe reconciliation procedure.
POINTER: sdn.cfg:1-5
OPENED TEXT:
/etc/pve/sdn/vnets.cfg:vnet: cirub0
/etc/pve/sdn/vnets.cfg:	zone cirub
/etc/pve/sdn/vnets.cfg-
/etc/pve/sdn/zones.cfg:simple: cirub
/etc/pve/sdn/zones.cfg-	nodes the RTX host
/etc/pve/sdn/zones.cfg-

[28] CLAIM: ** A4 shared-state changes lack a documented concurrency-safe reconciliation procedure.
POINTER: RUNBOOK.md:81-83
OPENED TEXT:
1. CF: delete Access app 493b20ef… (+policy), DNS CNAME `ssh-bench`, tunnel 4b1cfef2….
2. node2: `pct stop 204 205 && pct destroy 204 205`; remove SDN vnet `cirub0` +
   zone `cirub` (`pvesh delete /cluster/sdn/vnets/cirub0`, `.../zones/cirub`,
   then `pvesh set /cluster/sdn`).
3. GB10: `docker rm -f the external party-bench the external party-proxy the external party-ssh-relay`,
   `docker network rm cirubench`, `rm -rf /home/<login-user>/the external party-bench`, `docker rmi the external party-bench:1`.
4. Backup job: restore exclude to `114` only.

[29] CLAIM: ** A4 shared-state changes lack a documented concurrency-safe reconciliation procedure.
POINTER: gb10_setup.sh:5
OPENED TEXT:
# Outbound: squid proxy container with RFC1918 destinations denied.
# Docker-native isolation only — no host iptables/systemd changes.
set -euo pipefail

mkdir -p /home/<login-user>/the external party-bench/{home,scratch,build,proxy}

[30] CLAIM: ** A4 shared-state changes lack a documented concurrency-safe reconciliation procedure.
POINTER: RUNBOOK.md:25-28
OPENED TEXT:
on SDN vnet `cirub0` (zone `cirub`, node2-only, NO uplink — physical L2 island,
  no NAT anywhere). GPU via /dev/nvidia* bind mounts + cgroup allows c 195/510
  (same pattern as CT 201). NVIDIA user-space 610.43.02 via .run --no-kernel-modules
  (matches node2 host driver EXACTLY — if node2 driver is upgraded, rerun the .run
  in 205 with the new version). User `the external party` HAS passwordless sudo (his sandbox).
  Internet only via bastion tinyproxy (http/https, set in /etc/profile.d/proxy.sh,
  apt.conf.d/95proxy). Verified: HF reachable, Infisical/PBS NOT reachable.
  Excluded from the hourly PBS job (exclude=114,205) — scratch box, 120G churn.

[31] CLAIM: ** A5 lists the intended teardown inventory but does not provide reliable idempotent resume or executable complete teardown.
POINTER: RUNBOOK.md:80-86
OPENED TEXT:
1. CF: delete Access app 493b20ef… (+policy), DNS CNAME `ssh-bench`, tunnel 4b1cfef2….
2. node2: `pct stop 204 205 && pct destroy 204 205`; remove SDN vnet `cirub0` +
   zone `cirub` (`pvesh delete /cluster/sdn/vnets/cirub0`, `.../zones/cirub`,
   then `pvesh set /cluster/sdn`).
3. GB10: `docker rm -f the external party-bench the external party-proxy the external party-ssh-relay`,
   `docker network rm cirubench`, `rm -rf /home/<login-user>/the external party-bench`, `docker rmi the external party-bench:1`.
4. Backup job: restore exclude to `114` only.
5. If anything of ours was ever typed into his environment (it shouldn't be): rotate.

## Security posture summary

[32] CLAIM: ** A5 lists the intended teardown inventory but does not provide reliable idempotent resume or executable complete teardown.
POINTER: gb10_setup.sh:38-41
OPENED TEXT:
docker network inspect cirubench >/dev/null 2>&1 || \
  docker network create --internal --subnet 172.30.99.0/24 cirubench

docker rm -f the external party-bench the external party-proxy the external party-ssh-relay 2>/dev/null || true

docker run -d --name the external party-proxy --restart unless-stopped \
  -v /home/<login-user>/the external party-bench/proxy/squid.conf:/etc/squid/squid.conf:ro \

[33] CLAIM: ** A5 lists the intended teardown inventory but does not provide reliable idempotent resume or executable complete teardown.
POINTER: RUNBOOK.md:32
OPENED TEXT:
Excluded from the hourly PBS job (exclude=114,205) — scratch box, 120G churn.
- **GB10**: NOT built (permission classifier blocked remote execution on gb10; the owner
  runs it). Staged script: `<path> external party-bench/gb10_setup.sh` — runs as `<login-user>`
  (docker group, no sudo needed): `scp <path> external party-bench/gb10_setup.sh gb10:~ && ssh gb10 bash gb10_setup.sh`.
  Design: docker-native only (no host iptables/systemd): bench container on an

[34] CLAIM: ** A5 lists the intended teardown inventory but does not provide reliable idempotent resume or executable complete teardown.
POINTER: backup-job.json:1
OPENED TEXT:
{"all":1,"comment":"Hourly backup with deduplication - 24hr granularity","compress":"zstd","enabled":1,"exclude":"114,205","id":"cd5e47ba-81f1-4878-ac1d-aa685e22f6b1","mailnotification":"failure","mailto":"<email>","mode":"snapshot","prune-backups":{"keep-daily":"7","keep-hourly":"24","keep-monthly":"3","keep-weekly":"4"},"schedule":"hourly","storage":"iscsi-fast","type":"vzdump"}

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }
You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: The two GPU hosts do not implement the same isolation model: CT 205 is a no-uplink L2 island with the bastion as a real L3 chokepoint, while GB10 is docker-policy isolation whose SSH listener sits on the production LAN, so the bastion is theatre for that path.
POINTER: ct205.conf:8
OPENED TEXT:
nameserver: <island-address>
net0: name=eth0,bridge=cirub0,hwaddr=BC:24:11:57:D0:9E,ip=<island-address>/24,type=veth
onboot: 0
ostype: debian
rootfs: local-lvm:vm-205-disk-0,size=120G

[2] CLAIM: The two GPU hosts do not implement the same isolation model: CT 205 is a no-uplink L2 island with the bastion as a real L3 chokepoint, while GB10 is docker-policy isolation whose SSH listener sits on the production LAN, so the bastion is theatre for that path.
POINTER: ct204.conf:8-9
OPENED TEXT:
nameserver: 1.1.1.1
net0: name=eth0,bridge=vmbr0,gw=<lan-address>,hwaddr=BC:24:11:93:CA:E7,ip=<lan-address>/24,type=veth
net1: name=eth1,bridge=cirub0,hwaddr=BC:24:11:EC:45:25,ip=<island-address>/24,type=veth
onboot: 1
ostype: debian
rootfs: local-lvm:vm-204-disk-0,size=8G

[3] CLAIM: The two GPU hosts do not implement the same isolation model: CT 205 is a no-uplink L2 island with the bastion as a real L3 chokepoint, while GB10 is docker-policy isolation whose SSH listener sits on the production LAN, so the bastion is theatre for that path.
POINTER: bastion-nftables.conf:14
OPENED TEXT:
}
  chain forward { type filter hook forward priority 0; policy drop; }
  chain output {
    type filter hook output priority 0; policy accept;
    oif "lo" accept

[4] CLAIM: The two GPU hosts do not implement the same isolation model: CT 205 is a no-uplink L2 island with the bastion as a real L3 chokepoint, while GB10 is docker-policy isolation whose SSH listener sits on the production LAN, so the bastion is theatre for that path.
POINTER: gb10-docker-inspect.txt:3
OPENED TEXT:
/the external party-proxy nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e8416a31b996db6b4024","EndpointID":"efdb9650be3b86a3ead899a73511cac7bc4968ca5c7dbfb5eb364155a0c09d51","Gateway":"172.17.0.1","IPAddress":"172.17.0.6","MacAddress":"d6:af:c8:9a:6a:75","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":null},"cirubench":{"IPAMConfig":{"IPv4Address":"172.30.99.3"},"Links":null,"Aliases":[],"DriverOpts":{},"GwPriority":0,"NetworkID":"8c1484b020c6d40bcfb6500b9f14dafc3eb3db58bc8353cf576db62d526ed4bb","EndpointID":"cf116760b8078307321df092f27dacc895f7d12303663a5da0955e998ea946be","Gateway":"","IPAddress":"172.30.99.3","MacAddress":"b6:9d:a2:cd:e6:f7","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":["the external party-proxy","ceafb1148a9c"]}} ports={} mounts=["/home/<login-user>/the external party-bench/proxy/squid.conf:/etc/squid/squid.conf:ro"] mem=0 restart=unless-stopped
/the external party-ssh-relay nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e841

[5] CLAIM: The two GPU hosts do not implement the same isolation model: CT 205 is a no-uplink L2 island with the bastion as a real L3 chokepoint, while GB10 is docker-policy isolation whose SSH listener sits on the production LAN, so the bastion is theatre for that path.
POINTER: gb10_setup.sh:5
OPENED TEXT:
# Outbound: squid proxy container with RFC1918 destinations denied.
# Docker-native isolation only — no host iptables/systemd changes.
set -euo pipefail

mkdir -p /home/<login-user>/the external party-bench/{home,scratch,build,proxy}

[6] CLAIM: Cloudflare Access is load-bearing only on the WAN hostname; GB10 sshd is reachable on `<lan-address>` from any LAN peer, and bastion sshd accepts `:22` on vmbr0 with no source allowlist.
POINTER: cloudflare-objects.json:4-6
OPENED TEXT:
"ingress": [
   {
    "service": "ssh://localhost:22",
    "hostname": "the bench hostname"
   },
   {
    "service": "http_status:404"

[7] CLAIM: Cloudflare Access is load-bearing only on the WAN hostname; GB10 sshd is reachable on `<lan-address>` from any LAN peer, and bastion sshd accepts `:22` on vmbr0 with no source allowlist.
POINTER: cloudflare-objects.json:26-34
OPENED TEXT:
"access_policies": [
  {
   "name": "the external party-bench-allowed-emails",
   "decision": "allow",
   "include": [
    {
     "email": {
      "email": "<email>"
     }
    }
   ],
   "exclude": [],
   "require": [],

[8] CLAIM: Cloudflare Access is load-bearing only on the WAN hostname; GB10 sshd is reachable on `<lan-address>` from any LAN peer, and bastion sshd accepts `:22` on vmbr0 with no source allowlist.
POINTER: gb10-docker-inspect.txt:3
OPENED TEXT:
/the external party-proxy nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e8416a31b996db6b4024","EndpointID":"efdb9650be3b86a3ead899a73511cac7bc4968ca5c7dbfb5eb364155a0c09d51","Gateway":"172.17.0.1","IPAddress":"172.17.0.6","MacAddress":"d6:af:c8:9a:6a:75","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":null},"cirubench":{"IPAMConfig":{"IPv4Address":"172.30.99.3"},"Links":null,"Aliases":[],"DriverOpts":{},"GwPriority":0,"NetworkID":"8c1484b020c6d40bcfb6500b9f14dafc3eb3db58bc8353cf576db62d526ed4bb","EndpointID":"cf116760b8078307321df092f27dacc895f7d12303663a5da0955e998ea946be","Gateway":"","IPAddress":"172.30.99.3","MacAddress":"b6:9d:a2:cd:e6:f7","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":["the external party-proxy","ceafb1148a9c"]}} ports={} mounts=["/home/<login-user>/the external party-bench/proxy/squid.conf:/etc/squid/squid.conf:ro"] mem=0 restart=unless-stopped
/the external party-ssh-relay nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e841

[9] CLAIM: Cloudflare Access is load-bearing only on the WAN hostname; GB10 sshd is reachable on `<lan-address>` from any LAN peer, and bastion sshd accepts `:22` on vmbr0 with no source allowlist.
POINTER: bastion-nftables.conf:8
OPENED TEXT:
ct state established,related accept
    tcp dport 22 accept comment "sshd (cloudflared localhost + LAN mgmt)"
    ip saddr <island-address> tcp dport 8888 accept comment "bench -> tinyproxy"
    ip saddr <island-address> udp dport 53 accept comment "bench -> dns"
    ip saddr <island-address> tcp dport 53 accept

[10] CLAIM: Cloudflare Access is load-bearing only on the WAN hostname; GB10 sshd is reachable on `<lan-address>` from any LAN peer, and bastion sshd accepts `:22` on vmbr0 with no source allowlist.
POINTER: bastion-services.conf:6-7
OPENED TEXT:
X11Forwarding no
AllowAgentForwarding yes
AllowTcpForwarding yes
User tinyproxy
Group tinyproxy
Port 8888

[11] CLAIM: Highest-severity hole: GPU exclusivity is a runbook convention, not an enforcement — CT 205 keeps `/dev/nvidia*` for its lifetime and GB10 `the external party-bench` is already `--gpus all` / `unless-stopped` / 20g, so a guest job outside a window can drive the estate’s documented vLLM crashloop.
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

[12] CLAIM: Highest-severity hole: GPU exclusivity is a runbook convention, not an enforcement — CT 205 keeps `/dev/nvidia*` for its lifetime and GB10 `the external party-bench` is already `--gpus all` / `unless-stopped` / 20g, so a guest job outside a window can drive the estate’s documented vLLM crashloop.
POINTER: ct205.conf:9
OPENED TEXT:
net0: name=eth0,bridge=cirub0,hwaddr=BC:24:11:57:D0:9E,ip=<island-address>/24,type=veth
onboot: 0
ostype: debian
rootfs: local-lvm:vm-205-disk-0,size=120G
swap: 8192

[13] CLAIM: Highest-severity hole: GPU exclusivity is a runbook convention, not an enforcement — CT 205 keeps `/dev/nvidia*` for its lifetime and GB10 `the external party-bench` is already `--gpus all` / `unless-stopped` / 20g, so a guest job outside a window can drive the estate’s documented vLLM crashloop.
POINTER: RUNBOOK.md:67-75
OPENED TEXT:
- **RTX window open:** `pct exec 201 -- systemctl stop vllm` on node2 (HAL chat
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

[14] CLAIM: Highest-severity hole: GPU exclusivity is a runbook convention, not an enforcement — CT 205 keeps `/dev/nvidia*` for its lifetime and GB10 `the external party-bench` is already `--gpus all` / `unless-stopped` / 20g, so a guest job outside a window can drive the estate’s documented vLLM crashloop.
POINTER: gb10-docker-inspect.txt:1
OPENED TEXT:
/the external party-bench nets={"cirubench":{"IPAMConfig":{"IPv4Address":"172.30.99.10"},"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"8c1484b020c6d40bcfb6500b9f14dafc3eb3db58bc8353cf576db62d526ed4bb","EndpointID":"6db1f19a9564a0b47e9f73f6adeee0a5e853e302cee2c90787a8a79bbd307fe9","Gateway":"","IPAddress":"172.30.99.10","MacAddress":"b6:aa:82:d6:d6:55","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":["the external party-bench","2e58d4795efe"]}} ports={} mounts=["/home/<login-user>/the external party-bench/home:/home/the external party","/home/<login-user>/the external party-bench/scratch:/scratch"] mem=21474836480 restart=unless-stopped
/the external party-proxy nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e8416a31b996db6b4024","EndpointID":"efdb9650be3b86a3ead899a73511cac7bc4968ca5c7dbfb5eb364155a0c09d51","Gateway":"172.17.0.1","IPAddress":"172.17.0.6","MacAddress":"d6:af:c8:9a:6a:75","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":null},"cirubench":{"IPAMConfig":{"IPv4Address":"172.30.99.3"},"Links":null,"Aliases":[],"DriverOpts":{},"GwPriority":0,"Network

[15] CLAIM: Highest-severity hole: GPU exclusivity is a runbook convention, not an enforcement — CT 205 keeps `/dev/nvidia*` for its lifetime and GB10 `the external party-bench` is already `--gpus all` / `unless-stopped` / 20g, so a guest job outside a window can drive the estate’s documented vLLM crashloop.
POINTER: gb10_setup.sh:53-54
OPENED TEXT:
docker run -d --name the external party-bench --restart unless-stopped \
  --gpus all --network cirubench --ip 172.30.99.10 \
  --memory 20g \
  -v /home/<login-user>/the external party-bench/home:/home/the external party \
  -v /home/<login-user>/the external party-bench/scratch:/scratch \

[16] CLAIM: Highest-severity hole: GPU exclusivity is a runbook convention, not an enforcement — CT 205 keeps `/dev/nvidia*` for its lifetime and GB10 `the external party-bench` is already `--gpus all` / `unless-stopped` / 20g, so a guest job outside a window can drive the estate’s documented vLLM crashloop.
POINTER: RUNBOOK.md:12
OPENED TEXT:
→ CT 204 the external party-bastion (node2, <lan-address> + <island-address> on island)
        ├→ CT 205 the external party-bench-rtx  <island-address>:22  (isolated SDN vnet cirub0)
        └→ GB10 the external party-bench container  <lan-address>   [NOT YET BUILT]
```

[17] CLAIM: Guest→LAN from CT 205 is blocked by construction; guest→LAN from GB10 is only a squid RFC1918 deny on a dual-homed proxy that already has a docker0 default route, and the only LAN probe never tested the internal-net host gateway `172.30.99.1`.
POINTER: gb10-docker-inspect.txt:2
OPENED TEXT:
/the external party-bench nets={"cirubench":{"IPAMConfig":{"IPv4Address":"172.30.99.10"},"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"8c1484b020c6d40bcfb6500b9f14dafc3eb3db58bc8353cf576db62d526ed4bb","EndpointID":"6db1f19a9564a0b47e9f73f6adeee0a5e853e302cee2c90787a8a79bbd307fe9","Gateway":"","IPAddress":"172.30.99.10","MacAddress":"b6:aa:82:d6:d6:55","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":["the external party-bench","2e58d4795efe"]}} ports={} mounts=["/home/<login-user>/the external party-bench/home:/home/the external party","/home/<login-user>/the external party-bench/scratch:/scratch"] mem=21474836480 restart=unless-stopped
/the external party-proxy nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e8416a31b996db6b4024","EndpointID":"efdb9650be3b86a3ead899a73511cac7bc4968ca5c7dbfb5eb364155a0c09d51","Gateway":"172.17.0.1","IPAddress":"172.17.0.6","MacAddress":"d6:af:c8:9a:6a:75","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":null},"cirubench":{"IPAMConfig":{"IPv4Address":"172.30.99.3"},"Links":null,"Aliases":[],"DriverOpts":{},"GwPriority":0,"Network

[18] CLAIM: Guest→LAN from CT 205 is blocked by construction; guest→LAN from GB10 is only a squid RFC1918 deny on a dual-homed proxy that already has a docker0 default route, and the only LAN probe never tested the internal-net host gateway `172.30.99.1`.
POINTER: gb10-squid-dockerfile.txt:3
OPENED TEXT:
acl benchnet src 172.30.99.0/24
acl lan dst <lan-address>/16 10.0.0.0/8 172.16.0.0/12
acl SSL_ports port 443
acl Safe_ports port 80 443
http_access deny lan

[19] CLAIM: Guest→LAN from CT 205 is blocked by construction; guest→LAN from GB10 is only a squid RFC1918 deny on a dual-homed proxy that already has a docker0 default route, and the only LAN probe never tested the internal-net host gateway `172.30.99.1`.
POINTER: gb10-docker-inspect.txt:4
OPENED TEXT:
/the external party-ssh-relay nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e8416a31b996db6b4024","EndpointID":"54ce145d19c4a52f1fb450a78a02491a71d0dc8bb05024c2379a2315643c8d35","Gateway":"172.17.0.1","IPAddress":"172.17.0.5","MacAddress":"ce:ed:c7:2e:44:e7","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":null},"cirubench":{"IPAMConfig":{},"Links":null,"Aliases":[],"DriverOpts":{},"GwPriority":0,"NetworkID":"8c1484b020c6d40bcfb6500b9f14dafc3eb3db58bc8353cf576db62d526ed4bb","EndpointID":"173cbf3e4338c98993929cd0f755a092cc1d1e3135a19c225edadf7a6683f64d","Gateway":"","IPAddress":"172.30.99.2","MacAddress":"3e:d9:47:bf:c2:96","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":["the external party-ssh-relay","d81a094e45f0"]}} ports={"2222/tcp":[{"HostIp":"<lan-address>","HostPort":"2222"}]} mounts=null mem=0 restart=unless-stopped
cirubench internal=true subnet=[{"Subnet":"172.30.99.0/24","Gateway":"172.30.99.1"}]

[20] CLAIM: Guest→LAN from CT 205 is blocked by construction; guest→LAN from GB10 is only a squid RFC1918 deny on a dual-homed proxy that already has a docker0 default route, and the only LAN probe never tested the internal-net host gateway `172.30.99.1`.
POINTER: gb10_setup.sh:69-71
OPENED TEXT:
docker exec the external party-bench nvidia-smi --query-gpu=name,driver_version --format=csv,noheader || true
docker exec the external party-bench bash -c 'timeout 3 bash -c "echo > /dev/tcp/<lan-address>/8080" 2>/dev/null && echo "LAN REACHABLE (BAD)" || echo "LAN direct: blocked (good)"'
docker exec the external party-bench bash -c 'curl -s -x http://172.30.99.3:3128 -o /dev/null -w "internet-via-squid: %{http_code}\n" --max-time 20 https://huggingface.co/api/models?limit=1'
docker exec the external party-bench bash -c 'curl -s -x http://172.30.99.3:3128 -o /dev/null -w "squid-to-LAN: %{http_code}\n" --max-time 10 http://<lan-address> || true'

[21] CLAIM: A1: only cloudflared has a readiness/restart story; GB10 bring-up is racy and the island daemons have no units in-tree.
POINTER: gb10_setup.sh:48-51
OPENED TEXT:
docker run -d --name the external party-ssh-relay --restart unless-stopped \
  -p <lan-address>:2222 \
  alpine/socat tcp-listen:2222,fork,reuseaddr tcp:172.30.99.10:22
docker network connect cirubench the external party-ssh-relay

docker run -d --name the external party-bench --restart unless-stopped \
  --gpus all --network cirubench --ip 172.30.99.10 \

[22] CLAIM: A1: only cloudflared has a readiness/restart story; GB10 bring-up is racy and the island daemons have no units in-tree.
POINTER: gb10_setup.sh:43-46
OPENED TEXT:
docker run -d --name the external party-proxy --restart unless-stopped \
  -v /home/<login-user>/the external party-bench/proxy/squid.conf:/etc/squid/squid.conf:ro \
  ubuntu/squid
docker network connect --ip 172.30.99.3 cirubench the external party-proxy

docker run -d --name the external party-ssh-relay --restart unless-stopped \
  -p <lan-address>:2222 \

[23] CLAIM: A1: only cloudflared has a readiness/restart story; GB10 bring-up is racy and the island daemons have no units in-tree.
POINTER: gb10_setup.sh:60
OPENED TEXT:
sleep 3
docker exec the external party-bench bash -c '
  cp -rT /etc/skel /home/the external party 2>/dev/null || true
  mkdir -p /home/the external party/.ssh && touch /home/the external party/.ssh/authorized_keys

[24] CLAIM: A2/A3: immediate checks can false-pass, and no cache-proof DNS/Access/TLS validation is recorded.
POINTER: gb10_setup.sh:60-71
OPENED TEXT:
sleep 3
docker exec the external party-bench bash -c '
  cp -rT /etc/skel /home/the external party 2>/dev/null || true
  mkdir -p /home/the external party/.ssh && touch /home/the external party/.ssh/authorized_keys
  chown -R the external party:the external party /home/the external party && chmod 700 /home/the external party/.ssh && chmod 600 /home/the external party/.ssh/authorized_keys'

echo "=== verify ==="
docker ps --filter name=the external party --format '{{.Names}}\t{{.Status}}\t{{.Ports}}'
docker exec the external party-bench nvidia-smi --query-gpu=name,driver_version --format=csv,noheader || true
docker exec the external party-bench bash -c 'timeout 3 bash -c "echo > /dev/tcp/<lan-address>/8080" 2>/dev/null && echo "LAN REACHABLE (BAD)" || echo "LAN direct: blocked (good)"'
docker exec the external party-bench bash -c 'curl -s -x http://172.30.99.3:3128 -o /dev/null -w "internet-via-squid: %{http_code}\n" --max-time 20 https://huggingface.co/api/models?limit=1'
docker exec the external party-bench bash -c 'curl -s -x http://172.30.99.3:3128 -o /dev/null -w "squid-to-LAN: %{http_code}\n" --max-time 10 http://<lan-address> || true'

[25] CLAIM: A2/A3: immediate checks can false-pass, and no cache-proof DNS/Access/TLS validation is recorded.
POINTER: gb10_setup.sh:5
OPENED TEXT:
# Outbound: squid proxy container with RFC1918 destinations denied.
# Docker-native isolation only — no host iptables/systemd changes.
set -euo pipefail

mkdir -p /home/<login-user>/the external party-bench/{home,scratch,build,proxy}

[26] CLAIM: A2/A3: immediate checks can false-pass, and no cache-proof DNS/Access/TLS validation is recorded.
POINTER: cloudflare-objects.json:20
OPENED TEXT:
"type": "self_hosted",
  "session_duration": "24h",
  "auto_redirect_to_identity": false,
  "allowed_idps": [],
  "app_launcher_visible": true

[27] CLAIM: A2/A3: immediate checks can false-pass, and no cache-proof DNS/Access/TLS validation is recorded.
POINTER: RUNBOOK.md:40-46
OPENED TEXT:
## Onboarding the external party (when email + SSH pubkey arrive)

1. Add his email to the Access policy (script `cf_add_email.py` pattern, or dash:
   Zero Trust → Access → Applications → "the external party bench SSH" → policy → add email).
2. Append his pubkey to `/home/the external party/.ssh/authorized_keys` in CT 204 AND CT 205
   (`pct exec` from node2), and in the GB10 container home once built
   (`/home/<login-user>/the external party-bench/home/.ssh/authorized_keys` on gb10).
3. Send him the client config (needs `cloudflared` installed locally):

```

[28] CLAIM: A4: three shared mutable stores are written without a lock — PBS exclude `114,205`, cluster SDN, and Docker’s published-port/iptables — matching the estate RMW lesson; skipping or clobbering them is a co-tenant backup/SDN/LAN-port failure, not a local nit.
POINTER: backup-job.json:1
OPENED TEXT:
{"all":1,"comment":"Hourly backup with deduplication - 24hr granularity","compress":"zstd","enabled":1,"exclude":"114,205","id":"cd5e47ba-81f1-4878-ac1d-aa685e22f6b1","mailnotification":"failure","mailto":"<email>","mode":"snapshot","prune-backups":{"keep-daily":"7","keep-hourly":"24","keep-monthly":"3","keep-weekly":"4"},"schedule":"hourly","storage":"iscsi-fast","type":"vzdump"}

[29] CLAIM: A4: three shared mutable stores are written without a lock — PBS exclude `114,205`, cluster SDN, and Docker’s published-port/iptables — matching the estate RMW lesson; skipping or clobbering them is a co-tenant backup/SDN/LAN-port failure, not a local nit.
POINTER: gb10-docker-inspect.txt:3
OPENED TEXT:
/the external party-proxy nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e8416a31b996db6b4024","EndpointID":"efdb9650be3b86a3ead899a73511cac7bc4968ca5c7dbfb5eb364155a0c09d51","Gateway":"172.17.0.1","IPAddress":"172.17.0.6","MacAddress":"d6:af:c8:9a:6a:75","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":null},"cirubench":{"IPAMConfig":{"IPv4Address":"172.30.99.3"},"Links":null,"Aliases":[],"DriverOpts":{},"GwPriority":0,"NetworkID":"8c1484b020c6d40bcfb6500b9f14dafc3eb3db58bc8353cf576db62d526ed4bb","EndpointID":"cf116760b8078307321df092f27dacc895f7d12303663a5da0955e998ea946be","Gateway":"","IPAddress":"172.30.99.3","MacAddress":"b6:9d:a2:cd:e6:f7","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":["the external party-proxy","ceafb1148a9c"]}} ports={} mounts=["/home/<login-user>/the external party-bench/proxy/squid.conf:/etc/squid/squid.conf:ro"] mem=0 restart=unless-stopped
/the external party-ssh-relay nets={"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"DriverOpts":null,"GwPriority":0,"NetworkID":"d58bfad862ce40cbb9d6950eeb58dbaf6144dbf03d49e841

[30] CLAIM: A4: three shared mutable stores are written without a lock — PBS exclude `114,205`, cluster SDN, and Docker’s published-port/iptables — matching the estate RMW lesson; skipping or clobbering them is a co-tenant backup/SDN/LAN-port failure, not a local nit.
POINTER: gb10_setup.sh:5
OPENED TEXT:
# Outbound: squid proxy container with RFC1918 destinations denied.
# Docker-native isolation only — no host iptables/systemd changes.
set -euo pipefail

mkdir -p /home/<login-user>/the external party-bench/{home,scratch,build,proxy}

[31] CLAIM: A4: three shared mutable stores are written without a lock — PBS exclude `114,205`, cluster SDN, and Docker’s published-port/iptables — matching the estate RMW lesson; skipping or clobbering them is a co-tenant backup/SDN/LAN-port failure, not a local nit.
POINTER: ct205.conf:16-20
OPENED TEXT:
lxc.cgroup2.devices.allow: c 510:* rwm
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file

[32] CLAIM: A4: three shared mutable stores are written without a lock — PBS exclude `114,205`, cluster SDN, and Docker’s published-port/iptables — matching the estate RMW lesson; skipping or clobbering them is a co-tenant backup/SDN/LAN-port failure, not a local nit.
POINTER: RUNBOOK.md:25-26
OPENED TEXT:
on SDN vnet `cirub0` (zone `cirub`, node2-only, NO uplink — physical L2 island,
  no NAT anywhere). GPU via /dev/nvidia* bind mounts + cgroup allows c 195/510
  (same pattern as CT 201). NVIDIA user-space 610.43.02 via .run --no-kernel-modules
  (matches node2 host driver EXACTLY — if node2 driver is upgraded, rerun the .run
  in 205 with the new version). User `the external party` HAS passwordless sudo (his sandbox).
  Internet only via bastion tinyproxy (http/https, set in /etc/profile.d/proxy.sh,

[33] CLAIM: A5: `gb10_setup.sh` is resume-safe (`rm -f` + `network inspect || create`) but documented teardown does not restore production or remove every created object, and the external party keys/email are not installed yet so `from=` restrictions cannot be assumed.
POINTER: gb10_setup.sh:38-41
OPENED TEXT:
docker network inspect cirubench >/dev/null 2>&1 || \
  docker network create --internal --subnet 172.30.99.0/24 cirubench

docker rm -f the external party-bench the external party-proxy the external party-ssh-relay 2>/dev/null || true

docker run -d --name the external party-proxy --restart unless-stopped \
  -v /home/<login-user>/the external party-bench/proxy/squid.conf:/etc/squid/squid.conf:ro \

[34] CLAIM: A5: `gb10_setup.sh` is resume-safe (`rm -f` + `network inspect || create`) but documented teardown does not restore production or remove every created object, and the external party keys/email are not installed yet so `from=` restrictions cannot be assumed.
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

[35] CLAIM: A5: `gb10_setup.sh` is resume-safe (`rm -f` + `network inspect || create`) but documented teardown does not restore production or remove every created object, and the external party keys/email are not installed yet so `from=` restrictions cannot be assumed.
POINTER: ct205-services.conf:5
OPENED TEXT:
AllowUsers the external party
the external party ALL=(ALL) NOPASSWD:ALL
Defaults:the external party env_keep += "http_proxy https_proxy HTTP_PROXY HTTPS_PROXY no_proxy"
export http_proxy=http://<island-address>:8888
export https_proxy=http://<island-address>:8888

[36] CLAIM: A5: `gb10_setup.sh` is resume-safe (`rm -f` + `network inspect || create`) but documented teardown does not restore production or remove every created object, and the external party keys/email are not installed yet so `from=` restrictions cannot be assumed.
POINTER: gb10_setup.sh:15
OPENED TEXT:
useradd -m -s /bin/bash the external party && \
    echo 'the external party ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/the external party && chmod 440 /etc/sudoers.d/the external party && \
    printf 'PermitRootLogin no\nPasswordAuthentication no\nKbdInteractiveAuthentication no\nAllowUsers the external party\n' > /etc/ssh/sshd_config.d/90-bench.conf && \
    printf 'http_proxy=http://172.30.99.3:3128\nhttps_proxy=http://172.30.99.3:3128\nHTTP_PROXY=http://172.30.99.3:3128\nHTTPS_PROXY=http://172.30.99.3:3128\nno_proxy=localhost,127.0.0.1\n' >> /etc/environment
EXPOSE 22

[37] CLAIM: A5: `gb10_setup.sh` is resume-safe (`rm -f` + `network inspect || create`) but documented teardown does not restore production or remove every created object, and the external party keys/email are not installed yet so `from=` restrictions cannot be assumed.
POINTER: RUNBOOK.md:40-45
OPENED TEXT:
## Onboarding the external party (when email + SSH pubkey arrive)

1. Add his email to the Access policy (script `cf_add_email.py` pattern, or dash:
   Zero Trust → Access → Applications → "the external party bench SSH" → policy → add email).
2. Append his pubkey to `/home/the external party/.ssh/authorized_keys` in CT 204 AND CT 205
   (`pct exec` from node2), and in the GB10 container home once built
   (`/home/<login-user>/the external party-bench/home/.ssh/authorized_keys` on gb10).
3. Send him the client config (needs `cloudflared` installed locally):

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }
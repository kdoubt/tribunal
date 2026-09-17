# Oracles — g06 (orchestrator, non-LLM; pointer resolution only, no support ratings)

- S_claude.md: 8 claims, 19 file:line pointers, 19 open in the artifact (S_claude.grounding.json, from tools/grounding.py).
- S_codex.md: 8 claims, 34 pointers (`artifact/…` prefix, en-dash ranges, comma lists), 34 open (S_codex.grounding.json, resolved by an inline parser with the same rule because grounding.py splits only on plain `CLAIM:` headings).
- Verified by inspection (the cited text plainly states the fact):
  - synology-exports.txt:2-3: `/volume1/proxmoxbackup/…(rw,…,no_root_squash,…) *` and `/volume1/proxmox…(rw,…,crossmnt,no_root_squash,…) *`; lines 4-6 list a client IP and no options. (A1 facts, A8, B3, B4)
  - unified-nfs-design.md:16-22: one share holds workspace (RW to the party), models, and queue/<host>/{incoming,running,done}. (A4, B1)
  - unified-nfs-design.md:27-28: CT 206 "isolated, bastion-reachable" with no vnet named; RUNBOOK.md:24 pins CT 205 to `cirub0`, NO uplink. (A1 premise)
  - unified-nfs-design.md:44-45: `vers=3, soft, timeo=150, retrans=3, _netdev, nofail, nolock`, workspace/queue RW; gb10-nfs-precedent.txt:1 is `ro,…,soft`. (A3, A7, B5)
  - unified-nfs-design.md:46-47: anon uid 1025 vs sandbox users 1000/1001; "bind mounts run as that uid"; RUNBOOK.md:23 CT 205 unprivileged. (A2, B7)
  - unified-nfs-design.md:40-47 names no quota; design:10-11 "tens of GB"; exports show /volume1 shared with proxmoxbackup. (A5, B3)
  - CONTRACT.md:22-30 keeps arbiter.lock/state.json/window.flag/heartbeat on local paths; design:30-31 moves only the spool; CONTRACT.md:14-16 dead-man. (A6 premise, B8 pointers)
  - CONTRACT.md:73-75 atomic claim + interrupted-job rule; CONTRACT.md:12 single-writer. (A4, B1, B6)
  - CONTRACT.md:60-61 readiness probe; RUNBOOK.md:71 "active ≠ healthy". (A7, B5)
  - RUNBOOK.md:115-127 helper dual-homing residual; :96-101 teardown scope. (B2, B8)
- Not settled by any artifact text (stay PANELIST-CLAIM / EXTERNAL): CT 206's eventual network placement; uid-shift EPERM behaviour; NFSv3 rename/CTO semantics; whether existing scripts are NFS-independent (no scripts in the artifact).

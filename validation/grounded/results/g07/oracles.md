# Oracles — g07 (orchestrator, mechanical; no support ratings)

Pointer resolution (grounding.py): codex 15/15 open, grok 28/28 open. Every cited span was then opened by hand.

Settled by inspection (text plainly contains what the claim says):
- rtx:30-41 close = `systemctl start` + health loop; no `pct stop 205` anywhere in the script → A2 "does not revoke GPU access" holds for the script.
- gb10:35-48 close = `docker update --memory 20g the external party-bench` + `docker start` prod; the external party-bench is not stopped → A2 holds.
- RUNBOOK:103-105 verbatim: "A driver-crashing workload still hits prod vLLM; the window scripts' health gates are the guard, not a hardware boundary." → A2/B8 driver-crash fact.
- rtx:29 and gb10:34: failed open prints NO-GO and `exit 1`; no restart of what was stopped → A5 "no rollback on failed open".
- rtx:12-14, gb10:14-19: `ssh`/`curl` helpers carry no ConnectTimeout / `-m` → A5 "no deadline" (loop counts do not bound a blocked call).
- rtx:41, gb10:48: after 100×6 s without 200 → WARNING + `exit 1`, prod left in whatever state → A5/B6.
- rtx:11,26: FREE_MIB=4000 threshold; gb10:13,29-31: FREE_GB=60, `docker update ... >/dev/null` with no exit check → A6.
- rtx:19-20: `pvesh ... 2>/dev/null | grep -q ... && ABORT` — a pvesh error yields no match, so the open proceeds → A6 "fail-open backup check".
- qd:33-34: spool path `/scratch/queue/incoming/<id>.json` "INSIDE his sandbox" → B2 fact.
- qd:37-39, 40-47: window opened by operator; close by operator or max-window cap; budgets "enforced by the arbiter" — no second process named → B3 fact (extension path is the same process).
- qd:64-65: crash/reboot mid-drain listed as an open question → A8/B4.
- RUNBOOK:23: CT 205 `onboot=0`; RUNBOOK:28: user has passwordless sudo in the sandbox; RUNBOOK:38: GB10 state under the host home; gb10:30-31: cap 115g of 121G → A4/B7/B8 facts.
- qd:61-63: spool fill / wedge / outlive-timeout listed as open → A3.
- qd:16, 23-27: "minutes-scale" cold start; "twice" per job → A1/B1 text. A1's remark that "twice" is unsupported is a reading, not settled here.

Left PANELIST-CLAIM (recommendations or inferences, not settled by text): B2's "trusted-side spool is the better path"; A1's "transport not established"; severity rankings (A2 vs B4); A3's specific control list; B8's RTX disk-fill (self-labelled ASSUMPTION).

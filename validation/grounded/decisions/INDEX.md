# Grounded decision set - index

Ten decisions taken from the maintainer's panel archive (2026-08-19 to
2026-09-13), each with a known outcome. Arms see `gNN-brief.md` plus the
staged artifact; `sealed/gNN-truth.md` is never shown to an arm.

| id | name | artifact source | difficulty | must_catch | original panel missed a must_catch? | placeholders used |
|---|---|---|---|---|---|---|
| g01 | whipnode init onboarding flow | private repo @ 0def8e9 (staged full tree) | design | 6 | no | methodology repo |
| g02 | whipnode zero-click trial ladder | private repo @ 0def8e9 | subtle | 7 | no | none |
| g03 | onboarding spec review | private repo @ 3375cdf + `ONBOARDING_SPEC.md` | subtle | 7 | **yes** (which item is withheld here; see the sealed rubric) | none |
| g04 | whipnode landing page vs agent adopter | inline capture pack + repo @ 3375cdf (`apps/web`) | control | 7 | no | methodology repo |
| g05 | external-party GPU bench access | evidence dir (13 files) | subtle | 7 | no | RTX host, GB10 host, flat LAN, secrets store, backup server, bench hostname, LAN addresses, owner |
| g06 | unified NFS workspace for external party | evidence dir (5 files) | subtle | 6 | no | RTX host, GB10 host, arbiter host, NAS |
| g07 | benchmark job queue on shared GPUs | evidence dir (4 files; window scripts pre-fix) | subtle | 7 | no | RTX host, GB10 host + login user |
| g08 | NATS spine newsletter + 2.14.5 upgrade | evidence dir (8 files) | subtle | 8 | no (drain-time surprise is an outcome note, not a review miss) | vendor newsletter, hosts |
| g09 | public demo funnel fitness | artifact snapshot (110 files, deployed tree + captures) | subtle | 11 | no | capture date |
| g10 | interim LLM backstop during GPU lend | brief only | control | 7 | no | owner, secrets store, AI Gateway, custody router, assistant, another project |

Difficulty labels: `control` = both original seats agreed on everything at
Round 0 (expected finding: every arm gets it, panel adds cost); `subtle` = an
oracle-verified defect or a confident wrong claim existed in the original run;
`design` = no single correct call, scored on must_catch and landmines only.

Provenance rule: every `must_catch` item cites a `file:line` in the staged
artifact that the drafter opened, or is marked `unverifiable` in the truth.

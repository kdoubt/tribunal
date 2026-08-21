# Pilot 02 - results (n=3, ambiguous decisions, blind judge)

**Run:** 2026-08-21. **Seats:** Codex CLI (GPT-5-class), Grok CLI (Grok-4-class).
**Judge:** Claude CLI (third vendor, blind to arm identity, order pre-fixed - see
[`pilot-02/BLIND-ORDER.md`](pilot-02/BLIND-ORDER.md)). **Arms:** A1 = Codex alone;
A3 = full Tribunal (same Codex + Grok, R0 → R1 where disputed → verdict).
**Raw memos + seat rounds + judge outputs:** [`pilot-02/`](pilot-02/).

## Headline

**The panel did not win. Solo 2, panel 1.** On three genuinely-ambiguous
decisions, a blind judge preferred the single-model memo twice and the panel
memo once. More useful than the score are the *reasons the panel lost*, because
they cut at the method's core value claim.

| decision | seats agreed? | R1 ran? | solo call | panel call | judge total (solo / panel) | winner |
|----|----|----|----|----|----|----|
| P1 event-sourcing vs CRUD | yes (both B) | no (early stop) | B | B | 11 / 12 | **panel** |
| P2 service mesh now vs defer | **no** (A vs B) | **yes** | B | surviving dissent → decide-after-test | 11 / 10 | **solo** |
| P3 pgvector vs dedicated | yes (both A) | no (early stop) | A | A | 12 / 10 | **solo** |

Only **P2** exercised Round 1 at all - the other two early-stopped because both
seats independently agreed (as they did on all of pilot-01). So across both
pilots, **8 of 9 decisions produced no seat disagreement**; the heterogeneous
panel mostly just confirms what one strong model already concludes.

## Why the panel lost (the actual finding)

- **P2 - honesty about ambiguity read as indecision.** The seats genuinely
  disagreed (Codex: adopt a narrow mesh now; Grok: defer). Round 1 worked well -
  both conceded real points, neither flipped, and they **converged on the same
  discriminating test** and narrowed the dispute to one question (can uniform
  workload identity be achieved in-process, or does a hop need a sidecar?). The
  panel's honest output was *surviving dissent → run this 2-week spike, then
  decide*. The blind judge **docked it** (decisiveness 1/3) for "convert[ing] the
  decision into another two weeks of process" instead of committing to A or B as
  the brief demanded, and picked the solo memo's confident "B, with tripwires."
  **This is the load-bearing result: Tribunal's signature move - preserve
  surviving dissent, decide-after-check - scored *worse* than a decisive solo
  answer, even though the dissent was real.**
- **P3 - more evidence, more surface to be wrong.** Both agreed on A (pgvector).
  The panel memo carried the second seat's benchmark citations, one of which
  (a 70-160 ms filtered-query band) *contradicts* the brief's tens-of-ms target;
  the judge caught the unreconciled tension and called it false confidence. The
  tighter solo memo, carrying less, had less to get wrong. **Direct
  counter-evidence to "more vendors catch more."**
- **P1 - the one win, and it was narrow.** Both agreed on B. The panel memo won
  12-11 by converting the shared risk into a concrete enforcement mechanism
  (in-DB trigger + revoked UPDATE/DELETE) and a tighter test, with citations the
  judge deemed load-bearing. Real, but modest, and on an agreed call.

## Confounds (which cut both ways)

1. **n=3, single judge, same vendor as the orchestrator.** Small and not
   independent of the tool's own vendor. Memos are published so anyone can
   re-judge with any model.
2. **The A3 "panel memo" was orchestrator-assembled** from seat content. This
   could flatter the panel (better prose than the raw seats) *or* hurt it (P3's
   unreconciled benchmark was an assembly choice - a more careful synthesis might
   have dropped it). Either way, the panel's output is partly the orchestrator's,
   which is inherent to the method but a real confound here.
3. **Effort/length asymmetry:** the solo arm wrote to ~400 words; the panel drew
   on ~2×450 words of seat material. If anything this favored the panel, and it
   still lost the set.
4. **The rubric rewards committing to A/B** (per the briefs). A decision-maker
   who already knows a question is hard might *prefer* the panel's "here's the
   deciding test" over a confident commit - the P2 result might invert under a
   rubric that valued correctly-flagged ambiguity. That is a real open question,
   not a rescue: the briefs asked for a commitment and the panel didn't give one.

## What this means (honestly)

- **No evidence of panel lift, and some evidence against it.** Combined with
  pilot-01's null, two pilots now show the panel either matching or losing to a
  single strong model, at higher cost. The one win was on an agreed call and was
  marginal.
- **The seats rarely disagree** (1 of 9 across both pilots). Tribunal's engine
  assumes productive disagreement; frontier models mostly converge, so the
  machinery usually idles (early stop) and A3 collapses to A2 collapses to ~A1.
- **The method's honesty may be a scoring liability.** Preserved dissent and
  extra evidence - the things that make Tribunal *honest* - are exactly what cost
  it P2 and P3. That is worth stating plainly on the tin: Tribunal optimizes for
  *not being confidently wrong*, which is not the same as *scoring best on a
  "give me a committed answer" rubric*.
- **Where a real edge might still live** (untested here, and not to be assumed):
  a decision-maker who explicitly wants the strongest **counter-case** and the
  **cheapest discriminating test** rather than a confident recommendation. P2's
  panel memo delivered exactly that - and a rubric built for that buyer, not a
  commitment-seeking one, is the honest next experiment. It must be
  pre-registered before running, and it must be allowed to fail too.

## Bottom line

Two pre-registered pilots, published in full including the losses: **Tribunal has
not shown lift over a single strong model, on either decidable or ambiguous
decisions, and its dissent-preserving honesty can score worse than a decisive
solo answer.** That is the current honest state of the evidence. It argues for
narrowing Tribunal's claims to the specific buyer who wants an adversarial
counter-case plus a discriminating test - and for far more runs before any
stronger claim.

# Pilot 02 - results (n=3, ambiguous decisions, blind judge)

**Run:** 2026-08-21. **Seats:** Codex CLI (GPT-5-class), Grok CLI (Grok-4-class).
**Arms:** A1 = Codex alone; A3 = full Tribunal (same Codex + Grok, R0 → R1 where
disputed → verdict). **Judges:** one independent (Claude CLI, the third vendor -
run **twice**, order flipped) plus one **conflicted** robustness check (Grok,
whose own work is in the panel memos). All blind to arm identity; order pre-fixed
per pass - see [`pilot-02/BLIND-ORDER.md`](pilot-02/BLIND-ORDER.md).
**Raw memos + seat rounds + every judge output:** [`pilot-02/`](pilot-02/).

## Headline

**No robust signal either way - the outcome flips with the judge.** Under the
independent judge (Claude, both passes) the single-model memo won 2 of 3; under
a second judge *vendor* (Grok) it inverted to the panel winning 2 of 3. The two
judge vendors are **anti-correlated on all three decisions**. So the honest
result is not "solo wins" - it is that "*which memo is more decision-useful*" for
an ambiguous call is **judge-dependent**, and the single-judge reading in the
first version of this file did **not** survive a second judge.

| decision | seats agreed? | R1 ran? | Claude judge (×2) | Grok judge (conflicted) |
|----|----|----|----|----|
| P1 event-sourcing vs CRUD | yes (both B) | no (early stop) | **panel**, **panel** | solo |
| P2 service mesh now vs defer | **no** (A vs B) | **yes** | **solo**, **solo** | panel |
| P3 pgvector vs dedicated | yes (both A) | no (early stop) | **solo**, **solo** | panel |

Independent judge: **panel 1 / solo 2, stable under an order flip.** Conflicted
judge: **solo 1 / panel 2.** See "Robustness" below.

Only **P2** exercised Round 1 at all - the other two early-stopped because both
seats independently agreed (as they did on all of pilot-01). So across both
pilots, **8 of 9 decisions produced no seat disagreement**; the heterogeneous
panel mostly just confirms what one strong model already concludes.

## Why the *independent* judge scored as it did (and where the second judge disagreed)

The three bullets below explain the **Claude** judge's reasoning (the
independent one, panel 1 / solo 2). The **Grok** judge inverted every call - the
"Robustness" section next explains why that both matters and is contaminated.

- **P2 - honesty about ambiguity read (by Claude) as indecision.** The seats
  genuinely disagreed (Codex: adopt a narrow mesh now; Grok: defer). Round 1
  worked well - both conceded real points, neither flipped, and they **converged
  on the same discriminating test**, narrowing the dispute to one question (can
  uniform workload identity be achieved in-process, or does a hop need a
  sidecar?). The panel's honest output was *surviving dissent → run this 2-week
  spike, then decide*. **The two judges split precisely here:** Claude **docked
  it** (decisiveness 1/3) for "convert[ing] the decision into another two weeks
  of process" instead of committing to A or B as the brief demanded; the Grok
  judge **rewarded the same memo** for "isolat[ing] the one disagreement that
  would actually change the call" and naming "a cheaper test that compares both
  paths, while [the solo] decides first and measures regret later." That is not
  noise - it is the crux of Tribunal's value proposition: is *"here is the one
  test that settles it"* more or less useful than *"commit to B with tripwires"*?
  Reasonable reviewers genuinely disagree, and the brief asked for a commitment.
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

## Robustness - three judge passes, and a complete inversion

To finish the pilot, the judging was repeated (the pre-registration allowed for a
second judge):

- **Independent judge, second pass (Claude, order flipped): identical result -
  panel 1 / solo 2.** So within the independent judge, the outcome is *stable*
  against position/sampling; it is not a fluke of one pass.
- **Second judge vendor (Grok): inverts every decision - solo 1 / panel 2.**
  Grok is a **conflicted** judge (its own Round-0/Round-1 text is inside the panel
  memos), so its pro-panel lean cannot be cleanly separated from bias. Two things
  keep it informative anyway: it picked the *solo* memo on P1 (so it is not
  blindly pro-panel), and on P2 it articulated a coherent, defensible reason to
  prefer the panel's decide-after-test - the mirror image of Claude's reason to
  dock it.
- **Net:** the two judge *vendors* are **anti-correlated on all three
  decisions**. The apparent "solo 2 / panel 1" of the single-judge version is a
  Claude-judge reading, not a robust fact; a different judge vendor reverses it.

The honest conclusion is therefore stronger and humbler than "solo wins": on
ambiguous decisions, **whether the panel or the solo memo is 'more
decision-useful' is not robustly measurable - it depends on the judge, and the
judges disagree exactly on whether preserved dissent + a discriminating test
beats a committed answer.** A clean test would need an independent judge that is
*not* one of the seats and *not* the orchestrator's vendor (a fourth model) -
which this run could not reach.

## Confounds (which cut both ways)

1. **n=3; the only clean judge (Claude) shares the orchestrator's vendor.** If
   that biases anything it should favor the *panel* (the methodology's own
   vendor), yet Claude picked solo 2/3 - so the confound runs *against* the
   pro-solo reading, not for it. The one non-seat, non-orchestrator vendor needed
   for a clean second opinion was not reachable here. Memos are published so
   anyone can re-judge with any model.
2. **The A3 "panel memo" was orchestrator-assembled** from seat content. This
   could flatter the panel (better prose than the raw seats) *or* hurt it (P3's
   unreconciled benchmark was an assembly choice - a more careful synthesis might
   have dropped it). Either way, the panel's output is partly the orchestrator's,
   which is inherent to the method but a real confound here.
3. **Effort/length asymmetry:** the solo arm wrote to ~400 words; the panel drew
   on ~2×450 words of seat material. If anything this favored the panel, and it
   still lost the set.
4. **The rubric/judge rewards committing to A/B** (per the briefs) - and this is
   no longer hypothetical: the P2 result **did** invert between judges, with Grok
   valuing the panel's "here's the deciding test" exactly where Claude penalized
   its refusal to commit. That is not a rescue for the panel (the briefs asked for
   a commitment, and the clean judge weighted that), but it is direct evidence
   that the comparison is a values choice, not a fact.

## What this means (honestly)

- **No *robust* evidence of panel lift - the sign flips with the judge.** The
  independent judge favored solo 2/3 (stable across an order flip); a second judge
  vendor reversed it. Combined with pilot-01's null, the fair summary is: two
  pilots produced **no reliable lift signal**, at strictly higher cost for the
  panel.
- **The seats rarely disagree** (1 of 9 across both pilots). Tribunal's engine
  assumes productive disagreement; frontier models mostly converge, so the
  machinery usually idles (early stop) and A3 collapses to A2 collapses to ~A1.
  This is the most robust finding in either pilot, and it is judge-independent.
- **The method's honesty is judged both ways.** Preserved dissent + extra
  evidence - what makes Tribunal *honest* - was a liability under the
  commitment-weighting judge (Claude on P2/P3) and an asset under the
  test-weighting judge (Grok on P2/P3). Tribunal optimizes for *not being
  confidently wrong* and for *naming the deciding test*; whether that reads as
  "more useful" is exactly the thing the two judges disagree about.
- **Where a real edge might still live** (untested here, and not to be assumed):
  a decision-maker who explicitly wants the strongest **counter-case** and the
  **cheapest discriminating test** rather than a confident recommendation. P2's
  panel memo delivered exactly that - and a rubric built for that buyer, not a
  commitment-seeking one, is the honest next experiment. It must be
  pre-registered before running, and it must be allowed to fail too.

## Bottom line

Two pre-registered pilots, published in full including the losses and the judge
disagreement: **Tribunal has not shown *robust* lift over a single strong model.**
On decidable questions a strong model already gets them right (pilot-01 null); on
ambiguous ones, whether the panel or the solo memo is "more useful" **flips with
the judge**, because the judges disagree on whether a preserved dissent plus a
discriminating test beats a committed answer. The one judge-independent result is
that frontier seats rarely disagree at all (1 of 9), so the panel machinery mostly
idles. This argues for narrowing Tribunal's claims to the specific buyer who wants
an adversarial counter-case plus a discriminating test on a genuinely irreversible
call - and for a proper study (a fourth-vendor judge, larger n, real outcomes)
before any stronger claim. The narrowed README/METHODOLOGY (v0.6.5) already
reflect this.

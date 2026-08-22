# Pilot 02 - results (n=3, ambiguous decisions, blind judge)

**Run:** 2026-08-21. **Seats:** Codex CLI (GPT-5-class), Grok CLI (Grok-4-class).
**Arms:** A1 = Codex alone; A3 = full Tribunal (same Codex + Grok, R0 → R1 where
disputed → verdict). **Judges:** two independent - Claude CLI (third vendor,
run **twice**, order flipped) and **gpt-oss-120b** (a genuinely independent
fourth *model* (OpenAI-lineage), added once the confirmatory harness came online) - plus one
**conflicted** robustness check (Grok, whose own work is in the panel memos). All
blind to arm identity; order pre-fixed per pass - see
[`pilot-02/BLIND-ORDER.md`](pilot-02/BLIND-ORDER.md).
**Raw memos + seat rounds + every judge output:** [`pilot-02/`](pilot-02/).

## Headline

**No lift: on aggregate rubric score, solo ≥ panel under *every* judge.** An
earlier version of this file called the result "judge-dependent / flips with the
judge." A re-review (Codex) correctly showed that over-weighted the
forced-choice line; **corrected here.** On the rubric totals the two arms were
scored on:

| judge | solo aggregate | panel aggregate | winner (points) | forced-choice tally |
|----|----|----|----|----|
| Claude - pass 1 (independent) | **34** | 32 | solo | solo 2, panel 1 |
| Claude - pass 2 (independent, order flipped) | **34** | 31 | solo | solo 2, tie 1 |
| **gpt-oss-120b (independent 4th *model*, OpenAI-lineage, added later)** | 32 | 32 | **tie** | solo 1, panel 1, tie 1 |
| Grok (CONFLICTED - its own text is in the panel memos) | 32 | 32 | **tie** | solo 1, panel 2 |

**The panel never won on points under any judge** - including a genuinely
independent fourth *model* (gpt-oss-120b, OpenAI-lineage, neither seat nor the orchestrator's
vendor), added when the confirmatory harness came online: it scored a dead
**tie** on aggregate, corroborating "no panel lift" rather than reversing it. The only thing that inverted
was the *forced choice* of the **conflicted** judge (Grok, which authored part of
the panel memos) - and even that was on a tied aggregate score. So the honest
statement is: *solo scored at least as well as the panel under every judge,
including the conflicted one; only the conflicted judge's tie-break preferred the
panel.* That is a **stronger no-lift finding**, not a "flip."

Per-decision forced choices (for the earlier framing, now de-emphasized):

| decision | seats agreed? | R1 ran? | Claude ×2 | Grok (conflicted) |
|----|----|----|----|----|
| P1 event-sourcing vs CRUD | yes (both B) | no (early stop) | panel, panel | solo |
| P2 service mesh now vs defer | **no** (A vs B) | **yes** | solo, solo | panel |
| P3 pgvector vs dedicated | yes (both A) | no (early stop) | solo, solo | panel |

Only **P2** exercised Round 1 at all - the other two early-stopped because both
seats independently agreed (as they did on all of pilot-01). So across both
pilots, **8 of 9 decisions produced no seat disagreement** (strictly, 8 of 8
*measurable* - pilot-01 d3 had a dead seat, see that file); the heterogeneous
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

## Robustness - three judge passes (what actually moved, and what didn't)

To finish the pilot, the judging was repeated (the pre-registration allowed for a
second judge). Distinguish the two metrics the pre-registration named -
**aggregate rubric score** and **forced choice**:

- **Aggregate score never favored the panel** (34-32, 34-31, 32-32 in the table
  above). This is the robust metric and it says *solo ≥ panel under every judge*.
- **Independent judge, second pass (Claude, order flipped):** reproduced pass 1
  on both metrics (solo on points, solo 2 / panel 1 on forced choice). So the
  independent judge's result is *stable* against position/sampling.
- **Second judge vendor (Grok):** aggregate **tied**; only its *forced choice*
  went panel 2 / solo 1. Grok is a **conflicted** judge - its own Round-0/Round-1
  text is inside the panel memos - so any pro-panel lean cannot be separated from
  bias. It stays informative only because it picked the *solo* memo on P1 and gave
  a coherent reason on P2; it does **not** support the claim that "an independent
  judge reversed the result." No independent judge reversed anything.
- **Correction:** an earlier version of this file said the result "inverts /
  flips with the judge." That over-weighted the conflicted judge's forced choice.
  What actually happened: **the panel never out-scored the solo memo under any
  judge; a conflicted judge's tie-break preferred it on 2 of 3.**

The honest conclusion, corrected: on these three ambiguous decisions the panel
showed **no aggregate lift under any judge** - including the independent
independent gpt-oss judge (a fourth model, OpenAI-lineage), which tied. The one metric that ever favored the
panel (forced choice) moved only under the *conflicted* judge. This is still a
pilot (n=3, orchestrator-assembled memos); the pre-registered
[`../confirmatory/`](../confirmatory/) study - component-isolating arms, the
independent judge (fourth model) as primary, n ≥ 20, aggregate score as the primary endpoint -
is the confirmatory follow-up, and it inherits this no-lift prior.

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
4. **The rubric/judge rewards committing to A/B** (per the briefs). On P2 the
   *forced choice* split - Grok valued the panel's "here's the deciding test"
   where Claude penalized its refusal to commit - but on **aggregate score** the
   solo memo was ahead or tied for every judge, so this is a tie-break-level
   effect, not a reversal. It shows the commit-vs-name-the-test question is a
   values choice at the margin, not that the panel produces more useful output.
5. **The four arms don't cleanly isolate components** (a design limitation, not a
   reporting one): A1 may consult an oracle, A2 does not, A3 does - so A3-vs-A2
   changes both Round 1 *and* oracle access. The pilot compares *whole arms*, not
   isolated mechanisms; read it as "does the full panel beat a solo model," not
   "what does Round 1 add." And Round 1 itself ran on only 1 of 3 decisions here
   (P2), so its specific value is essentially untested.

## What this means (honestly)

- **No aggregate lift under any judge.** On the rubric totals, solo ≥ panel for
  all three judges (solo won under both independent Claude passes, tied under the
  conflicted Grok judge). Only a conflicted judge's *forced choice* preferred the
  panel. Combined with pilot-01's null, two pilots produced **no lift signal**, at
  strictly higher cost for the panel. (An earlier version of this file called
  this "judge-dependent / flips" - that over-weighted forced choice and is
  corrected.)
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

Two pre-registered pilots, published in full including the losses:
**Tribunal has shown no lift over a single strong model.** On decidable questions
a strong model already gets them right (pilot-01 null). On ambiguous ones the
solo memo scored **≥** the panel on aggregate under every judge (including a
conflicted one that only tie-broke toward the panel on forced choice). The one
judge-independent result is that frontier seats rarely disagree at all (1 of 8
measurable), so the panel machinery mostly idles. This argues for narrowing
Tribunal's claims to the specific buyer who wants an adversarial counter-case plus
a discriminating test on a genuinely irreversible call - and for a proper
confirmatory study (a non-OpenAI independent judge, larger n, real outcomes,
component-isolating arms) before any stronger claim. The narrowed
README/METHODOLOGY (v0.6.5) already reflect the no-lift finding; this file's
earlier "flips with the judge" framing was corrected after a re-review.

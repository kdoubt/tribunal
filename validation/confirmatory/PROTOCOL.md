# Confirmatory study - protocol (pre-registration)

**Status: pre-registration.** Committed before any confirmatory run. The two
[pilots](../results/) were exploratory and showed **no lift**; a re-review
(Codex) correctly listed four reasons they cannot *confirm* anything. This study
is built to remove all four, so its result - lift or no lift - is load-bearing.
If a later commit changes an arm, a decision, the scorer, or the endpoint, that
is flagged in `CHANGELOG.md` and the affected runs are redone.

## The four exploratory weaknesses this fixes

1. **Judge was the orchestrator's vendor** (Claude), and the only clean judge.
   → **Fix:** the judge is a **fourth *model* (independent of the seats and orchestrator, though OpenAI-lineage, so not a fourth vendor) that is neither seat nor the
   orchestrator** (see Judge). No seat judges its own work; the orchestrator never
   judges.
2. **Arms didn't isolate components** - oracle access varied across arms, so
   "A3 vs A2" changed both Round 1 *and* oracle access.
   → **Fix:** oracle access is **held constant (available) in every arm**, so each
   comparison moves exactly one variable (see Arms).
3. **The panel memo was orchestrator-assembled**, which could flatter or hurt it.
   → **Fix:** the panel's judged output is a **mechanical template fill** from the
   ledger (seat-sourced sentences only, no orchestrator prose); a **control
   subset** is *also* judged as the raw two-seat concatenation, to detect whether
   assembly is doing the work (see Panel-output control).
4. **n was tiny (3-5), scorer undisclosed, blinding was instruction-based.**
   → **Fix:** **n ≥ 20** across a fixed set; a **disclosed independent scorer**
   (not the orchestrator); **access-separated** sealed truth (see Blinding).

## Arms (oracle access constant in all four)

Every arm may consult the oracle (tests / compiler / primary docs); nothing else
differs between the arms it is compared against.

| arm | composition | isolates |
|----|----|----|
| **A_solo** | the *stronger* of two single-seat runs (Codex-solo, Grok-solo), each with oracle access; pick the higher-scored solo **per decision, blind**, as the baseline | one strong model + tools |
| **A_2seat** | Codex + Grok, isolated Round 0, oracle, **no Round 1** (positions taken as-is / agreement) | + the second vendor (vs A_solo) |
| **A_panel** | Codex + Grok, isolated Round 0 → ledger → **Round 1** cross-exam → oracle → verdict | + Round 1 debate (vs A_2seat) |

**The two pre-registered comparisons:**

- **Primary — value of Round 1:** `A_panel − A_2seat`. Oracle and vendor-count are
  identical; the *only* difference is the cross-examination. This is the method's
  load-bearing claim and it was never cleanly tested in the pilots.
- **Secondary — value of the second vendor:** `A_2seat − A_solo`. Oracle held
  constant; the only difference is one vs two vendors.

Running Grok-solo as well as Codex-solo removes the pilots' Codex-only-baseline
bias and gives each vendor a fair shot at being the strong solo.

## Judge (independent of the seats and orchestrator; a fourth model, not a fourth vendor)

- **Primary judge: `gpt-oss-120b`** served locally (OpenAI-compatible endpoint),
  **temperature 0** for reproducibility. It is **not** one of the seats (Codex /
  Grok) and **not** the orchestrator (Claude). Caveat, stated: it shares OpenAI
  lineage with the Codex seat - but Codex appears in *both* A_solo and A_panel, so
  any GPT-style affinity affects both arms and largely cancels; still, a
  **non-OpenAI second judge (Meta Llama or Alibaba Qwen via the gateway) is the
  designated upgrade** and will be added when a gateway key is available.
- **Blind:** memos are stripped of arm labels, given neutral names, order
  **randomized per decision** and recorded before judging.
- **Both metrics reported** (the pilot correction): **aggregate rubric score**
  (the primary endpoint) *and* forced choice. The pilots' error was leaning on
  forced choice; here the aggregate score is primary and forced choice is
  secondary.
- The judge is a plain API call, so its inputs/outputs are saved verbatim and
  the score is fully reproducible by re-running the same prompt.

## Scoring & the independent scorer

- **Ambiguous decisions** (no ground truth): scored by the judge model on the
  fixed usefulness rubric (risks / discrimination / honesty+decisiveness /
  freedom-from-padding), 0-3 each.
- **Known-answer decisions:** `decision_correct` is settled by the **oracle**
  (a runnable test or a primary-doc quote in the sealed rubric), not opinion;
  `must_catch_rate` is computed per arm and **reported per arm** (the pilot-01
  gap). The party applying the rubric is the **judge model + the mechanical
  oracle**, explicitly **not the orchestrator**.

## Blinding (access-separated)

Sealed truth lives in `confirmatory/sealed/` (git-ignored during a run is not
enough - the seats never receive that path and run with a **cwd that does not
contain it**). Seats get the brief **inline in the prompt only**. This closes the
pilots' "the truth file was in the same workspace" gap.

## Decision set (n ≥ 20, fixed before running)

A fixed set spanning:

- **Known-answer, discriminating** (~10): engineering decisions with an
  authoritative oracle *and* a plausible confident-wrong answer, chosen so a
  strong solo model has a real chance of erring (the pilots' set was too easy;
  these are harder - subtle spec/edge cases, concurrency, security defaults,
  version-specific behavior). Lift here = the panel catches a solo error.
- **Ambiguous, no-ground-truth** (~10): genuinely contested architecture /
  migration / tradeoff calls (the P-series style). Lift here = a
  judge-scored-more-useful memo.
- **Real-outcome subset** (as many as feasible): decisions whose outcome is
  publicly documented (post-mortems, resolved standards debates), so `verdict_held`
  can eventually be scored against reality, not just an oracle.

Each decision = `dNN-brief.md` (what arms see) + `sealed/dNN-truth.md` (oracle,
correct call, must_catch, landmine). The set is committed **before** any run; a
first batch is committed with this protocol and the set is completed to n ≥ 20
before scoring begins.

## Pre-registered primary endpoint & decision rule

- **Primary endpoint:** mean `A_panel − A_2seat` aggregate score across all
  decisions (paired per decision).
- **Lift is declared only if** that mean is **positive and holds under both the
  primary judge and the designated non-OpenAI second judge**, *and* the panel does
  not lose on `false_objections`. A null or negative result is reported as the
  finding and the README's "no lift" stance stands.
- Secondary endpoints (`A_2seat − A_solo`, per-decision forced choice, known-
  answer `decision_correct` and `must_catch_rate`) are reported but do not by
  themselves establish lift.

## Panel-output control (removes the assembly confound)

A_panel's judged text is a **mechanical fill** of `core/templates/verdict.md`
from the ledger: buckets + seat-verbatim claims + the discriminating test as the
seats worded it. The orchestrator adds **no** arguments. On a **random 25%
subset**, the panel is *also* judged as the **raw concatenation** of both seats'
final positions (no orchestrator touch at all); if the two panel renderings score
materially differently, assembly is confounding the result and is reported as
such.

## Honesty commitments (unchanged from the pilots, tightened)

- Set fixed before running; judge blind; **both** metrics reported; every
  decision published including panel losses.
- **Both judge vendors' scores published**; no cherry-picking a friendlier judge.
- Confounds and any protocol deviation stated in the results file.
- This is powered for a *pilot-plus* (n≈20); it is a real confirmatory attempt,
  not proof-for-all-time. Real-world-outcome validation at larger n remains the
  ceiling.

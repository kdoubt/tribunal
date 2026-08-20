# Tribunal

<p align="center">
  <a href="LICENSE"><img alt="License: MIT" src="https://img.shields.io/badge/license-MIT-blue.svg"></a>
  <a href="https://github.com/kdoubt/tribunal/tags"><img alt="Release" src="https://img.shields.io/github/v/tag/kdoubt/tribunal?label=release&color=green"></a>
  <img alt="Runtime: none" src="https://img.shields.io/badge/runtime-none%20·%20docs%20only-8A2BE2">
  <img alt="Adapters" src="https://img.shields.io/badge/adapters-claude--code%20·%20shell-informational">
</p>

> A multi-agent red-team tribunal for your hardest engineering decisions:
> AI CLIs from different vendors debate in structured rounds and return
> adjudicated verdicts that preserve dissent. A methodology, not a
> framework.

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/workflow-dark.svg">
    <img alt="Tribunal workflow: scouted from your project, a frozen brief goes to isolated parallel seats from different vendors; their claims land in a provenance-tagged ledger; disputed claims are cross-examined via verbatim relay while checkable claims go to oracles; a rare triggered Round 2 restates positions when a load-bearing claim flips; the verdict separates independent agreement, resolved claims, and surviving dissent; a retrospective feeds a template delta back into the next brief." src="assets/workflow-light.svg" width="960">
  </picture>
</p>

The loop, in one glance:

```
scout project ─▶ frozen brief ─▶ ROUND 0  seats answer in isolation (parallel, no peeking)
                             ─▶ LEDGER   claims + falsifiers; pre-exposure agreement = settled
                             ─▶ ROUND 1  disputed claims only, relayed verbatim - attack/concede/revise
                             ─▶ ROUND 2  only if a load-bearing claim flips - delta-only restate (rare)
                             ─▶ ORACLES  tests · compilers · primary docs settle the checkable
                             ─▶ VERDICT  1 agreement · 2 resolved · 3 surviving dissent (kept, not averaged)
                             ─▶ RETRO    did it hold? whose dissent was right? → template delta → next brief
```

Built for developers who work with agentic CLIs and have no human review
board: the panel is your reviewers. One model reviewing its own plan
converges on its own blind spots; two *different* vendors' models, forced
to take independent positions and then cross-examine each other claim by
claim, catch what neither would alone - **if** the process prevents them
from politely agreeing their way to a wrong answer. Tribunal is that
process. You need two or more independently configured model/agent
commands from *different* vendors or model families (local or hosted;
subscriptions only where a provider requires one).

## When to use it

High-stakes, ambiguous, or irreversible decisions: architecture calls,
security boundaries, migrations, "is this even the right design." A panel
costs several model invocations and minutes of wall-clock per round -
spend it only where the expected loss of deciding wrong justifies it (see
`core/METHODOLOGY.md`, "When to convene").

Skip it for anything a test, compiler, or grep can settle. The first rule
of the methodology: **run the oracle before convening a debate.**

## How this differs from multi-agent debate / LLM-as-judge

Multi-agent debate and LLM-as-judge are well-studied, with mixed results -
mostly because agreement between models that have read each other is cheap.
Tribunal's mechanisms target exactly that:

- **Only pre-exposure agreement counts as consensus.** Round 0 is isolated
  and parallel; anything models agree on *after* seeing each other is
  never promoted to "the panel concludes".
- **Disputes route to oracles, not more debate.** Checkable claims go to
  tests, compilers, and primary documents; debate is reserved for what
  can't be checked.
- **Dissent survives into the verdict.** Unresolved disagreement is
  reported with the cheapest discriminating test - never averaged away.

## Getting started (allow 20-40 minutes the first time)

1. **Find your first panel.** Paste `core/templates/scout.md` into your own
   agentic CLI from your project's root: it reads *your* project and
   returns project-specific panel-worthy decisions (and what to leave to
   plain oracles), then drafts your first frozen brief.
2. **Pick an adapter.** For a first panel use one of the two maintained
   adapters - [`adapters/shell/`](adapters/shell/README.md) (any
   orchestrator, plain bash) or
   [`adapters/claude-code/`](adapters/claude-code/SKILL.md). The other
   adapter directories are contribution stubs.
   (`git clone https://github.com/kdoubt/tribunal.git`)
3. **Run it.** Follow your adapter: freeze the brief
   ([template](core/templates/brief.md)), Round 0 in parallel with no
   cross-exposure, relay disputed claims verbatim, verify, adjudicate.
   Keep [`core/CONTRACT.md`](core/CONTRACT.md),
   [`core/LEDGER.md`](core/LEDGER.md), and
   [`core/VERDICT.md`](core/VERDICT.md) open as references; read
   [`core/METHODOLOGY.md`](core/METHODOLOGY.md) in full before your first
   *high-stakes* panel.

## Proof

The panel that designed this methodology is in
[`examples/sample-run/`](examples/sample-run/): seat A proposed three
debate rounds and blinded relay; seat B a two-round cap and mandatory
attribution - neither saw the other first. Both conceded specific points
under cross-examination; attribution and tie-breaking stayed contested and
are recorded as surviving dissent, not smoothed over. **Note:** it is a
historical bootstrap transcript - it predates the finalized templates and
relays full Round 0 essays, which the finalized method bans; learn the
loop from `core/templates/`, not from its file shapes.

## Repository layout

```
core/METHODOLOGY.md     the method: rounds, stop rules, failure modes
core/CONTRACT.md        orchestrator + seat obligations (MUST/SHOULD)
core/LEDGER.md          the claim ledger: fields, status enum, dispute rule
core/VERDICT.md         verdict format: three buckets, mode tags
core/templates/         scout, brief, per-round prompts, ledger, verdict, retro
examples/sample-run/    the real (historical) panel that designed the method
adapters/claude-code/   run panels from Claude Code (installable skill)
adapters/shell/         run panels from any shell - no orchestrator CLI needed
adapters/*/             other orchestrators (stubs - see CONTRIBUTING.md)
```

(`core/` is normative and orchestrator-neutral; adapters translate host
mechanics only and may never redefine rounds, ledger states, grounding, or
verdict rules.)

## FAQ

**What is multi-agent orchestration?**
Coordinating several AI models on one task. Most orchestration aims at
collaboration; Tribunal deliberately aims at *structured dispute* - the
orchestrator is a switchboard that isolates models, relays disputed claims
verbatim, and never adds its own arguments (see
[`core/CONTRACT.md`](core/CONTRACT.md)).

**Multi-agent vs single agent - when is a panel actually worth it?**
A single strong model is cheaper and usually right; use it, plus a test
suite. A panel pays off only when the decision is irreversible, ambiguous,
or hard to observe going wrong - the cases where one model's confident
blind spot is exactly the risk. Tribunal's first rule cuts the other way
too: if a compiler, test, or grep can settle it, never convene a panel.

**Why do multi-agent systems fail?**
Mostly because agreement between models that have read each other's output
is cheap: sycophancy, politeness convergence, and one model's hallucination
becoming shared "fact." Tribunal's whole design targets this - isolated
Round 0, provenance-tagged claims, verbatim-only relay, and dissent that
survives into the verdict. The full failure-mode table with mitigations is
in [`core/METHODOLOGY.md`](core/METHODOLOGY.md).

**What does red teaming mean for AI-assisted engineering?**
Red teaming is paying someone to attack your plan before reality does.
Tribunal applies that structure to engineering decisions: every claim needs
a falsifier, every disputed claim gets cross-examined by a model from a
different vendor, and the verdict reports what survived the attack - not
what everyone was happy to sign.

**How do I set up a multi-agent panel in Claude Code?**
Install the [`adapters/claude-code/`](adapters/claude-code/SKILL.md) skill,
authenticate two seat CLIs from different vendors, and start with the
scout prompt (`core/templates/scout.md`) - it reads your project and
drafts your first panel brief. The [`adapters/shell/`](adapters/shell/README.md)
recipe does the same from any terminal, no Claude Code required.

## Status

v0.1.0, and every rule in `core/` was earned the hard way: the methodology
was designed by running it on itself, then hardened by a ten-seat,
three-vendor review panel before release. The receipts are in the repo -
the founding debate lives in `examples/sample-run/`, and the failure modes
documented in METHODOLOGY (silent seat deaths, politeness convergence,
context poisoning) were each caught in real runs, not imagined. It is in
active use by its maintainer, Square Post Labs Inc., where it has already
overturned its own authors' designs more than once.

What it wants next is independent runs. Each adapter's README states its
own status; `CONTRIBUTING.md` has the support policy and the most-wanted
contribution: a sanitized panel run on a real engineering decision.

## License

MIT - see `LICENSE`.

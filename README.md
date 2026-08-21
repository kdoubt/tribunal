# Tribunal

<p align="center">
  <a href="LICENSE"><img alt="License: MIT" src="https://img.shields.io/badge/license-MIT-blue.svg"></a>
  <a href="https://github.com/kdoubt/tribunal/tags"><img alt="Release" src="https://img.shields.io/github/v/tag/kdoubt/tribunal?label=release&color=green"></a>
  <a href="https://github.com/kdoubt/tribunal/actions/workflows/checks.yml"><img alt="checks" src="https://img.shields.io/github/actions/workflow/status/kdoubt/tribunal/checks.yml?branch=main&label=checks"></a>
  <img alt="Runtime: none" src="https://img.shields.io/badge/runtime-none%20·%20docs%20only-8A2BE2">
  <img alt="Adapters" src="https://img.shields.io/badge/adapters-claude--code%20·%20shell-informational">
</p>

> A multi-agent red-team tribunal for your hardest engineering decisions:
> AI CLIs from different vendors debate in structured rounds and return
> adjudicated verdicts that preserve dissent. A methodology, not a
> framework.

<p align="center"><b>Get it — docs and prompt templates, nothing to build:</b></p>

```sh
git clone https://github.com/kdoubt/tribunal.git ~/tribunal
```

<p align="center"><sub>Then, from your project's root, hand the scout straight to your agent — it drafts your first panel:</sub></p>

```sh
# Pick ONE line below (each hands the scout to a different agent to draft your brief):
claude -p  "$(~/tribunal/scout)"      # Claude Code
codex exec "$(~/tribunal/scout)"      # Codex CLI
grok  -p   "$(~/tribunal/scout)"      # Grok CLI
```

<p align="center"><sub><code>~/tribunal/scout</code> just prints the scouting prompt with your clone path filled in — it runs no panel and writes nothing to your project. (Its once-a-week self-update check may <code>git fetch</code> inside the clone, or touch a cache-dir timestamp for zip installs; silence with <code>TRIBUNAL_NO_UPDATE_CHECK=1</code>.) That's the whole install.</sub></p>

<p align="center"><sub><b>One prerequisite to run a panel:</b> two <i>already-authenticated</i> <b>seat</b> CLIs from <i>different</i> vendors or model families (local or hosted). A single CLI is enough to <i>scout</i>, but a real panel needs two seats — that's the whole point. The <i>orchestrator</i> is a separate role: either you at a terminal (the <a href="adapters/shell/README.md">shell adapter</a>), or a driver tool like Claude Code — which is then a <b>third</b> binary, distinct from the two seats it relays between.</sub></p>

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/workflow-dark.svg">
    <img alt="Tribunal workflow: two required seats (A and B) plus an optional dashed Seat C, each a frontier CLI from a different vendor. A scouted frozen brief goes to these isolated parallel seats; their claims land in a provenance-tagged ledger; disputed claims are cross-examined via verbatim relay while checkable claims go to oracles; a rare triggered Round 2 restates positions when a load-bearing claim flips; the verdict separates independent agreement, resolved claims, and surviving dissent; a retrospective feeds a template delta back into the next brief." src="assets/workflow-light.svg" width="960">
  </picture>
</p>

<p align="center"><sub>Two seats (A, B) is the floor; the dashed third seat is optional — add it only for the highest-stakes, most irreversible calls.</sub></p>

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

## Getting started

Nothing to install - this is docs and prompt templates. Your first
*complete* panel takes about 20-40 minutes of **panel wall-clock** (scouting,
two rounds, adjudication) - authenticating your two CLIs is separate,
one-time vendor setup that is *not* counted in that estimate. Once you know
the loop it is mostly the models' wall-clock - a few minutes per round. The
one prerequisite is two *seat* CLIs from different vendors, already
authenticated (the orchestrator is a separate role - you, or a driver tool
like Claude Code, which is then a third binary).

**If a run stalls,** the usual causes are a missing or not-logged-in CLI
(check `command -v` and re-authenticate), macOS needing GNU `timeout`
(`brew install coreutils`), or a seat exiting *silently* on a permission
prompt. Each adapter's README documents these silent-seat-killers and a smoke
test that catches them before a real run.

Two seats is the floor, not the ceiling. The diagram shows three, but a
panel is any N ≥ 2 - two is the floor and the third seat is optional, added
for the highest-stakes, most irreversible calls. The biggest gain is 1 → 2
(self-review to cross-vendor); each seat past that adds diminishing value
at linear cost, so scale to the stakes. Giving seats distinct review
*lenses* is a separate, optional layer with its own rules (see
`core/METHODOLOGY.md`, "Assigning lenses") - by default you name the
surfaces in the shared brief rather than slicing one per seat.

1. **Find your first panel.** From your project's root, run
   `claude -p "$(~/tribunal/scout)"` (or `codex exec` / `grok -p`, or pipe
   `~/tribunal/scout` into any agent). It reads *your* project and returns
   project-specific panel-worthy decisions (and what to leave to plain
   oracles), then drafts your first frozen brief. **Save that brief as a file
   in your project** (e.g. `frozen-brief.md`) - that is what the adapter runs.
   (`scout` just prints the prompt with your clone path resolved; you can
   still open `core/templates/scout.md` and paste it by hand if you prefer.)
2. **Pick an adapter.** For a first panel use one of the two maintained
   adapters - [`adapters/shell/`](adapters/shell/README.md) (any
   orchestrator, plain bash) or
   [`adapters/claude-code/`](adapters/claude-code/SKILL.md). The other
   adapter directories are contribution stubs.
3. **Run it.** Follow your adapter: freeze the brief
   ([template](core/templates/brief.md)), Round 0 in parallel with no
   cross-exposure, relay disputed claims verbatim, verify, adjudicate.
   Keep [`core/CONTRACT.md`](core/CONTRACT.md),
   [`core/LEDGER.md`](core/LEDGER.md), and
   [`core/VERDICT.md`](core/VERDICT.md) open as references; read
   [`core/METHODOLOGY.md`](core/METHODOLOGY.md) in full before your first
   *high-stakes* panel.

## Staying current

Tribunal is a git clone of documentation, so updating is just a pull:

```sh
git -C ~/tribunal pull --ff-only
```

`scout` checks for you at most once a week: if your clone is behind origin
it prints that one line to *stderr* (never into the prompt it pipes to your
agent) and nothing more - it never pulls on its own. Silence it with
`export TRIBUNAL_NO_UPDATE_CHECK=1`.

If you installed from a **zip or tarball** instead of `git clone`, there is
no `origin` to pull from - so `scout` can't check versions, and updating
means re-downloading. It will, at most once a week, note which version you
have and point you at the git-clone install (which *does* self-check) and
the [releases page](https://github.com/kdoubt/tribunal/releases). The
git-clone install is recommended precisely because it makes every future
update a one-line `git pull`.

**Your own files are safe.** `--ff-only` only fast-forwards - it never
merges or rewrites history, and it leaves untracked files (your notes,
briefs, outputs) untouched. If you have a local change to a file Tribunal
also changed, git *aborts the pull and changes nothing* rather than
overwriting your edit. The update only advances the docs Tribunal ships;
it is clean-or-abort, never silent loss.

The reliable way to keep it that way: **don't edit files inside the clone.**
Use Tribunal by reference - point your agent at `~/tribunal` and write your
briefs, ledgers, and verdicts in *your own* project (that is already how the
method works; `scout` surveys your project's root, not the clone). A
pristine clone always fast-forwards cleanly; when you want a local tweak,
copy the template into your project and edit the copy.

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

Two runs in the **current** template format bracket the two outcomes the method
produces:

- [`examples/api-auth-jwt-vs-sessions/`](examples/api-auth-jwt-vs-sessions/) -
  two *neutral* heterogeneous seats (Codex CLI + Grok CLI) independently chose
  the same auth design in isolation, so the panel *early-stopped* at Round 0:
  pre-exposure **agreement**, honest empty-dissent bucketing, residual risk sent
  to a test.
- [`examples/repo-monorepo-vs-polyrepo/`](examples/repo-monorepo-vs-polyrepo/) -
  the mirror image: a **role-incentivized** stress-test (one seat assigned each
  side; roles as incentives, not personas) that forces a real **Round 1**
  cross-examination. Both seats concede points and revise confidence, neither is
  overturned, and the verdict keeps the load-bearing **surviving dissent** - then
  routes it to the cheapest discriminating test *both seats independently
  proposed*.

## Repository layout

```
scout                   one-command helper: prints the scouting prompt (and a weekly update notice)
flywheel-export         local helper: reduces your retro.md archive to de-identified metadata (stdout only)
data/                   the de-identification schema + tooling (no intake open; see data/README.md)
CHANGELOG.md            what changed per version; the update notice points here
core/METHODOLOGY.md     the method: rounds, stop rules, failure modes
core/CONTRACT.md        orchestrator + seat obligations (MUST/SHOULD)
core/LEDGER.md          the claim ledger: fields, status enum, dispute rule
core/VERDICT.md         verdict format: three buckets, mode tags
core/templates/         scout, brief, per-round prompts, ledger, verdict, retro, ach (competing-hypotheses mode)
examples/sample-run/    the real (historical) panel that designed the method
examples/api-auth-jwt-vs-sessions/   a current-format run: two vendors, independent agreement, early stop
examples/repo-monorepo-vs-polyrepo/  a current-format run: role-incentivized, full Round 1, surviving dissent
adapters/claude-code/   run panels from Claude Code (installable skill)
adapters/shell/         run panels from any shell - no orchestrator CLI needed
adapters/*/             other orchestrators - codex-cli, gemini-cli, opencode, buzz (stubs)
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

Every rule in `core/` was earned the hard way (the badge above shows the
current version; `CHANGELOG.md` records what each release changed). The
methodology was designed by running it on itself; the founding debate is in
[`examples/sample-run/`](examples/sample-run/) - a historical bootstrap,
labeled as such. Two runs in the current template format bracket the method's
outcomes:
[`examples/api-auth-jwt-vs-sessions/`](examples/api-auth-jwt-vs-sessions/)
(two vendors, independent agreement, early stop) and
[`examples/repo-monorepo-vs-polyrepo/`](examples/repo-monorepo-vs-polyrepo/)
(role-incentivized, full Round 1, surviving dissent). The clearest
in-repo receipt that the method overturns its own author is the CHANGELOG
itself: several `core/` rules landed only after a cross-vendor panel rejected
their first draft (the lens doctrine and the v0.4-0.5 hardening each carry
that note). The failure modes documented in METHODOLOGY (silent seat deaths,
politeness convergence, context poisoning) were caught in real runs, not
imagined. The ten-seat pre-release review and the maintainer's operational
runs are not published in-tree. In active use by its maintainer, Square Post
Labs Inc.

What it wants next is independent runs. Each adapter's README states its
own status; `CONTRIBUTING.md` has the support policy and the most-wanted
contribution: a sanitized panel run on a real engineering decision.

## License

MIT - see `LICENSE`.

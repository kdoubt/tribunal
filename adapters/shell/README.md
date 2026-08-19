# Shell Adapter - run a panel from any orchestrator (or by hand)

**Status: maintained recipe (not a runner). The human fills the ledger.
Verified 2026-08 on Linux/bash. No SLA.**

The methodology needs no framework: each round is a fresh, stateless,
single-shot CLI invocation, and the ledger - a file you maintain - is the
memory. Any orchestrator that can run shell commands (or a human with a
terminal) can run a compliant panel.

## Seat invocation - examples (verified 2026-08; flags change)

Any CLI with a non-interactive single-shot mode can be a seat. Confirm your
CLI's current headless flag with `<cli> --help` before trusting these:

```bash
codex exec --skip-git-repo-check "<prompt>"   # Codex CLI
grok -p "<prompt>"                             # Grok CLI
claude -p "<prompt>"                           # Claude Code (as a seat)
gemini -p "<prompt>"                           # Gemini CLI
ollama run <model> "<prompt>"                  # local model as a seat
```

Local models are first-class seats - heterogeneity is about *model
families and training lineages*, not billing. Two caveats: a local
distill of the same family as another seat is fake heterogeneity (see
CONTRACT: identity theater), and the model-selection principle still
applies - a seat must be frontier-class for a debate round, local or not.
Note that plain model runners (like `ollama run`) have no file tools, so
the orchestrator must inline the artifact excerpts the brief needs into
the prompt for that seat.

Seats must be genuinely heterogeneous - different vendors or model
families, local or hosted (CONTRACT: no identity theater). Authentication
is vendor-controlled and changes; consult each vendor's current docs. One
durable gotcha (as of 2026-08 - verify against your CLI's precedence
rules): environment API keys (`OPENAI_API_KEY`, `XAI_API_KEY`, …) can
silently take precedence over subscription auth and bill per token - keep
them unset for panel runs.

If your CLI offers a host-level read-only or sandbox mode, use it - the
prompt's read-only fence is an instruction, not a security control (see
CONTRACT "Seat fencing").

**Silent seat killers.** Exit code 0 does not mean the seat produced a
position. Three failure shapes produce plausible-looking output and must
be caught by *reading* the output, not by exit codes:

1. **Headless permission death** - an agentic CLI hits an interactive
   tool-approval prompt, auto-cancels it, and exits 0 with only its
   opening narration ("I'll read the files…" and nothing else).
   Pre-authorize the read-only tools your seats need (e.g. Grok:
   `--allow 'Bash(git *)'`; check your CLI's allow-rule syntax) and tell
   seats to prefer built-in read tools over shell commands. Some CLIs
   also accept the prompt from a file (e.g. Grok's `--prompt-file`),
   which avoids argv size/visibility limits.
2. **Usage/quota exhaustion mid-panel** - subscription CLIs expose no
   "remaining quota" query, so a cap hit between rounds surfaces only as
   an error message or truncated prose in the output, often with exit 0.
   The smoke test catches a seat that is *already* exhausted; for
   mid-panel hits, scan each seat file for limit signatures before
   ledgering (case-insensitive: "usage limit", "rate limit", "quota",
   "try again later", "upgrade to") - any hit is a dead seat for that
   round.
3. **Context overflow / truncation** - output that stops mid-sentence or
   omits the required sections.

All three resolve the same way, via the CONTRACT: output that is not in
the brief's required shape (CLAIM blocks / the ATTACK-CONCEDE-REVISE
structure) is **rejected and the seat re-run once** - after the cause is
fixed - and never paraphrased, summarized, or synthesized around. A dead
seat re-runs on the *same* vendor; substituting another vendor's model
sacrifices the heterogeneity the seat exists to provide.

## A complete two-seat panel

Save as a script (do not paste into an interactive shell - it uses
`exit`). Two roots, kept separate: the pressure-test **clone** (templates)
and the **artifact root** - the project under review, where seats must run
so the brief's relative paths resolve:

```bash
#!/usr/bin/env bash
set -euo pipefail

PRESSURE_TEST_ROOT=${PRESSURE_TEST_ROOT:?set to your pressure-test clone}
ARTIFACT_ROOT=${ARTIFACT_ROOT:?set to the project under review}
[[ -f "$PRESSURE_TEST_ROOT/core/templates/r0-seat.md" ]] || { echo "not a pressure-test clone: $PRESSURE_TEST_ROOT" >&2; exit 1; }
cd "$ARTIFACT_ROOT"
mkdir -p panel && cd panel   # or point PANEL_OUT anywhere outside the review scope

# --- Seats: name + single-shot command (see examples block above)
SEAT_A_CMD=(codex exec --skip-git-repo-check)
SEAT_B_CMD=(grok -p)

# --- GNU timeout (macOS: brew install coreutils), always with a kill-after
if   command -v timeout  >/dev/null; then T=(timeout  -k 10 600)
elif command -v gtimeout >/dev/null; then T=(gtimeout -k 10 600)
else echo "need GNU timeout (macOS: brew install coreutils)" >&2; exit 1
fi

# --- 0. Smoke-test each seat with a VERIFIABLE answer (a canned "reply OK"
#     can pass a misconfigured seat)
for s in A B; do
  cmd="SEAT_${s}_CMD[@]"
  ans=$("${T[@]}" "${!cmd}" "What is 19+23? Reply with the number only.") || { echo "seat $s dead" >&2; exit 1; }
  [[ "$ans" == *42* ]] || { echo "seat $s failed smoke test: $ans" >&2; exit 1; }
done

# --- 1. Freeze the brief. Fill core/templates/brief.md, save as
#     frozen-brief.md here; assemble the R0 prompt mechanically.
#     (Use quoted heredocs - <<'EOF' - if you generate prompts in-script,
#     so backticks and $() in prompt text aren't expanded.)
[[ -s frozen-brief.md ]] || { echo "write frozen-brief.md first (copy core/templates/brief.md)" >&2; exit 1; }
{ cat "$PRESSURE_TEST_ROOT/core/templates/r0-seat.md"; echo; cat frozen-brief.md; } > r0-prompt.md

# --- 2. Round 0: identical prompt, PARALLEL, no cross-exposure.
#     Prompts pass via stdin where the CLI supports it (argv leaks to `ps`
#     and hits ARG_MAX); the file stays as the audit trail either way.
"${T[@]}" "${SEAT_A_CMD[@]}" "$(cat r0-prompt.md)" > seat-a-r0.md 2> seat-a-r0.err & A_PID=$!
"${T[@]}" "${SEAT_B_CMD[@]}" "$(cat r0-prompt.md)" > seat-b-r0.md 2> seat-b-r0.err & B_PID=$!
fail=0
wait "$A_PID" || { echo "seat A failed ($?)" >&2; fail=1; }
wait "$B_PID" || { echo "seat B failed ($?)" >&2; fail=1; }
(( fail == 0 )) || exit 1
[[ -s seat-a-r0.md && -s seat-b-r0.md ]] || { echo "empty seat output" >&2; exit 1; }
# limit-signature scan (see "Silent seat killers"; repeat after Round 1):
if grep -liE 'usage limit|rate limit|quota|try again later|upgrade to' seat-*-r0.md; then
  echo "seat hit a usage/rate limit - dead seat, fix and re-run it" >&2; exit 1
fi

# --- 3. Ledger: copy the template, fill per core/LEDGER.md, mark
#     agreed-r0 / disputed / open. If everything decision-relevant is
#     agreed-r0: STOP - write the verdict now.
cp "$PRESSURE_TEST_ROOT/core/templates/ledger.md" ledger.md
${EDITOR:?set EDITOR to your editor command} ledger.md

# --- 4. Extract each seat's DISPUTED claims VERBATIM (their own
#     sentences + EVIDENCE lines, numbered - never your paraphrase).
#     Mechanical extraction: read the R0 file, then e.g.
#         sed -n '12,19p' seat-b-r0.md > disputed-from-b.md
#     A disputed-from file looks like:
#         DISPUTED CLAIMS FROM SEAT B (verbatim):
#         B2. CLAIM: The cache layer is unnecessary because ...
#             EVIDENCE: src/cache.py:88 "..."
#         B4. CLAIM: Migration order X-then-Y corrupts ...
#             EVIDENCE: migrations/0042.sql:12 "..."
${EDITOR} disputed-from-a.md disputed-from-b.md
#     Also extract each seat's OWN R0 claims (stateless seats need them):
${EDITOR} own-r0-a.md own-r0-b.md

# --- 5. Round 1: assemble mechanically - template + frozen brief +
#     own claims + opponent's disputed claims. Re-read each assembled
#     prompt before sending (verbatim-relay check).
{ cat "$PRESSURE_TEST_ROOT/core/templates/r1-seat.md"; echo; cat frozen-brief.md; echo; cat own-r0-a.md; echo; cat disputed-from-b.md; } > r1-for-a.md
{ cat "$PRESSURE_TEST_ROOT/core/templates/r1-seat.md"; echo; cat frozen-brief.md; echo; cat own-r0-b.md; echo; cat disputed-from-a.md; } > r1-for-b.md
"${T[@]}" "${SEAT_A_CMD[@]}" "$(cat r1-for-a.md)" > seat-a-r1.md 2> seat-a-r1.err & A_PID=$!
"${T[@]}" "${SEAT_B_CMD[@]}" "$(cat r1-for-b.md)" > seat-b-r1.md 2> seat-b-r1.err & B_PID=$!
fail=0
wait "$A_PID" || { echo "seat A failed ($?)" >&2; fail=1; }
wait "$B_PID" || { echo "seat B failed ($?)" >&2; fail=1; }
(( fail == 0 )) || exit 1
[[ -s seat-a-r1.md && -s seat-b-r1.md ]] || { echo "empty seat output" >&2; exit 1; }

# --- 6. Update ledger; run oracles on checkable residuals; write the
#     verdict from core/templates/verdict.md. Round 2 only if a
#     load-bearing claim was overturned (core/templates/r2-revision.md).
```

Durable rules regardless of CLI:

- **cwd = artifact root** when invoking seats, so the brief's relative
  paths resolve. If a seat reports it cannot read a listed path, its
  dependent claims are `SPECULATIVE` - never relay them as `USER-FACT`.
- **Parallel & isolated Round 0** - never run seat B after reading seat
  A's output into your own context; that's a relay chain, not a panel.
- **Files, not ad-hoc strings** - every prompt lives in a file (audit
  trail of exactly what each seat saw).
- **Prompts contain no secrets** - they cross process boundaries and land
  in logs.

## N > 2 seats

Add more `SEAT_*_CMD` entries; Round 1 sends each seat one combined,
attributed packet of every *other* seat's disputed claims (plus its own R0
claims), and `agreed-r0` requires all seats.
